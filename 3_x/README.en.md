# Project Guide

This project provides a guide for building and executing a stock data pipeline using Apache Airflow and Spark in a local environment. 🇰🇷 [한국어 버전](README.md)

## 1. Prerequisites

To run this project locally, you must have the following tools installed:

*   **Docker**: Required to run the containerized sub-environment.
    *   [Docker Official Installation Guide](https://docs.docker.com/get-docker/)
*   **Astro CLI**: A command-line tool to easily configure, run, and manage your local Airflow environment.
    *   [Astro CLI Official Installation Guide](https://docs.astronomer.io/astro/cli/install-cli)

## 2. Docker Image Build

You need to build the custom Docker images that will be used by the Airflow tasks for data processing. Ensure you are in the current project directory in your terminal and run the following command:

```bash
docker build -t airflow/spark-master ./spark/master && docker build -t airflow/spark-worker ./spark/worker && docker build -t airflow/stock-app ./spark/notebooks/stock_transform  
```
*(This command sequence builds the `spark-master`, `spark-worker`, and data processing `stock-app` images required for distributed processing.)*

## 3. Manage Local Airflow Environment (Astro)

Use the Astro CLI commands to control your local Airflow instances.

*   **Start the Environment**
    ```bash
    astro dev start
    ```
    This command starts the necessary Airflow containers (Webserver, Scheduler, Postgres, etc.) in the background and allows you to access the local Airflow UI at `http://localhost:8080`.

*   **Stop the Environment**
    ```bash
    astro dev stop
    ```
    Use this command to temporarily pause your work and stop the containers. Existing data files and workflows will be preserved.

*   **⚠️ Kill the Environment (WARNING)**
    ```bash
    astro dev kill
    ```
    **WARNING:** This command not only cleanly terminates running containers but also **completely deletes the Airflow metadata DB (Postgres)**. All DAG execution history (Run History), Connections, and Variables configuration will be reset. Use this command ONLY when you want to wipe everything out and start completely from scratch.

## 4. Verify Data Load (After Successful Workflow Execution)

If the entire DAG pipeline executed successfully in the Airflow UI, you can connect to the built-in Postgres Database via the terminal to physically verify the processed stock data.

**1) Connect to Postgres Terminal**
```bash
PGPASSWORD=postgres psql -h localhost -p 5432 -U postgres -d postgres
```

**2) Verify Processed Stock Data (SQL Query)**

*   **Check the last 10 days of stock data (Order by Descending):**
    ```sql
    SELECT date, open, high, low, close, volume
    FROM stock_market
    ORDER BY date DESC
    LIMIT 10;
    ```
*   **Check the total row count currently loaded in the table:**
    ```sql
    SELECT COUNT(*) FROM stock_market;
    ```
*   **Exit Database connection:**
    ```text
    \q
    ```
