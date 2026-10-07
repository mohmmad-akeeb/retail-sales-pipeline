# Progress Log

## Day 1

### Completed

- Created the Retail Sales Pipeline repository structure.
- Configured Docker and Docker Compose.
- Downloaded all 9 Olist source CSVs into `data/raw/`.
- Preserved the unmodified official Airflow Compose configuration as `docker-compose.official.yaml`.
- Configured Airflow to use LocalExecutor.
- Added a separate PostgreSQL service for the retail warehouse.
- Mapped the warehouse PostgreSQL service to host port 5433.
- Verified the base Airflow image uses Debian 12.
- Built a custom Airflow 3.3.1 image with Java 17 and PySpark 4.0.1.
- Verified Java inside the Airflow scheduler container.
- Verified PySpark 4.0.1 imports inside the Airflow scheduler container.
- Started the customized Docker Compose stack successfully.
- Verified Airflow API server, DAG processor, scheduler, metadata PostgreSQL and warehouse PostgreSQL are healthy.
- Verified the Airflow UI and login.

### Currently working on

Day 1 — Airflow practice DAG.

### Not yet completed

- First practice DAG.
- DAG execution and task-log inspection.
- PySpark smoke test.
- Olist data exploration.
- Data relationship notes.
- Learning/concept documentation review.
- Day 1 Git commits and final proof update.