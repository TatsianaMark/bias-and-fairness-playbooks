# Validation Framework for the Fairness Action Playbook

## Overview

The Validation Framework provides guidance for verifying the effectiveness of the entire fairness intervention workflow. It covers baseline assessment, sequential validation after each intervention stage, tracking of key metrics, acceptance thresholds, and ongoing monitoring. Intersectional fairness is considered at each stage to ensure equitable outcomes across combined protected groups.

---

## 1. Baseline Assessment

Before applying any interventions, collect:

- Model performance metrics such as accuracy, AUC, F1
- Fairness metrics across all protected attributes (e.g., protected_attr_1, protected_attr_2)
- Intersectional group metrics for combined subgroups (e.g., subgroup_AB, subgroup_AC)
- Calibration metrics like Expected Calibration Error
- Subgroup distributions and approval rates

Purpose: Establish a reference to measure improvement after interventions.

---

## 2. Sequential Validation

Validate after each toolkit stage.

### 2.1 Post-Causal Analysis

- Verify identified mediators, proxies, and legitimate predictors
- Ensure intersectional bias pathways are mapped
- Document potential risk points for intervention

### 2.2 Post Pre-Processing

- Check fairness metrics including equal opportunity gap and demographic parity difference
- Evaluate predictive performance including accuracy, AUC, and F1
- Validate calibration for sensitive groups
- Monitor synthetic data or reweighted samples for stability and plausibility

### 2.3 Post In-Processing

- Assess fairness metrics for each protected and intersectional subgroup
- Evaluate performance trade-offs including AUC and feature importance stability
- Conduct robustness checks across random seeds, domain-specific segments, and minor distribution shifts

### 2.4 Post Post-Processing

- Evaluate residual bias in outputs including approval gaps, probability calibration, and subgroup disparities
- Compare fairness metrics against baseline and prior stages
- Confirm regulatory constraints are satisfied
- Test dynamic thresholds or score adjustments for stability

---

## 3. Metrics to Track

| Metric Type | Examples | Notes | Library |
|------------|---------|------|---------|
| Fairness | Equal opportunity gap, demographic parity difference, subgroup error rates | Include intersectional subgroups | `fairlearn.metrics.MetricFrame` |
| Performance | Accuracy, AUC, F1, calibration | Ensure minimal trade-off with fairness | `sklearn.metrics` |
| Calibration | Expected Calibration Error, reliability diagrams | Track per subgroup | `sklearn.calibration.calibration_curve` |
| Robustness | Sensitivity to hyperparameters, random seed variations, distribution shifts | Check both performance and fairness stability | `fairlearn.metrics.MetricFrame` |
| Intersectional Outcomes | Outcome rates for combined subgroups (e.g., subgroup_AB, subgroup_AC) | Monitor residual biases that simple group metrics may miss | `fairlearn.metrics.MetricFrame` with multi-column sensitive features |

### Automated Subgroup Evaluation Example

```python
from fairlearn.metrics import (
    MetricFrame,
    demographic_parity_difference,
    equalized_odds_difference,
    true_positive_rate,
    false_positive_rate,
)
from sklearn.metrics import accuracy_score
import pandas as pd

# Single protected attribute
mf = MetricFrame(
    metrics={
        "accuracy": accuracy_score,
        "tpr": true_positive_rate,
        "fpr": false_positive_rate,
    },
    y_true=y_test,
    y_pred=y_pred,
    sensitive_features=df_test["sex"]
)
print(mf.by_group)
print("Equal opportunity gap:", mf.difference(method="between_groups")["tpr"])

# Intersectional groups (sex × age_bucket)
sensitive_intersectional = df_test[["sex", "age_bucket"]]
mf_intersectional = MetricFrame(
    metrics={"accuracy": accuracy_score},
    y_true=y_test,
    y_pred=y_pred,
    sensitive_features=sensitive_intersectional
)
print(mf_intersectional.by_group)
```

---

## 4. Acceptance Thresholds

Thresholds are intentional decisions — set them based on domain risk level, regulatory requirements, and your baseline disparity, not as arbitrary numbers.

