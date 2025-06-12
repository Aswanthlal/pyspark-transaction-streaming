# PySpark Data Engineer Assignment - Aswanth Lal

## Overview

This project simulates a streaming transaction pipeline using Databricks PySpark, AWS S3, and PostgreSQL. It involves two main mechanisms:

- **Mechanism X**: Simulates streaming by generating transaction data chunks
- **Mechanism Y**: Processes each chunk, detects defined patterns, and writes results to S3 and PostgreSQL

---

## Architecture

### Mechanism X: Streaming Simulation

- Reads a master transactions CSV file from GDrive
- Writes chunks of 10,000 rows every second into an S3 bucket under `/stream_chunks`
- Each file is uniquely named with a timestamp suffix to simulate real-time ingestion

### Mechanism Y: Stream Processing

- Monitors the S3 bucket for new files
- For each unprocessed file:
  - Reads the chunk
  - Runs three pattern detection functions (PatId1, PatId2, PatId3)
  - Aggregates results and writes them in **batches of 50** to uniquely named S3 files under `/detected_patterns`
  - Tracks processed files in a PostgreSQL table to avoid reprocessing
  - Also writes full results to a PostgreSQL table for easy querying

---

## Technologies Used

- **Databricks** (PySpark for ETL and detection logic)
- **AWS S3** (storage for input/output files)
- **PostgreSQL** (tracking processed chunks and storing detected patterns)
- **Loom** (for video walkthroughs)

---

## How to Run

### Pre-requisites

- AWS account with S3 bucket created
- PostgreSQL instance (e.g., on AWS RDS)
- Databricks workspace

### Step 1: Setup PostgreSQL

```sql
CREATE TABLE processed_chunks (
    chunk_name TEXT PRIMARY KEY
);

CREATE TABLE detected_patterns (
    customer TEXT,
    merchant TEXT,
    txn_count INT,
    avg_weight DOUBLE PRECISION,
    avg_amount DOUBLE PRECISION,
    txn_rank DOUBLE PRECISION,
    weight_rank DOUBLE PRECISION,
    F INT,
    M INT,
    patternId TEXT,
    actionType TEXT,
    chunk_name TEXT,
    processed_at TEXT
);
```

### Step 2: Run Mechanism X

- Run the `mechanism_x.py` script on Databricks or as a job
- It will stream data to S3 (`/stream_chunks` folder)

### Step 3: Run Mechanism Y

- Run the `mechanism_y.py` script continuously
- It processes all new chunks, detects patterns, and writes output

---

## Output Schema

Each detection result contains:

- `YStartTime`: When Mechanism Y started (IST)
- `detectionTime`: Time of detection (IST)
- `patternId`: PatId1 / PatId2 / PatId3
- `actionType`: Action to be taken (e.g., UPGRADE, CHILD, DEI-NEEDED)
- `customerName`: Customer ID (if applicable)
- `merchantId`: Merchant ID (if applicable)

Fields that don't apply are filled as empty strings.

---

## Assumptions

- 10,000 rows per chunk simulates a realistic data stream
- Weight and category match is considered sufficient for PatId1
- Gender imbalance logic in PatId3 considers only 'F' and 'M'
- Only new files (not in processed\_chunks) are picked for pattern detection


