# Bias Root Cause Analysis

## Overview

A structured framework for identifying, analysing, and prioritising sources of bias in AI systems. Translates abstract bias concepts into actionable methods applicable during model development, evaluation, and deployment.

Bias can enter at multiple stages — from data collection to labelling, modelling, and deployment. This component helps teams find it, score it, and decide what to address first.

## 1. Intersectionality

**Why It Matters:**

Single-axis analysis misses bias that only appears at the intersection of identities — for example, race × gender, or age × disability × socioeconomic status. A model may appear fair when each attribute is examined alone but produce discriminatory outcomes for specific combinations. See the [Fairness Assessment Playbook Intro](./fairness_assessment_playbook_intro.md#4-intersectionality) for the shared foundation.

**Key Benefits:**

- **Uncover Hidden Bias** — Intersectional subgroups often carry the highest-severity bias yet remain invisible in single-attribute audits. Representation bias may be undetectable at the gender level but severe for women of colour.
- **Accurate Prioritisation** — Priority scores based on single attributes underestimate real harm. Scoring across intersectional groups ensures critical patterns are not buried by aggregation.
- **Targeted Remediation** — Knowing which intersection carries the bias directs interventions more precisely, avoiding broad fixes that fail specific subgroups.

**Implementation Guidance:**

- Apply all detection methods (Sections 2.a–2.b) across intersectional subgroups, not just individual protected attributes.
- Score each bias source in the prioritisation framework (Section 2.c) for intersectional groups separately where data allows.
- Flag small subgroup sample sizes and apply statistical techniques (bootstrapping, Bayesian estimation) to handle them.
- Document intersectional findings in the output template alongside single-attribute results.

## 2. Implementation Framework

### 2.a. Bias Type Taxonomy

Pre-deployment — detectable during development and testing:

| Bias Type | Definition | Indicators |
| --- | --- | --- |
| **Historical Bias** | Pre-existing social inequities encoded in data, regardless of sampling or feature selection. | Target variables reflecting historical discrimination; correlations mirroring societal inequities. |
| **Representation Bias** | Populations sampled or measured unequally in training data. | Demographic imbalances vs. target population; quality disparities across groups; systematic measurement differences. |
| **Measurement Bias** | Attributes measured, proxied, or operationalised differently across groups. | Different measurement approaches per group; proxy variables with varying accuracy; inconsistent label quality. |
| **Aggregation Bias** | Distinct populations with different feature-outcome relationships combined into a single model. | One-size-fits-all models for heterogeneous populations; unexplained subgroup performance gaps. |
| **Learning Bias** | Modelling choices that amplify or create disparities. | Overfitting majority patterns; regularisation penalising minority patterns; optimisation misaligned with fairness goals. |
| **Evaluation Bias** | Testing procedures that do not reflect real-world performance. | Test datasets differing from deployment context; metrics ignoring fairness dimensions; insufficient subgroup disaggregation. |

Post-deployment — detectable only after the system is live:

| Bias Type | Definition | Indicators |
| --- | --- | --- |
| **Deployment Bias** | Bias arising from how the system is used in practice once live. | Context shifts between training and deployment; user interactions reinforcing bias; feedback loops amplifying disparities over time. |

> Deployment Bias cannot be fully audited before launch. Establish monitoring thresholds at deployment and treat it as an ongoing check, not a pre-launch gate.

### 2.b. Detection Methodology

| Bias Type | Detection Methods |
| --- | --- |
| **Historical Bias** | Historical outcome comparison; correlation analysis with protected attributes; counterfactual testing; label provenance review. |
| **Representation Bias** | Demographic distribution vs. population benchmarks; sample size checks; missing data pattern analysis; data quality audits. |
| **Measurement Bias** | Proxy validation across groups; feature distribution comparison; label consistency checks; audit of data collection instruments. |
| **Aggregation Bias** | Group-wise performance analysis; conditional feature-outcome testing; segmented model experiments. |
| **Learning Bias** | Fairness metric evaluation (equal opportunity gaps); regularisation impact analysis; feature importance comparison; stress testing on minority subpopulations. |
| **Evaluation Bias** | Train-test-deploy comparison; metric adequacy review; intersectional evaluation; threshold sensitivity analysis. |
| **Deployment Bias** | Post-deployment monitoring; user interaction analysis; feedback loop detection; context shift monitoring. |

### 2.c. Prioritisation Framework

Score each identified bias source on four dimensions (1–5). Intervention Feasibility is tracked separately to inform sequencing, not urgency.

| Dimension | Scale |
| --- | --- |
| **Severity** — Potential harm if unaddressed | 1 = minimal harm → 5 = severe, life-affecting harm |
| **Scope** — Proportion of decisions or individuals affected | 1 = very few → 5 = affects most users |
| **Persistence** — Whether effects compound over time | 1 = isolated → 5 = self-reinforcing feedback loop |
| **Historical Alignment** — Connection to known historical discrimination patterns | 1 = no connection → 5 = directly rooted in historical inequity |

**Priority Score = (Severity × 0.35) + (Scope × 0.25) + (Persistence × 0.25) + (Historical Alignment × 0.15)**

| Score | Priority |
| --- | --- |
| ≥ 4.0 | High — Address before deployment |
| 3.0–3.9 | Medium — Address post-launch or in next sprint |
| < 3.0 | Low — Monitor; no immediate action required |

**Intervention Feasibility** (1 = very hard, 5 = easy to fix) — record separately in the output template. Use it to sequence work: among High-priority items, tackle the most feasible first to build momentum while harder fixes are planned.

## 3. Output Template

Complete one row per identified bias source.

| Bias Type | Description / Evidence | System Component Affected | Severity (1–5) | Scope (1–5) | Persistence (1–5) | Historical Alignment (1–5) | Priority Score | Priority Level | Feasibility (1–5) | Recommended Action |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| | | | | | | | | | | |
| | | | | | | | | | | |
| | | | | | | | | | | |

Next step: Take High-priority rows directly into the [Fairness Action Playbook](../fairness_action_playbook/fairness_action_playbook_intro.md) to select the appropriate pre-processing, in-processing, or post-processing remedy.

## 4. Usage Guide

### Step-by-Step

1. Review Historical Context Assessment outputs — identify high-risk groups and systemic patterns.
2. Map patterns to the model — examine how historical inequities may influence features, labels, or model design, including intersectional effects.
3. Select relevant bias types — use the taxonomy to identify which bias types apply to your system.
4. Apply detection methods — run the recommended analyses per bias type; audit intersectional groups.
5. Score and prioritise — complete the output template; categorise as High, Medium, or Low.
6. Hand off — feed High-priority findings into the Fairness Action Playbook; log Medium items as fairness debt for the next sprint.

### Tips

- Audit intersectionally — examine multiple protected attributes together.
- Document decisions — record the rationale for detection choices and scores.
- Link to fairness definitions — connect each bias to the definition it threatens.
- Monitor continuously — deployment bias in particular requires ongoing checks after launch.
