# Customer Analytics ETL Pipeline

A containerized data engineering project that ingests customer transactions from CSV, transforms them with PySpark, orchestrates the workflow with Apache Airflow, and loads analytics tables into PostgreSQL.

Built and maintained by **Chetan**.

[![GitHub](https://img.shields.io/badge/GitHub-Chetan1930-181717?logo=github)](https://github.com/Chetan1930)

## Architecture

The pipeline runs PySpark inside the Airflow container, so the demo is self-contained and does not require a separate Spark cluster:

    customers.csv -> Airflow DAG -> PySpark transformations -> PostgreSQL analytics tables
                          |                    ^                       |
                          +-- verification ----+-----------------------+

## What it demonstrates

- Schema-defined CSV ingestion with PySpark
- Customer-level and product-category aggregations
- Idempotent PostgreSQL upserts
- Airflow scheduling, retries, task dependencies, and verification
- Reproducible local development with Docker Compose
- Configuration through environment variables instead of committed credentials

## Repository layout

    .
    ├── airflow/
    │   ├── dags/customer_pipeline.py  # Airflow DAG
    │   ├── Dockerfile                  # Airflow + Java + PostgreSQL client image
    │   └── requirements.txt
    ├── data/customers.csv              # Sample input data
    ├── docker-compose.yml              # Local orchestration
    ├── images/                         # Example screenshots
    ├── postgres/init.sql               # Analytics database schema
    └── spark/
        ├── Dockerfile                  # Optional standalone Spark image
        └── scripts/process_data.py     # PySpark transformation and load job

## Technology stack

- Apache Airflow 2.8.1
- PySpark 3.5.0
- PostgreSQL 13
- Docker and Docker Compose
- Python 3.8+

## Quick start

### Prerequisites

- Docker Desktop or Docker Engine with the Compose plugin
- At least 4 GB of memory available to Docker

### 1. Configure local credentials

    cp .env.example .env

Edit .env and replace the placeholder values. Generate an Airflow Fernet key with:

    python -c "from cryptography.fernet import Fernet; print(Fernet.generate_key().decode())"

The .env file is ignored by Git and must never be committed.

### 2. Start the stack

    docker compose up -d --build
    docker compose logs -f airflow-init

The first startup initializes the Airflow metadata database and creates the admin account configured in .env.

### 3. Run the pipeline

1. Open Airflow at [http://localhost:8080](http://localhost:8080).
2. Sign in with `AIRFLOW_ADMIN_USERNAME` and `AIRFLOW_ADMIN_PASSWORD` from .env.
3. Enable `customer_data_pipeline`.
4. Trigger the DAG manually and follow the task logs.

PostgreSQL is exposed on `localhost:5433` for local inspection.

## Pipeline output

The DAG creates two analytics tables in the `customer_analytics` database:

| Table | Purpose |
| --- | --- |
| `customer_summary` | Total spend, transaction count, average transaction, and latest transaction per customer |
| `product_category_stats` | Revenue, transaction count, and average transaction by product category |

Query the results from the PostgreSQL container:

    docker compose exec postgres sh -c 'psql -U "$POSTGRES_USER" -d customer_analytics -c "SELECT * FROM customer_summary ORDER BY total_spent DESC;"'

    docker compose exec postgres sh -c 'psql -U "$POSTGRES_USER" -d customer_analytics -c "SELECT * FROM product_category_stats ORDER BY total_revenue DESC;"'

## Useful commands

    # Check service health
    docker compose ps

    # Follow all logs
    docker compose logs -f

    # Stop services while keeping database volumes
    docker compose down

    # Reset the local database and rerun initialization
    docker compose down -v

## Development notes

- Update `data/customers.csv` to test new input records.
- Update `spark/scripts/process_data.py` when changing transformations or load behavior.
- Update `airflow/dags/customer_pipeline.py` when changing orchestration or validation.
- Rebuild after dependency or Dockerfile changes with `docker compose up -d --build`.

## License

This project is released under the MIT License. See [LICENSE](LICENSE).
