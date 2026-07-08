# Clinical Trial AI Data Insight System

An RBQM-inspired AI data insight system for clinical trial risk detection, CRC task prioritization, and evidence-grounded action recommendations.

## Overview

Clinical Trial AI Data Insight System is an ML-ready data product designed to transform clinical trial operation data into actionable risk insights for Clinical Research Coordinators (CRCs).

Instead of using AI only to summarize reports, this project focuses on building a structured risk triage pipeline:

1. Extract operational data from DCT-style clinical trial workflows.
2. Convert raw records into subject-level, site-level, and task-level risk features.
3. Score and rank CRC tasks based on operational risk.
4. Generate evidence-grounded action recommendations that CRC users can understand and act on.

The long-term goal is to build a reusable AI data insight system that supports Risk-Based Quality Management (RBQM) workflows in clinical trial operations.

---

## Project Motivation

Clinical Research Coordinators often manage many time-sensitive tasks at the same time, including patient visits, EDC queries, form completion, safety follow-ups, document tracking, and protocol deviation prevention.

Traditional reporting systems usually show what has already happened. However, CRC users need a system that can answer more operational questions:

* Which subjects are most likely to miss their visit window?
* Which visits may soon become protocol deviations?
* Which EDC queries should be handled first?
* Which forms remain incomplete and may affect data quality?
* Which AE/SAE-related records may require urgent review?
* Which daily tasks should CRCs prioritize under limited working time?

This project aims to convert clinical trial operation data into a daily risk-based task list, helping CRC users decide what to handle first and why.

---

## Core Idea

The system is designed around three layers:

### 1. Data Layer

The data layer organizes DCT-style clinical trial operation data into structured tables, such as:

* Subject information
* Visit schedules
* Visit windows
* EDC queries
* Form completion records
* Site-level operation metrics
* AE/SAE-related signals
* CRC task records

For this public GitHub project, all data is synthetic and does not contain real patient information.

### 2. Risk Modeling Layer

The risk modeling layer converts raw clinical operation records into features and risk scores.

Example risk modeling tasks include:

* Visit window deviation risk prediction
* Query priority ranking
* Form completion risk detection
* AE/SAE anomaly signal detection
* CRC daily task prioritization

The current MVP starts with transparent rule-based scoring as a baseline. Future versions can extend the pipeline with LightGBM, XGBoost, survival analysis, learning-to-rank models, anomaly detection, and model explainability methods such as SHAP.

### 3. LLM Insight Layer

The LLM insight layer converts structured model evidence into CRC-readable action recommendations.

The system is designed to avoid unsupported free-form generation. Instead, the LLM receives structured evidence, such as risk type, risk score, deadline, historical delay count, site deviation rate, and missing task details.

Example:

```json
{
  "patient_id": "P01008",
  "risk_type": "visit_window_risk",
  "risk_score": 0.87,
  "evidence": [
    "Visit window ends in 3 days",
    "Subject had 2 historical delayed visits",
    "Site 003 has a 30-day visit deviation rate of 18%"
  ]
}
```

The generated insight should be grounded in the evidence:

```text
Subject P01008 has a high visit-window risk because the visit window ends in 3 days, the subject has a history of delayed visits, and the site has a relatively high recent deviation rate. The CRC should confirm the appointment today and coordinate physician availability.
```

---

## Current MVP Scope

The current first-stage MVP focuses on:

# Visit Window Risk Prediction

Business question:

```text
Which subjects are most likely to miss their upcoming visit window in the next 3 to 7 days?
```

The MVP uses synthetic visit schedule data and rule-based risk scoring to generate a ranked list of high-risk subjects.

### MVP Inputs

The MVP uses sample CSV files:

```text
data/sample/patients.csv
data/sample/visits.csv
data/sample/sites.csv
```

Example fields:

```text
patients.csv
- patient_id
- site_id
- enroll_date
- status

visits.csv
- patient_id
- visit_name
- planned_date
- window_start
- window_end
- actual_date
- previous_delay_count

sites.csv
- site_id
- site_recent_deviation_rate
```

