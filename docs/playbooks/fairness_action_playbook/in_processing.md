# In-Processing (Model)

## Overview

This document provides a structured toolkit for embedding fairness directly into model training. It applies to internal models, where training can be controlled, and includes considerations for 3rd-party APIs, where in-training interventions are not possible and pre-/post-processing must be relied on.

---

## 1. Model Architecture Analysis Template

### 1.1 Model Type Classification

* Linear models: logistic regression, linear SVM
* Tree-based models: decision trees, random forests, gradient boosting
* Neural networks: feedforward, convolutional, recurrent
* Other: ____________

### 1.2 Model Characteristics

* Training approach (batch, online, incremental)
* Loss function
* Regularization methods
* Hyperparameter tuning approach

### 1.3 Technical Constraints

* Available computational resources
* Maximum acceptable training time increase
* Explainability requirements
* Deployment environment limitations
* Regulatory or auditing constraints

### 1.4 Compatibility Matrix

| Fairness Technique        | Linear Models | Tree-based Models | Neural Networks |
|---------------------------|---------------|-----------------|----------------|
| Constraint Optimization   | High          | Low             | Medium         |
| Adversarial Debiasing     | Low           | Low             | High           |
| Fairness Regularization   | High          | Medium          | High           |
| Fair Representations (pre-processing step — see Pattern 5 note) | Medium | Low | High |
| Specialized Algorithms    | Medium        | High            | Low            |

> **Guidance:** Use this matrix to select techniques compatible with your architecture and constraints.

---

## 2. Technique Selection Decision Tree

### Step 1: Model Architecture Assessment

* Linear model → Step 2A
* Tree-based model → Step 2B
* Neural network → Step 2C
* Other → Consider model-agnostic fairness approaches

### Step 2A: Linear Model Approaches

* Demographic parity → Constraint optimization with fairness-aware objective (Pattern 1)
* Equal opportunity → Constraint optimization or adjusted thresholds (Pattern 1)
* Logistic regression with label bias → Fairness-aware regularization (Pattern 6: PrejudiceRemover)
* Individual fairness → Similarity-based regularization

### Step 2B: Tree-based Model Approaches

* Demographic parity → Fair splitting criteria
* Equal opportunity → Fair splitting with weighted samples
* Individual fairness → Regularized tree induction

### Step 2C: Neural Network Approaches

* Demographic parity → Adversarial debiasing
* Equal opportunity → Multi-task learning with fairness head
* Individual fairness → Gradient penalties or contrastive learning

> For **3rd-party APIs**, in-training interventions are not possible. Instead, combine pre-processing transformations and post-processing adjustments to achieve fairness.

---

## 3. Implementation Pattern Catalog

### Pattern 1: Constraint Optimization for Linear Models

* Approach: Add fairness constraints to the objective function.
* Components:
    * Modified loss function including fairness penalty
    * Slack variables or relaxation parameters
    * Learning rate adjustments for constrained optimization
* Parameters:
    * Constraint weight (controls fairness-performance trade-off)
    * Slack bounds
    * Convergence criteria adjustments
* Considerations: Requires specialized solvers; increases training time; works best with convex loss.
* Library: `fairlearn.reductions.ExponentiatedGradient`

```python
from fairlearn.reductions import ExponentiatedGradient, DemographicParity
from sklearn.linear_model import LogisticRegression

estimator = LogisticRegression()
constraint = DemographicParity()

mitigator = ExponentiatedGradient(estimator, constraint, eps=0.01)
mitigator.fit(X_train, y_train, sensitive_features=df_train["sex"])
y_pred = mitigator.predict(X_test)
```

### Pattern 2: Adversarial Debiasing for Neural Networks

* Approach: Train main predictor while minimizing adversary's ability to infer protected attributes.
* Components:
    * Main predictor
    * Adversary network
    * Gradient reversal layer
    * Combined loss function
* Parameters:
    * `adversary_loss_weight`: controls the trade-off between prediction accuracy and debiasing strength — increase to enforce stronger fairness, decrease to preserve more accuracy
    * Architecture complexity of both predictor and adversary
    * Gradient scaling factor on the reversal layer
* Considerations: Needs careful balancing; larger datasets preferred; may be unstable without tuning. Evaluate on both fairness metrics and accuracy — adversarial training can oscillate if the adversary outpaces the predictor early in training.
* Library: PyTorch (custom implementation) — `aif360`'s built-in `AdversarialDebiasing` requires TensorFlow 1.x which is end-of-life. Use the pattern below directly.

