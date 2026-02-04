# DataSmith CLI Commands Reference

Complete guide to all DataSmith CLI commands, flags, and usage examples.

---

## Table of Contents

1. [Installation](#installation)
2. [Quick Start](#quick-start)
3. [Commands](#commands)
   - [clean](#clean-command)
   - [rollback](#rollback-command)
4. [Input Sources](#input-sources)
5. [Output Options](#output-options)
6. [PostgreSQL Operations](#postgresql-operations)
7. [Advanced Usage](#advanced-usage)

---

## Installation

```bash
# Clone repository
git clone <repo-url>
cd datasmith

# Install dependencies
pip install -r requirements.txt

# Set up environment variables
echo "OPENAI_API_KEY=your_key_here" > .env
echo "ANTHROPIC_API_KEY=your_key_here" >> .env
```

---

## Quick Start

```bash
# Clean a CSV file (creates new file in saved_files/)
python cli.py clean --input data.csv

# Clean with custom instructions
python cli.py clean --input data.csv --instructions "remove rows with missing emails"

# Replace file in-place (creates automatic backup)
python cli.py clean --input data.csv --replace --instructions "fill missing ages with median"
```

---

## Commands

### `clean` Command

Main command for cleaning datasets using AI. Supports CSV, JSON, PostgreSQL, Kaggle datasets, and SFTP sources.

**Basic Usage:**
```bash
python cli.py clean [OPTIONS]
```

---

#### Input Source Flags

##### Local Files

**`--input <path>` / `-i <path>`**
- Path to local CSV or JSON file
- Example:
  ```bash
  python cli.py clean --input data.csv
  python cli.py clean -i customers.json
  ```

##### Kaggle Datasets

**`--kaggle <dataset>` / `-k <dataset>`**
- Download and clean a Kaggle dataset
- Format: `username/dataset-name`
- Requires: Kaggle API credentials configured
- Example:
  ```bash
  python cli.py clean --kaggle titanic
  python cli.py clean -k username/housing-prices
  ```

##### SFTP Sources

**`--sftp <url>`**
- Download file from SFTP server
- Format: `sftp://user@host.com/path/file.csv`
- Example:
  ```bash
  python cli.py clean --sftp "sftp://user@server.com/data/sales.csv" --sftp-key ~/.ssh/id_rsa
  ```

**`--sftp-key <path>`**
- Path to SSH private key for authentication
- Default: `~/.ssh/id_rsa`
- Example:
  ```bash
  python cli.py clean --sftp "sftp://user@host/data.csv" --sftp-key ~/.ssh/my_key
  ```

**`--sftp-password <password>`**
- SFTP password (if not using SSH key)
- Example:
  ```bash
  python cli.py clean --sftp "sftp://user@host/data.csv" --sftp-password "mypass123"
  ```

---

#### Output Flags

**`--output <path>` / `-o <path>`**
- Path to save cleaned file
- If not specified, creates UUID-based file in `saved_files/csv/` or `saved_files/json/`
- Example:
  ```bash
  python cli.py clean --input data.csv --output cleaned_data.csv
  python cli.py clean -i data.csv -o results/clean.csv
  ```

**`--replace`**
- Modify input file in-place
- Automatically creates backup in `.datasmith/backups/`
- Only works with local `--input` files (not Kaggle/SFTP)
- Example:
  ```bash
  python cli.py clean --input data.csv --replace --instructions "remove duplicates"
  ```

**`--output-sftp <url>`**
- Upload cleaned file to SFTP server after processing
- Format: `sftp://user@host.com/path/output.csv`
- Example:
  ```bash
  python cli.py clean --input data.csv \
    --output-sftp "sftp://user@server.com/cleaned/data.csv" \
    --sftp-key ~/.ssh/id_rsa
  ```

**`--pipeline <path>` / `-p <path>`**
- Path to save pipeline JSON metadata
- Default: `pipeline.json`
- Example:
  ```bash
  python cli.py clean --input data.csv --pipeline my_pipeline.json
  ```

---

#### PostgreSQL Flags

##### Connection String Method

**`--postgres <connection_string>`**
- Full PostgreSQL connection string
- Format: `postgresql://user:password@host:port/database`
- Example:
  ```bash
  python cli.py clean --postgres "postgresql://admin:pass@localhost:5432/mydb" \
    --postgres-table users --instructions "clean nulls"
  ```

##### Individual Parameters Method

**`--postgres-host <host>`**
- PostgreSQL server hostname
- Example: `localhost`, `db.example.com`

**`--postgres-port <port>`**
- PostgreSQL server port
- Default: `5432`

**`--postgres-user <username>`**
- PostgreSQL username

**`--postgres-password <password>`**
- PostgreSQL password

**`--postgres-database <database>`**
- PostgreSQL database name

**`--postgres-table <table>` / `-t <table>`**
- Table name to clean
- Required for cleaning existing tables
- Not required when creating new tables

**Example (individual parameters):**
```bash
python cli.py clean \
  --postgres-host localhost \
  --postgres-port 5432 \
  --postgres-user admin \
  --postgres-password mypass \
  --postgres-database production \
  --postgres-table customers \
  --instructions "fill missing phone numbers with N/A"
```

---

#### Processing Flags

**`--instructions <text>`**
- Natural language instructions for cleaning
- Default: `"Clean this dataset for machine learning."`
- Examples:
  ```bash
  # Remove rows with missing data
  python cli.py clean --input data.csv --instructions "remove rows with any missing values"

  # Fill missing values
  python cli.py clean --input data.csv --instructions "fill missing ages with median age"

  # Replace values
  python cli.py clean --input data.csv --instructions "replace Female with Women in gender column"

  # Multiple operations
  python cli.py clean --input data.csv --instructions "remove duplicates, fill nulls with 0, convert dates to YYYY-MM-DD format"
  ```

**`--verbose` / `--quiet`**
- Enable or disable verbose output
- Default: `--verbose`
- Example:
  ```bash
  python cli.py clean --input data.csv --quiet
  python cli.py clean --input data.csv --verbose
  ```

---

### `rollback` Command

Restore a file from automatic backups created by `--replace` flag.

**Basic Usage:**
```bash
python cli.py rollback <file_path> [OPTIONS]
```

---

#### Rollback Flags

**`--list` / `-l`**
- List all available backups without restoring
- Shows timestamp, size, operation, and row counts
- Example:
  ```bash
  python cli.py rollback data.csv --list
  ```

**`--index <number>` / `-i <number>`**
- Backup index to restore
- `0` = most recent backup
- `1` = second most recent backup
- Default: `0`
- Example:
  ```bash
  # Restore most recent backup
  python cli.py rollback data.csv

  # Restore second most recent backup
  python cli.py rollback data.csv --index 1
  ```

---

## Input Sources

### Priority Order

When multiple input sources are provided, DataSmith uses this priority:
1. SFTP (`--sftp`)
2. Kaggle (`--kaggle`)
3. Local file (`--input`)
4. PostgreSQL (`--postgres`)

### Examples

**CSV File:**
```bash
python cli.py clean --input sales_data.csv
```

**JSON File:**
```bash
python cli.py clean --input config.json --instructions "standardize date formats"
```

**Kaggle Dataset:**
```bash
# Requires Kaggle API setup (~/.kaggle/kaggle.json)
python cli.py clean --kaggle titanic --output titanic_clean.csv
```

**SFTP Server:**
```bash
# With SSH key
python cli.py clean \
  --sftp "sftp://user@server.com/data/file.csv" \
  --sftp-key ~/.ssh/id_rsa

# With password
python cli.py clean \
  --sftp "sftp://user@server.com/data/file.csv" \
  --sftp-password "mypassword"
```

---

## Output Options

### Auto-Generated Output (Default)

When no `--output` is specified, DataSmith creates a UUID-based filename:
```bash
python cli.py clean --input data.csv
# Output: saved_files/csv/abc123-data-cleaned-20240201_143052.csv
```

### Custom Output Path

```bash
python cli.py clean --input data.csv --output results/cleaned_data.csv
```

### In-Place Replacement (with backup)

```bash
# Replaces data.csv but creates backup in .datasmith/backups/
python cli.py clean --input data.csv --replace --instructions "remove duplicates"

# Check backups
python cli.py rollback data.csv --list

# Restore if needed
python cli.py rollback data.csv
```

### Upload to SFTP After Cleaning

```bash
python cli.py clean --input data.csv \
  --output-sftp "sftp://user@server.com/cleaned/data.csv" \
  --sftp-key ~/.ssh/id_rsa
```

---

## PostgreSQL Operations

### Clean Existing Table

```bash
# Using connection string
python cli.py clean \
  --postgres "postgresql://user:pass@localhost:5432/mydb" \
  --postgres-table users \
  --instructions "remove rows where email is null"

# Using individual parameters
python cli.py clean \
  --postgres-host localhost \
  --postgres-user admin \
  --postgres-password mypass \
  --postgres-database production \
  --postgres-table orders \
  --instructions "fill missing quantities with 1"
```

### Create New Table

```bash
# No --postgres-table needed for table creation
python cli.py clean \
  --postgres "postgresql://user:pass@localhost:5432/mydb" \
  --instructions "create a table called employees with id, name, email, and salary columns"

# Create table from natural language
python cli.py clean \
  --postgres "postgresql://user:pass@localhost:5432/mydb" \
  --instructions "make a table named products with product_id (int), name (text), price (decimal), and in_stock (boolean)"
```

---

## Advanced Usage

### Complex Cleaning Instructions

**Remove specific rows:**
```bash
python cli.py clean --input data.csv \
  --instructions "remove rows where age is less than 18 or email is missing"
```

**Fill missing values with logic:**
```bash
python cli.py clean --input data.csv \
  --instructions "fill missing salaries with department average, missing dates with today's date"
```

**Data type conversions:**
```bash
python cli.py clean --input data.csv \
  --instructions "convert price column to float, dates to YYYY-MM-DD format, and remove $ symbols"
```

**Multiple operations:**
```bash
python cli.py clean --input data.csv \
  --instructions "1) remove duplicates 2) fill missing ages with median 3) standardize country names 4) convert dates to ISO format"
```

---

### Workflow Examples

**Download from Kaggle, clean, upload to SFTP:**
```bash
python cli.py clean \
  --kaggle "username/housing-data" \
  --instructions "remove outliers and fill missing values" \
  --output-sftp "sftp://user@server.com/cleaned/housing.csv" \
  --sftp-key ~/.ssh/id_rsa
```

**Download from SFTP, clean, save locally:**
```bash
python cli.py clean \
  --sftp "sftp://source@server.com/raw/data.csv" \
  --sftp-key ~/.ssh/source_key \
  --output cleaned_local.csv \
  --instructions "remove duplicates and standardize formats"
```

**In-place cleaning with version control:**
```bash
# Clean and replace
python cli.py clean --input production_data.csv --replace \
  --instructions "remove test data and fill missing values"

# Check what changed
python cli.py rollback production_data.csv --list

# Rollback if needed
python cli.py rollback production_data.csv --index 0
```

**PostgreSQL data pipeline:**
```bash
# Create staging table
python cli.py clean \
  --postgres "postgresql://user:pass@localhost:5432/analytics" \
  --instructions "create table staging_users with user_id, name, email, created_at"

# Clean production table
python cli.py clean \
  --postgres "postgresql://user:pass@localhost:5432/analytics" \
  --postgres-table staging_users \
  --instructions "remove duplicate emails, standardize phone formats"
```

---

## Pipeline Metadata

DataSmith saves metadata about each cleaning operation in a JSON file:

```bash
# Save custom pipeline file
python cli.py clean --input data.csv --pipeline my_operation.json
```

**Pipeline file structure:**
```json
{
  "agent_type": "oneshot",
  "actions_taken": [
    "Applied replace_values",
    "Applied drop_rows"
  ],
  "warnings": [],
  "recommendations": [],
  "input_shape": [1000, 10],
  "output_shape": [950, 10]
}
```

---

## Backup System

### Automatic Backups

Backups are created automatically when using `--replace`:
- Stored in `.datasmith/backups/`
- Includes metadata (timestamp, operation, row counts)
- Up to 10 most recent backups tracked per file

### Backup Metadata

Located in `.datasmith/metadata.json`:
```json
{
  "backups": {
    "data.csv": {
      "current_backup": ".datasmith/backups/data-20240201_143052.csv",
      "history": [
        {
          "backup_path": ".datasmith/backups/data-20240201_143052.csv",
          "timestamp": "2024-02-01T14:30:52",
          "operation": "replace",
          "instruction": "remove duplicates",
          "size_bytes": 45632,
          "rows_before": 1000,
          "rows_after": 950
        }
      ]
    }
  }
}
```

---

## Error Handling

### Common Errors

**No input source:**
```bash
❌ No data source provided. Please specify one of:
  --input <file.csv>
  --kaggle <username/dataset>
  --sftp <sftp://user@host/...>
  --postgres <connection-string>
```

**Invalid --replace usage:**
```bash
❌ --replace requires --input to be specified.
Cannot modify remote sources (Kaggle/SFTP) in-place.
```

**Missing PostgreSQL table:**
```bash
❌ --postgres-table is required when cleaning an existing table
```

---

## Tips & Best Practices

1. **Always test on a copy first** before using `--replace`
2. **Use descriptive instructions** - more detail = better results
3. **Check pipeline.json** to see what operations were performed
4. **Use --verbose** to see detailed operation logs
5. **Leverage backups** - `--replace` is safe with automatic backups
6. **Organize outputs** - default UUID naming prevents accidental overwrites

---

## Environment Variables

Create a `.env` file in the project root:

```bash
# Required for AI operations
OPENAI_API_KEY=sk-...
ANTHROPIC_API_KEY=sk-ant-...

# Optional: Kaggle credentials (or use ~/.kaggle/kaggle.json)
KAGGLE_USERNAME=your_username
KAGGLE_KEY=your_api_key
```

---

## File Organization

```
project/
├── saved_files/           # Auto-generated cleaned files (organized by type)
│   ├── csv/              # UUID-named CSV outputs
│   └── json/             # UUID-named JSON outputs
├── .datasmith/           # Internal DataSmith data (in .gitignore)
│   ├── backups/          # Automatic backups from --replace
│   └── metadata.json     # Backup tracking metadata
├── pipeline.json         # Latest operation metadata (in .gitignore)
└── .env                  # API keys (in .gitignore)
```

---

## Support

For issues or questions:
- GitHub Issues: [your-repo-url]
- Documentation: This file
- Examples: See [Advanced Usage](#advanced-usage) section

---

**Last Updated:** 2024-02-02