### MVP Outputs

The MVP outputs a ranked risk table:

```text
outputs/visit_risk_output.csv
```

Example output:

```text
patient_id | site_id | visit_name | window_end | days_until_window_end | previous_delay_count | site_recent_deviation_rate | risk_score | risk_level | reason
P01008     | S003    | V5         | 2026-07-15 | 3                     | 2                    | 0.18                       | 0.87       | High       | Visit window ends soon; subject has historical visit delays; site deviation rate is elevated.
```

---

## Planned System Modules

The full project is designed as a multi-module clinical trial AI data insight system.

### Module 1: Visit Window Risk Prediction

Predicts which subjects are likely to miss their visit window or become protocol deviation risks.

Potential features:

* Days until visit window end
* Historical visit delay count
* Whether the subject has missed visits before
* Site-level visit deviation rate
* CRC workload
* Weekend or holiday effects
* Appointment confirmation status

Potential models:

* Rule-based baseline scoring
* Logistic regression
* LightGBM / XGBoost
* Survival analysis
* Random Survival Forest
* SHAP-based explanation

---

### Module 2: Query Priority Ranking

Ranks EDC queries based on urgency, clinical importance, and operational impact.

Potential features:

* Query open days
* Query category
* Whether the query is safety-related
* Whether the query is AE/SAE-related
* Whether the query affects primary endpoint data
* Historical closure time for similar queries
* Site-level query backlog

Potential methods:

* Rule-based priority scoring
* Text embeddings
* Similar query retrieval
* Learning-to-rank
* LightGBM Ranker / LambdaMART

Example ranking logic:

```text
A normal date mismatch query open for 8 days may be less urgent than a newly opened SAE-related query because the SAE query has higher clinical and compliance risk.
```

---

### Module 3: AE/SAE Signal and Anomaly Detection

Detects potential under-reporting, late reporting, or abnormal reporting patterns related to AE/SAE data.

Potential features:

* AE report rate by site
* SAE report delay
* Lab abnormality frequency
* Symptom-related text signals
* Visit frequency
* Concomitant medication records
* Site-level safety reporting pattern

Potential methods:

* Bayesian hierarchical modeling
* Isolation Forest
* Local Outlier Factor
* EWMA / CUSUM
* Risk-adjusted benchmarking
* Clinical NLP

This module is planned as a later-stage extension because safety-related data requires careful design, validation, and privacy protection.

---

### Module 4: CRC Daily Task Ranking

Combines multiple risk signals into a single daily prioritized task list for CRC users.

Example task types:

* Upcoming visit window risk
* Overdue visit
* Open high-risk query
* Incomplete critical form
* Potential AE/SAE follow-up
* Missing documentation
* Patient loss-to-follow-up risk

Example output:

```json
[
  {
    "task_id": "T001",
    "task_type": "visit_window_risk",
    "patient_id": "P01008",
    "priority": "High",
    "risk_score": 0.87,
    "reason": "Visit window ends in 3 days and the subject has historical visit delays.",
    "recommended_action": "Confirm the visit appointment today and reserve physician availability."
  },
  {
    "task_id": "T002",
    "task_type": "query_priority_risk",
    "patient_id": "P00421",
    "priority": "High",
    "risk_score": 0.82,
    "reason": "The open query is related to SAE data completeness.",
    "recommended_action": "Review SAE start date, severity, outcome, and action taken fields."
  }
]
```

---

### Module 5: Evidence-Grounded LLM Insight Generator

Generates CRC-readable insights from structured risk evidence.

Design principles:

* The LLM should not invent facts.
* Each recommendation should be grounded in structured evidence.
* The output should be short, actionable, and suitable for CRC workflows.
* The system should preserve an audit trail showing which data points supported the insight.

Example structured input:

