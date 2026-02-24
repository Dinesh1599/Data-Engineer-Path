![[Airflow Architecture.png]]
# Airflow 3 Architecture

## Core Idea

Airflow has **one central brain (Metadata Database)** and multiple **components that communicate through APIs**, not directly.

Airflow 3 introduced **API-based communication for better scalability and security.**

---

# Main Components

## 1. Metadata Database (Brain)

Stores everything:

- DAG definitions
- Task status
- Task history
- Scheduling information

Important:

> In Airflow 3, components DO NOT access this DB directly. They use APIs.

---

## 2. Scheduler (Planner)

Responsibility:

- Decides **what DAG to run**
- Decides **when to run**
- Sends tasks to Executor

Think:

> Project Manager

---

## 3. Executor (Dispatcher)

Responsibility:

- Receives tasks from Scheduler
- Sends tasks to Workers

Think:

> Task Distributor

Examples:

- LocalExecutor
- CeleryExecutor
- KubernetesExecutor

---

## 4. Workers (Do the actual work)

Responsibility:

- Executes your task code

Examples:

- Python scripts
- SQL queries
- Spark jobs
- ETL pipelines

Think:

> Workers doing the job

---

## 5. DAG Processor (Reads DAG files)

Responsibility:

- Reads DAG Python files
- Converts them into DAG objects
- Sends metadata via API

Think:

> Code Interpreter

---

## 6. Triggerer (Handles waiting tasks)

Responsibility:

Manages async / waiting tasks

Examples:

- Waiting for file
- Waiting for API response
- Waiting for event

Think:

> Waiting Manager

---

## 7. API Server (Communication layer)

NEW and important in Airflow 3

Responsibility:

- All components communicate through API
- No direct DB access

Benefits:

- More secure
- More scalable
- More stable

---

# Execution Flow (Step-by-Step)

1. DAG file created
2. DAG Processor reads DAG
3. DAG metadata stored in DB via API
4. Scheduler checks database
5. Scheduler sends task to Executor
6. Executor sends task to Worker
7. Worker executes task
8. Worker updates status via API
9. Metadata Database updated
