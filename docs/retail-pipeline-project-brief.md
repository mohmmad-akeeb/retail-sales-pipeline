# Retail Sales Pipeline

## 1. Project Overview

Build a production-style local data engineering pipeline using the Olist Brazilian E-Commerce dataset.

The project is designed to demonstrate practical skills in:

- Apache Airflow
- PySpark
- PostgreSQL
- ETL/ELT pipeline design
- Medallion architecture
- Data quality and data validation
- Idempotent processing
- Date-partitioned ingestion
- Backfills and retries
- Star-schema dimensional modeling
- SQL analytics and window functions
- Docker-based local infrastructure

The entire project must be buildable and runnable locally without paid cloud services or paid infrastructure.

The project should be implemented strongly enough that every claim made on the resume can be demonstrated through source code, database objects, DAG runs, SQL queries, test results, and documented evidence.

---

## 2. Dataset

### Source

Olist Brazilian E-Commerce Public Dataset.

The dataset contains approximately 100K orders covering the 2016–2018 period.

### Source tables

The pipeline must work with these nine source tables:

1. `olist_customers_dataset.csv`
2. `olist_geolocation_dataset.csv`
3. `olist_order_items_dataset.csv`
4. `olist_order_payments_dataset.csv`
5. `olist_order_reviews_dataset.csv`
6. `olist_orders_dataset.csv`
7. `olist_products_dataset.csv`
8. `olist_sellers_dataset.csv`
9. `product_category_name_translation.csv`

The raw CSV files are stored under:

```text
data/raw/
```

The raw data should be treated as immutable source data. Transformations must create derived datasets rather than modifying the original CSV files.

---

## 3. Project Objective

Build an end-to-end pipeline:

```text
Olist CSV files
      |
      v
   Bronze
      |
      v
   Silver
      |
      v
    Gold
      |
      v
PostgreSQL Warehouse
      |
      v
 SQL Analytics
```

Airflow is responsible for orchestration.

PySpark is responsible for large-scale data transformation logic.

PostgreSQL is responsible for the analytical warehouse and SQL-based analytics.

Docker Compose provides the local execution environment.

---

## 4. Resume Claims to Prove

The project is intended to substantiate the following four resume claims.

### Claim 1 — ETL, Medallion Architecture, Idempotency and Backfills

> Engineered an idempotent ETL pipeline with Apache Airflow and PySpark on a medallion (bronze -> silver -> gold) architecture, ingesting 100K+ Olist orders across 9 source tables via date-partitioned loads, with automated retries and backfill support for 2016-2018 history.

Implementation must demonstrate:

- Airflow orchestration
- PySpark transformations
- Bronze → Silver → Gold layers
- Date-partitioned processing
- Idempotent reruns
- Airflow retries
- Historical backfill capability
- Processing of the nine Olist source tables

This claim is not considered complete until there is executable evidence for each component.

---

### Claim 2 — Star Schema, Join Fan-Out and Customer Identity

> Modeled a star-schema warehouse in PostgreSQL (order-item-grain fact table, customer/product/seller/date dimensions) and fixed join fan-out and customer-identity issues in PySpark, preventing inflated revenue and incorrect repeat-purchase metrics.

Implementation must demonstrate:

- PostgreSQL star schema
- Order-item-grain fact table
- Customer dimension
- Product dimension
- Seller dimension
- Date dimension
- Correct handling of one-to-many relationships
- Prevention of order-item/payment join fan-out
- Correct use of `customer_unique_id` for customer-level repeat-purchase analysis
- Validation that revenue is not artificially inflated

---

### Claim 3 — Data Quality, Quarantine and Audit

> Enforced data-quality gates in the DAG (duplicate orders, negative prices, orphaned customers, payment-to-item reconciliation), quarantining bad records with reason codes and logging every run's results to an audit table.

The pipeline must implement checks for:

- Duplicate orders
- Negative prices
- Orphaned customer records
- Payment-to-item reconciliation

Invalid records should be quarantined rather than silently discarded.

Each quarantined record should have an appropriate reason code.