```json
{
  "risk_type": "visit_window_risk",
  "risk_score": 0.87,
  "risk_level": "High",
  "patient_id": "P01008",
  "evidence": [
    "Visit window ends in 3 days",
    "Previous delayed visit count: 2",
    "Site recent deviation rate: 18%"
  ]
}
```

Example generated insight:

```text
Subject P01008 has a high risk of missing the V5 visit window. The visit window ends in 3 days, and this subject has a history of delayed visits. The CRC should confirm the appointment today and coordinate site availability.
```

---

## Proposed Repository Structure

```text
clinical-trial-ai-data-insight-system/
│
├── data/
│   ├── sample/
│   │   ├── patients.csv
│   │   ├── visits.csv
│   │   ├── queries.csv
│   │   ├── forms.csv
│   │   └── sites.csv
│   │
│   ├── raw/
│   │   └── private data not tracked by Git
│   │
│   └── processed/
│       └── processed data not tracked by Git
│
├── src/
│   ├── data_loader.py
│   ├── feature_engineering.py
│   ├── risk_scoring.py
│   ├── task_ranker.py
│   ├── insight_generator.py
│   │
│   ├── visit_window_risk/
│   │   ├── features.py
│   │   ├── scoring.py
│   │   └── model.py
│   │
│   ├── query_priority_ranking/
│   │   ├── features.py
│   │   ├── scoring.py
│   │   └── ranker.py
│   │
│   ├── ae_sae_signal_detection/
│   │   ├── features.py
│   │   ├── anomaly_detection.py
│   │   └── rules.py
│   │
│   └── crc_task_ranking/
│       ├── task_builder.py
│       └── ranker.py
│
├── notebooks/
│   ├── 01_visit_window_risk.ipynb
│   ├── 02_query_priority_ranking.ipynb
│   └── 03_ae_sae_signal_detection.ipynb
│
├── outputs/
│   ├── visit_risk_output.csv
│   ├── query_priority_output.csv
│   └── ranked_crc_tasks.csv
│
├── app/
│   └── streamlit_app.py
│
├── tests/
│   └── test_risk_scoring.py
│
├── README.md
├── requirements.txt
├── .gitignore
└── LICENSE
```

---

## Installation

Clone the repository:

```bash
git clone https://github.com/YOUR_USERNAME/clinical-trial-ai-data-insight-system.git
cd clinical-trial-ai-data-insight-system
```

Create a virtual environment:

```bash
python -m venv .venv
```

Activate the environment:

On macOS or Linux:

```bash
source .venv/bin/activate
```

On Windows:

```bash
.venv\Scripts\activate
```

Install dependencies:

```bash
pip install -r requirements.txt
```

---

## Usage

Run the MVP pipeline:

```bash
python src/main.py
```

Expected output:

```text
outputs/visit_risk_output.csv
```

The output contains ranked visit-window risk records with risk scores, risk levels, and explanation reasons.

---

## Example Risk Scoring Logic

The initial MVP uses transparent rule-based scoring.

Example logic:

```python
def calculate_visit_risk_score(days_until_window_end, previous_delay_count, site_deviation_rate):
    score = 0.0

    if days_until_window_end <= 2:
        score += 0.5
    elif days_until_window_end <= 5:
        score += 0.3
    elif days_until_window_end <= 7:
        score += 0.1

    score += min(previous_delay_count * 0.1, 0.3)
    score += min(site_deviation_rate, 0.2)

    return min(score, 1.0)
```

This rule-based baseline is intentionally simple and interpretable. It can later be replaced or enhanced with machine learning models.

---

## Evaluation Plan

The system can be evaluated at multiple levels.

### Risk Prediction Evaluation

For supervised prediction tasks:

* AUC
* Precision@K
* Recall@K
* F1 score
* Calibration curve

### Task Ranking Evaluation

For task prioritization:

* Precision@Top 10
* NDCG
* Mean Reciprocal Rank
* Time-to-resolution improvement
* Percentage of high-risk tasks surfaced early

### Insight Quality Evaluation

