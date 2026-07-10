# BigQuery Contribution Analysis & Data Processing Report

## 1. Core Objective & Analysis Scope

### Objective

The primary objective of this project was to perform an automated diagnostic analysis on a complex General Ledger dataset to identify the root cause of significant financial variance between two different accounting periods.

Instead of writing numerous manual `GROUP BY` queries or complex pivot tables to analyze every possible combination of dimensions, **BigQuery ML Contribution Analysis** (`ML.GET_INSIGHTS`) was used to automatically identify the combinations of categorical variables responsible for the observed variance.

### Analysis Configuration

| Parameter | Value |
|-----------|-------|
| **Target Variable** | `SUM(amount)` |
| **Control Population (Baseline)** | Transactions from **May 2026** (`metric_control`) |
| **Test Population (Comparison)** | Transactions from **June 2026** (`metric_test`) |

---

# 2. Data Preparation & Engineering

Before executing the BigQuery ML model, the raw General Ledger transaction dataset (`sample_ledger`) underwent several preprocessing steps.

## Step 1 — Categorical Isolation

The original ledger contained multiple financial statement categories, including Balance Sheet and Income Statement transactions.

To restrict the analysis to operational profitability, the dataset was filtered using:

```sql
WHERE financial_statement_helper = 'income_statement'
```

This isolated:

- Revenue
- Other Income
- Operating Expenses

while excluding unrelated Balance Sheet entries.

---

## Step 2 — Population Definition

The date dimension was used to divide transactions into two comparison groups.

| Population | Date Range |
|------------|------------|
| **Control** | May 2026 |
| **Test** | June 2026 |

These two populations became the baseline and comparison datasets for Contribution Analysis.

---

## Step 3 — Feature Selection

The model analyzed multiple categorical dimensions simultaneously to identify structural shifts.

Selected input dimensions included:

- `account_name`
- `account_sub_type`
- `account_type`
- `transaction_type`
  - DEBIT
  - CREDIT
- `vendor_name`

These dimensions were automatically combined into different segment combinations during analysis.

---

# 3. Executing the BigQuery ML Engine

The analysis was performed using **BigQuery ML Contribution Analysis** via the `ML.GET_INSIGHTS()` function.

The executed query followed the structure below:

```sql
SELECT
    contributors,
    metric_control,
    metric_test,
    difference,
    unexpected_difference,
    apriori_support
FROM
    ML.GET_INSIGHTS(
        MODEL `website-tracking-analytics.financial_analysis.net_profit_variance_model`
    )
ORDER BY
    ABS(difference) DESC;
```

---

## Output Columns Explained

### `contributors`

An array containing the exact combination of categorical dimensions responsible for the detected variance.

Example:

```text
[
    "account_name=Entertainment",
    "transaction_type=DEBIT"
]
```

---

### `metric_control`

The aggregated value of `SUM(amount)` for the selected segment during the **May 2026** control period.

---

### `metric_test`

The aggregated value of `SUM(amount)` for the same segment during the **June 2026** comparison period.

---

### `difference`

The absolute financial change between the two periods.

Formula:

```text
difference = metric_test - metric_control
```

---

### `unexpected_difference`

Measures how much the segment deviated from the overall expected trend.

Rather than simply measuring raw change, this metric highlights segments that changed significantly more (or less) than the global dataset trend.

---

### `apriori_support`

Represents how frequently a segment appears within the overall dataset.

Higher support values indicate business-critical segments, while lower values often represent niche or isolated cases.

---

# 4. Engineering Challenges & Solutions

## Issue A — Overlapping Segment Combinations

### Problem

The model produced **1,086 result rows**, many of which appeared duplicated.

Example:

One row identified:

```text
account_name = Bering Capital Revenue Share
```

while another identified:

```text
vendor_name = Ian Johnson
```

A third row combined both dimensions.

---

### Root Cause

This behavior is intentional.

BigQuery Contribution Analysis automatically evaluates:

- Individual dimensions
- Pairs of dimensions
- Triplets
- Higher-order combinations

to determine which combinations best explain the observed variance.

---

### Resolution

To avoid interpreting overlapping segments multiple times, more specific segments were isolated by selecting rows with larger contributor arrays.

Example approach:

```sql
ARRAY_LENGTH(contributors)
```

Longer contributor arrays generally represented more precise micro-segments.

---

## Issue B — Global Aggregate Dominating Results

### Problem

The highest-ranked rows were broad categories such as:

```text
transaction_type = DEBIT
```

or

```text
all
```

showing very large financial differences.

These broad categories masked more meaningful operational insights.

---

### Root Cause

Rows are ranked using:

```text
ABS(difference)
```

Because global categories include nearly every transaction, they naturally have the largest absolute financial movement.

---

### Resolution

Additional filtering was applied to remove overly broad segments.

Examples included:

- Excluding rows where contributors contained `"all"`
- Ignoring contributor arrays of length 1

This allowed detailed vendor-level and account-level variance drivers to become visible.

---

# 5. Key Learnings & Best Practices

## Array Handling

The `contributors` field is stored as a native **BigQuery ARRAY** rather than plain text.

Searching within this field requires array operations such as:

```sql
WHERE 'vendor_name=Ian Johnson'
IN UNNEST(contributors)
```

or

```sql
EXISTS(...)
```

rather than traditional string matching.

---

## Transaction Sign Conventions

Always verify the financial interpretation of transaction types.

Examples:

- A decrease in **DEBIT** transactions typically indicates reduced spending, positively impacting net profit.
- A decrease in **CREDIT** transactions generally indicates reduced revenue, negatively impacting net profit.

Understanding accounting sign conventions is essential before interpreting model outputs.

---

## Interpreting `apriori_support`

A segment may exhibit a large `unexpected_difference` while having extremely low support.

Such segments represent statistical outliers rather than meaningful business drivers.

For executive reporting, prioritize segments that demonstrate:

- High absolute financial difference
- Significant `unexpected_difference`
- Moderate to high `apriori_support`

These segments provide the greatest practical business insights.

---

# Conclusion

This project demonstrated how **BigQuery ML Contribution Analysis** can automate multidimensional financial variance investigation without requiring extensive manual SQL aggregation.

By leveraging `ML.GET_INSIGHTS()`, the analysis efficiently identified the combinations of accounts, vendors, and transaction types responsible for changes in financial performance between May and June 2026.

The preprocessing pipeline, engineered filtering strategy, and interpretation of contribution metrics enabled the extraction of meaningful operational insights while avoiding misleading aggregate-level results.