Each pipeline execution should produce an audit record containing sufficient information to understand:

- What ran
- When it ran
- Which partition/date was processed
- Whether it succeeded or failed
- Relevant row counts
- Data-quality results

---

### Claim 4 — Analytical SQL and Business Finding

> Delivered window-function SQL analytics (LAG, RANK, rolling averages, cohort retention) covering month-over-month revenue, category rankings by state, and seller late-delivery rates, surfacing the link between late deliveries and low review scores.

The analytical layer must include SQL demonstrating:

- `LAG`
- `RANK`
- Rolling/windowed averages
- Cohort retention analysis
- Month-over-month revenue
- Category rankings by state
- Seller late-delivery rates
- Relationship between late deliveries and review scores

The final business finding must be based on the actual output of the implemented queries rather than being assumed in advance.

---

## 5. Architecture

### Infrastructure

The local environment consists of:

```text
Docker Compose
│
├── Airflow API Server
├── Airflow Scheduler
├── Airflow DAG Processor
├── Airflow Metadata PostgreSQL
└── Warehouse PostgreSQL
```

Airflow uses `LocalExecutor`.

The Airflow image is customized to include:

- Java 17
- PySpark 4.0.1

Spark runs locally inside the Airflow environment rather than using a separate Spark cluster.

The warehouse PostgreSQL instance is exposed locally on port `5433`.

The Airflow metadata database and analytical warehouse database are intentionally separated.

---

## 6. Medallion Architecture

### Bronze

Purpose:

- Preserve source data in a structured processing layer.
- Perform minimal transformation.
- Maintain traceability to raw data.

Expected characteristics:

- Source-oriented structure
- Partitioning where appropriate
- Minimal business transformation

Location:

```text
data/bronze/
```

---

### Silver

Purpose:

- Clean and standardize source data.
- Resolve data-quality issues.
- Normalize data types.
- Apply business transformation rules.
- Prepare reliable datasets for analytical modeling.

Location:

```text
data/silver/
```

---

### Gold

Purpose:

- Produce business-ready datasets.
- Support dimensional modeling and analytics.
- Feed the PostgreSQL warehouse.

Location:

```text
data/gold/
```

If `data/gold/` is added to the repository, it should be documented consistently with the other medallion layers.

---

## 7. Warehouse Model

The PostgreSQL warehouse should use a star schema.

### Fact

Primary fact table:

```text
fact_order_items
```

Grain:

```text
one row per order item
```

The fact table should contain the measures and foreign keys required for analytical queries.

### Dimensions

At minimum:

```text
dim_customer
dim_product
dim_seller
dim_date
```

The model must explicitly account for the distinction between:

```text
customer_id
```

and:

```text
customer_unique_id
```

`customer_id` identifies a customer record associated with an order, while `customer_unique_id` is required for identifying the underlying customer across multiple orders for repeat-purchase analysis.

---

## 8. Important Data-Modeling Risks

### Join fan-out

The Olist dataset contains one-to-many relationships.

For example:

```text
orders
  |
  +---- order_items
  |
  +---- order_payments
```

Joining both child tables directly to orders can multiply rows.

The pipeline must aggregate at the appropriate grain before joining where necessary.

Incorrect:

```text
orders
JOIN order_items
JOIN order_payments
```

without accounting for multiplicity.

The implementation must include validation demonstrating that the final revenue calculations are not inflated.

---

### Customer identity

Repeat-purchase analysis must not treat every `customer_id` as a unique real-world customer.

The project must investigate and document:

```text
customer_id
customer_unique_id
```

and use the appropriate identifier for customer-level metrics.

---

## 9. Data Quality

The pipeline must include explicit data-quality gates.

Required checks:

| Check | Expected behavior |
|---|---|
| Duplicate orders | Detect and reject/quarantine |
| Negative prices | Detect and quarantine |
| Orphaned customers | Detect and quarantine |
| Payment/item reconciliation | Detect discrepancies |
| Required keys | Validate where applicable |
| Null critical fields | Validate where applicable |

Quarantine data should be stored under:

