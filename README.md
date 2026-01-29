# ELT Pipeline (dbt, Snowflake, Airflow)

This project demonstrates a robust ELT (Extract, Load, Transform) pipeline orchestrating dbt Core models using Apache Airflow and Astronomer Cosmos, with Snowflake as the data warehouse.

## Overview

-   **Orchestration:** Apache Airflow (running locally via Astro CLI)
-   **Transformation:** dbt Core
-   **Data Warehouse:** Snowflake
-   **Integration:** Astronomer Cosmos (renders dbt projects as Airflow DAGs)

## Prerequisites

-   [Docker Desktop](https://www.docker.com/products/docker-desktop)
-   [Astro CLI](https://docs.astronomer.io/astro/cli/install-cli)
-   A **Snowflake Account** (Trial or Standard)

## Project Structure

-   `dags/dbt/data_pipeline`: The dbt project containing models, seeds, and tests.
-   `dags/dbt_dag.py`: The Airflow DAG definition using Cosmos to load the dbt project.
-   `airflow_settings.yaml`: Local configuration for Airflow connections (Snowflake credentials).
-   `Dockerfile`: Custom runtime image with dbt-snowflake installed in a virtual environment.

## Setup & Configuration

1.  **Clone the repository:**
    ```bash
    git clone <repository-url>
    cd dbt-dag
    ```

2.  **Configure Snowflake Connection:**
    Open `airflow_settings.yaml` and update the `snowflake_conn` entry with your credentials.

    **Important:** Check your Snowflake URL to determine your region.
    -   If your URL is structure like `https://xyz123.us-east-1.snowflakecomputing.com`:
        -   `conn_host`: `xyz123.us-east-1.snowflakecomputing.com`
        -   `account` (in extra): `xyz123.us-east-1`

    ```yaml
    airflow:
      connections:
        - conn_id: snowflake_conn
          conn_type: snowflake
          conn_host: <your_account>.<region>.snowflakecomputing.com
          conn_schema: dbt_schema
          conn_login: <your_username>
          conn_password: <your_password>
          conn_extra:
            account: <your_account>.<region>
            warehouse: <your_warehouse>
            database: dbt_db
            role: <your_role>
    ```

3.  **Ensure Ports are Available:**
    This project uses the following local ports (configured in `.astro/config.yaml` to avoid conflicts):
    -   **Airflow Webserver:** 8080
    -   **Postgres Metadata:** 5434 (Modified from default 5432)

## Running the Pipeline

1.  **Start Airflow:**
    ```bash
    astro dev start
    ```
    *This usually takes 1-2 minutes to spin up the Docker containers.*

2.  **Access the UI:**
    Go to [http://localhost:8080](http://localhost:8080).
    -   **Username:** `admin`
    -   **Password:** `admin`

3.  **Trigger the DAG:**
    -   Find `dbt_dag` in the DAGs list.
    -   Toggle it **ON**.
    -   Click the **Play** button to trigger a manual run.

4.  **Monitor Progress:**
    Click on the DAG name to see the Grid view. You will see individual tasks representing your dbt models (`dbt_run`, `dbt_test`, etc.) executing in specific Airflow tasks.

## Troubleshooting

-   **Port Conflicts:** If you see "address already in use", check `.astro/config.yaml` to change the `postgres.port`.
-   **Snowflake 404:** Verify your `conn_host` and `account` in `airflow_settings.yaml` match your specific Snowflake region URL exactly.
-   **Path Errors:** The DAG uses dynamic path resolution (`pathlib`), so it should work regardless of where the project is mounted.


