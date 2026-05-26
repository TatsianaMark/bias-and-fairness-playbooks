# Pre-Processing (Data)

## Overview

The Pre-Processing Fairness Toolkit addresses bias at the data level before model training or before sending data to a 3rd-party API.  

- For internal models, interventions occur directly on training data.  
- For 3rd-party APIs, pre-processing is applied to input features before requests are sent. This reduces bias in API outputs even though the model cannot be modified.  

Pre-processing is the first intervention step in the Fairness Action Playbook. Its goal is to reduce bias while preserving predictive signal.

---

## 1. Technique Catalog

### 1.1 Reweighting Approaches

**Instance Weighting**  
- Internal Models: Adjusts training example influence to balance representation.  
- 3rd-Party APIs: Simulate weighting by preparing input batches that overrepresent underrepresented groups. Use these to evaluate outputs and guide post-processing.  
- Strengths: Keeps all data points; adjustable intervention strength.  
- Limitations: For APIs, effects are approximate since model weights cannot be changed.  
- Library: `aif360.algorithms.preprocessing.Reweighing`

```python
from aif360.algorithms.preprocessing import Reweighing
from aif360.datasets import BinaryLabelDataset

rw = Reweighing(unprivileged_groups=[{"sex": 0}],
                privileged_groups=[{"sex": 1}])
dataset_transformed = rw.fit_transform(dataset)  # dataset: BinaryLabelDataset
```

**Label Massaging**
- Internal Models: Flips labels of borderline instances near the decision boundary — promoting disadvantaged-group negatives and demoting privileged-group positives — to reduce label-protected attribute correlation before training.
- 3rd-Party APIs: Apply to evaluation or surrogate datasets to test API fairness under corrected labels; not applicable to live API inputs.
- Strengths: Lightweight, model-agnostic, interpretable label corrections.
- Limitations: Sensitive to the number of flips; too many can degrade predictive accuracy.

```python
import numpy as np
from sklearn.linear_model import LogisticRegression

def label_massaging(X, y, protected, privileged_val=1, n_flip=100):
    scores = LogisticRegression().fit(X, y).predict_proba(X)[:, 1]

    # Promote: disadvantaged group, negative label, closest to boundary
    promote_idx = np.where((protected != privileged_val) & (y == 0))[0]
    y[promote_idx[np.argsort(-scores[promote_idx])[:n_flip]]] = 1

    # Demote: privileged group, positive label, closest to boundary
    demote_idx = np.where((protected == privileged_val) & (y == 1))[0]
    y[demote_idx[np.argsort(scores[demote_idx])[:n_flip]]] = 0

    return y
```

### 1.2 Transformation Techniques

**Disparate Impact Removal**  
- Internal Models: Transforms features to minimize correlation with protected attributes.  
- 3rd-Party APIs: Apply same transformations to inputs before sending requests, ensuring proxies or protected attributes are less predictive.  
- Library: `aif360.algorithms.preprocessing.DisparateImpactRemover`

```python
from aif360.algorithms.preprocessing import DisparateImpactRemover

di = DisparateImpactRemover(repair_level=1.0, sensitive_attribute="sex")
dataset_transformed = di.fit_transform(dataset)
# repair_level: 0.0 = no repair, 1.0 = full repair (tune for utility trade-off)
```

**Fair Representations**  
- Internal Models: Create new feature space masking protected attributes while preserving predictive signal.  
- 3rd-Party APIs: Map inputs to fairness-aware feature space before API requests using embeddings or latent features.  
- Library: `aif360.algorithms.preprocessing.LFR` (Learning Fair Representations)
- Note: LFR has its own `fit` phase on training data, but it is a pre-processing transformation — the downstream model trains on the transformed output, not jointly with LFR.

```python
from aif360.algorithms.preprocessing import LFR

lfr = LFR(unprivileged_groups=[{"sex": 0}],
           privileged_groups=[{"sex": 1}],
           k=10, Ax=0.1, Ay=1.0, Az=2.0)
lfr.fit(dataset_train, maxiter=5000)
dataset_transformed = lfr.transform(dataset_train)
```

### 1.3 Sampling and Synthetic Data Generation

**Resampling Methods**  
- Internal Models: Oversample or undersample to balance representation.  
- 3rd-Party APIs: Prepare balanced input batches simulating equal representation across protected attributes. Send these to the API to measure fairness in outputs.  
- Library: `imbalanced-learn` (`imblearn`)

```python
from imblearn.over_sampling import SMOTE
import numpy as np

smote = SMOTE(sampling_strategy="minority", random_state=42)

# Apply SMOTE within each protected group to avoid synthesizing samples
# across group boundaries, which would blur group-level distributions.
# Requires at least k+1 minority examples per group (default k=5 → need ≥ 6).
X_parts, y_parts = [], []
for group in np.unique(protected_train):
    mask = protected_train == group
    X_g, y_g = smote.fit_resample(X_train[mask], y_train[mask])
    X_parts.append(X_g)
    y_parts.append(y_g)

X_resampled = np.vstack(X_parts)
y_resampled = np.concatenate(y_parts)
```

