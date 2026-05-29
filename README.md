# Bias & Fairness Playbooks

![License](https://img.shields.io/badge/License-CC%20BY%204.0-lightgrey)
![MkDocs](https://img.shields.io/badge/Built%20with-MkDocs-blue)
![Status](https://img.shields.io/badge/Status-Active-green)
![GitHub stars](https://img.shields.io/github/stars/TatsianaMark/bias-and-fairness-playbooks)
![GitHub forks](https://img.shields.io/github/forks/TatsianaMark/bias-and-fairness-playbooks)

- [Bias & Fairness Playbooks](https://tatsianamark.github.io/bias-and-fairness-playbooks)


---
AI fairness work is often theoretical. These playbooks 
translate fairness theory and regulatory requirements 
into practical guidance — giving teams a structured 
process for detecting and mitigating bias in classical 
ML systems, from data preparation through deployment.

Built for engineering and data science teams working 
in healthcare, fintech, and other regulated industries.

---

## Who Is This For

Primarily for teams who need to implement a fairness audit 
on their own systems:

- Engineering and data science teams building or maintaining AI systems
- Product managers and heads of product in regulated industries
- Compliance officers navigating EU AI Act requirements
- QA and assurance practitioners moving into AI systems
- Founders of AI startups in healthcare, fintech, or public sector
- Teams doing their first fairness audit with no prior framework

## Overview

AI fairness work is often fragmented: audits happen without follow-through, and interventions are applied inconsistently. This project closes that gap with two complementary playbooks that work as an end-to-end pipeline:

| Playbook | Purpose |
|---|---|
| Fairness Assessment Playbook | Detect and diagnose bias: understand what is wrong and why |
| Fairness Action Playbook | Apply targeted interventions: correct bias at the data, model, or output stage |

Both playbooks are designed to be modular: teams can apply them to a specific component, use case, or development phase rather than implementing everything at once.

---

## Playbooks

### Fairness Assessment Playbook

Standardized guidance for evaluating bias and fairness across an AI system's full lifecycle.

| Component | What it covers |
|---|---|
| Context & History Analysis | Examines historical, social, and organizational factors that shaped the system's data and design |
| Fairness Definition Guide | Helps teams select and apply the right fairness definitions given stakeholder priorities and error-impact trade-offs |
| Bias Root Cause Analysis | Categorizes bias sources, applies structured detection methods, and prioritizes interventions |
| Fairness Metrics | Guides metric selection, statistical validation, and visualization across demographic and intersectional groups |

---

### Fairness Action Playbook

A structured framework for implementing fairness interventions consistently and at scale — including systems using 3rd-party AI APIs where model internals are inaccessible.

| Component | What it covers |
|---|---|
| Causal Analysis | Maps how protected attributes influence downstream outcomes |
| Pre-Processing | Corrects biased data representations before model training |
| In-Processing | Embeds fairness constraints directly into model training |
| Post-Processing | Adjusts predictions or thresholds to reduce unfair outcomes after training |
| Validation Framework | Validates interventions across fairness, accuracy, and business metrics |
| Implementation Guide | Organizational integration and team workflow guidance |

---

## Case Studies

Two end-to-end examples demonstrating the playbooks applied to real-world scenarios:

- Medical Health AI Agent — Applying the Assessment Playbook to a clinical decision-support system serving a diverse patient population, with attention to historical healthcare discrimination patterns.
- Loan Approval (End-to-End) — Applying the full Action Playbook to a credit scoring model, covering ECOA, Fair Housing Act, and GDPR compliance requirements.

---

## Regulatory Coverage

The playbooks reference obligations and documentation requirements for:

| Framework | Jurisdiction | Domain |
|---|---|---|
| EU AI Act (2024) | European Union | High-risk AI systems |
| GDPR Art. 22 | European Union | Automated decision-making |
| HIPAA | United States | Healthcare AI |
| ECOA / Fair Housing Act | United States | Credit and lending |

---

## Documentation Site

This project is built with [MkDocs](https://www.mkdocs.org/) using the [Material theme](https://squidfunk.github.io/mkdocs-material/), with light/dark mode support.

### Prerequisites

```bash
pip install mkdocs-material
```

### Local development

```bash
mkdocs serve
```

Then open `http://127.0.0.1:8000` in your browser.

### Build static site

```bash
mkdocs build
```

Output is written to the `site/` directory.

---

## Project Structure

```
docs/
├── index.md                          # Home page
├── playbooks/
│   ├── fairness_assessment_playbook/ # Assessment Playbook (4 components)
│   └── fairness_action_playbook/     # Action Playbook (6 components)
├── case_studies/
│   ├── medical_health_ai_agent.md
│   └── loan_approval_fairness_e2e_case.md
├── images/                           # Diagrams and visual assets
└── stylesheets/
    └── extra.css
mkdocs.yml                            # Site configuration
```

---

## Scope Note (2026)

The Action Playbook is currently optimized for classical ML systems — classification and regression models with structured inputs. LLM and generative AI fairness challenges (embedding bias, RLHF annotator bias, prompt sensitivity disparities) are partially covered in the Adaptability Guidelines section.

## Contributing

Contributions, corrections, and suggestions are welcome.  
[![Contributing](https://img.shields.io/badge/GitHub-Contributing-181717?logo=github)](https://github.com/TatsianaMark/bias-and-fairness-playbooks/blob/main/CONTRIBUTING.md)


## Author

Built by Tatsiana Donkhina: AI Fairness, Assurance & LLM Testing · Advancing Responsible AI · Certified in AI Ethics

- [LinkedIn](https://www.linkedin.com/in/tatsianamark/)
- [GitHub](https://github.com/TatsianaMark)

## License

Documentation: [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/)  
Code examples: [MIT](https://opensource.org/licenses/MIT)  
© 2026 Tatsiana Donkhina