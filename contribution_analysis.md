# Technical Data Report: BigQuery Contribution Analysis & Data Processing
**Focus Area:** Backend Ingestion, Population Segments, and ML Diagnostic Modeling

---

## 1. Core Objective & Analysis Scope
The primary objective of this project was to architect an automated diagnostic data pipeline to isolate the definitive root-cause drivers behind a significant monthly financial variance between two distinct operational windows. 

Instead of deploying traditional, resource-heavy multi-variable SQL pivot loops or manual spreadsheet cross-tabulations, we leveraged **BigQuery ML's Contribution Analysis** capabilities via the `ML.GET_INSIGHTS` function. This approach treats overlapping categorical parameter groups as unique multi-dimensional micro-segments, exposing hidden operational anomalies instantly.

### Core Analysis Architecture
* **Target Metric (Continuous Variable):** `SUM(amount)`
* **Control Population (Baseline Cohort):** General ledger transactions recorded across **May 2026** (`metric_control`)
* **Test Population (Variance Cohort):** General ledger transactions recorded across **June 2026** (`metric_test`)

---

## 2. Raw Data Ingestion & Engineering Steps
Before executing the machine learning engine, the raw, unstructured general ledger transaction history (`sample_ledger`) underwent specific preprocessing constraints within the cloud database:

### Step 1: Categorical Ledger Isolation
The raw dataset contained transactional logs tracking various balance sheet changes. To ensure the ML model focused strictly on operational income, expenses, and overhead variables, we isolated targeted records using the financial matrix flag:
```sql
WHERE financial_statement_helper = 'income_statement'
