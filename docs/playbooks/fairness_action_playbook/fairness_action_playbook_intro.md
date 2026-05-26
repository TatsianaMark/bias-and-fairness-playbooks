# Fairness Action Playbook

## Overview

AI systems increasingly support decision-making across multiple domains, but growing concerns highlight potential systemic bias. Existing fairness tools often focus only on audits, leaving interventions inconsistent and ad hoc.

The Fairness Action Playbook provides a structured framework to help engineering and data science teams implement fairness interventions consistently, effectively, and at scale. For 3rd-party AI APIs, the playbook guides system-level interventions,including input handling, output evaluation, and post-processing adjustments — even when model internals are inaccessible.

> **Scope (2026):** This playbook covers classical ML systems — classification and regression models with structured inputs and discrete outputs. LLM and generative AI systems introduce distinct fairness challenges (embedding bias, output bias in text generation, RLHF annotator bias, prompt sensitivity disparities) that are not fully addressed by the techniques in this playbook. See the Adaptability Guidelines for guidance on where classical interventions transfer and where they do not.

The playbook aims to:

- Standardize fairness interventions across AI systems.
- Provide clear, actionable workflows without requiring constant expert oversight.
- Integrate causal analysis, data transformation, model constraints, and threshold adjustments into a coherent strategy.
- Ensure intersectional fairness is considered across all components.
- Support practical, scalable adoption in real-world organizational contexts.

Using this playbook, organizations can:

- Develop adaptable workflows for fairness interventions across ML systems.
- Communicate technical fairness trade-offs clearly to both executives and technical teams.
- Translate mathematical fairness concepts into actionable implementation steps.
- Balance thorough fairness assessments with practical time and resource constraints.
- Validate interventions across fairness, model performance, and business outcomes.

---

## Components

The playbook integrates four core toolkits:

1. Causal Analysis – Maps how protected attributes influence downstream outcomes.
2. Pre-Processing methods – Corrects biased data representations before model training.
3. In-Processing methods – Embeds fairness considerations directly into model training.
4. Post-Processing methods – Adjusts predictions or thresholds to reduce unfair outcomes after model training.

Workflow: Each component builds on the previous one, creating an end-to-end fairness intervention pipeline. Teams can implement interventions consistently while reserving expert input for complex cases.

---

## How to Use This Playbook

### 1. General Workflow

1. Identify dataset and model type; map bias pathways using the Causal Analysis.
2. Select toolkit(s) based on bias type and deployment constraints using the Decision Checklist below.
3. Apply Pre-Processing methods to correct biased input data.
4. Use In-Processing strategies to embed fairness into model training.
5. Adjust predictions with Post-Processing methods as needed.
6. Validate outcomes across fairness, accuracy, and intersectional metrics.
7. Document decisions, trade-offs, and monitoring plan for accountability and reproducibility.
8. Reassess periodically as data or model changes.

### 2. Decision Checklist

| Bias Type / Scenario                        | Recommended Toolkit(s)         | Key Metrics to Check           | Notes / Risks                                           |
|--------------------------------------------|-------------------------------|-------------------------------|--------------------------------------------------------|
| Representation disparities                  | Pre-Processing               | Distribution parity, subgroup coverage | May require synthetic data if underrepresented groups are too small |
| Proxy discrimination                        | Pre-Processing / In-Processing | Correlation with protected attributes | Transformation may reduce predictive power |
| Label bias / historical discrimination      | Pre-Processing / In-Processing | False positive/negative rates by group | Prejudice removal may introduce new bias if miscalibrated |
| Model learned bias despite fair inputs      | In-Processing                | True positive / false positive rates | Adversarial training may be unstable or require tuning |
| Cannot retrain model / using 3rd-party API | Post-Processing              | Threshold-adjusted fairness metrics | Limited to score transformation or thresholding only |

---

### 3. Risk & Trade-Off Guidance

- Performance trade-offs: Interventions may reduce accuracy or AUC; monitor impact on key business metrics.  
- Overfitting / instability: Weighting, adversarial debiasing, or constraint optimization may overfit minority groups; validate carefully.  
- Intersectional fairness: Always evaluate combined groups (e.g., gender × age, gender × foreign worker).  
- Regulatory compliance: Explainability requirements may limit complex in-processing or post-processing transformations.  
- Monitoring: Establish ongoing checks and thresholds for drift or emerging bias in new data.

#### Fairness Definitions Cannot All Be Satisfied Simultaneously

Before selecting a fairness definition, teams must understand a fundamental constraint: demographic parity, equalized odds, and calibration cannot all hold at the same time when base rates differ across groups (Chouldechova, 2017; Kleinberg et al., 2016).

