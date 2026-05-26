# Fairness Definition Guide

Fairness definition selection is especially important in high-stakes domains such as lending, hiring, and medical assessment, where different definitions lead to materially different outcomes.

---

## Contents

1. Intersectionality in Fairness Definitions
2. Implementation Framework
   1. Fairness Definition Catalog
   2. Definition Selection Decision Tree
   3. Conflict Resolution
3. Usage Guide
   1. Implementation Process
   2. Multi-Dimensional Fairness Assessment

---

## 1. Intersectionality in Fairness Definitions

Intersectionality ensures fairness definitions account for overlapping identities (race, gender, age, disability, socioeconomic status) rather than treating each attribute in isolation.

**Why it matters:**

- A hiring algorithm may show no gender bias and no racial bias individually, but Latina women could still face disproportionately lower selection rates due to combined effects.
- In healthcare, a diagnostic AI trained mainly on older men may perform worse for young women of colour, even if single-axis metrics appear acceptable.

**In practice:**

- Apply fairness metrics to intersectional subgroups, not just individual protected attributes.
- Monitor outputs continuously and document assumptions.
- Engage domain experts to interpret intersectional outcomes and guide corrective action.

---

## 2. Implementation Framework

### 2.a. Fairness Definition Catalog

| Definition | When to Use | Key Limitation | Example |
| --- | --- | --- | --- |
| **Demographic Parity** — Equal probability of a positive outcome across groups. `P(Ŷ=1\|A=a) = P(Ŷ=1\|A=b)` | Equal representation is the primary goal; domains with historical exclusion. | May reduce accuracy if base rates differ legitimately. | STEM career ads shown at equal rates across gender groups. |
| **Equal Opportunity** — Qualified individuals receive positive predictions at equal rates across groups. `P(Ŷ=1\|Y=1,A=a) = P(Ŷ=1\|Y=1,A=b)` | False negatives are more harmful than false positives; labels are reliable. | Does not address false positive disparities. | Medical screening tool identifying patients with a condition equally across races. |
| **Equalized Odds** — Both true positive and false positive rates equal across groups. `P(Ŷ=1\|Y=y,A=a) = P(Ŷ=1\|Y=y,A=b)` | Both false positives and false negatives have significant consequences. | More complex to implement; typically reduces accuracy. | Recidivism system where both wrongful detention and wrongful release must be distributed fairly. |
| **Predictive Equality** — False positive rates equal across groups. | False positives are more harmful (e.g., wrongly flagging low-risk individuals). | Does not address false negative disparities. | Fraud detection where incorrectly flagging legitimate transactions is the primary concern. |
| **Calibration / Sufficiency** — Predicted probabilities match actual outcomes equally across groups. | Probabilistic scores are used directly in decisions. | Can conflict with equal opportunity or equalized odds. | Credit scoring where a 70% default probability means the same across demographic groups. |
| **Individual Fairness** — Similar individuals receive similar predictions. | Fine-grained fairness matters more than group-level parity. | Requires a meaningful similarity metric, which is hard to define. | Two candidates with near-identical CVs receiving comparable scores. |
| **Bounded Group Loss** — Maximum error for any group is capped. | Regression tasks; preventing any group from being systematically worse off. | Setting the bound requires domain judgement. | Salary prediction where no demographic group has a mean error exceeding a defined threshold. |
| **Exposure Parity** — Groups receive proportional exposure in rankings or recommendations. | Ranking or recommendation systems. | Exposure equality may conflict with relevance-based ranking. | Job recommendation engine surfacing roles to candidates across demographic groups at proportional rates. |

---

### 2.b. Definition Selection Decision Tree

Use this to select your primary fairness definition.

```
Is the output a ranking or recommendation?
├── YES → Use Exposure Parity (± Representation Parity for top-k)
└── NO
    Is the output continuous (regression)?
    ├── YES → Use Bounded Group Loss or Individual Fairness
    └── NO (classification)
        Are false negatives more harmful than false positives?
        ├── YES → Use Equal Opportunity
        └── NO
            Are both false positives AND false negatives harmful?
            ├── YES → Use Equalized Odds
            └── NO (false positives more harmful)
                Is equal representation the primary goal?
                ├── YES → Use Demographic Parity
                └── NO → Use Predictive Equality or Calibration
```

> The decision tree selects a primary definition. Always use at least one secondary metric to capture what the primary misses. Document why.

---

### 2.c. Conflict Resolution

When two stakeholders want incompatible fairness definitions — which happens — follow these steps:

1. **Name the conflict explicitly.** Write down which definitions are in tension and why (e.g., "Legal wants Demographic Parity; clinical team wants Equal Opportunity — these cannot both be fully satisfied simultaneously").
2. **Assess harm direction.** Which definition, if deprioritised, causes more harm to affected groups? Prioritise minimising the worst harm.
3. **Document the trade-off.** Record the decision, rationale, alternatives considered, and who approved it. This is a compliance requirement under EU AI Act Article 13.
4. **Escalate if unresolved.** If the team cannot agree, escalate to the ethics review body or Fairness Guild. Do not proceed without a documented decision.

---

## 3. Usage Guide

### 3.a. Implementation Process

1. **Confirm the selected fairness definition** — Record the primary definition chosen and the decision tree path that led to it.
2. **Reference historical context findings** — Note the key patterns from the Historical Context Assessment that influenced the choice.
3. **Assess error impact** — Identify which errors (false positives, false negatives, or both) cause the greatest harm and confirm the definition addresses them.
4. **Incorporate stakeholder perspectives** — Document priorities from users, affected communities, business owners, and compliance teams.
5. **Acknowledge trade-offs explicitly** — List fairness properties *not* satisfied by the chosen definition and the risks this creates.
6. **Define the monitoring plan** — Specify which metrics are tracked, how often, and what threshold change triggers further investigation.

---

### 3.b. Multi-Dimensional Fairness Assessment

Use this when a single definition is insufficient — typically in high-stakes or multi-stakeholder systems.

| Dimension | What to do |
| --- | --- |
| **Philosophical tensions** | Identify competing fairness values (equality vs. equity). Document your organisation's position and why. |
| **Contextual factors** | Analyse historical data for patterns of advantage/disadvantage. Map relevant legal requirements for your domain. |
| **Stakeholder conflicts** | Run structured engagement (surveys, focus groups). Give extra weight to perspectives of groups most affected by the system. |
| **Intersectional considerations** | Audit outcomes for users at the intersection of multiple marginalised identities. Implement specialist review where historical representation is limited. |
| **Impossibility constraints** | Document explicitly which fairness definitions conflict. Set a primary metric and minimum thresholds for secondary metrics. Track all of them. |