```python
import torch
import torch.nn as nn
import torch.optim as optim
from torch.utils.data import DataLoader, TensorDataset

# --- Architecture ---
class Predictor(nn.Module):
    def __init__(self, input_dim):
        super().__init__()
        self.net = nn.Sequential(
            nn.Linear(input_dim, 64), nn.ReLU(),
            nn.Linear(64, 32),        nn.ReLU(),
            nn.Linear(32, 1),         nn.Sigmoid()
        )
    def forward(self, x):
        return self.net(x)

class Adversary(nn.Module):
    def __init__(self):
        super().__init__()
        self.net = nn.Sequential(
            nn.Linear(1, 32), nn.ReLU(),
            nn.Linear(32, 1), nn.Sigmoid()
        )
    def forward(self, pred_prob):
        return self.net(pred_prob)

# --- Training loop ---
ADVERSARY_LOSS_WEIGHT = 0.5  
N_EPOCHS = 30

predictor = Predictor(input_dim=X_train.shape[1])
adversary  = Adversary()

opt_pred = optim.Adam(predictor.parameters(), lr=1e-3)
opt_adv  = optim.Adam(adversary.parameters(),  lr=1e-3)
bce = nn.BCELoss()

dataset = TensorDataset(
    torch.tensor(X_train, dtype=torch.float32),
    torch.tensor(y_train, dtype=torch.float32),
    torch.tensor(sensitive_train, dtype=torch.float32)  # protected attribute
)
loader = DataLoader(dataset, batch_size=128, shuffle=True)

for epoch in range(N_EPOCHS):
    for X_batch, y_batch, s_batch in loader:

        # ── Step 1: update adversary ──────────────────────────────────────────
        # Detach predictor output so predictor parameters receive no gradients
        # from the adversary loss.
        pred_prob = predictor(X_batch).detach()
        adv_loss  = bce(adversary(pred_prob).squeeze(), s_batch)
        opt_adv.zero_grad()
        adv_loss.backward()
        opt_adv.step()

        # ── Step 2: update predictor ──────────────────────────────────────────
        # Freeze adversary parameters before backward so total_loss.backward()
        # does not corrupt the adversary with gradients from the predictor loss.
        for p in adversary.parameters():
            p.requires_grad_(False)

        pred_prob  = predictor(X_batch)
        pred_loss  = bce(pred_prob.squeeze(), y_batch)
        adv_loss_p = bce(adversary(pred_prob).squeeze(), s_batch)
        total_loss = pred_loss - ADVERSARY_LOSS_WEIGHT * adv_loss_p

        opt_pred.zero_grad()
        total_loss.backward() 
        opt_pred.step()

        # Restore adversary parameters for next Step 1
        for p in adversary.parameters():
            p.requires_grad_(True)

# Inference
predictor.eval()
with torch.no_grad():
    y_prob = predictor(torch.tensor(X_test, dtype=torch.float32)).squeeze().numpy()
y_pred = (y_prob >= 0.5).astype(int)
```

### Pattern 3: Fair Splitting / Regularization for Tree-based Models

* Approach: Modify splitting criteria to penalize splits that increase disparity.
* Components:
    * Custom impurity metric including fairness penalty
    * Weighted sample adjustments
    * Early stopping based on fairness validation
* Parameters:
    * Fairness penalty weight
    * Maximum tree depth constraints
    * Minimum samples per node for stable fairness metrics
* Considerations: Maintains explainability; compatible with gradient boosting and random forests; limited flexibility for complex interactions.
* Library: `fairlearn.reductions.GridSearch`

```python
from fairlearn.reductions import GridSearch, EqualizedOdds
from sklearn.tree import DecisionTreeClassifier

estimator = DecisionTreeClassifier(max_depth=4)
constraint = EqualizedOdds()

sweep = GridSearch(estimator, constraint, grid_size=20)
sweep.fit(X_train, y_train, sensitive_features=df_train["sex"])

# Select model with best accuracy that meets fairness threshold
best_model = sweep.predictors_[sweep.best_index_]
y_pred = best_model.predict(X_test)
```

### Pattern 4: Fairness Regularization for Neural Networks

* Approach: Add fairness regularization term to standard loss.
* Components:
    * Loss = standard prediction loss + fairness regularization
    * Regularization based on protected attribute correlations or subgroup metrics
* Parameters:
    * Regularization coefficient
    * Subgroup sampling strategy
* Considerations: Can generalize to multiple fairness definitions; may slightly reduce predictive performance.
* Library: `fairlearn.reductions.ExponentiatedGradient` with `EqualizedOdds`

```python
from fairlearn.reductions import ExponentiatedGradient, EqualizedOdds
from sklearn.neural_network import MLPClassifier

estimator = MLPClassifier(hidden_layer_sizes=(64, 32), max_iter=500)
constraint = EqualizedOdds(difference_bound=0.05)

mitigator = ExponentiatedGradient(estimator, constraint)
mitigator.fit(X_train, y_train, sensitive_features=df_train["sex"])
y_pred = mitigator.predict(X_test)
```

