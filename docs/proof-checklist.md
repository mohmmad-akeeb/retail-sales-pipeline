# Proof Checklist

Every resume claim must have implementation evidence before it is considered true.

| Bullet | Proof (file / DAG task / query) | Done? |
|---|---|---|
| 1. Idempotent ETL, medallion architecture, partitions, retries, backfill | TBD — Airflow DAG + Bronze/Silver/Gold PySpark jobs + partitioning implementation + retry configuration + successful rerun/backfill evidence | ☐ |
| 2. Star schema, fan-out fix, customer identity | TBD — PostgreSQL schema + Gold PySpark transformation + join validation + customer_unique_id logic + fact/dimension tables | ☐ |
| 3. DQ gates, quarantine, audit table | TBD — Airflow DQ tasks + quarantine implementation + reason codes + PostgreSQL audit table + demonstrated DQ results | ☐ |
| 4. Window-function analytics, late-delivery vs review finding | TBD — SQL files containing LAG, RANK, rolling-window and cohort queries + seller delivery/review analysis + actual result supporting the finding | ☐ |

## Infrastructure Evidence

| Capability | Proof | Done? |
|---|---|---|
| Docker Compose | `docker-compose.yaml` | ✓ |
| Airflow LocalExecutor | `docker-compose.yaml` | ✓ |
| Separate Airflow metadata PostgreSQL | `docker-compose.yaml` | ✓ |
| Separate warehouse PostgreSQL | `docker-compose.yaml` | ✓ |
| Custom Airflow image | `Dockerfile` | ✓ |
| Java runtime | Docker image + `java -version` verification | ✓ |
| PySpark runtime | Docker image + PySpark import verification | ✓ |
| Airflow UI | Successful browser login | ✓ |

## Resume-Bullet Evidence Status

Bullet 1: ☐ Not yet proven  
Bullet 2: ☐ Not yet proven  
Bullet 3: ☐ Not yet proven  
Bullet 4: ☐ Not yet proven