- **Fairness metrics:** Default ceiling is equal opportunity gap ≤ 0.05 and demographic parity gap ≤ 0.07 (see Section 5.3 hard thresholds). Tighten to ≤ 0.03 / ≤ 0.05 for high-stakes domains (healthcare, credit, hiring). Note: the [Fairness Assessment Playbook](../fairness_assessment_playbook/fairness_metrics.md#2b-default-thresholds) sets demographic parity at ≤ 0.10 for initial assessment; 0.07 is intentionally stricter here because this framework governs production monitoring and mandatory re-intervention, not one-time evaluation.
- **Performance degradation:** Default ceiling is AUC drop ≤ 0.04 below baseline (see Section 5.3). This reflects the business cost your organisation is willing to accept in exchange for fairness improvements; document and justify any deviation.
- **Calibration:** Default ceiling is ECE < 0.10 per subgroup; investigate at ECE > 0.05 (see Section 5.3). Miscalibration in a minority group can cause systematic over- or under-prediction even when group-level accuracy looks acceptable.
- **Document the rationale:** Record why each threshold was chosen — regulatory requirement, domain norm, stakeholder agreement, or baseline-relative target. Undocumented thresholds become arbitrary over time.

---

## 5. Monitoring and Periodic Reassessment

Production fairness failures are almost always one of three types. Each has a different cause and a different response:

| Failure Type | What It Looks Like | Likely Cause | Response |
|---|---|---|---|
| **Group distribution shift** | A protected group's share of incoming requests changes | Population change, product expansion, seasonal effect | Recalibrate post-processing thresholds; re-evaluate synthetic data coverage |
| **Outcome rate drift** | Approval/rejection rate for a group shifts without model change | Upstream data pipeline change, proxy feature drift | Re-run pre-processing audit; check proxy correlations |
| **Calibration drift** | Model probabilities stop reflecting true outcomes for a subgroup | Concept drift, label distribution change | Refit group-specific calibrators on recent data |

---

### 5.1 What to Log

Log the following fields on every prediction. Without these, you cannot reconstruct a fair lending examination, respond to a GDPR subject access request, or diagnose which stage caused a drift.

```python
import hashlib
import json
from datetime import datetime

def log_prediction(record_id, input_features, raw_score, calibrated_score,
                   decision, model_version, threshold, sensitive_group=None):
    """
    sensitive_group: only logged for offline monitoring analysis,
                     NOT used in the decision path.
    """
    entry = {
        "record_id":        record_id,
        "timestamp":        datetime.utcnow().isoformat(),
        "model_version":    model_version,
        "raw_score":        round(float(raw_score), 6),
        "calibrated_score": round(float(calibrated_score), 6),
        "threshold":        threshold,
        "decision":         int(decision),
        "sensitive_group":  sensitive_group,   # None at decision time if legally restricted
        "feature_hash":     hashlib.sha256(
                                json.dumps(input_features, sort_keys=True).encode()
                            ).hexdigest()
    }
    # Write to your logging backend (e.g., BigQuery, S3, Elasticsearch)
    return entry
```

---

### 5.2 Rolling Fairness Metric Tracking

Run this on a sliding window of recent predictions (e.g., last 1,000 or last 7 days). Compare against your baseline metrics from Section 1.

```python
import pandas as pd
from fairlearn.metrics import MetricFrame, true_positive_rate, selection_rate
from sklearn.metrics import accuracy_score, roc_auc_score

def compute_fairness_snapshot(df_window: pd.DataFrame,
                               y_true_col: str,
                               y_pred_col: str,
                               sensitive_col: str,
                               y_score_col: str | None = None) -> dict:
    """
    df_window:   recent prediction logs with ground truth outcomes attached.
    y_score_col: column of predicted probabilities; required to populate "auc".
                 If None, "auc" is omitted and AUC alerts cannot fire.
    Returns a dict of per-group metrics and disparity gaps.

    eop_gap  = equal opportunity gap  (TPR difference across groups)
    dp_gap   = demographic parity gap (selection rate difference across groups)
               Uses selection_rate, not FPR — FPR only measures false positives
               among negatives, not overall positive prediction rate.
    auc      = overall ROC-AUC; tracked to detect performance degradation over time.
    """
    mf = MetricFrame(
        metrics={
            "accuracy":       accuracy_score,
            "tpr":            true_positive_rate,
            "selection_rate": selection_rate,
        },
        y_true=df_window[y_true_col],
        y_pred=df_window[y_pred_col],
        sensitive_features=df_window[sensitive_col]
    )
    result = {
        "by_group":           mf.by_group.to_dict(),
        "eop_gap":            mf.difference()["tpr"],             # equal opportunity gap
        "dp_gap":             mf.difference()["selection_rate"],  # demographic parity gap
        "accuracy_gap":       mf.difference()["accuracy"],
        "group_sample_sizes": df_window[sensitive_col].value_counts().to_dict()
    }
    if y_score_col is not None:
        result["auc"] = roc_auc_score(df_window[y_true_col], df_window[y_score_col])
    return result

# Example: run weekly on last 7 days of logs
snapshot = compute_fairness_snapshot(
    df_window=recent_logs,
    y_true_col="actual_outcome",
    y_pred_col="decision",
    sensitive_col="sensitive_group",
    y_score_col="raw_score"        # omit if probability scores are unavailable
)
print(snapshot)
```

> **Minimum sample size warning:** if any group has fewer than 50 samples in the window, flag the metrics as unreliable — do not trigger an alert based on them alone.

---

### 5.3 Alert Thresholds and Re-Intervention Triggers

Do not alert on every fluctuation. Use a two-tier system: a soft alert for investigation, a hard threshold for mandatory re-intervention.

| Metric | Soft Alert | Hard Threshold — Re-Intervene | Re-Intervention Stage |
|---|---|---|---|
| Equal opportunity gap | > baseline + 0.02 | > 0.05 | Recalibrate post-processing thresholds |
| Demographic parity gap | > baseline + 0.03 | > 0.07 | Re-audit proxy features (pre-processing) |
| Group calibration ECE | > 0.05 for any group | > 0.10 | Refit group-specific calibrators |
| Group sample size drop | < 100 in window | < 50 in window | Flag metrics as unreliable; human review |
| Overall AUC drop | > 0.02 below baseline | > 0.04 below baseline | Full model retraining |

> **Threshold alignment note:** The demographic parity hard ceiling of 0.07 is stricter than the 0.10 default in the [Fairness Assessment Playbook](../fairness_assessment_playbook/fairness_metrics.md#2b-default-thresholds). The assessment default covers initial evaluation where a wider acceptable range allows teams to prioritise the most severe disparities first. The 0.07 ceiling here applies to a live system: once in production, a smaller gap can still compound over time across many decisions, and the cost of re-intervention is lower than waiting until 0.10 is breached.

```python
BASELINE = {"eop_gap": 0.009, "dp_gap": 0.015, "auc": 0.81}

# Soft alert: fires when the metric drifts this far *above baseline* — triggers investigation.
SOFT_DELTA   = {"eop_gap": 0.02,  "dp_gap": 0.03}

# Hard ceiling: fires when the metric exceeds this *absolute value* — triggers mandatory re-intervention.
HARD_CEILING = {"eop_gap": 0.05,  "dp_gap": 0.07}

# AUC drop: fires when AUC falls this far *below baseline* — direction is reversed vs. gap metrics.
AUC_SOFT_DROP = 0.02   # soft: investigate
AUC_HARD_DROP = 0.04   # hard: full model retraining

def check_alerts(snapshot: dict, baseline: dict) -> list:
    alerts = []

    checks = [
        (
            "eop_gap",
            "Recalibrate post-processing thresholds immediately",
            "Investigate — check group distribution and recent data pipeline changes",
        ),
        (
            "dp_gap",
            "Re-audit proxy features (pre-processing) immediately",
            "Investigate — check for proxy feature drift or pipeline change",
        ),
    ]

    for metric, hard_action, soft_action in checks:
        value = snapshot[metric]
        if value > HARD_CEILING[metric]:                          # absolute ceiling
            alerts.append({
                "level":  "HARD",
                "metric": metric,
                "value":  value,
                "action": hard_action,
            })
        elif value > baseline[metric] + SOFT_DELTA[metric]:      # drift from baseline
            alerts.append({
                "level":  "SOFT",
                "metric": metric,
                "value":  value,
                "action": soft_action,
            })

    # AUC is a drop metric — alert when it falls below baseline, not when it rises.
    if "auc" in snapshot:
        auc_drop = baseline["auc"] - snapshot["auc"]
        if auc_drop > AUC_HARD_DROP:
            alerts.append({
                "level":  "HARD",
                "metric": "auc",
                "value":  snapshot["auc"],
                "action": "Full model retraining required — AUC has degraded significantly",
            })
        elif auc_drop > AUC_SOFT_DROP:
            alerts.append({
                "level":  "SOFT",
                "metric": "auc",
                "value":  snapshot["auc"],
                "action": "Investigate — AUC drop detected; check for data or distribution changes",
            })

    return alerts

alerts = check_alerts(snapshot, BASELINE)
for a in alerts:
    print(f"[{a['level']}] {a['metric']} = {a['value']:.4f} — {a['action']}")
```

---

### 5.4 Re-Intervention Decision Logic

When a hard threshold is breached, this determines which pipeline stage to re-run — not always a full retraining.

```
Fairness metric breaches hard threshold
│
├── Outcome rate drift (group approval rates shifted, AUC stable)
│     └── Start at POST-PROCESSING: recalibrate thresholds on recent data
│
├── Proxy correlation increased (feature drift detected)
│     └── Start at PRE-PROCESSING: re-audit proxy features, refit transformations
│           └── If not resolved → proceed to IN-PROCESSING retraining
│
├── AUC dropped AND fairness degraded
│     └── Full retraining required: restart from PRE-PROCESSING
│
└── Group sample size too small for reliable metrics
      └── Do NOT trigger automated re-intervention
            └── Flag for human review; consider synthetic data augmentation
```

---

### 5.5 Monitoring Cadence

| Check | Frequency | Tool |
|---|---|---|
| Rolling fairness snapshot | Daily (or per batch if batch system) | `fairlearn.metrics.MetricFrame` on prediction logs |
| Feature distribution drift | Weekly | `evidently` `DataDriftPreset` or `alibi-detect` |
| Calibration ECE per group | Weekly | `sklearn.calibration.calibration_curve` |
| Alert threshold check | Every snapshot | `check_alerts()` above |
| Full re-intervention review | Quarterly or on hard trigger | Full pipeline re-run per Section 2 |
| Documentation update | After every intervention | Record parameter choices, outcomes, trigger reason |

---

## See Also

- [Organizational Implementation Guidelines](./org_guide.md) — team roles, monitoring cadence table, and required tooling
- [Post-Processing Methods](./post_processing.md) — recalibration steps to run when a hard threshold is breached
- [Loan Approval Case Study — Cross-Stage Metrics](../../case_studies/loan_approval_fairness_e2e_case.md#5-cross-stage-metrics-summary) — example of how metrics should evolve across all four intervention stages