```text
data/quarantine/
```

Each quarantined record should include a reason code.

Examples:

```text
DUPLICATE_ORDER
NEGATIVE_PRICE
ORPHAN_CUSTOMER
PAYMENT_ITEM_MISMATCH
```

The exact reason-code scheme may evolve during implementation and must be recorded in the relevant decision/progress documentation.

---

## 10. Airflow Requirements

Airflow must orchestrate the pipeline rather than merely execute isolated scripts.

The DAG should eventually contain logically separated stages such as:

```text
Ingestion
   |
   v
Bronze transformation
   |
   v
Silver transformation
   |
   v
Data-quality gates
   |
   v
Gold transformation
   |
   v
Warehouse load
   |
   v
Analytics / validation
```

The exact task decomposition should be decided during implementation rather than being artificially forced to match this example.

The pipeline must demonstrate:

- Task dependencies
- Retries
- Date-based execution
- Catchup/backfill behavior
- Idempotent processing
- Failure handling
- Logging
- Data-quality gates

---

## 11. Idempotency

The pipeline must be safe to rerun for the same logical processing period.

For example:

```text
Process 2017-05-01
       |
       v
Run succeeds
       |
       v
Run 2017-05-01 again
       |
       v
No duplicate records
```

The implementation must choose and document an explicit idempotency strategy.

Possible approaches include:

- Partition overwrite
- Scoped delete + insert
- Database upsert

The chosen strategy must preserve correctness when a processing period is rerun.

---

## 12. Partitioning and Backfills

The pipeline should process order history using a date-based partitioning strategy.

The historical period is:

```text
2016–2018
```

The pipeline must support processing an individual period and processing historical periods through Airflow backfill/catchup mechanisms.

The exact partition granularity should be decided during implementation based on the dataset and pipeline design.

---

## 13. Analytical SQL

The SQL analytics layer should contain separate query files rather than one large SQL script.

Planned queries:

```text
sql/analytics/
├── monthly_revenue.sql
├── category_rankings.sql
├── rolling_metrics.sql
├── cohort_retention.sql
└── seller_delivery_reviews.sql
```

Required analytical concepts:

### Month-over-month revenue

Use:

```sql
LAG(...)
```

to compare revenue between periods.

### Category ranking by state

Use:

```sql
RANK(...)
```

or an appropriate ranking function to rank categories within states.

### Rolling metrics

Use SQL window frames to calculate rolling averages or comparable metrics.

### Cohort retention

Group customers by acquisition/first-purchase cohort and measure subsequent activity.

### Seller delivery and review analysis

Calculate seller delivery performance and compare late deliveries with customer review scores.

The final conclusion must come from actual query results.

---

## 14. Repository Structure

Target repository structure:

```text
retail-sales-pipeline/
│
├── .env
├── .gitignore
├── Dockerfile
├── docker-compose.yaml
├── docker-compose.official.yaml
├── README.md
│
├── dags/
│   ├── hello_retail.py
│   └── ...
│
├── spark_jobs/
│   └── ...
│
├── sql/
│   ├── schema.sql
│   ├── ...
│   └── analytics/
│       ├── monthly_revenue.sql
│       ├── category_rankings.sql
│       ├── rolling_metrics.sql
│       ├── cohort_retention.sql
│       └── seller_delivery_reviews.sql
│
├── data/
│   ├── raw/
│   ├── bronze/
│   ├── silver/
│   └── quarantine/
│
├── tests/
│   └── ...
│
├── scripts/
│   └── explore_data.py
│
├── docs/
│   ├── concepts.md
│   ├── learning-log.md
│   ├── data-notes.md
│   ├── decisions-log.md
│   ├── progress-log.md
│   ├── proof-checklist.md
│   ├── interview-cheat-sheet.md
│   ├── architecture.md
│   └── retail-pipeline-project-brief.md
│
├── config/
│   └── airflow.cfg
│
└── plugins/
```

`docker-compose.official.yaml` is retained as a known-good rollback/reference copy of the original official Airflow Compose setup.

---

## 15. Constraints

