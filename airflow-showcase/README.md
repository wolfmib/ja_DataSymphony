
---

# Apache Airflow Demo – macOS M2 Setup

This guide details how to set up and run Apache Airflow on macOS (M2), using Python 3.10 and PostgreSQL for the metadata database. It also covers troubleshooting, debugging, and useful Airflow commands.

---

## Table of Contents

1. [Environment Setup](#environment-setup)
2. [Installing Apache Airflow](#installing-apache-airflow)
3. [PostgreSQL Setup for Airflow Metadata](#postgresql-setup-for-airflow-metadata)
4. [Configuring Airflow](#configuring-airflow)
5. [Initializing and Running Airflow](#initializing-and-running-airflow)
6. [Debugging & Logging](#debugging--logging)
7. [Useful Airflow Commands](#useful-airflow-commands)
8. [Final Notes](#final-notes)

---

## 1. Environment Setup

### 1.1. Ensure Python 3.10 is Installed

Verify your Python version:
```bash
python3 --version
```
_Expected output:_
```
Python 3.10.16
```

### 1.2. Update Your PATH

If Python 3.10 isn’t recognized as the default, update your `PATH`:

1. Open your `~/.zshrc` file:
   ```bash
   nano ~/.zshrc
   ```
2. Add this line at the top:
   ```bash
   export PATH="/opt/homebrew/opt/python@3.10/libexec/bin:$PATH"
   ```
3. Save (`Ctrl+O` then `Enter`) and exit (`Ctrl+X`).

4. Apply changes:
   ```bash
   source ~/.zshrc
   ```

5. Verify the version:
   ```bash
   python --version
   python3 --version
   ```

---

## 2. Installing Apache Airflow

### 2.1. Set Airflow Home Directory

Define your Airflow home:
```bash
export AIRFLOW_HOME=~/airflow  # You can choose another directory if needed
```

### 2.2. Install Airflow with Pinned Dependencies

Determine your Airflow version and use the constraints file:
```bash
AIRFLOW_VERSION=2.6.3
PYTHON_VERSION="$(python --version | cut -d ' ' -f 2 | cut -d '.' -f 1-2)"
CONSTRAINT_URL="https://raw.githubusercontent.com/apache/airflow/constraints-${AIRFLOW_VERSION}/constraints-${PYTHON_VERSION}.txt"

pip install "apache-airflow[pandas,http]==${AIRFLOW_VERSION}" --constraint "${CONSTRAINT_URL}"
```

### 2.3. Verify Airflow Installation

Check the installation:
```bash
airflow version
pip freeze
```
_Expected output for Airflow:_
```
2.6.3
```

---

## 3. PostgreSQL Setup for Airflow Metadata

### 3.1. Install PostgreSQL

Using Homebrew:
```bash
brew install postgresql
brew services start postgresql@14
```

### 3.2. Create the Airflow Database

1. Connect to PostgreSQL:
   ```bash
   psql -d postgres
   ```
2. Create the database:
   ```sql
   CREATE DATABASE airflow;
   ```
3. List databases to verify:
   ```sql
   \l
   ```

### 3.3. Set Default PostgreSQL Database

For zsh:
```bash
export PGDATABASE=airflow
```

### 3.4. Create an Airflow User

In the PostgreSQL shell:
```bash
psql
```
Then run:
```sql
CREATE USER airflow_user WITH PASSWORD 'airflow123!';
ALTER ROLE airflow_user CREATEDB;
```

### 3.5. Install PostgreSQL Connector

Install `psycopg2`:
```bash
pip install psycopg2
```

---

## 4. Configuring Airflow

### 4.1. Update `airflow.cfg` to Use PostgreSQL

Edit your configuration file:
```bash
nano ~/airflow/airflow.cfg
```
Find and modify the following line:
```
sql_alchemy_conn = postgresql+psycopg2://airflow_user:airflow123!@localhost/airflow
```
Save the file.

---

## 5. Initializing and Running Airflow

### 5.1. Initialize the Airflow Database

Run:
```bash
airflow db init
```

### 5.2. Create an Admin User

Create an admin account:
```bash
airflow users create --username admin --firstname Admin --lastname User --role Admin --email admin@example.com --password admin
```

### 5.3. Fork Safety for macOS M1/M2

If you encounter fork safety issues, edit the virtual environment activation script:
```bash
nano airflow_venv/bin/activate
```
Add this line:
```bash
export OBJC_DISABLE_INITIALIZE_FORK_SAFETY=YES
```

### 5.4. Start Airflow Services

In separate terminal windows, run:
```bash
# Start the scheduler:
airflow scheduler

# Start the webserver on port 8080:
airflow webserver --port 8080
```

---

## 6. Debugging & Logging

### 6.1. Viewing Task Logs

To view logs for a specific DAG run:
```bash
cat ~/airflow/logs/dag_id=<dag_id>/run_id=<run_id>/task_id=<task_id>/attempt=1.log
```
_Example:_
```bash
cat ~/airflow/logs/dag_id=run_hello_script/run_id=manual__2025-02-28T18:36:09.786710+00:00/task_id=run_hello_task/attempt=1.log
```

### 6.2. Common Debugging Steps

- **Check Logs:**  
  Inspect the `~/airflow/logs` directory for error messages.
- **Restart Services:**  
  Restart the scheduler and webserver after any changes.
- **Verify Database:**  
  Confirm that PostgreSQL is running and accessible.

---

## 7. Useful Airflow Commands

- **List Available DAGs:**
  ```bash
  airflow dags list
  ```
- **List DAG Jobs:**
  ```bash
  airflow dags list-jobs
  ```

---

## 8. Final Notes

- **PostgreSQL:**  
  Ensure that PostgreSQL is running before starting Airflow.
- **Configuration Changes:**  
  Restart Airflow after any modifications to `airflow.cfg` or user settings.
- **OS Compatibility:**  
  Refer to the OS recommendations to resolve any issues specific to macOS M2.
- **Debug Regularly:**  
  Regularly inspect logs to troubleshoot and verify your setup.

---

This document provides a complete walkthrough for setting up your Airflow demo on macOS M2. Thanks 
John MARCH-2025

---

