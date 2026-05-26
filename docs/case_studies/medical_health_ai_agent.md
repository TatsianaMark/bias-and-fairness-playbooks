# Applying the Fairness Audit Playbook to a Medical Health AI Agent

## Context

The system under review is a Medical Health AI Agent designed to answer patient questions about symptoms, treatment options, medication usage, and care navigation.
It is intended to support clinical decision-making and is deployed to a diverse patient population across age, gender, race, disability status, and socioeconomic backgrounds.
Because healthcare systems have a long history of unequal access, treatment disparities, and biased medical knowledge, a historical-context-aware fairness audit is required before deployment.

## Scenario Note

This case study is an illustrative scenario, not based on a specific real-world dataset. It is designed to demonstrate how the Fairness Assessment Playbook applies to a high-risk healthcare AI system. Historical references to discrimination patterns (e.g., the Tuskegee Syphilis Study, J. Marion Sims' experiments) cite documented events and are included to ground the audit in verified historical context.

## Step 1. Historical Context Assessment

---

### Historical Pattern Identification

In healthcare, historical discrimination patterns primarily disadvantaged racial and ethnic minorities, particularly Black and other people of color, alongside women, low-income groups, and those with disabilities. These patterns stem from slavery-era experiments, segregation, and policy exclusions that embedded inequities in access, treatment, and medical knowledge.

---

### Key Disadvantaged Groups

| Group | Historical Pattern |
|-------|-------------------|
| Racial and ethnic minorities (e.g., Black Americans) | Enslaved Black women endured non-consensual surgeries (e.g., J. Marion Sims' VVF experiments without anesthesia). The Tuskegee Syphilis Study (1932–1972) withheld treatment from Black men, fostering lasting distrust. |
| Women | Particularly women of color faced compounded biases in gynecology and obstetrics due to experimental exploitation during slavery. |
| Low socioeconomic status | Jim Crow laws and occupational segregation limited insurance and care access. Medicaid flexibility allowed underfunding for minorities. |

---

### Patterns of Discrimination

- Segregation persisted via "separate but equal" hospitals until Medicare's 1965 enforcement
- Race-based clinical adjustments (e.g., lung function) perpetuated flawed genetic myths
- Structural racism shaped Medicare/Medicaid inequities, denying admissions despite nondiscriminatory policies
- Pain assessment biases underrate minorities' needs, leading to systematic undertreatment

---

### Implications for the AI Agent

Training data likely inherits these biases, risking inaccurate symptom advice or treatment recommendations for historically marginalized groups such as Black patients or low-income users. Audits must trace data provenance to these patterns for fairness.

---

### How Previous Technologies Reflected and Reinforced Social Hierarchies

**Reflection and Reinforcement**

- Early medical tools such as pulse oximeters and spirometers incorporated race-based corrections that assumed biological inferiority of Black patients, leading to inaccurate readings and undertreatment for minorities
- Hospital segregation technologies — separate wards enabled by Hill-Burton Act funding (1946–1960s) — physically entrenched racial divides, limiting Black patients' access to advanced equipment

**Amplification via Diffusion**

- Innovative devices like CT scanners and dialysis machines followed a hierarchical diffusion pattern: higher socioeconomic groups adopted them first, widening gaps before eventual trickle-down reduced disparities over decades
- Expensive insulin pumps and robotic surgery systems amplified inequalities by favouring affluent users, entrenching class-based hierarchies
- Digital health apps and EHRs mirrored this — low-income and minority groups faced access barriers, reinforcing historical exclusion

**Rare Challenges to the Pattern**

- Community Health Centers (1965 onward) and basic technologies like manual blood pressure cuffs democratized care for underserved populations, narrowing some gaps by design
- However, these were exceptions — most tools had neutral-to-negative effects on biases without intentional equity safeguards

---

### AI Agent Relevance

The Medical Health AI Agent risks inheriting these patterns if trained on biased historical data from race-adjusted tools or segregated records, potentially underdiagnosing symptoms in disadvantaged groups. Audits should probe training data provenance for such reinforcements.

Recurring bias mechanisms include:

- Data feedback loops — biased historical records training new tools
- Implicit clinical stereotypes embedded in labels and proxy features (e.g., zip codes for SES)
- Systematic data missingness from disadvantaged groups due to access barriers

---

### Pattern-to-Risk Mapping

| Historical Pattern | AI System Risk | Target Audit Area |
|-------------------|---------------|-------------------|
| Racial undertreatment | Training data skewed by records from segregated care systems underrepresenting minorities | Data provenance |
| Gender bias in pain assessment | Models undervalue women's symptom reports via historical clinical labels | Input features and labeling |
| Socioeconomic disparities | Missingness in low-income groups biases model toward affluent outcomes | Data completeness |
| Race-based clinical adjustments | Model architecture inherits legacy race-based corrections | Feature selection and encoding |
| Intersectional subgroups | Evaluation metrics overlook combined race-gender-SES interactions | Subgroup validation |

Target audits on data provenance, feature selection, and subgroup validation to mitigate these mappings.

---

### Feature Encoding and Historical Classification Systems

Race-based systems such as eGFR and spirometer corrections — rooted in 19th-century pseudoscience assuming Black patients' biological inferiority — influence feature definitions by embedding race as a binary or categorical proxy variable in ML pipelines. These legacy encodings carry over into symptom mappings or treatment recommendation features, where race uplifts kidney function estimates or downgrades lung capacity for minorities, skewing AI advice toward White-majority norms.

Key risks for the Medical Health AI Agent:

- Symptom queries may encode biased thresholds (e.g., higher pain tolerance assumed for men of colour), perpetuating undertreatment via feature engineering inherited from clinical norms
- Encodings often use zip code or surname as implicit race proxies, masking disparities while amplifying historical hierarchies in model inputs

Mitigation: Audit features for race-derived scalars and replace with social determinants such as SES or access metrics.

---

### Historical Performance Disparities

Historical performance gaps manifest in healthcare AI models as:

- Lower accuracy for racial minorities, particularly Black patients, due to underrepresented or mislabeled data from unequal testing and treatment
- Higher error rates in predicting illness severity — underestimating sepsis risk or chronic conditions — because training data reflects less frequent testing and lower recorded care costs as proxies for health needs
- 20–30% worse sensitivity in diagnostics for women and low-SES groups compared to White affluent males in pain and symptom recognition

For the Medical Health AI Agent, this risks flawed symptom advice or care navigation for disadvantaged users, amplifying historical undertreatment via poor subgroup precision.

Mitigation: Evaluate with intersectional metrics such as equalized odds across race-gender-SES strata to detect and flag disparities pre-deployment.

---

### Evaluation Metric Bias

| Issue | Detail |
|-------|--------|
| Aggregate metrics mask disparities | AUC and F1-score, inherited from clinical trials dominated by White males, optimize for well-represented groups while undervaluing sensitivity for symptoms in women or minorities |
| Biased threshold calibration | Thresholds calibrated on biased data (e.g., lower pain scores for Black patients) lead to conservative treatment recommendations, perpetuating undertreatment in AI outputs |
| Overall accuracy prioritised over subgroup fairness | Historical optimization priorities in healthcare — favouring overall accuracy and cost-efficiency for majority-White, affluent patients — embed higher diagnostic thresholds for minorities |

Mitigation: Adopt demographic parity and equalized odds metrics with intersectional thresholds to realign evaluations.
 
## Prioritization Framework

| Dimension | Rating | Summary |
|-----------|--------|---------|
| Historical Connection | Very Strong | Training data directly embeds clinical biases (e.g., eGFR race-adjusted algorithms, segregated-era records) |
| Potential Harm | Severe and Immediate | Misdiagnoses, delayed treatments, deepened healthcare distrust among vulnerable groups |
| Bias Visibility | Low | Subtle patterns hidden in aggregate metrics — only detectable through targeted disaggregated audits |

---

## Historical Connection — Very Strong

The link between historical healthcare inequities and the Medical Health AI Agent is very strong:

- Symptom advice and treatment recommendations directly rely on clinical data steeped in historical bias patterns
- Training datasets from segregated eras and race-adjusted algorithms (e.g., eGFR) embed biases into core components
- Evidence from analogous AI systems shows 20–50% worse performance for minorities
- Real-time clinical decision support for diverse users amplifies risks, mirroring past tools' failures in equitable care

Action: Prioritize high-mitigation efforts here over weaker historical ties in less data-dependent applications.

---

## Potential Harm — Severe and Immediate

If historical patterns recur in the Medical Health AI Agent, harm is severe and immediate:

| Affected Group | Risk | Consequence |
|----------------|------|-------------|
| Racial minorities | Undertriaged symptom advice (e.g., ignoring Black patients' pain) | Worsened outcomes — untreated sepsis, 20–30% higher mortality gaps |
| Low-SES users | Compounded errors in care navigation | Exacerbated access barriers, long-term chronic disease disparities |
| Female users | Gender-biased pain assessment | Systematic undertreatment, misdiagnosis |
| All groups at scale | Societal damage from mass deployment | Eroded trust, legal liabilities under EU AI Act |

Action: Mitigation urgency is critical — consequences are high-stakes and irreversible.

---

## Bias Visibility — Low

Potential bias exhibits low visibility for the following reasons:

- Subtle historical patterns (e.g., race-adjusted clinical encodings, underrepresented minority data) manifest indirectly in symptom advice rather than as overt flags
- Aggregate metrics (e.g., overall accuracy) conceal subgroup disparities — targeted audits reveal 20–30% error gaps for Black or low-SES users
- Proxy features such as zip codes obscure racial biases inherited from segregated records, evading standard testing
- High deployment scale across diverse patients heightens hidden risks

Action: Low visibility elevates prioritization for comprehensive, disaggregated fairness checks.

---

## Prioritization Order

| Priority | Bias Type | Rationale |
|----------|-----------|-----------|
| 1st | Racial undertreatment and race-based clinical adjustments | Directly embedded in training data — drives 20–50% accuracy drops for Black patients in symptom triage |
| 2nd | Gender-biased pain recognition | Historical undervaluation of women's symptoms skews model thresholds across diverse users |
| 3rd | Socioeconomic data missingness | Amplifies disparities in care navigation for low-SES groups via incomplete representations |
| Critical | Intersectional combinations (e.g., Black women) | Compounded risks demand dedicated subgroup testing | 

## **Historical Pattern Risk Classification Matrix**

### **Matrix Components**

1. **Historical Pattern**: Specific documented pattern of discrimination with historical evidence.
2. **Severity**: Impact of this bias if perpetuated (High/Medium/Low):
    - High: Directly impacts fundamental rights or life outcomes.
    - Medium: Creates significant disparities in opportunities or resources.
    - Low: Creates differential experiences but with limited material impact.
3. **Likelihood**: Probability of this pattern manifesting in AI systems:
    - High: Pattern frequently appears in similar systems.
    - Medium: Pattern occasionally appears in similar systems.
    - Low: Pattern rarely appears in similar systems.
4. **Relevance**: Applicability to the specific AI system being developed:
    - High: Direct applicability to system's domain/purpose.
    - Medium: Partial applicability to certain system components.
    - Low: Limited applicability but potential for manifestation.
5. **Priority Score**: Calculated as Severity + Likelihood + Relevance (sum, max = 9):
    - 7–9: Critical — Requires immediate mitigation.
    - 5–6: High — Requires mitigation before launch.
    - 3–4: Medium — Monitor after launch.
    - 1–2: Low — No action needed.

### **Historical Pattern Risk Classification Matrix**

| **Historical Pattern** | **Severity (1-3)** | **Likelihood (1-3)** | **Relevance (1-3)** | **Priority Score (Sum)** | **Priority Level** |
| --- | --- | --- | --- | --- | --- |
| Racial undertreatment of minorities, especially Black patients, via experiments like Tuskegee and biased pain management | 3 | 3 | 3 | 9 | Critical |
| Race-based clinical adjustments in tools like eGFR and spirometers, assuming biological differences | 3 | 2 | 2 | 7 | High |
| Gender disparities in symptom recognition, undervaluing women's pain reports historically | 2 | 3 | 2 | 7 | High |
| Segregated hospital access and care under Jim Crow laws and Hill-Burton funding, limiting minority exposure to advanced treatments | 2 | 2 | 2 | 6 | Medium |
| Socioeconomic barriers causing data missingness for low-income groups in medical records | 2 | 2 | 2 | 6 | Medium |
| Biased clinical trials dominated by White male participants, skewing diagnostic norms | 2 | 1 | 1 | 4 | Low |


### **Using the Historical Pattern Risk Classification Matrix**

Once you have completed the **Historical Pattern Risk Classification Matrix**, it becomes a central tool to guide your fairness assessment and mitigation efforts. Here's how to use it effectively:

1. **Identify High-Risk Components**
    - Review the matrix to see which system components (data, features, model outputs, evaluation metrics) are linked to historical biases with **high severity and high visibility**.
    - Focus your initial mitigation efforts on these areas, as they pose the greatest risk to fairness.
2. **Prioritize Interventions**
    - Use the risk scores in the matrix to determine which patterns require immediate attention versus monitoring.
    - Patterns with high harm potential but low visibility may need **specialized evaluation methods** or additional stakeholder review.
3. **Guide Feature and Data Audits**
    - Map high-priority historical patterns to specific datasets or feature definitions.
    - Investigate whether these features reflect historical inequities and whether they require re-encoding, balancing, or augmentation.
4. **Inform Model Testing and Evaluation**
    - Align your fairness tests with the highest-risk historical patterns identified.
    - Ensure that evaluation metrics capture potential disparities across affected groups.
5. **Support Stakeholder Communication**
    - The matrix provides a visual and structured summary of risk areas, making it easier to **communicate findings and proposed interventions** to non-technical stakeholders.
6. **Update Iteratively**
    - Treat the matrix as a living document. As new data or insights emerge, update the classifications to reflect evolving risks.
    - Integrate updates into your CI/CD pipeline or sprint planning to ensure ongoing fairness monitoring.

**Outcome:** Using the matrix this way ensures that your fairness efforts are data-driven, risk-prioritized, and actionable, directly linking historical context to practical model interventions.

# Step 2 Fairness Definitions

After completing Step 1: Historical Pattern Identification, you now have a clear view of the groups, features, and system components most affected by past inequities. This step provides the evidence base for selecting the right fairness definitions

## Demographic Parity (Statistical Parity)

**Definition**: The probability of receiving a positive outcome (e.g., treatment recommendation) must be identical across protected groups like race, gender, or SES.

**Mathematical form**: P(Ŷ=1 | A=a) = P(Ŷ=1 | A=b) for groups a, b.

**When to Use**: Prioritizes equal access in symptom advice, countering historical exclusion in healthcare.

**Limitations**: Ignores differing symptom base rates; risks over-recommending to low-risk minorities.

**Example**: AI Agent assigns care navigation prompts equally to Black and White patients reporting chest pain, addressing segregation legacies.

## Equal Opportunity

**Definition**: Qualified patients (true condition present) receive positive predictions at equal rates across groups.

**Mathematical form**: P(Ŷ=1 | Y=1, A=a) = P(Ŷ=1 | Y=1, A=b).

**When to Use**: Critical for avoiding false negatives in treatment options, where labels reflect true need despite biased history.

**Limitations**: Relies on potentially skewed ground truth from undertreated groups.

**Example**: Ensures patients with actual infections get medication advice equally, regardless of gender or SES data gaps.

## Equalized Odds

**Definition**: True positive and false positive rates equal across groups for all outcomes.

**Mathematical form**: P(Ŷ=1 | Y=y, A=a) = P(Ŷ=1 | Y=y, A=b) for y ∈ {0,1}.

**When to Use**: Balances both over- and under-treatment risks in high-stakes clinical navigation.

**Limitations**: Demands accuracy trade-offs; complex for intersectional checks (e.g., Black women).

**Example**: Matches error rates in flagging urgent symptoms for low-SES vs. affluent users, mitigating trial biases.

Fairness selection here embodies equity over equality, prioritizing harm reduction from historical disparities in diverse patient support

## Selected Definition Documentation for Medical Health AI Agent

**Primary Fairness Definition Selected**: Equal Opportunity (Equality of True Positive Rates)

**Mathematical Formulation**: P(Ŷ=1 | Y=1, A=a) = P(Ŷ=1 | Y=1, A=b) for protected groups (race, gender, SES).

**Selection Rationale**:

**Connection to historical context**: Racial undertreatment (e.g., Tuskegee, pain biases) and race-based adjustments (e.g., eGFR) systematically overlooked qualified needs in minorities, per Historical Context Assessment. This ensures patients with true conditions (Y=1) get equal symptom/treatment advice despite past disparities.

**Application requirements**: Prioritizes avoiding false negatives in high-stakes care navigation—missing real symptoms harms more than over-flagging mild cases.

**Stakeholder priorities**: Clinical teams seek reliable positives; patients/DEI focus on equitable access countering segregation legacies.

**Trade-off Acknowledgment**:

**Fairness properties not satisfied**: Skips demographic parity (equal recommendation rates) and equalized odds (both error types); representation may vary by base rates.

**Performance implications**: May lower precision by 10-15% via broader recommendations for underrepresented groups, but maintains high recall with negligible outcome impact.

**Monitoring approach**: Track true positive equality across race-gender-SES intersections monthly, plus secondary demographic parity; flag false negative drifts >5% for audit.

# Step 3. Bias Root Cause Analysis

Once you have completed Step 2: Fairness Definition Selection, you have a set of explicit fairness goals tailored to the AI system based on historical patterns and identified risks. The next step is to operationalize these goals by identifying where bias can enter the system.

| **Bias Type**        | **Example in AI Medical Agent**                                                     | **Risk / Impact**                                                    | **Detection Method**                                                                 |
|----------------------|-----------------------------------------------------------------------------------|----------------------------------------------------------------------|--------------------------------------------------------------------------------------|
| Historical Bias       | Historical healthcare disparities: Black and Hispanic patients underrepresented in clinical trials | Systematic underdiagnosis or misclassification for marginalized groups | Compare predictions to documented healthcare inequities; correlation analysis by demographics |
| Representation Bias   | Training data mainly from young adults; elderly and rare conditions underrepresented | Lower accuracy and generalization for underrepresented patients      | Compare dataset demographics to target population; analyze missing data and quality metrics |
| Measurement Bias      | Use of proxies like insurance type; inconsistent symptom coding across hospitals   | Misinterpretation of patient inputs; biased recommendations          | Validate proxy accuracy across groups; audit labeling processes                        |
| Aggregation Bias      | Single model applied to all age groups/comorbidities                               | Performance gaps between subgroups; overgeneralized guidance         | Evaluate subgroup performance; conditional feature analysis                            |
| Learning Bias         | Model overfits majority patterns; penalizes rare conditions                        | False negatives for minority or rare cases; reinforces majority patterns | Evaluate fairness metrics (equal opportunity, equalized odds); analyze false negative/positive rates by group |
| Evaluation Bias       | Validation set lacks demographic diversity; metrics ignore fairness                | Model appears accurate overall but underperforms for specific populations | Disaggregate evaluation metrics by demographics; threshold sensitivity analysis        |
| Deployment Bias       | Patients with low digital literacy or different language use                        | Unequal access to guidance; feedback loops reinforce disparities      | Monitor real-world usage by demographics; detect patterns indicating misrecommendations |


### Bias Source Prioritization Table

Priority Score = (Severity × 0.35) + (Scope × 0.25) + (Persistence × 0.25) + (Historical Alignment × 0.15). Feasibility is tracked separately to sequence work — among High-priority items, address the most feasible first.

| **Bias Source** | **Severity (1-5)** | **Scope (1-5)** | **Persistence (1-5)** | **Historical Alignment (1-5)** | **Priority Score** | **Priority Level** | **Feasibility (1-5)** |
|-----------------|------------------|----------------|----------------------|-------------------------------|-------------------|-------------------|-----------------------|
| Historical Bias in Clinical Data | 5 | 5 | 4 | 5 | 4.75 | High | 3 |
| Representation Imbalance (Age, Gender, Ethnicity) | 5 | 4 | 3 | 4 | 4.2 | High | 3 |
| Learning Bias from Overfitting Majority Patterns | 5 | 4 | 4 | 4 | 4.45 | High | 3 |
| Measurement Bias in Symptom Encoding | 4 | 4 | 3 | 4 | 3.85 | Medium | 4 |
| Aggregation Bias Across Patient Groups | 3 | 4 | 3 | 3 | 3.3 | Medium | 4 |
| Evaluation Bias from Limited Validation Sets | 3 | 4 | 3 | 3 | 3.2 | Medium | 4 |

**Deployment Bias** (digital literacy gaps, language barriers) is post-deployment only and cannot be fully scored pre-launch. Monitoring plan: track recommendation rate disparities by demographic group; alert if divergence exceeds 10% within 30 days of launch.

**Next step:** High-priority items (Historical, Representation, Learning Bias) feed into the [Fairness Action Playbook](../playbooks/fairness_action_playbook/fairness_action_playbook_intro.md) — Pre-Processing for data imbalances, In-Processing for learning bias.


# Step 4. Fairness Metrics

Once your team has completed Step 3: Bias Source Identification and prioritized the most critical bias sources, the next step is to define how to measure and monitor fairness. This ensures that identified risks are not just theoretical but are actively tracked and mitigated during development, deployment, and operation.

| Bias Source | Problem Type | Fairness Definition | Recommended Metrics | Error Direction Importance | Uncertainty / Score Usage | Audit Verification / Notes |
| --- | --- | --- | --- | --- | --- | --- |
| Historical Bias in Clinical Diagnoses | Classification | Equal Opportunity | True Positive Rate Difference | False negatives more harmful → Negative Residual Difference | Probabilistic risk scores → Calibration by group, Prediction Interval Coverage | Retrospective validation: compare historical underdiagnosis rates to model TPR across groups |
| Representation Imbalance in Training Data | Classification | Demographic Parity | Selection Rate Difference, Statistical Parity Difference | Both error types harmful → Absolute Residual Difference | Document limitations if no uncertainty estimates | Monitor minority patient representation; compare to population benchmarks; use audit coverage metrics |
| Measurement Bias in Symptom Encoding | Classification | Individual Fairness | Individual Consistency, Input–Output Sensitivity | Underprediction more harmful → Negative Residual Difference | Probabilistic outputs → Prediction Interval Coverage | Cross-annotator consistency checks; validate predictions against known clinical outcomes |
| Aggregation Bias from Mixed Populations | Classification | Equalized Odds | TPR Difference, FPR Difference | Both errors harmful → Absolute Residual Difference | Probabilistic outputs → Calibration metrics | Subgroup performance audits; compare aggregated vs. disaggregated performance |
| Learning Bias from Model Regularization | Classification | Equalized Odds | TPR Difference, FPR Difference | False negatives more harmful → Negative Residual Difference | Probabilistic outputs → Calibration by group | Test audit predictions against minority patient outcomes; track if fairness constraints are enforced |
| Evaluation Bias due to Non-Representative Test Set | Classification | Predictive Equality | False Positive Rate Difference | False positives more harmful → Positive Residual Difference | Document limitations | Audit test set diversity; compare predictions to real-world deployment outcomes |
| Deployment Bias from Patient Interaction | Ranking / Recommendation | Exposure Parity | Exposure Ratio, NDCG Difference | Both errors harmful → Absolute Residual Difference | Probabilistic risk scores → Prediction Interval Coverage | Continuous monitoring metrics; track interaction disparities; calibrate system periodically |

### **How to Use This Table in Practice**

1. Start with Bias Sources: Pull your high-priority bias sources from Step 3: Bias Source Identification.
2. Identify Problem Type: Classify each bias according to your system outputs: Classification, Regression, Ranking.
3. Map Fairness Definition: Use the selected fairness definitions from Step 2 to ensure alignment between ethical priorities and measurement.
4. Select Metrics: Choose metrics that quantify the impact of each bias, considering both error type importance and whether probabilistic outputs are provided.
5. Validate Statistically: Apply bootstrapping, Bayesian approaches for small samples, and calculate confidence or credible intervals.
6. Visualize & Communicate: Use Fairness Disparity Charts and Intersectional Heatmaps to track disparities by demographic and intersectional groups.
7. Monitor Continuously: Integrate metrics into post-deployment monitoring to detect emerging biases and adjust the system as needed.

**Thresholds:** Apply defaults from the [Fairness Metrics playbook](../playbooks/fairness_assessment_playbook/fairness_metrics.md#2b-default-thresholds). Given this is a high-risk healthcare system, tighten TPR Difference to ≤ 0.03 before production.

**Tooling:** Fairlearn for TPR/FPR metric computation and disparity charts; Evidently AI for post-deployment drift monitoring.

**Re-audit triggers:**
- Model retrained on new patient data → re-run full Step 4
- Monitoring alert exceeds threshold → re-run affected metrics within 5 days
- New patient demographic added to scope → re-run intersectional evaluation

## Next Step: Fairness Action Playbook

The assessment above produces three High-priority bias sources, one primary fairness definition (Equal Opportunity), and a set of metrics to track. These feed directly into the [Fairness Action Playbook](../playbooks/fairness_action_playbook/fairness_action_playbook_intro.md).

| Bias Source | Priority | Recommended Intervention Stage |
|---|---|---|
| Historical Bias in Clinical Data | High | Pre-Processing — reweighting to reduce label–protected-attribute correlation |
| Representation Imbalance (Age, Gender, Ethnicity) | High | Pre-Processing — resampling or synthetic data generation for underrepresented groups |
| Learning Bias from Overfitting Majority Patterns | High | In-Processing — fairness-constrained training with an Equal Opportunity constraint |
| Measurement Bias in Symptom Encoding | Medium | Pre-Processing — proxy variable transformation (Disparate Impact Removal) |
| Aggregation Bias Across Patient Groups | Medium | In-Processing — subgroup-specific evaluation and weighted sampling during training |
| Evaluation Bias from Limited Validation Sets | Medium | Post-Processing — group-specific threshold calibration on a representative held-out set |
| Deployment Bias (digital literacy, language barriers) | Monitor | Post-deployment only — track recommendation rate disparities by demographic group; alert if divergence exceeds 10% within 30 days of launch |

The primary fairness definition (Equal Opportunity) maps to the In-Processing stage using a True Positive Rate Parity constraint, followed by Post-Processing calibration to correct residual gaps. For a fully worked end-to-end example using the same intervention sequence, see the [Loan Approval Case Study](./loan_approval_fairness_e2e_case.md).

---

## References

- Tuskegee Syphilis Study (1932–1972) — U.S. Public Health Service
- J. Marion Sims' gynecological experiments on enslaved women (1840s)
- Hill-Burton Act (1946) — Hospital Survey and Construction Act
- eGFR race-based adjustments — National Kidney Foundation (revised 2021)
- Obermeyer, Z. et al. (2019) — "Dissecting racial bias in an algorithm 
  used to manage the health of populations." Science, 366(6464), 447–453.