This means every fairness intervention involves an explicit prioritisation decision:

| If you prioritise… | You may give up… | Typical use case |
|---|---|---|
| Demographic parity | Calibration — scores lose consistent meaning across groups | Hiring, university admissions |
| Equalized odds (equal TPR + FPR) | Demographic parity — selection rates may still differ | Credit, medical screening |
| Calibration | Equal opportunity — high-risk groups may face higher thresholds | Risk scoring, insurance |

**What to do:**

1. Choose one primary fairness definition based on the harm you most need to prevent.
2. Document why that definition was selected and what was traded off.
3. Monitor the sacrificed metrics — an acceptable trade-off today may become unacceptable as data or context changes.
4. Never attempt to simultaneously optimise all three without acknowledging that the result will violate at least one.

---

## Intersectional Fairness

Intersectional fairness ensures AI decisions are equitable across combinations of protected attributes, not just individual attributes. Examples include gender × age or gender × foreign worker status, where combined effects may create compounded disparities.

### Metrics

- Measure equal opportunity difference, demographic parity difference, and subgroup error rates for each intersectional group.  
- Track calibration metrics such as Expected Calibration Error for each intersectional subgroup.  
- Report both absolute disparities and relative gaps compared to majority groups.  

### Workflow Guidance

- Causal Analysis: Identify all relevant intersectional groups and potential bias pathways.  
- Pre-Processing: Apply transformations, reweighting, or synthetic data to underrepresented intersectional subgroups.  
- In-Processing: Monitor subgroup representation during training and adjust fairness constraints for intersectional groups.  
- Post-Processing: Calibrate scores and thresholds for each intersectional group to correct residual bias.  

### Handling Small Subgroups

- Generate synthetic samples when intersectional groups are too small for reliable metrics.  
- Use weighting or smoothing to avoid overfitting on small groups.  
- Monitor confidence intervals or statistical bounds to ensure stability.  

### Reporting and Monitoring

- Include intersectional fairness metrics in all evaluation dashboards.  
- Establish alerts for disproportionate errors in small or vulnerable subgroups.  
- Periodically review and adjust interventions as data distributions or group definitions change.  

---

## Regulatory Compliance Reference

Different regulatory frameworks impose different obligations on AI systems. The table below maps each framework to the fairness metrics it most directly implicates, the intervention stage where compliance work is concentrated, and the key documentation requirement. Use this as a starting point — consult legal counsel for jurisdiction-specific interpretation.

| Framework | Jurisdiction | Applies To | Key Fairness Obligations | Relevant Metrics | Primary Intervention Stage | Documentation Required |
|---|---|---|---|---|---|---|
| **EU AI Act** (2024) | European Union | High-risk AI systems (credit, hiring, education, law enforcement, healthcare) | Bias testing before deployment; ongoing monitoring; human oversight for high-risk decisions; transparency to affected individuals | Demographic parity difference, equalized odds, subgroup error rates | All stages — causal analysis mandatory for high-risk systems | Conformity assessment, technical documentation, incident logs |
| **GDPR** Art. 22 | European Union | Any automated decision-making with legal or significant effects | Right to explanation; prohibition on solely automated decisions without human review; purpose limitation for sensitive data | Individual fairness, calibration per group | Post-processing (explainable outputs); causal analysis (data minimisation) | Data Protection Impact Assessment (DPIA), lawful basis for processing sensitive attributes |
| **HIPAA** | United States | Healthcare AI using protected health information | Non-discrimination in treatment recommendations; data minimisation; access controls | Equalized odds across demographic groups, subgroup false negative rates | Pre-processing (data handling), post-processing (outcome equity) | Privacy notices, Business Associate Agreements, audit trails |
| **Fair Lending** (ECOA / FHA) | United States | Credit scoring, loan approval, mortgage decisions | Prohibition on disparate treatment and disparate impact on protected classes (race, sex, national origin, age, religion) | Adverse action rates by group, demographic parity difference, equal opportunity gap | Pre-processing (proxy removal), post-processing (threshold calibration) | Adverse action notices, model risk management documentation, fair lending exam readiness |

### How to Use This Table

1. Identify applicable frameworks for your deployment jurisdiction and domain before selecting fairness definitions.
2. Align your primary fairness metric to the framework's obligation — e.g., Fair Lending's disparate impact focus maps to demographic parity difference; HIPAA's treatment equity focus maps to equalized odds.
3. Check documentation requirements early — conformity assessments and DPIAs must be completed before deployment, not after.
4. When multiple frameworks apply (e.g., a European healthcare credit product), take the union of obligations and satisfy the stricter constraint where they conflict.

