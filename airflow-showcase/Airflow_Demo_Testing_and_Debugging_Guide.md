
---

# Airflow Demo Testing and Debugging Guide

This document explains how to check and manage the Airflow scheduler process, test new DAG files before deploying them, and review log outputs to ensure your Airflow tasks are executing as expected.

---

## 1. Scheduler Process Management

Before testing any new changes, it’s important to check that no stale Airflow scheduler processes are running. To verify, run:

```bash
ps aux | grep 'airflow scheduler' | grep -v grep
```

If you find any processes (for example, with PIDs `15832` and `15828`), terminate them with:

```bash
kill -9 15832 15828
```

---

## 2. Testing New Files

Before moving a new DAG file into the Airflow `dags/` folder, you can test it using the following methods:

### 2.1. Test Python Functions Locally

Run the script manually to check for syntax errors or function execution issues:

```bash
python hello_hour.py
```

### 2.2. Use Airflow’s DAG Test Command

You can test the DAG in the Airflow environment without moving it into `dags/`:

```bash
airflow dags test hello_hour 2025-02-28
```

This command runs the DAG for a specific execution date so you can verify that it executes without errors.

---

## 3. DAG Code Examples

### 3.1. DAG with Hourly Scheduling

This DAG demonstrates running two tasks sequentially every hour. It prints "Hello!" then waits one minute (displaying "Waiting..." every second) before printing "Finished waiting!" and finally prints "Bonjour!" once the first task completes.

```python
import time
from airflow import DAG
from airflow.operators.python_operator import PythonOperator
from datetime import datetime, timedelta

def run_hello_script():
    print("Hello!")
    # Wait for 1 minute (60 seconds)
    for i in range(60):
        print("Waiting...")
        time.sleep(1)
    print("Finished waiting!")

def run_bonjour_script():
    print("Bonjour!")

with DAG(
    'hello_hour',  # DAG ID
    default_args={
        'owner': 'airflow',
        'depends_on_past': False,
        'start_date': datetime(2025, 2, 28),
        'retries': 1,
        'retry_delay': timedelta(minutes=5),
    },
    description='Run hello.py and bonjour.py sequentially every hour',
    schedule_interval='@hourly',  # Runs every hour
    catchup=False,  # Do not backfill
) as dag:
    task_hello = PythonOperator(
        task_id='run_hello_task',
        python_callable=run_hello_script,
    )
    task_bonjour = PythonOperator(
        task_id='run_bonjour_task',
        python_callable=run_bonjour_script,
    )
    task_hello >> task_bonjour
```

### 3.2. DAG for Manual Testing (No Schedule)

This alternative DAG lets you test your scripts manually without a scheduled interval:

```python
import time
from airflow import DAG
from airflow.operators.python_operator import PythonOperator
from datetime import datetime

def run_hello_script():
    print("Hello!")
    for i in range(60):
        print("Waiting...")
        time.sleep(1)
    print("Finished waiting!")

def run_bonjour_script():
    print("Bonjour!")

with DAG(
    'run_hello_script',  # DAG ID
    default_args={'owner': 'airflow'},
    description='Run hello.py and bonjour.py sequentially',
    schedule_interval=None,  # No automatic scheduling
    start_date=datetime(2025, 2, 28),
    catchup=False,
) as dag:
    task_hello = PythonOperator(
        task_id='run_hello_task',
        python_callable=run_hello_script,
    )
    task_bonjour = PythonOperator(
        task_id='run_bonjour_task',
        python_callable=run_bonjour_script,
    )
    task_hello >> task_bonjour
```

---

## 4. Log Observations

After running your DAGs, review the logs to confirm successful execution:

### Airflow Run Log Notes

- **Status:** All tasks executed successfully.
- **Observation:** The log shows "Bonjour" at the end, confirming that the `run_bonjour_script` task ran after `run_hello_script`.

### Sample Commands to View Logs

```bash
cat ~/airflow/logs/dag_id=run_hello_script/run_id=manual__2025-02-28T18:36:09.786710+00:00/task_id=run_hello_task/attempt=1.log
```

```bash
cat ~/airflow/logs/dag_id=run_hello_script/run_id=manual__2025-02-28T18:36:09.786710+00:00/task_id=run_bonjour_task/attempt=1.log
```

Both logs should indicate that the tasks ran without error.

---

## 5. Final Notes

- **Testing is Crucial:**  
  Always test your DAG files outside of the `dags/` folder to catch errors early.
- **Log Review:**  
  Regularly check your log files to ensure tasks are executing as expected.
- **Scheduler Management:**  
  Ensure you clear out old scheduler processes to prevent conflicts.

✅ **Everything is working fine!**

---

This guide is intended to help you efficiently test and debug your Airflow DAGs before full deployment. 
Merci et Xiae Xiae
John March 2025

---

