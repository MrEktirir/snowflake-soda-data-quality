# Snowflake Data Quality Monitoring with Soda Core

A hands-on data engineering project demonstrating automated data quality validation in Snowflake using **Soda Core** and **SodaCL**.

## Project Overview

Data pipelines can successfully load data while still containing missing values, duplicate records, or violations of business rules.

This project demonstrates how Soda Core can validate data quality directly in Snowflake using declarative checks defined in YAML.

The implementation uses the **Tasty Bytes** dataset and focuses on the `CUSTOMER_LOYALTY` table.

## Architecture

```text
Snowflake
  └── TASTY_BYTES_DBT_DB
       └── RAW
            └── CUSTOMER_LOYALTY
                    │
                    ▼
              Soda Core
                    │
                    ▼
                checks.yml
                    │
                    ▼
          Data Quality Validation
                    │
             ┌──────┴──────┐
             ▼             ▼
           PASSED         FAILED
```

Soda Core connects to Snowflake, executes queries based on the configured SodaCL checks, and evaluates the results against the expected conditions.

## Technology Stack

| Technology | Purpose |
|---|---|
| Snowflake | Cloud data warehouse |
| Soda Core 3.5.6 | Data quality validation |
| SodaCL | Declarative data quality checks |
| Python 3.12 | Runtime environment |
| GitHub Codespaces | Development environment |
| Git & GitHub | Version control |

## Project Structure

```text
snowflake-soda-data-quality/
├── .gitignore
├── checks.yml
├── configuration.example.yml
└── README.md
```

The local `configuration.yml` file and Python virtual environment are excluded from version control.

## Data Quality Checks

The project implements three checks on `CUSTOMER_LOYALTY`.

```yaml
checks for CUSTOMER_LOYALTY:
  - row_count > 0
  - missing_count(CUSTOMER_ID) = 0
  - duplicate_count(CUSTOMER_ID) = 0
```

These checks verify that:

1. The table contains at least one record.
2. Customer identifiers do not contain missing values.
3. Customer identifiers are unique.

## Setup and Installation

### 1. Create a Python virtual environment

```bash
python3.12 -m venv .venv
source .venv/bin/activate
```

### 2. Install Soda Core

```bash
python -m pip install soda-core-snowflake setuptools
```

### 3. Configure Snowflake

Copy the example configuration:

```bash
cp configuration.example.yml configuration.yml
```

Update the configuration with your Snowflake connection settings using environment variables.

The example configuration requires:

- `SNOWFLAKE_ACCOUNT`
- `SNOWFLAKE_USERNAME`
- `SNOWFLAKE_PASSWORD`
- `SNOWFLAKE_DATABASE`
- `SNOWFLAKE_WAREHOUSE`
- `SNOWFLAKE_ROLE`
- `SNOWFLAKE_SCHEMA`

Set these variables in your environment before running Soda.

**Security:** Do not commit passwords, access tokens, or private connection configuration files to GitHub.

### 4. Test the Snowflake connection

```bash
soda test-connection -d snowflake -c configuration.yml
```

Expected successful output:

```text
Successfully connected to 'snowflake'.
Connection 'snowflake' is valid.
```

### 5. Execute data quality checks

```bash
soda scan -d snowflake -c configuration.yml checks.yml
```

## Validation Results

The initial scan completed successfully.

| Data Quality Check | Result |
|---|---|
| `row_count > 0` | PASSED |
| `missing_count(CUSTOMER_ID) = 0` | PASSED |
| `duplicate_count(CUSTOMER_ID) = 0` | PASSED |

**Result: 3/3 checks passed.**

The validated dataset contained **222,540 records**, with no missing or duplicate `CUSTOMER_ID` values.

### Negative Test

An intentionally failing condition was introduced:

```yaml
- row_count < 100
```

Since the dataset contained 222,540 records, Soda correctly reported:

```text
3/4 checks PASSED
1/4 checks FAILED

row_count < 100 [FAILED]
check_value: 222540

1 failures. 0 warnings. 0 errors.
```

This demonstrated Soda's ability to distinguish between a data quality rule violation and a technical execution error.

The intentionally failing rule was subsequently removed.

## Key Learnings

- Connecting Soda Core to Snowflake
- Writing declarative SodaCL validation rules
- Detecting missing and duplicate identifiers
- Distinguishing successful scans from failed quality checks
- Testing failure scenarios without modifying source data
- Managing connection credentials through environment variables
- Preparing a reproducible data quality project for GitHub

## Future Improvements

Potential extensions include scheduled scans, additional column-level validation rules, CI/CD integration, and automated alerting.

These features are not implemented in the current version.

## References

- [Soda Documentation](https://docs.soda.io/)
- [Snowflake Documentation](https://docs.snowflake.com/)
- [Snowflake Soda Quickstart](https://www.snowflake.com/en/developers/guides/soda/)

## License

This project is intended for educational and portfolio demonstration purposes.