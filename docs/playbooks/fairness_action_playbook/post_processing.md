# Post-Processing (Outcomes)

## Overview

This toolkit guides applying post-processing interventions to models for fairness, without modifying the underlying model. It works for internal models as well as 3rd-party APIs.

---

## 1. Threshold Optimization Framework

Post-processing techniques adjust model outputs (scores or decisions) to achieve fairness goals.

### 1.1 Fairness Definition Formulations

- Demographic Parity: Equal selection rates across groups  
  P(Ŷ=1 | A=a) = P(Ŷ=1 | A=b) for all groups a,b

- Equal Opportunity: Equal true positive rates  
  P(Ŷ=1 | Y=1, A=a) = P(Ŷ=1 | Y=1, A=b)

- Equalized Odds: Equal true positive and false positive rates  
  P(Ŷ=1 | Y=y, A=a) = P(Ŷ=1 | Y=y, A=b) for y ∈ {0,1}

- Intersectional Fairness: Apply same metrics across subgroups defined by multiple protected attributes (e.g., gender × age × foreign worker)

### 1.2 Threshold Search Algorithm

1. Split validation data by protected groups  
2. For each group:  
   a. Compute ROC points across threshold values  
   b. Identify thresholds satisfying the fairness definition  
   c. From valid thresholds, select one maximizing utility (e.g., accuracy, business metric)  
3. Document thresholds and fairness impacts  

**Special Cases for 3rd-party APIs**:  
- If protected attributes are unavailable at inference, rely on pre-processed inputs and group-agnostic thresholds.  
- For APIs with only binary outputs, use decision flipping or sampling-based methods.

---

## 2. Calibration Implementation Template

Calibration ensures model scores mean the same across groups.

### 2.1 Calibration Disparity Assessment

1. Divide validation data by protected groups  
2. For each group:  
   - Plot reliability diagram (predicted vs actual probabilities)  
   - Compute Expected Calibration Error (ECE)  
   - Identify regions of miscalibration  

3. Compare calibration metrics across groups and intersectional subgroups  

### 2.2 Group-Specific Calibration Methods

- Platt Scaling: Logistic regression transformation  
  P(Y=1 | s) = 1 / (1 + exp(-(A * s + B)))  

- Isotonic Regression: Non-parametric, preserves rank order  

- Temperature Scaling: Scales neural network logits  
  P(Y=1 | s) = 1 / (1 + exp(-s/T))  

### 2.3 Implementation Workflow

- Fit calibration models per group or intersectional subgroup  
- Apply transformations to raw outputs  
- Verify improvement on validation data  

> 3rd-party API note: Apply calibration on API outputs post-inference if protected attributes are known for analysis. Otherwise, rely on pre-processing interventions.

---

## 3. Transformation Selection System

Guides which post-processing technique to use.

### 3.1 Decision Tree for Technique Selection

1. Primary Fairness Goal
    - Demographic parity → threshold adjustment or score transformation
    - Equal opportunity → threshold adjustment or calibration
    - Individual fairness → score normalization

2. Deployment Constraints
    - Protected attributes unavailable → transform scores using pre-processing
    - Regulatory restrictions → prefer model-agnostic methods
    - Real-time decisions → choose computationally light techniques

3. Model Outputs Available
    - Probability scores → use calibration or score transformation
    - Raw scores → transform before thresholding
    - Binary decisions → use decision flipping or rejection options 

### 3.2 Transformation Technique Catalog

| Technique | Description | Best Use | Limitations | Library |
|-----------|-------------|----------|------------|---------|
| Threshold Adjustment | Adjust group-specific thresholds | Binary or probabilistic outputs | Requires group labels | `fairlearn.postprocessing.ThresholdOptimizer` |
| Probability Calibration | Ensure consistent probability interpretation | Probabilistic outputs | Needs sufficient validation data | `sklearn.calibration.CalibratedClassifierCV` |
| Score Transformation | Adjust scores for fairness before threshold | Probabilistic outputs | May distort rank order | `aif360.algorithms.postprocessing.CalibratedEqOddsPostprocessing` |
| Decision Flipping | Flip individual decisions to achieve fairness | Binary outputs, small datasets | May reduce predictive performance | `aif360.algorithms.postprocessing.RejectOptionClassification` |
| Rejection Option Classification | Flag uncertain decisions for human review | High-risk or regulated domains | Requires human resources | `aif360.algorithms.postprocessing.RejectOptionClassification` |