---

## Adaptability Guidelines

This section provides guidance on adapting the Fairness Action Playbook across different domains, model types, and regulatory contexts.

### 1. Domain Considerations

- While case studies are based on banking and loan approvals, the playbook principles apply to healthcare, insurance, human resources, and other decision-making domains.  
- Identify domain-specific protected attributes (e.g., race, age, gender, disability status) and relevant intersectional groups.  
- Consider regulatory constraints for each domain, such as GDPR in Europe, Fair Lending in finance, or HIPAA in healthcare.  
- Adjust intervention thresholds and fairness metrics to align with domain-specific risk tolerance and operational requirements.

### 2. Model Type Considerations

- Pre-processing techniques such as feature transformation, reweighting, and synthetic data generation are generally applicable across classification and regression models.  
- In-processing techniques must consider model architecture:
  - Tree-based models: specialized algorithms and fairness regularization are most effective.  
  - Linear models: constraint optimization and fairness regularization are often feasible.  
  - Neural networks: adversarial debiasing and fair representation learning are more suitable.  
- Post-processing techniques such as calibration, score transformation, and threshold adjustments are model-agnostic and can be applied regardless of underlying architecture.  

### 3. Generalization and Domain Tuning

- **Generalizable techniques**: threshold adjustment, score transformation, probability calibration, feature scaling, and disparate impact removal.  
- **Domain-specific tuning required**: mediator identification, causal graph construction, synthetic data generation for small subgroups, fairness-aware tree induction, and hyperparameter adjustment in in-processing.  
- Always validate metrics for both fairness and model performance within the specific domain context.  


### 4. LLM and Generative AI Systems

As of 2026, LLMs and generative AI systems are deployed in high-stakes decision contexts (hiring assistants, medical triage chatbots, loan explanation generators) where fairness obligations apply just as they do to classical models. The bias mechanisms and intervention points differ significantly.

**Where classical interventions still apply:**

| Classical Technique | Transfers to LLMs? | How |
|---|---|---|
| Pre-processing (input transformation) | Partially | Prompt engineering and input sanitisation can reduce proxy signals before sending to the model |
| Post-processing (output filtering) | Yes | Output classifiers or rule-based filters can flag or suppress biased generations |
| Causal analysis (proxy identification) | Yes | Identify which prompt elements act as proxies for protected attributes |
| Monitoring (output drift detection) | Yes | Track output quality and refusal rates across demographic groups over time |

**Where classical interventions do not transfer:**

| LLM-specific Bias Type | Description | Intervention |
|---|---|---|
| **Embedding bias** | Protected attributes encoded in token or sentence embeddings skew downstream outputs | Evaluate with WEAT / StereoSet; consider fine-tuning on debiased corpora |
| **Output quality disparity** | Model generates lower-quality, shorter, or less accurate responses for certain demographic groups given equivalent prompts | Counterfactual prompt testing: swap demographic signals, compare output quality |
| **RLHF annotator bias** | Human feedback used in RLHF reflects annotator demographics and cultural norms | Diverse annotator pools; stratified reward model evaluation across groups |
| **Prompt sensitivity** | Equivalent requests phrased differently for different groups produce inconsistent outputs | Systematic prompt variation testing across protected attribute signals |
| **RAG retrieval bias** | Retrieval-augmented systems may consistently surface biased or underrepresentative documents for certain groups | Audit retrieved document distributions by query group; diversify retrieval corpus |

**Evaluation tools for LLM fairness (2026):**

- `lm-evaluation-harness` — standardised benchmark evaluation including BBQ (Bias Benchmark for QA) and WinoBias
- `langfair` — counterfactual fairness testing for LLM outputs
- Manual counterfactual audit: generate responses for demographically varied but semantically equivalent prompts, score for quality parity

**Key principle:** For LLMs used as decision-support tools (not final decision-makers), the fairness obligation shifts to the human-in-the-loop process and the system design, not only the model output. Audit the full workflow, not just the generation.


### 5. Practical Recommendations

1. Start by mapping protected and intersectional attributes relevant to the domain.  
2. Identify mediators and proxies specific to domain processes.  
3. Apply pre-processing interventions where feasible, especially for 3rd-party or black-box models.  
4. Select in-processing techniques based on model type and deployment constraints.  
5. Apply post-processing interventions for residual bias and for models that cannot be retrained.  
6. Establish monitoring and recalibration plans to handle evolving data distributions and domain-specific changes.
7. Stay current with the tooling landscape. Fairness libraries and frameworks are evolving fast. Regularly evaluate new approaches as they mature — what works best today may be improved or replaced as the field develops.