### Pattern 5: Fair Representations

> **Note:** LFR (`aif360.algorithms.preprocessing.LFR`) fits a prototype-based encoder on training data and transforms it before any downstream model is trained. This is a pre-processing step, not an in-processing one — it lives in `aif360.algorithms.preprocessing` for this reason. It is listed in the compatibility matrix below because the choice of downstream model architecture affects how well LFR representations transfer.
>
> For implementation details and code, see [Pre-Processing — Section 1.2: Fair Representations](./pre_processing.md#12-transformation-techniques).

### Pattern 6: Fairness-Aware Regularization for Logistic Regression (PrejudiceRemover)

* Approach: Adds a fairness regularization term to logistic regression's training objective, penalizing mutual information between predictions and the protected attribute.
* Components:
    * Standard logistic regression objective
    * Fairness penalty term controlled by `eta`
* Parameters:
    * `eta`: regularization strength — higher values enforce stronger fairness at the cost of predictive accuracy; start at 1.0 and tune upward
    * `sensitive_attr`: name of the protected attribute column in the dataset
* Considerations: Specific to logistic regression — not transferable to other model types. Provides a single-step training solution without a separate fairness wrapper. Verify calibration after applying, since aggressive regularization can shift probability estimates for minority groups.
* Library: `aif360.algorithms.inprocessing.PrejudiceRemover`

```python
from aif360.algorithms.inprocessing import PrejudiceRemover

model = PrejudiceRemover(eta=25.0, sensitive_attr="sex")
model.fit(dataset_train)
predictions = model.predict(dataset_test)
```

> **3rd-party APIs:** These patterns cannot be applied directly in training. Pre-processing input features and post-processing outputs are required to simulate fairness adjustments.

---

## 4. Integration Verification Framework

### 4.1 Validation Testing Protocol

* Baseline Establishment: Train model without fairness interventions; document metrics.
* Intervention Validation: Apply selected in-processing technique; measure fairness improvements and performance impact.
* Robustness Testing:
    * Across data subsets (intersectional groups)
    * Sensitivity to hyperparameter changes
    * Behavior under distribution shifts

### 4.2 Success Criteria

* Fairness metric improvement ≥ 10% relative to baseline (tighten to ≥ 20% for high-risk domains such as healthcare, credit, or hiring)
* Performance reduction ≤ 5% on primary metric (AUC, F1, or accuracy); document and justify any deviation above this threshold
* Consistent improvements across subgroups, including intersectional groups — a technique that improves one group while degrading another does not meet this criterion
* Stable behavior under minor hyperparameter changes — fairness gains that disappear with small perturbations should not be accepted

### 4.3 3rd-Party API Adaptation

In-training interventions are not available for 3rd-party APIs. Fairness must rely entirely on pre-processing inputs and post-processing outputs.

**Pre-processing:**
* Transform features to remove bias correlations (normalization, masking, or weighting)
* Generate synthetic input examples for minority or intersectional subgroups
* Apply the technique logic from Patterns 1–5 to input data before sending API requests

**Post-processing:**
* Apply thresholds or score adjustments to API outputs to reduce disparities
* Evaluate corrections across intersectional subgroups, not just individual protected attributes

**Validation:**
* Monitor API outputs for fairness metrics after each pre-processing change
* Iterate pre- and post-processing until fairness thresholds from Section 4.2 are met
* Re-test after any API version update — vendor model changes can shift disparity patterns

---

## 5. Key Considerations and Best Practices

1. Hyperparameter Tuning: Always tune both performance and fairness hyperparameters.  
2. Intersectionality: Evaluate improvements not only for single protected attributes but also for combined subgroups.  
3. Explainability: Preserve interpretability when required, particularly in regulated domains.  
4. Monitoring: Continuously track fairness metrics post-deployment, especially for 3rd-party APIs.  
5. Pipeline Integration: Ensure in-processing techniques integrate with existing model training pipelines with minimal disruption.  
6. Fallback Strategies: Use pre- and post-processing when in-training modification is impossible (e.g., external APIs).

---

## See Also

* [Pre-Processing Methods](./pre_processing.md) — prior stage; apply before in-processing
* [Post-Processing Methods](./post_processing.md) — next stage; handles residual bias after training
* [Validation Framework](./validation_framework.md) — how to validate in-processing outcomes (Section 2.3)
* [Loan Approval Case Study — In-Processing](../../case_studies/loan_approval_fairness_e2e_case.md#3-in-processing-toolkit) — worked example with `fairlearn.reductions.GridSearch` on gradient boosting
