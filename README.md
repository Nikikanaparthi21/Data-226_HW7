# 📊 DATA 226 - Assignment 7 (dbt + Snowflake)

### Author: Naman Chheda  
### Course: DATA 226 - ELT Deep Dive (Week 10)

---

## 🎯 Objective
This assignment demonstrates the complete setup of a **dbt project** using the **Snowflake connector**, following the Week 10 class instructions.  
It includes input and output models, snapshots, and testing — all connected to Snowflake.

---

## 🧱 Project Structure
```
dbt/
├── dbt_project.yml
├── models/
│ ├── input/
│ │ ├── user_session_channel.sql
│ │ └── session_timestamp.sql
│ ├── output/
│ │ └── session_summary.sql
│ ├── schema.yml
│ └── sources.yml
├── snapshots/
│ └── snapshot_session_summary.sql
├── .gitignore
└── README.md
```


---

## 🧰 Setup Instructions

### 1️⃣ Create a dbt project with Snowflake connector (1 pt)
```bash
dbt init build_mau
```
During setup, choose Snowflake and provide your connection details.
Profiles are defined in profiles.yml.

### 2️⃣ Set up Input Models (2 pt)

Requirement: Input tables are built as CTEs (ephemeral models).

Configured in dbt_project.yml:
```bash
models:
  build_mau:
    input:
      +materialized: ephemeral
```
## 📂 models/input/user_session_channel.sql
```
WITH src AS (
  SELECT * FROM {{ source('raw', 'user_session_channel') }}
)
SELECT
  userId,
  sessionId,
  channel
FROM src
WHERE sessionId IS NOT NULL;
```
## 📂 models/input/session_timestamp.sql
```
WITH src AS (
  SELECT * FROM {{ source('raw', 'session_timestamp') }}
)
SELECT
  sessionId,
  ts
FROM src
WHERE sessionId IS NOT NULL;
```

### 3️⃣ Set up Output Model (1 pt)
## 📂 models/output/session_summary.sql
```
SELECT
  u.userId,
  u.sessionId,
  u.channel,
  st.ts
FROM {{ ref('user_session_channel') }} AS u
JOIN {{ ref('session_timestamp') }} AS st
  ON u.sessionId = st.sessionId;
```

### 4️⃣ Add Snapshot for Output Table (2 pt)
## 📂 snapshots/snapshot_session_summary.sql
```{% snapshot snapshot_session_summary %}
{{
  config(
    target_schema='snapshot',
    unique_key='sessionId',
    strategy='timestamp',
    updated_at='ts',
    invalidate_hard_deletes=True
  )
}}
SELECT * FROM {{ ref('session_summary') }}
{% endsnapshot %}
```

Executed with:
```
dbt snapshot
```

### 5️⃣ Add at least 2 tests to sessionId (2 pt)
## 📂 models/schema.yml
```
version: 2

models:
  - name: session_summary
    description: "Session summary table joined with timestamp"
    columns:
      - name: sessionId
        description: "Unique identifier for each session"
        tests:
          - unique
          - not_null
```
Run tests:
```
dbt test --select model:session_summary
```

### 🧪 Commands Summary
```
# Validate connection
dbt debug

# Build models
dbt run

# Run snapshot
dbt snapshot

# Test constraints
dbt test --select model:session_summary
```

