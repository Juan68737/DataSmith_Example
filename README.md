# DataSmith: AI-Powered Data Cleaning Platform

**Enterprise-grade data transformation using natural language. Built for data engineers at scale (Meta, Google, Stripe patterns).**

[![Python 3.9+](https://img.shields.io/badge/python-3.9+-blue.svg)](https://www.python.org/downloads/)
[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)

---

## Table of Contents

1. [Overview](#overview)
2. [Key Features](#key-features)
3. [Installation](#installation)
4. [Quick Start](#quick-start)
5. [Commands Reference](#commands-reference)
6. [Input Sources](#input-sources)
7. [Output Options](#output-options)
8. [YAML Configuration System](#yaml-configuration-system)
9. [Core Features](#core-features)
10. [PostgreSQL Operations](#postgresql-operations)
11. [Advanced Usage](#advanced-usage)
12. [Performance](#performance)
13. [Architecture](#architecture)
14. [Troubleshooting](#troubleshooting)

---

## Overview

DataSmith automates data cleaning and preparation using natural language instructions. Instead of writing repetitive pandas/SQL code, describe what you need and let the AI handle the implementation.

**What makes it different:**
-  **8x faster** than Pandas (Polars/Rust backend)
-  **Auto schema discovery** for multi-table joins
-  **Enterprise PII protection** (GDPR/CCPA/HIPAA)
-  **Incremental processing** (process only changed rows)
-  **LLM-powered conditional logic** (no hardcoding)
-  **YAML config system** (manage credentials and profiles)

---

## Key Features

| Feature | Description | Use Case |
|---------|-------------|----------|
| **Multi-Table Joins** | Auto-discovers table relationships, generates optimal JOINs | E-commerce order analysis, financial reconciliation |
| **Conditional Logic** | LLM generates transformation rules dynamically | Tax jurisdiction assignment, fraud scoring |
| **PII Detection** | Detects 20+ PII types (SSN, credit cards, medical IDs) | HIPAA compliance, GDPR right to erasure |
| **Incremental Processing** | Process only new/changed rows with checkpointing | Meta-scale daily cleanup (3B → 250K rows) |
| **YAML Configuration** | Manage connections, profiles, secrets in YAML | Multi-environment deployments |
| **Data Quality** | ML-readiness assessment, leakage detection | Pre-model training validation |
| **Missing Values** | Intelligent imputation strategies | Fill customer data gaps |
| **Outlier Detection** | IQR, Isolation Forest methods | Remove anomalous transactions |
| **21 Specialized Tools** | Transform, filter, validate, anonymize | Full data engineering pipeline |


## Quick Start

### CSV Cleaning

```bash
# Basic cleaning
datasmith clean --input data.csv --instructions "Remove nulls and normalize phone numbers"

# Save to specific location
datasmith clean --input data.csv --output cleaned.csv

# In-place replacement (creates backup)
datasmith clean --input data.csv --replace --instructions "Fill missing ages with median"
```

### PostgreSQL Cleaning

```bash
# Clean a table
datasmith clean \
  --postgres "postgresql://user:pass@localhost:5432/mydb" \
  --postgres-table users \
  --instructions "Fill missing emails and remove test accounts"
```

### Using YAML Config

```bash
# Use named connection and profile from config file
datasmith clean \
  --config datasmith.yaml \
  --connection prod_users_db \
  --profile standard_user_cleanup \
  --table users
```

---

## Commands Reference

### `clean` Command

Main command for cleaning datasets using AI.

**Syntax:**
```bash
datasmith clean [INPUT_OPTIONS] [OUTPUT_OPTIONS] [PROCESSING_OPTIONS]
```

**All flags:**
```bash
datasmith clean \
  # Config system
  --config datasmith.yaml \
  --connection prod_db \
  --profile standard_cleanup \
  --table users \
  # Or direct input
  --input data.csv \
  --kaggle "user/dataset" \
  --sftp "sftp://user@host/file.csv" \
  --postgres "postgresql://..." \
  --postgres-table users \
  # Output
  --output cleaned.csv \
  --replace \
  --output-sftp "sftp://user@host/output.csv" \
  # Incremental processing
  --incremental \
  --where "updated_at > '2024-01-01'" \
  --target-table users_cleaned \
  --chunk-size 100000 \
  --resume \
  # Processing
  --instructions "Clean nulls and normalize" \
  --verbose
```

---

## Input Sources

### Local Files

**`--input <path>` / `-i <path>`**

Load local CSV, Parquet, JSON, or text files.

**Examples:**
```bash
# CSV file
datasmith clean --input customers.csv

# Parquet file
datasmith clean --input events.parquet

# JSON file
datasmith clean --input config.json
```

---

### Kaggle Datasets

**`--kaggle <dataset>` / `-k <dataset>`**

Download and clean Kaggle datasets directly.

**Example:**
```bash
# Official Titanic dataset
datasmith clean --kaggle titanic --output titanic_clean.csv

# User dataset
datasmith clean --kaggle "username/housing-prices" \
  --instructions "Fill missing values and remove outliers"
```

---

### SFTP Sources

**`--sftp <url>`**

Download files from SFTP servers.

**Flags:**
- `--sftp-key <path>` - SSH private key path
- `--sftp-password <password>` - Password authentication

**Example:**
```bash
# Using SSH key
datasmith clean \
  --sftp "sftp://dataeng@server.company.com/exports/sales_2024.csv" \
  --sftp-key ~/.ssh/company_key
```

---

## Output Options

### Custom Output Path

**`--output <path>` / `-o <path>`**

```bash
datasmith clean --input data.csv --output results/cleaned_data.csv
```

### In-Place Replacement

**`--replace`**

Modify input file in-place. **Automatically creates backup** in `.datasmith/backups/`.

```bash
datasmith clean --input data.csv --replace \
  --instructions "Remove duplicates and fill missing values"
```

### Upload to SFTP

**`--output-sftp <url>`**

```bash
datasmith clean --input raw_data.csv \
  --output-sftp "sftp://user@server.com/cleaned/data.csv" \
  --sftp-key ~/.ssh/id_rsa
```

---

## YAML Configuration System

### Why Use YAML Config?

-  **Manage credentials** securely (no hardcoded passwords)
-  **Reusable profiles** (standard cleaning instructions)
-  **Multi-environment** support (dev, staging, prod)
-  **Secret managers** (AWS Secrets, 1Password, Vault)
-  **Environment variables** (${DB_PASSWORD})
-  **Team collaboration** (share configs without exposing secrets)

---

### Creating Config File

**Step 1: Create `datasmith.yaml`**

```yaml
version: "1.0"

# CONNECTIONS - Your data sources
connections:
  # Production database
  prod_users_db:
    type: postgres
    connection_string: "postgresql://${PROD_DB_USER}:${PROD_DB_PASS}@prod-db.company.com:5432/users"

  # Staging database
  staging_users_db:
    type: postgres
    host: staging-db.company.com
    port: 5432
    database: users
    username: ${STAGING_DB_USER}
    password: ${STAGING_DB_PASS}

  # Local development
  local_postgres:
    type: postgres
    connection_string: "postgresql://localhost:5432/mydb"

  # SFTP server
  data_export_server:
    type: sftp
    host: exports.company.com
    port: 22
    username: dataeng
    key_path: ~/.ssh/company_key

# PROFILES - Reusable cleaning instructions
profiles:
  standard_user_cleanup:
    instructions: |
      Remove test accounts (email contains 'test' or 'staging').
      Fill missing emails with 'noemail@example.com'.
      Normalize phone numbers to E.164 format.
      Standardize country codes to ISO 3166-1 alpha-2.
      Remove duplicates based on email.

  quick_cleanup:
    instructions: |
      Fill all missing values with appropriate defaults.
      Remove obvious outliers.

  gdpr_compliance:
    instructions: |
      Detect all PII in the dataset.
      Anonymize emails with partial masking (j***@gmail.com).
      Hash SSNs and passport numbers.
      Generate compliance report for audit.

  ml_preparation:
    instructions: |
      Fill missing numerical values with median.
      Fill missing categorical values with mode.
      Detect and handle outliers using IQR method.
      Validate data quality score >80.
      Flag data leakage risks.

# DEFAULTS - Global settings
defaults:
  verbose: true
  backup_enabled: true
  chunk_size: 100000

# ENVIRONMENTS - Environment-specific configuration
environments:
  development:
    default_connection: local_postgres
    log_level: DEBUG

  staging:
    default_connection: staging_users_db
    log_level: INFO

  production:
    default_connection: prod_users_db
    log_level: WARNING
```

**Step 2: Set environment variables**

```bash
# In .env or export
export PROD_DB_USER="admin"
export PROD_DB_PASS="secret123"
export STAGING_DB_USER="staging_user"
export STAGING_DB_PASS="staging_pass"
```

---

### Using Config File

#### Example 1: Named Connection + Profile

```bash
# Use prod database with standard cleanup profile
datasmith clean \
  --config datasmith.yaml \
  --connection prod_users_db \
  --profile standard_user_cleanup \
  --table users
```

**What it does:**
1. Loads connection from `prod_users_db` in config
2. Loads instructions from `standard_user_cleanup` profile
3. Replaces `${PROD_DB_USER}` and `${PROD_DB_PASS}` with env vars
4. Executes cleaning on `users` table

---

#### Example 2: Multi-Environment Setup

```bash
# Development environment
DATASMITH_ENV=development datasmith clean \
  --connection local_postgres \
  --table users \
  --instructions "Quick test cleanup"

# Staging environment
DATASMITH_ENV=staging datasmith clean \
  --connection staging_users_db \
  --table users \
  --profile quick_cleanup

# Production environment
DATASMITH_ENV=production datasmith clean \
  --connection prod_users_db \
  --table users \
  --profile standard_user_cleanup
```

---

#### Example 3: GDPR Compliance Profile

```bash
datasmith clean \
  --config datasmith.yaml \
  --connection prod_users_db \
  --profile gdpr_compliance \
  --table customer_data
```

Uses the `gdpr_compliance` profile which automatically:
- Detects PII
- Anonymizes with appropriate methods
- Generates audit report

---

### Secret Manager Integration

#### AWS Secrets Manager
Akylai, if you see this, text back with a ping pong racket

```yaml
connections:
  prod_db:
    type: postgres
    connection_string: "postgresql://${aws_secrets:prod/postgres/credentials}@host:5432/db"
```

#### 1Password CLI

```yaml
connections:
  prod_db:
    type: postgres
    password: "${1password:op://Production/Database/password}"
```

#### HashiCorp Vault

```yaml
connections:
  prod_db:
    type: postgres
    password: "${vault:secret/data/prod/postgres}"
```

---

### Config File Search Order

DataSmith searches for config files in this order:

1. Explicitly provided path: `--config /path/to/datasmith.yaml`
2. Current directory: `./datasmith.yaml`
3. User home: `~/.datasmith/datasmith.yaml`
4. System-wide: `/etc/datasmith/datasmith.yaml`

---

### Validation

```bash
# Validate config file
python -c "from agents.config import DataSmithConfig; \
  config = DataSmithConfig.load('datasmith.yaml'); \
  errors = config.validate(); \
  print('✓ Valid config' if not errors else f' Errors: {errors}')"
```

---

## Core Features

### 1. Multi-Table Operations

**Automatically discovers table relationships and generates optimal JOINs.**

#### Example 1.1: E-Commerce Order Analysis

```bash
datasmith clean \
  --postgres "postgresql://admin:pass@db:5432/ecommerce" \
  --postgres-table orders \
  --instructions "Join users, orders, payments, and shipping_addresses tables.
                  Calculate total revenue per user.
                  Flag orders with failed payments.
                  Add customer lifetime value (CLV)."
```

---

#### Example 1.2: Financial KYC Pipeline (using YAML config)

```bash
datasmith clean \
  --config datasmith.yaml \
  --connection prod_kyc_db \
  --table customers \
  --instructions "Join customers, bank_accounts, transactions, and identity_verifications.
                  Calculate 90-day transaction volume per customer.
                  Flag high-risk customers: >$50K monthly volume AND <3 months account age."
```

---

### 2. Conditional Business Logic

**LLM dynamically generates transformation rules from world knowledge.**

#### Example 2.1: International Tax Compliance

```bash
datasmith clean \
  --postgres "postgresql://billing:pass@db:5432/payments" \
  --postgres-table transactions \
  --instructions "Fill tax_jurisdiction based on:
                  1. If billing_country is set, use that
                  2. Else if email domain maps to country (.uk → UK, .de → Germany), use that
                  3. Else if IP geolocation available, use that
                  4. Else if phone country code exists (+1 → US, +44 → UK), use that
                  5. Default to UNKNOWN"
```

---

#### Example 2.2: Fraud Detection Scoring

```bash
datasmith clean \
  --postgres "postgresql://fraud:pass@db:5432/transactions" \
  --postgres-table payments \
  --instructions "Add fraud_risk_score column:
                  - High risk (80-100): First transaction >$500 AND foreign IP AND mismatched billing/shipping
                  - Medium risk (40-79): Account <7 days AND transaction >$200 AND velocity >5/hour
                  - Low risk (0-39): Account >90 days AND verified email/phone AND transaction <$100"
```

---

### 3. PII Detection & Anonymization

**Enterprise-grade PII protection with Microsoft Presidio.**

#### Example 3.1: Healthcare HIPAA Compliance

```bash
datasmith clean \
  --config datasmith.yaml \
  --connection prod_hipaa_db \
  --profile gdpr_compliance \
  --table patient_records
```

**Or with custom instructions:**

```bash
datasmith clean \
  --postgres "postgresql://hipaa:pass@db:5432/patients" \
  --postgres-table patient_records \
  --instructions "Detect and anonymize all PII for HIPAA compliance:
                  - Hash medical record numbers
                  - Synthetic names
                  - Partial mask dates (keep year/month, mask day)
                  - Redact SSNs completely
                  Generate compliance report"
```

---

#### Example 3.2: GDPR Right to Erasure

```bash
datasmith clean \
  --postgres "postgresql://prod:pass@db:5432/users" \
  --postgres-table users \
  --instructions "For user_id = 12345, anonymize all PII:
                  - Replace email with hashed version
                  - Replace name with 'REDACTED_USER_12345'
                  - Replace phone with NULL
                  - Keep metadata (signup_date, last_login)"
```

---

#### Example 3.3: PII Detection Report (Audit Mode)

```bash
datasmith clean \
  --postgres "postgresql://prod:pass@db:5432/customers" \
  --postgres-table customer_data \
  --instructions "Generate PII compliance report.
                  Do not modify data - detection only.
                  Report GDPR/CCPA/HIPAA compliance status."
```

**Report includes:**
- Total PII entities by type (SSN: 1000, Emails: 5000, etc.)
- Columns containing PII
- Confidence scores
- Recommended anonymization methods
- Compliance status

---

### 4. Incremental Processing

**Process only new/modified rows with automatic checkpointing.**

#### Example 4.1: Meta-Scale Daily Cleanup

```bash
# First run: processes all 3 billion rows
datasmith clean \
  --postgres "postgresql://readonly:pass@prod-replica:5432/social" \
  --postgres-table users \
  --incremental \
  --chunk-size 500000 \
  --target-table users_cleaned \
  --instructions "Fill missing profiles, normalize phones, flag suspicious accounts"

# Next day: processes only 250K changed rows (12,000x faster!)
datasmith clean \
  --postgres "postgresql://readonly:pass@prod-replica:5432/social" \
  --postgres-table users \
  --incremental \
  --chunk-size 500000 \
  --target-table users_cleaned \
  --instructions "Fill missing profiles, normalize phones, flag suspicious accounts"
```

---

#### Example 4.2: Custom WHERE Clause

```bash
datasmith clean \
  --postgres "postgresql://prod:pass@db:5432/analytics" \
  --postgres-table events \
  --incremental \
  --where "event_timestamp >= '2024-01-01' AND event_type = 'purchase'" \
  --chunk-size 100000 \
  --instructions "Calculate session boundaries, geolocate IPs, flag bots"
```

---

#### Example 4.3: Resume Failed Job

```bash
# Job failed at 60% - just run same command again
datasmith clean \
  --postgres "postgresql://warehouse:pass@db:5432/analytics" \
  --postgres-table user_events \
  --incremental \
  --chunk-size 200000 \
  --resume \
  --instructions "Aggregate user events into daily summaries"
```

**Automatically resumes from last successful chunk!**

---

#### Incremental Processing Flags

| Flag | Default | Description |
|------|---------|-------------|
| `--incremental` | False | Enable incremental mode |
| `--where` | Auto-generated | Custom WHERE clause |
| `--target-table` | `{table}_cleaned` | Target table for cleaned data |
| `--incremental-column` | `updated_at` | Column to track changes |
| `--primary-key` | `id` | Primary key for upsert |
| `--chunk-size` | 100,000 | Rows per chunk |
| `--resume` | True | Resume from checkpoint |

---

### 5. Data Quality Validation

#### Example 5.1: Pre-ML Pipeline Validation

```bash
datasmith clean \
  --config datasmith.yaml \
  --connection prod_ml_db \
  --profile ml_preparation \
  --table customer_features
```

**Or with custom validation:**

```bash
datasmith clean \
  --postgres "postgresql://ml:pass@db:5432/training_data" \
  --postgres-table customer_features \
  --instructions "Validate data quality for ML pipeline:
                  - Check missing values in critical features
                  - Detect outliers (income >$10M, age >120)
                  - Flag data leakage risks (future timestamps)
                  - Assess ML readiness score
                  Fail pipeline if score <80"
```

---

### 6. Missing Value Handling

#### Example 6.1: Customer Data Cleanup

```bash
datasmith clean \
  --postgres "postgresql://prod:pass@db:5432/crm" \
  --postgres-table customers \
  --instructions "Fill missing values:
                  - Age: use median age
                  - Country: use mode (most common)
                  - Income: use mean income
                  - Email: mark as 'MISSING_EMAIL@placeholder.com'"
```

---

### 7. Outlier Detection

#### Example 7.1: Financial Transaction Anomalies

```bash
datasmith clean \
  --postgres "postgresql://fraud:pass@db:5432/transactions" \
  --postgres-table payments \
  --instructions "Detect outliers in transaction_amount using IQR method.
                  Cap outliers at 99th percentile.
                  Flag transactions >3 std deviations for review."
```

---

### 8. Data Transformations

#### Example 8.1: Column Operations

```bash
datasmith clean \
  --input customers.csv \
  --instructions "Rename column 'old_name' to 'customer_id'.
                  Drop columns 'temp_field' and 'debug_info'.
                  Add column 'full_name' by concatenating first_name and last_name."
```

---

#### Example 8.2: Value Replacement

```bash
datasmith clean \
  --postgres "postgresql://prod:pass@db:5432/users" \
  --postgres-table users \
  --instructions "Replace values in gender column:
                  - 'F' → 'Female'
                  - 'M' → 'Male'
                  - 'N/A' → NULL
                  Replace 'United States' with 'USA' in country column"
```

---

## PostgreSQL Operations

### Connection Methods

#### Method 1: Direct Connection String

```bash
datasmith clean \
  --postgres "postgresql://user:password@host:port/database" \
  --postgres-table table_name \
  --instructions "..."
```

#### Method 2: Individual Parameters

```bash
datasmith clean \
  --postgres-host localhost \
  --postgres-port 5432 \
  --postgres-user admin \
  --postgres-password mypass \
  --postgres-database production \
  --postgres-table customers \
  --instructions "..."
```

#### Method 3: YAML Config (Recommended)

```bash
datasmith clean \
  --config datasmith.yaml \
  --connection prod_users_db \
  --profile standard_user_cleanup \
  --table users
```

---

### Clean Existing Table

```bash
datasmith clean \
  --postgres "postgresql://admin:pass@prod-db:5432/myapp" \
  --postgres-table users \
  --instructions "Remove rows where email is NULL.
                  Fill missing phone numbers with 'N/A'.
                  Remove test users (email contains 'test')."
```

---

### Create New Table

```bash
datasmith clean \
  --postgres "postgresql://admin:pass@localhost:5432/mydb" \
  --instructions "Create a table called employees with:
                  - employee_id (integer, primary key)
                  - name (text, not null)
                  - email (text, unique)
                  - salary (decimal)"
```

---

## Advanced Usage

### Airflow DAG Integration

```python
from airflow import DAG
from airflow.operators.bash import BashOperator
from datetime import datetime, timedelta

dag = DAG('datasmith_daily', schedule_interval='0 2 * * *')

cleanup = BashOperator(
    task_id='incremental_cleanup',
    bash_command="""
        datasmith clean \
          --config /etc/datasmith/datasmith.yaml \
          --connection prod_users_db \
          --profile standard_user_cleanup \
          --table users \
          --incremental
    """,
    dag=dag,
)
```

---

### Blue-Green Deployment

```bash
# Step 1: Clean to new table
datasmith clean \
  --config datasmith.yaml \
  --connection prod_db \
  --table users \
  --target-table users_v2 \
  --incremental \
  --instructions "Apply new cleaning logic"

# Step 2: Validate
datasmith clean \
  --connection prod_db \
  --table users_v2 \
  --instructions "Validate quality score >90"

# Step 3: Swap tables
psql -c "ALTER TABLE users RENAME TO users_old;
         ALTER TABLE users_v2 RENAME TO users;"
```

---

## Performance

### Benchmarks

**Dataset**: 500K rows, 50 columns

| Operation | Pandas | Polars (DataSmith) | Speedup |
|-----------|--------|-------------------|---------|
| Read CSV | 2.3s | 0.3s | **8x** |
| Filter rows | 1.8s | 0.2s | **9x** |
| Group by | 3.1s | 0.4s | **8x** |
| Join tables | 4.2s | 0.5s | **8x** |

---

### Incremental Processing Performance

**Scenario:** Meta-scale user table (3B rows, 250K daily updates)

| Mode | Rows | Time | Cost | Frequency |
|------|------|------|------|-----------|
| Regular | 3B | 4 hours | $50 | Weekly |
| Incremental | 250K | 30 seconds | $0.004 | Hourly |
| **Speedup** | **12,000x** | **480x** | **12,500x** | ∞ |

---

## Architecture

```
┌────────────────────────────────────────────────────────┐
│                   DataSmith CLI                        │
├────────────────────────────────────────────────────────┤
│  Input                Agent Core         Output        │
│  ┌─────────┐        ┌─────────┐       ┌─────────┐    │
│  │PostgreSQL│───────▶│LangGraph│──────▶│PostgreSQL│    │
│  │CSV/SFTP │        │Claude3.5│       │CSV/SFTP  │    │
│  │Kaggle   │        │         │       │          │    │
│  └─────────┘        └─────────┘       └─────────┘    │
│                          │                            │
│                          ▼                            │
│               ┌─────────────────────┐                │
│               │ 21 Specialized Tools│                │
│               │ • PII Detection     │                │
│               │ • Transformations   │                │
│               │ • Validation        │                │
│               └─────────────────────┘                │
└────────────────────────────────────────────────────────┘
```

**Tech Stack:**
- **Agent**: LangGraph + Claude 3.5 Sonnet
- **Data**: Polars (Rust-based, 8x faster)
- **Database**: PostgreSQL
- **PII**: Microsoft Presidio
- **State**: SQLite
- **Config**: YAML with env var interpolation

---

## Troubleshooting

### Out of Memory

```bash
# Reduce chunk size
--chunk-size 50000
```

### Slow Performance

```sql
-- Add indexes
CREATE INDEX CONCURRENTLY idx_users_updated_at ON users(updated_at);
```

### Connection Timeouts

```bash
--postgres "postgresql://user:pass@host/db?connect_timeout=30"
```

### Resume Failed Job

```bash
# Automatically resumes from last checkpoint
--incremental --resume
```

---

## Environment Variables

```bash
# Required
ANTHROPIC_API_KEY=sk-ant-...

# Optional
KAGGLE_USERNAME=your_username
KAGGLE_KEY=your_api_key

# Database credentials (for YAML config)
PROD_DB_USER=admin
PROD_DB_PASS=secret123
```

---

## File Organization

```
project/
├── datasmith.yaml           # Config file (connections, profiles)
├── saved_files/             # Auto-generated cleaned files
│   ├── csv/
│   └── json/
├── .datasmith/              # Internal state
│   ├── backups/             # Automatic backups
│   ├── state.db             # Incremental processing state
│   └── schema_cache.json    # Multi-table schema cache
└── .env                     # API keys and secrets
```

---

## Complete Example Workflow

```bash
# 1. Create config file
cat > datasmith.yaml <<EOF
version: "1.0"
connections:
  prod_db:
    type: postgres
    connection_string: "postgresql://\${DB_USER}:\${DB_PASS}@prod-db:5432/users"
profiles:
  daily_cleanup:
    instructions: |
      Fill missing values, normalize phones, remove test accounts
EOF

# 2. Set credentials
export DB_USER="admin"
export DB_PASS="secret123"

# 3. Run incremental cleanup
datasmith clean \
  --config datasmith.yaml \
  --connection prod_db \
  --profile daily_cleanup \
  --table users \
  --incremental

# 4. Schedule with cron (daily at 2 AM)
echo "0 2 * * * cd /path/to/datasmith && datasmith clean --config datasmith.yaml --connection prod_db --profile daily_cleanup --table users --incremental" | crontab -
```

---

## Support

- **Issues**: [GitHub Issues](https://github.com/yourusername/datasmith/issues)
- **Docs**: This README + [INCREMENTAL_PROCESSING.md](INCREMENTAL_PROCESSING.md)

---

## License

MIT License

---

Akylai, if you see this, text back with a unicorn

**Built with ❤️ for Data Engineers - Jhonathan**

*Stop writing repetitive data cleaning code. Start building features that matter.*