The project should remain:

- Local
- Free
- Reproducible
- Docker-based
- Suitable for the available laptop resources
- Suitable for interview demonstration

No paid cloud service should be required.

Cloud architecture may be discussed as a future production-scale design, but it must not be represented as implemented unless it is actually implemented.

---

## 16. Implementation Roadmap

### Day 1 — Foundations and Environment

Goal:

Establish the local development environment and understand the dataset.

Definition of done:

- Docker Compose stack works
- Airflow UI works
- Airflow login works
- First practice DAG runs successfully
- Java works inside the Airflow environment
- PySpark works inside the Airflow environment
- All nine source files are explored
- Dataset structure and important relationships are documented

---

### Day 2 — Bronze Layer and Airflow Fundamentals

Goal:

Build the first real ingestion/transformation workflow.

Focus:

- Airflow DAG structure
- Scheduling
- Tasks and dependencies
- Retries
- Bronze ingestion
- Date partitioning

---

### Day 3 — Silver Layer, Idempotency and Backfill

Goal:

Create reliable cleaned datasets.

Focus:

- PySpark transformations
- Schema handling
- Cleaning
- Idempotent processing
- Backfills
- Rerun validation

---

### Day 4 — Gold Layer and Star Schema

Goal:

Create analytical datasets and PostgreSQL warehouse model.

Focus:

- Fact/dimension modeling
- Order-item grain
- Join fan-out
- Customer identity
- Gold transformations
- PostgreSQL loading

---

### Day 5 — Data Quality and Audit

Goal:

Make the pipeline trustworthy.

Focus:

- DQ gates
- Duplicate detection
- Negative-price detection
- Orphan detection
- Payment/item reconciliation
- Quarantine
- Reason codes
- Audit table

---

### Day 6 — Analytical SQL and Review Finding

Goal:

Produce business-facing analytics.

Focus:

- `LAG`
- `RANK`
- Rolling windows
- Cohort retention
- Seller delivery analysis
- Review-score analysis
- Validate the final business finding

---

### Day 7 — Buffer, Documentation and Interview Preparation

Goal:

Make the project defensible.

Focus:

- Fix remaining issues
- Complete README
- Update proof checklist
- Review architecture
- Review decisions
- Prepare interview answers
- Validate all resume claims
- Push clean Git history

Day 7 is intentionally a buffer and should not be treated as an additional build day.

---

## 17. Definition of Project Completion

The project is complete only when:

1. The complete pipeline runs successfully.
2. The pipeline can be rerun without creating duplicate data.
3. Historical processing/backfill works.
4. Bronze, Silver and Gold layers exist and are actually used.
5. PostgreSQL contains the intended star schema.
6. Join fan-out has been demonstrated and corrected.
7. Customer identity is handled correctly.
8. Data-quality checks run as pipeline gates.
9. Invalid records are quarantined with reason codes.
10. Pipeline execution results are stored in an audit table.
11. Required SQL analytics execute successfully.
12. The final late-delivery/review finding is supported by actual data.
13. Every resume claim has implementation evidence.
14. Documentation reflects the implementation rather than planned-but-unbuilt functionality.
15. The user can explain the architecture and major design decisions without relying on memorized statements.

---

## 18. Documentation Rules

This file defines what the project is supposed to contain.

It should not become a running diary.

Use the other documentation files for implementation-specific information:

- `docs/learning-log.md` — what was personally learned during the project.
- `docs/concepts.md` — technical concepts and explanations.
- `docs/data-notes.md` — actual findings from exploring the Olist data.
- `docs/decisions-log.md` — architectural/design decisions and their rationale.
- `docs/progress-log.md` — chronological implementation progress.
- `docs/proof-checklist.md` — evidence proving each resume claim.
- `docs/interview-cheat-sheet.md` — interview explanations and questions.
- `docs/architecture.md` — architecture diagrams and detailed system design.
- `README.md` — concise public-facing project documentation and instructions for running the project.

The project brief should only be changed when the agreed project requirements, scope, architecture, constraints, or roadmap materially change.