For LLM-generated recommendations:

* Evidence grounding
* Actionability
* Clinical operation relevance
* Hallucination rate
* Human CRC review score

---

## Tech Stack

Current MVP:

* Python
* pandas
* NumPy
* scikit-learn-ready feature pipeline
* CSV-based synthetic data
* Rule-based risk scoring

Planned extensions:

* LightGBM / XGBoost
* SHAP
* sentence-transformers
* pgvector or Milvus
* FastAPI
* Streamlit
* PostgreSQL
* MLflow
* Evidently
* Docker
* GitHub Actions
* OpenAI API or other LLM APIs

---

## Roadmap

### Phase 1: Visit Window Risk MVP

* [ ] Create synthetic patient, visit, and site datasets
* [ ] Build data loading pipeline
* [ ] Engineer visit-window risk features
* [ ] Implement rule-based risk scoring
* [ ] Output ranked visit-window risk table
* [ ] Write unit tests for scoring logic

### Phase 2: Query Priority Ranking

* [ ] Create synthetic EDC query dataset
* [ ] Identify query risk categories
* [ ] Implement query priority scoring
* [ ] Add query text classification or embedding features
* [ ] Output ranked query priority table

### Phase 3: CRC Daily Task Ranking

* [ ] Convert visits, queries, forms, and safety follow-ups into unified task records
* [ ] Build task-level risk features
* [ ] Rank daily CRC tasks by risk score
* [ ] Output top-k recommended tasks

### Phase 4: LLM Insight Generator

* [ ] Define structured evidence schema
* [ ] Generate CRC-readable action recommendations
* [ ] Add guardrails to prevent unsupported claims
* [ ] Store evidence and generated insights for auditability

### Phase 5: Dashboard and API

* [ ] Build Streamlit dashboard
* [ ] Add filters by site, patient, risk type, and priority
* [ ] Build FastAPI endpoint for risk scoring
* [ ] Add Docker support

### Phase 6: ML Model Extensions

* [ ] Train LightGBM model for visit-window deviation prediction
* [ ] Add model evaluation metrics
* [ ] Add SHAP explanation
* [ ] Compare ML model against rule-based baseline

---

## Data Privacy and Compliance Notes

This public repository uses synthetic data only.

The following data should never be committed to this repository:

* Real patient names
* Real phone numbers
* Real patient IDs
* Real clinical trial project IDs
* Real site names
* Real investigator names
* Real EDC query contents
* Real AE/SAE narratives
* Real company database exports
* Real SQL queries from internal systems
* Real screenshots from production systems
* Any protected health information or confidential business information

Recommended `.gitignore` rules:

```gitignore
.env
.venv/
venv/
__pycache__/
*.pyc

data/raw/
data/private/
data/processed/

outputs/*.csv
outputs/*.xlsx
outputs/*.json

.DS_Store
Thumbs.db
```

Only synthetic sample data under `data/sample/` should be included in the public repository.

---

## Disclaimer

This project is a technical prototype for educational and portfolio purposes.

It is not a medical device, not a validated clinical decision support system, and should not be used to make real clinical trial safety, compliance, or patient care decisions without proper validation, governance, and domain expert review.

---

## Potential Resume Description

Built an RBQM-inspired Clinical Trial AI Data Insight System that transforms DCT-style clinical trial operation data into subject-level and task-level risk features, ranks CRC daily tasks by visit-window, query, and form-completion risks, and generates evidence-grounded action recommendations.

Developed a Python-based risk triage pipeline with pandas, rule-based baseline scoring, ML-ready feature engineering, task ranking, and structured LLM insight generation for clinical trial operations.

---

## Project Status

This project is currently in the MVP development stage.

Current focus:

```text
Visit Window Risk Prediction
```

Planned next modules:

```text
Query Priority Ranking
CRC Daily Task Ranking
Evidence-Grounded LLM Insight Generation
AE/SAE Signal Detection
Dashboard and API
```