**Synthetic Data Generation**  
- Internal Models: Generate additional data for underrepresented groups.  
- 3rd-Party APIs: Generate synthetic input examples for minority or intersectional groups, then use API outputs to evaluate and correct residual bias with post-processing.  
- Library: `sdv` (Synthetic Data Vault) or `aif360.algorithms.preprocessing.OptimPreproc`

```python
from sdv.single_table import GaussianCopulaSynthesizer
from sdv.metadata import SingleTableMetadata

metadata = SingleTableMetadata()
metadata.detect_from_dataframe(df_minority_group)

synthesizer = GaussianCopulaSynthesizer(metadata)
synthesizer.fit(df_minority_group)
synthetic_data = synthesizer.sample(num_rows=500)
```

### 1.4 Data Auditing

- Description: Systematic analysis of distributions, correlations, and subgroup representation.  
- Internal Models: Identify bias sources before training; guide pre-processing strategy.  
- 3rd-Party APIs: Audit input feature distributions and output predictions; use results to decide which pre-processing transformations and post-processing corrections are necessary.  
- Library: `fairlearn.metrics.MetricFrame`

```python
from fairlearn.metrics import MetricFrame, demographic_parity_difference
from sklearn.metrics import accuracy_score

mf = MetricFrame(
    metrics={"accuracy": accuracy_score},
    y_true=y_true,
    y_pred=y_pred,
    sensitive_features=df["sex"]
)
print(mf.by_group)          
print(mf.difference())  
```  

---

## 2. Selection Decision Tree

Guides pre-processing technique selection based on causal bias analysis. Steps apply to internal models and 3rd-party APIs with adjusted methods.

### Step 1: Bias Pattern Identification

- Representation disparities → Step 2  
- Proxy discrimination → Step 3  
- Label bias → Step 4  
- Multiple bias types → Combine approaches

### Step 2: Representation Disparities

- Model supports instance weights?  
  - Yes → Instance Weighting for internal models  
  - For APIs → Prepare balanced input batches or simulate weights  
- No → Enough samples in underrepresented groups?  
  - Yes → Resampling  
  - No → Synthetic Data Generation

### Step 3: Proxy Discrimination

- Can you identify proxy features?  
  - Yes → Is interpretability critical?  
    - Yes → Disparate Impact Removal  
    - No → Fair Representations  
  - No → Fair Representations or advanced transformations

### Step 4: Label Bias

- Historical discrimination causing bias?  
  - Yes → Label Massaging (see also [In-Processing — PrejudiceRemover](./in_processing.md) for a training-time alternative)
  - No → Reconsider data collection process

---

## 3. Configuration Guidelines

### 3.1 Instance Weighting

- Weighting schemes:  
  - Demographic parity → Inverse frequency weighting  
  - Equal opportunity → Conditional inverse frequency weighting  
- Intervention strength: Start moderate, gradually increase, cap maximum weight.  
- Validation: Check loss stability and fairness improvements on validation data.  
- 3rd-Party APIs: Validate fairness effects by sending weighted input batches and measuring output disparities.

### 3.2 Disparate Impact Removal

- Feature selection: Target features strongly correlated with protected attributes; preserve legitimate predictors.  
- Transformation intensity: Start partial, adjust based on fairness-utility trade-offs.  
- Validation: Measure correlation reduction and predictive power preservation.  
- 3rd-Party APIs: Apply transformations to inputs before API calls; evaluate output distributions to confirm bias reduction.

### 3.3 Fair Representations and Synthetic Data

- Representation learning: Mask protected attributes while preserving predictive signals.  
- Synthetic data: Ensure realism and alignment with original distributions.  
- Validation: Evaluate predictive performance, subgroup representation, and downstream fairness.  
- 3rd-Party APIs: Send fairness-aware or synthetic inputs to the API and use outputs to guide post-processing.

---

## 4. Evaluation Framework

### 4.1 Fairness Assessment

- Measure fairness metrics such as demographic parity and equalized odds.  
- Evaluate intersectional fairness across multiple protected attributes.  
- Conduct statistical significance testing.  
- 3rd-Party APIs: Compare outputs across pre-processed inputs for residual disparities.

### 4.2 Information Preservation

- Track predictive performance including accuracy, AUC, and F1.  
- Check rank ordering and calibration within groups.  
- Assess feature importance stability.

### 4.3 Computational Efficiency

- Document processing time and memory usage.  
- Test pipeline scaling with dataset size.  
- Evaluate deployment implications.

---

## See Also

- [Causal Analysis](./causal_fairness_toolkit.md) — identify which features to transform before applying pre-processing
- [In-Processing Methods](./in_processing.md) — next stage after pre-processing
- [Validation Framework](./validation_framework.md) — how to validate pre-processing outcomes (Section 2.2)
- [Loan Approval Case Study — Pre-Processing](../../case_studies/loan_approval_fairness_e2e_case.md#2-pre-processing) — worked example with code on the German Credit dataset