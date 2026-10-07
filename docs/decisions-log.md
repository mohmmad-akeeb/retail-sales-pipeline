# Decisions Log

| Date | Decision | Options considered | Why I chose it |
|---|---|---|---|
| 2026-10-07 | Use Apache Airflow LocalExecutor | LocalExecutor; CeleryExecutor with Redis, worker and Flower | The project is small and runs on a 12 GB RAM machine. LocalExecutor removes unnecessary Redis/Celery/Flower services while retaining real Airflow task execution. |
| 2026-10-07 | Run Spark in local mode inside the Airflow container | PySpark inside Airflow container; separate Spark container; Spark on host | Keeps the development environment reproducible and avoids another Spark container. The Olist dataset is small enough for Spark local mode. |
| 2026-10-07 | Use two PostgreSQL services | Two PostgreSQL services; one PostgreSQL service with two databases | Separates Airflow metadata from the retail warehouse while remaining manageable on the available hardware. |
| 2026-10-07 | Build a custom Airflow image | Official image without modification; custom Dockerfile | The pipeline requires Java and PySpark, so the Airflow runtime needs a reproducible custom image. |
| 2026-10-07 | Pin Airflow to 3.3.1 | Floating/latest version; pinned version | A pinned version makes the development environment reproducible and prevents unexpected changes during the project. |
| 2026-10-07 | Use Java 17 with PySpark 4.0.1 | Other Java/PySpark combinations | The base Airflow image was verified to use Debian 12. Java 17 was installed and PySpark 4.0.1 was successfully imported inside the Airflow container. |
| 2026-10-07 | Expose warehouse PostgreSQL on host port 5433 | Host port 5432; host port 5433 | The Airflow PostgreSQL service uses PostgreSQL port 5432 internally. Port 5433 avoids a host-port collision. |
| 2026-10-07 | Keep the official Airflow Compose file as a rollback/reference copy | Delete it after customization; retain it | The unmodified configuration is a known-good reference if the customized stack develops an environment problem. |
| 2026-10-07 | Build the infrastructure before writing application DAGs | DAG first; infrastructure first | DAG execution depends on a functioning Airflow scheduler, metadata database, executor and runtime environment. |