### 3.3 Code Examples

**Threshold Adjustment**

```python
from fairlearn.postprocessing import ThresholdOptimizer

optimizer = ThresholdOptimizer(
    estimator=trained_model,
    constraints="equalized_odds",
    objective="balanced_accuracy_score",
    predict_method="predict_proba"
)
optimizer.fit(X_val, y_val, sensitive_features=df_val["sex"])
y_pred = optimizer.predict(X_test, sensitive_features=df_test["sex"])
```

**Probability Calibration per group**

```python
from sklearn.calibration import CalibratedClassifierCV

# Fit a separate calibrator per protected group
calibrators = {}
for group in df_val["sex"].unique():
    mask = df_val["sex"] == group
    cal = CalibratedClassifierCV(trained_model, method="sigmoid", cv="prefit")
    cal.fit(X_val[mask], y_val[mask])
    calibrators[group] = cal

# Guard against unseen groups before applying calibrators —
# a missing key here would raise a cryptic KeyError mid-loop.
unknown_groups = set(df_test["sex"].unique()) - calibrators.keys()
if unknown_groups:
    raise ValueError(
        f"No calibrator fitted for group(s) {unknown_groups}. "
        "Ensure all test groups appear in validation data."
    )

# Apply group-specific calibration at inference
y_proba_calibrated = [
    calibrators[g].predict_proba(x.reshape(1, -1))[0, 1]
    for x, g in zip(X_test, df_test["sex"])
]
```

**Equalized Odds Post-Processing (AIF360)**

```python
from aif360.algorithms.postprocessing import CalibratedEqOddsPostprocessing

cpp = CalibratedEqOddsPostprocessing(
    privileged_groups=[{"sex": 1}],
    unprivileged_groups=[{"sex": 0}],
    cost_constraint="weighted"
)
cpp.fit(dataset_val_true, dataset_val_pred)
dataset_test_fair = cpp.predict(dataset_test_pred)
```

---

## 4. Implementation Patterns

- Intersectional Thresholding: Adjust thresholds for each intersectional subgroup (e.g., Female_>60)  
- Dynamic Threshold Updates: Update thresholds periodically to account for population or data drift  
- Monitoring Pipeline: Track subgroup selection rates, ECE, and accuracy metrics continuously  

> 3rd-party API note: Apply pre-processing and post-processing transformations to API inputs/outputs, monitor bias metrics over time, and adjust thresholds externally.

---

## 5. Integration Verification Framework

### 5.1 Baseline Assessment

- Collect model outputs and subgroup metrics without intervention  
- Document fairness gaps and business performance  

### 5.2 Intervention Validation

- Apply chosen post-processing technique  
- Measure fairness improvement across primary and intersectional metrics  
- Check performance impact (accuracy, AUC, F1, or business KPIs)  

### 5.3 Robustness Testing

- Test across multiple random splits  
- Simulate distribution shifts to ensure threshold stability  
- Monitor for subgroup reversals or unintended consequences  

### 5.4 Success Criteria

- Fairness metric gap reduction ≥ 10% relative to baseline (tighten to ≥ 20% for high-risk domains such as healthcare, credit, or hiring)
- Performance degradation ≤ 5% on primary metric (AUC, F1, or accuracy); document and justify any deviation above this threshold
- Consistent improvements across subgroups and time — gains that benefit one group while degrading another do not meet this criterion  

---

## 6. Case Study: Loan Approval System – Post-Processing

### 6.1 System Context

