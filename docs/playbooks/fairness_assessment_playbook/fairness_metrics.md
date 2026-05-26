# Fairness Metrics

## Overview

This section defines how fairness metrics should be selected, validated, and communicated. It is most heavily relied upon in regulated, decision-dense domains where outcomes must be continuously monitored, audited, and defended with quantitative evidence.

---

## 1. Intersectionality

**Why It Matters:**

Applying metrics only to individual protected attributes masks disparities that emerge at intersections — for example, a model may satisfy equal opportunity for women and for Black individuals separately, yet systematically underperform for Black women. See the [Fairness Assessment Playbook Intro](./fairness_assessment_playbook_intro.md#4-intersectionality) for the shared foundation.

**Key Benefits:**

- Complete Coverage — Intersectional evaluation reveals disparity patterns that single-axis metrics cannot detect, reducing the risk of approving a system that harms specific subgroups.
- Stronger Compliance Posture — Regulators increasingly expect disaggregated reporting. Intersectional metrics provide the evidence trail needed for audit and accountability.
- Informed Threshold Setting — Disparity thresholds calibrated against intersectional groups reflect real-world harm more accurately than group-level averages.

**Implementation Guidance:**

- Apply all selected metrics (Section 2.a) to intersectional subgroups, not just individual protected attributes.
- Include intersectional breakdowns in all disparity charts and heatmaps (Section 2.d).
- Flag small intersectional subgroup sizes explicitly; use Bayesian estimation or bootstrapping where sample sizes are limited.
- Document intersectional findings in every fairness report alongside single-attribute results.

---

## 2. Implementation Framework

### 2.a. Metric Selection

Metric selection must be driven by problem type, fairness definition, and error impact — not convenience.

**Step 1: Classify problem type**

- Classification — binary or multi-class decisions (approve / deny)
- Regression — continuous outputs (risk scores, prices)
- Ranking / Recommendation — ordered outputs or exposure allocation

**Step 2: Map fairness definition to metrics**

**Classification**

| Fairness Definition | Recommended Metrics |
| --- | --- |
| Demographic Parity | Selection Rate Difference, Statistical Parity Difference |
| Equal Opportunity | True Positive Rate Difference |
| Equalized Odds | TPR Difference, FPR Difference |
| Predictive Equality | False Positive Rate Difference |
| Calibration / Sufficiency | Calibration Error by Group |

**Regression**

| Fairness Definition | Recommended Metrics |
| --- | --- |
| Statistical Parity | Group Outcome Difference, Distribution Comparison |
| Bounded Group Loss | Maximum Group Loss, Group Error Ratio |
| Individual Fairness | Individual Consistency, Input–Output Sensitivity |

**Ranking / Recommendation**

| Fairness Definition | Recommended Metrics |
| --- | --- |
| Exposure Parity | Exposure Ratio, NDCG Difference |
| Representation Parity | Top-k Proportion Difference |
| Individual Fairness | Rank Consistency, Similar-Item Rank Distance |

**Step 3: Assess error direction**

- Overprediction more harmful → add Positive Residual Difference
- Underprediction more harmful → add Negative Residual Difference
- Both harmful → add Absolute Residual Difference

**Step 4: Account for uncertainty**

- Uncertainty estimates available → add Prediction Interval Coverage, Calibration Metrics
- No uncertainty → document the limitation and plan post-deployment monitoring

---

### 2.b. Default Thresholds

These are starting-point thresholds. Tighten them for high-risk domains (healthcare, credit, hiring) or where regulation requires stricter standards.

| Metric | Default Acceptable Threshold | Notes |
| --- | --- | --- |
| Selection Rate Difference (Demographic Parity) | ≤ 0.10 | EU AI Act high-risk: consider ≤ 0.05. The [Fairness Action Playbook validation framework](../fairness_action_playbook/validation_framework.md#5-monitoring-and-periodic-reassessment) applies a stricter 0.07 production monitoring ceiling — see that document for the rationale. |
| True Positive Rate Difference (Equal Opportunity) | ≤ 0.05 | Tighten to ≤ 0.03 for life-affecting decisions |
| False Positive Rate Difference (Predictive Equality) | ≤ 0.05 | |
| TPR + FPR Difference (Equalized Odds) | ≤ 0.05 each | Both must be within threshold |
| Calibration Error by Group | ≤ 0.05 | |
| Maximum Group Loss (Bounded Group Loss) | ≤ 1.25× average group loss | |
| Exposure Ratio | ≥ 0.80 | i.e., no group receives less than 80% of proportional exposure |

> These thresholds are defaults, not absolutes. Document any deviation and the rationale. If your system is in a regulated domain, cross-reference the [Fairness Action Playbook](../fairness_action_playbook/fairness_action_playbook_intro.md) for binding requirements.

---

### 2.c. Statistical Validation

Fairness metrics must be statistically validated to avoid false conclusions, especially for small or underrepresented groups.

**Bootstrap Confidence Intervals**

- Resample the dataset with replacement.
- Recalculate fairness metrics for each resample.
- Report 95% confidence intervals alongside point estimates.

**Small Sample Handling**

- Use Bayesian estimation with weakly informative priors.
- Report credible intervals instead of frequentist confidence intervals.
- Flag small sample sizes explicitly in charts and reports.
- Avoid definitive conclusions when uncertainty is high.

---

### 2.d. Visualization and Reporting

**Fairness Disparity Charts**

- Bar charts showing primary metrics by group.
- Error bars for confidence or credible intervals.
- Reference lines at acceptable thresholds.
- Visual markers for statistically significant disparities.

**Intersectional Heatmaps**

- Metric values across intersecting protected attributes.
- Color intensity = disparity magnitude.
- Cell opacity = sample size.
- Explicit labelling of sparsely populated intersections.

**Each fairness report must include:**

- Selected fairness definitions and rationale
- Metrics used and validation methods applied
- Key findings per group and intersectional subgroup
- Identified risks and recommended actions
- Limitations (small samples, missing data, proxy variables)
- Monitoring plan: metrics tracked, frequency, alert thresholds, owner, escalation path

---

## 3. Tooling

| Tool | Best For |
| --- | --- |
| **Fairlearn** | Metric computation, mitigation algorithms, disparity charts |
| **Aequitas** | Bias reporting across groups, audit-ready outputs |
| **What-If Tool** (Google) | Interactive visualization, counterfactual exploration |
| **Evidently AI** | Post-deployment drift monitoring, automated reports |

All four are open-source. For smaller teams, start with Fairlearn — it covers metric computation and basic mitigation in one library.

---

## 4. Re-audit Triggers

Do not wait for a scheduled review if any of the following occur:

| Trigger | Action |
| --- | --- |
| Monitoring alert exceeds threshold | Re-run affected metrics immediately; escalate if unresolved within 5 days |
| Model retrained on new data | Re-run full Fairness Metrics component |
| Demographic shift detected in user base | Re-run Representation checks + all primary metrics |
| New protected attribute added to scope | Re-run full component for new attribute and its intersections |
| Regulatory audit initiated | Full metrics re-run with formal documentation |
| Annually (minimum) | Full metrics re-run |

---

## 5. Usage Guide

### Step-by-Step

1. Review Historical Context and Fairness Definitions — focus on high-risk groups and selected definitions.
2. Classify problem type (Classification / Regression / Ranking).
3. Select metrics aligned with fairness definition and error direction.
4. Apply intersectional evaluation — disaggregate by subgroups.
5. Validate statistically — bootstrap CIs or Bayesian estimation for small groups.
6. Compare results against default thresholds; document any deviations.
7. Visualise — disparity charts and intersectional heatmaps.
8. Complete the fairness report — include all required fields.
9. Feed findings into the [Fairness Action Playbook](../fairness_action_playbook/fairness_action_playbook_intro.md) for remediation.
10. Set monitoring alerts and schedule re-audit per triggers above.

### Tips

- Use at least two metrics. No single metric captures the full picture.
- Document trade-offs. When metrics conflict, record the conflict and the decision — this is a compliance requirement.
- Communicate uncertainty. Always report confidence intervals alongside point estimates, especially for small groups.
- Prioritise high-impact errors. Focus mitigation on errors with the largest potential harm first.