- Gradient boosting model predicts default risk  
- Approval threshold: 0.15 (15% risk)  
- Gender bias observed: men approved 67%, women 63%  
- Previous interventions: pre-processing and in-processing applied  

### 6.2 Step 1: Technique Selection

- Fairness goal: Equal opportunity  
- Deployment constraints: Protected attributes available for analysis but not for decision-making  
- Model outputs: probability scores (0-1)  
- Selected: Platt scaling calibration + score transformation  

### 6.3 Step 2: Implementation

```python
import numpy as np
from sklearn.calibration import CalibratedClassifierCV, calibration_curve

# ── Step 1: Assess calibration disparity per group ───────────────────────────
for group, label in [(1, "Men"), (0, "Women")]:
    mask = df_val["sex"] == group
    fraction_pos, mean_pred = calibration_curve(
        y_val[mask], y_prob_val[mask], n_bins=10
    )
    mean_error = (mean_pred - fraction_pos).mean()
    print(f"{label}: mean calibration error = {mean_error:+.3f}")
# Men:   mean calibration error = +0.008
# Women: mean calibration error = +0.031  ← women's default risk overestimated by ~3%

# ── Step 2: Fit per-group Platt scaling (sigmoid calibration) ─────────────────
# method='sigmoid' fits logistic regression on (raw_score, label) pairs:
# P(Y=1 | score) = 1 / (1 + exp(-(A * score + B)))
calibrators = {}
for group in df_val["sex"].unique():
    mask = df_val["sex"] == group
    cal = CalibratedClassifierCV(trained_model, method="sigmoid", cv="prefit")
    cal.fit(X_val[mask], y_val[mask])
    calibrators[group] = cal

# ── Step 3: Inspect fitted parameters ────────────────────────────────────────
for group, label in [(1, "Men"), (0, "Women")]:
    lr = calibrators[group].calibrated_classifiers_[0].calibrators[0]
    A, B = lr.coef_[0][0], lr.intercept_[0]
    print(f"{label}: A = {A:.4f}, B = {B:.4f}")
# Men:   A = 1.0200, B = -0.0100  → P(default) = 1/(1 + exp(-(1.02·score - 0.01)))
# Women: A = 0.9800, B = -0.0400  → P(default) = 1/(1 + exp(-(0.98·score - 0.04)))

# ── Step 4: Apply calibrated scores and uniform threshold at inference ────────
unknown_groups = set(df_test["sex"].unique()) - calibrators.keys()
if unknown_groups:
    raise ValueError(f"No calibrator fitted for group(s) {unknown_groups}.")

y_proba_calibrated = np.array([
    calibrators[g].predict_proba(x.reshape(1, -1))[0, 1]
    for x, g in zip(X_test, df_test["sex"])
])
y_pred = (y_proba_calibrated >= 0.15).astype(int) 
```

### 6.4 Step 3: Evaluation

- Equal opportunity gap reduced from 4% → 0.8%  
- Gender approval gap: 4% → 0.5%  
- Overall approval rate maintained at 65%  
- AUC unchanged (<0.01)  

**3rd-party API note:** Apply calibration and score transformation externally using collected outputs. Monitor subgroup disparities over time.  

---

## 7. Key Considerations

1. Post-processing can achieve fairness without retraining the model.  
2. Intersectional fairness requires careful thresholding across subgroups.  
3. Calibration ensures probability estimates are meaningful and consistent.  
4. Dynamic monitoring and updates maintain fairness over time.  
5. For external models or APIs, post-processing is the primary intervention mechanism, combined with pre-processing when protected attributes are unavailable at decision time.

---

## See Also

- [In-Processing Toolkit](./in_processing.md) — prior stage; reduces bias that post-processing then needs to correct
- [Validation Framework](./validation_framework.md) — how to validate post-processing outcomes (Section 2.4) and set up production monitoring (Section 5)
- [Loan Approval Case Study — Post-Processing](../../case_studies/loan_approval_fairness_e2e_case.md#4-post-processing-outcomes) — worked example with Platt scaling, group calibration, and the protected-attribute-at-inference clarification

---