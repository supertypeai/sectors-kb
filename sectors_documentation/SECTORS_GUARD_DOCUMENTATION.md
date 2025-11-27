# Sectors Guard Documentation

> **Last Updated**: November 2025
> **Repositories**: `sectors_guard` (Frontend), `sectors_guard_validator` (Backend)

---

## Table of Contents

1. [Overview](#overview)
2. [System Architecture](#system-architecture)
3. [Technology Stack](#technology-stack)
4. [Backend API Reference](#backend-api-reference)
5. [Validation Tables & Rules](#validation-tables--rules)
6. [GitHub Actions Schedules](#github-actions-schedules)
7. [RPC Functions Validation](#rpc-functions-validation)
8. [Local Development Setup](#local-development-setup)
9. [Environment Variables](#environment-variables)
10. [Deployment](#deployment)

---

## Overview

**Sectors Guard** is an internal data quality monitoring platform for validating Indonesian Stock Exchange (IDX) and Singapore Exchange (SGX) financial data. The system automatically detects anomalies in critical data tables and sends email alerts when issues are found.

### Core Features

| Feature | Description |
|---------|-------------|
| **Automated Validation** | Scheduled validation jobs via GitHub Actions |
| **Manual Triggers** | On-demand validation via API or dashboard |
| **Anomaly Detection** | Rule-based detection for data quality issues |
| **Email Alerts** | Automatic notifications when anomalies exceed thresholds |
| **Dashboard** | Real-time visualization of validation status and trends |
| **RPC Validation** | Validation of Supabase RPC functions for data freshness |

### What Sectors Guard Does

1. **Validates Data Integrity** - Checks financial tables for inconsistencies, missing data, and extreme changes
2. **Monitors Data Freshness** - Ensures daily data is up-to-date and complete
3. **Detects Anomalies** - Flags unusual patterns like >35% price changes or accounting rule violations
4. **Stores Results** - Persists validation results in Supabase for historical analysis
5. **Sends Notifications** - Emails operations team when critical issues are detected

---

## System Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                        GitHub Actions                            │
│  (Scheduled validation jobs - cron-based triggers)              │
└─────────────────────────┬───────────────────────────────────────┘
                          │ HTTP POST (Bearer Auth)
                          ▼
┌─────────────────────────────────────────────────────────────────┐
│                  Backend API (Fly.io)                            │
│  sectors-guard-validator.fly.dev                                │
│  ┌─────────────┐  ┌──────────────────┐  ┌──────────────────┐   │
│  │ FastAPI     │  │ IDXFinancial     │  │ Email            │   │
│  │ Routes      │──│ Validator        │──│ Service          │   │
│  └─────────────┘  └──────────────────┘  └──────────────────┘   │
└─────────────────────────┬───────────────────────────────────────┘
                          │
                          ▼
┌─────────────────────────────────────────────────────────────────┐
│                     Supabase (PostgreSQL)                        │
│  - IDX/SGX data tables                                          │
│  - validation_results (stores anomalies)                        │
│  - validation_configs (table-specific settings)                 │
└─────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────┐
│                   Frontend Dashboard (Vercel)                    │
│  React + Material-UI                                            │
│  - Dashboard overview                                           │
│  - Validation results viewer                                    │
│  - Manual validation triggers                                   │
│  - GitHub Actions status monitor                                │
└─────────────────────────────────────────────────────────────────┘
```

### Repository Structure

```
sectors_guard_be/
├── app/
│   ├── api/
│   │   ├── routes.py          # Main API endpoints
│   │   └── sheet_router.py    # Google Sheets endpoints
│   ├── validators/
│   │   ├── data_validator.py           # Base validator class
│   │   ├── idx_financial_validator.py  # IDX/SGX validation logic
│   │   └── notification_validator.py   # Validation with notifications
│   ├── database/
│   │   └── connection.py      # Supabase client
│   ├── notifications/
│   │   ├── email_helper.py              # Email sending
│   │   └── validation_email_service.py  # Email templates
│   ├── auth.py               # Bearer token auth
│   ├── config.py             # App configuration
│   └── main.py               # FastAPI app entry
├── .github/workflows/
│   ├── data-validation.yml   # Scheduled validation jobs
│   ├── daily-summary.yml     # Daily summary email
│   ├── check-api.yml         # Periwatch API check
│   └── fetch-sheet.yml       # Google Sheets fetch
├── requirements.txt
├── Dockerfile
├── fly.toml
└── Procfile

sectors_guard_fe/
├── src/
│   ├── pages/
│   │   ├── Dashboard.js           # Main dashboard
│   │   ├── ValidationResults.js   # Results viewer (regular tables only)
│   │   ├── TableConfiguration.js  # Config editor
│   │   ├── Visualization.js       # Data viz
│   │   ├── Workflows.js           # GitHub Actions status
│   │   ├── RPCValidation.js       # RPC function validation
│   │   └── Access.js              # Login page
│   ├── services/
│   │   └── api.js                 # API client + auth
│   └── components/
├── package.json
└── vercel.json
```

---

## Technology Stack

### Backend

| Component | Technology | Version |
|-----------|------------|---------|
| Framework | FastAPI | 0.104.1 |
| Server | Gunicorn + Uvicorn | 21.2.0 / 0.24.0 |
| Database | Supabase (PostgreSQL) | 2.0.3 |
| ORM | SQLAlchemy | 2.0.43 |
| Data Processing | Pandas, NumPy | latest |
| Validation | Pydantic, Pandera | latest |
| Email | AWS SES (boto3) | 1.34.0 |
| HTTP Client | httpx | 0.24.1 |

### Frontend

| Component | Technology | Version |
|-----------|------------|---------|
| Framework | React | 18.2.0 |
| UI Library | Material-UI (MUI) | 5.x |
| State Management | React Query | 3.39.3 |
| Charts | Recharts | 2.5.0 |
| HTTP Client | Axios | 1.3.4 |
| Routing | React Router | 6.8.1 |

---

## Backend API Reference

**Base URL**: `https://sectors-guard-validator.fly.dev` (Production)  
**Local**: `http://localhost:8000`

### Authentication

All protected endpoints require Bearer token authentication:

```
Authorization: Bearer <BACKEND_API_TOKEN>
```

Token is configured via `BACKEND_API_TOKEN` environment variable.

### Health Check

| Endpoint | Method | Auth | Description |
|----------|--------|------|-------------|
| `/health` | GET | No | Returns `{"status": "healthy"}` |

### Validation API (`/api/validation`)

| Endpoint | Method | Auth | Description |
|----------|--------|------|-------------|
| `/tables` | GET | No | List all available validation tables with last validated time |
| `/run/{table_name}` | POST | Yes | Run validation for specific table |
| `/run-all` | POST | Yes | Run validation for all tables |
| `/run/rpc-functions` | POST | Yes | Run validation for all RPC functions |
| `/run/rpc-functions/{function_name}` | POST | Yes | Run validation for specific RPC function |
| `/config/{table_name}` | GET | No | Get validation config for table |
| `/config/{table_name}` | POST | Yes | Save validation config for table |

#### Run Validation Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| `start_date` | string (YYYY-MM-DD) | Optional. Filter data from this date |
| `end_date` | string (YYYY-MM-DD) | Optional. Filter data until this date |

**Example Request:**
```bash
curl -X POST "https://sectors-guard-validator.fly.dev/api/validation/run/idx_daily_data?start_date=2025-11-01&end_date=2025-11-27" \
  -H "Authorization: Bearer <token>"
```

**Example Response:**
```json
{
  "table_name": "idx_daily_data",
  "validation_timestamp": "2025-11-27T10:30:00.000000",
  "total_rows": 5420,
  "anomalies_count": 3,
  "anomalies": [...],
  "status": "error",
  "date_filter": {
    "start_date": "2025-11-01",
    "end_date": "2025-11-27"
  }
}
```

### Dashboard API (`/api/dashboard`)

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/stats` | GET | Get dashboard statistics (total tables, validated today, anomalies) |
| `/results` | GET | Get recent validation results for regular tables (excludes RPC) |
| `/results/by-table/{table_name}` | GET | Get validation results for specific table |
| `/rpc-results` | GET | Get validation results for RPC functions only |
| `/rpc-results/by-function/{function_name}` | GET | Get validation results for specific RPC function |
| `/charts/validation-trends` | GET | Get 7-day validation trend data |
| `/charts/table-status` | GET | Get table health status distribution |
| `/table-data/{table_name}` | GET | Get raw table data with filters |
| `/github-actions` | GET | Get GitHub Actions workflow status |

#### RPC Results Endpoints

The `/results` and `/rpc-results` endpoints are separated to allow:
- **`/results`**: Returns only regular table validation results (uses `NOT LIKE 'rpc%'` filter at database level)
- **`/rpc-results`**: Returns only RPC function validation results (uses `LIKE 'rpc%'` filter at database level)

**Query Parameters:**

| Endpoint | Parameter | Type | Description |
|----------|-----------|------|-------------|
| `/rpc-results` | `function_name` | string | Optional. Filter by specific RPC function |
| `/rpc-results` | `limit` | int | Number of results (default: 10, max: 100) |
| `/rpc-results/by-function/{function_name}` | `limit` | int | Number of results (default: 5, max: 100) |

**Example Requests:**
```bash
# Get all RPC validation results
curl "https://sectors-guard-validator.fly.dev/api/dashboard/rpc-results"

# Get results for specific RPC function
curl "https://sectors-guard-validator.fly.dev/api/dashboard/rpc-results?function_name=get_top_gainers&limit=5"

# Get recent results by function name
curl "https://sectors-guard-validator.fly.dev/api/dashboard/rpc-results/by-function/get_top_gainers?limit=5"
```

---

## Validation Tables & Rules

### IDX Tables

| Table | Validation Type | Rules | Default Window |
|-------|----------------|-------|----------------|
| `idx_combine_financials_annual` | Financial Performance | YoY change >75% AND >2x avg (min 2); revenue > earnings; identity & ratio checks | All time |
| `idx_combine_financials_quarterly` | Financial Performance | QoQ change >100% AND >2.5x avg (min 2); revenue > earnings; identity & ratio checks | Last 1 year |
| `idx_financial_sheets_annual` | Accounting Identity | Minority check; revenue positivity | Last 2 years |
| `idx_financial_sheets_quarterly` | Accounting Identity | Minority check; revenue positivity | Last 2 years |
| `idx_daily_data` | Price Movement | Close price change >35% day-over-day | Last 7 days |
| `idx_daily_data_completeness` | Data Completeness | All active symbols present; close, volume, market_cap not null | Previous business day |
| `index_daily_data` | Index Coverage | Each date must have exactly 18 index_code entries | Last 7 days |
| `idx_dividend` | Dividend Yield | Average yield ≥30% OR yield change ≥10% per year | All time |
| `idx_all_time_price` | Price Hierarchy | 90d < YTD < 52w < all-time consistency; cross-check with daily data | All time |
| `idx_filings` | Filing Price | Filing price difference ≥50% vs daily close; duplicate detection (<3 days, same holder) | All time |
| `idx_stock_split` | Split Timing | Multiple splits within 14 days for same symbol | All time |
| `idx_news` | Subsector Tagging | subsector list length ≤5; no duplicates (case-insensitive) | All time |
| `idx_company_profile` | Shareholders | share_percentage sum ≈100% (±1%); sector/industry not empty | All time |
| `idx_sector_reports` | Data Freshness | mcap_summary.monthly_performance has yesterday/today data (Jakarta TZ) | Today |

### SGX Tables

| Table | Validation Type | Rules |
|-------|----------------|-------|
| `sgx_company_report` | Fundamentals & Freshness | market_cap & volume not null; latest close = today (Singapore TZ); historical_financials >100% AND >2.5x avg (min 2) |
| `sgx_manual_input` | Business Logic | customer_breakdown sum ≤ total_revenue; property_counts sum ≤ total_revenue |
| `sgx_filings` | Duplicates & Completeness | Composite key duplicate check; missing transaction details when transaction_type present |

**Note**: SGX company report validation is limited to top 50 companies by market capitalization.

### Special Validations

| Validation | Rules |
|------------|-------|
| `rpc_functions` | 16 Supabase RPC functions - date freshness and data availability checks |

*See [SECTORS_GUARD_VALIDATION_RULES.md](./SECTORS_GUARD_VALIDATION_RULES.md) for detailed RPC function validation rules.*

### Validation Rules Detail

#### Financial Sheets Accounting Rules

1. **~~Net Income Flow~~** *(Disabled)*: `net_income = pretax_income - income_taxes + minorities`
   - *Note: Commented out in code due to data inconsistencies*
2. **Minority Check**: If `minorities = 0`, then `net_income = profit_attributable_to_parent` (10% tolerance or 1B absolute)
3. **Revenue Positivity**: `total_revenue > 0` always (strict check)

#### Identity Checks (for financial annual/quarterly with >10 rows)

| Identity | Formula | Tolerance |
|----------|---------|----------|
| Assets = Liabilities + Equity | `total_assets = total_liabilities + total_equity` | 10% or 1B |
| Net Loan | `net_loan = gross_loan - abs(allowance)` | 2% or 1B |
| EBT | `EBT ≈ earnings + tax (+ minorities)` | 5% or 1B |
| Net Cash Flow | `NCF = CFO + CFI + CFF` | 5% or 1B |
| Free Cash Flow | `FCF = CFO - capex` (skip sub_sector_id=19) | 5% or 500M |

#### Banking-Specific Ratio Checks

| Ratio | Valid Range | Severity |
|-------|-------------|----------|
| LDR (Loan-to-Deposit) | 40-130% | warning |
| CASA Ratio | 0-100% | warning |
| CAR (Capital Adequacy) | ≥10% | warning |
| NIM (Net Interest Margin) | -2% to 25% | info |
| Cost-to-Income | 0-300% | warning |
| Coverage Ratio | 0-50% | info |

**Note**: Islamic banks (BANK.JK, BRIS.JK, BSIM.JK, PNBS.JK, BTPS.JK) are excluded from LDR and Assets=Liabilities+Equity checks.

---

## GitHub Actions Schedules

### data-validation.yml

Main validation workflow with multiple schedules:

| Validation | Schedule (UTC) | Cron Expression |
|------------|----------------|-----------------|
| `idx_daily_data_completeness` | Daily at 12:00 AM | `0 0 * * *` |
| `idx_combine_financials_annual` | Mar, Apr, Jun 16th at 2:00 AM | `0 2 16 3,4,6 *` |
| `idx_combine_financials_quarterly` | Monthly 1st at 3:00 AM | `0 3 1 * *` |
| `idx_daily_data` | Weekly Monday at 1:00 AM | `0 1 * * 1` |
| `idx_dividend` | Monthly 1st at 4:00 AM | `0 4 1 * *` |
| `idx_all_time_price` | Daily at 5:00 AM | `0 5 * * *` |
| `sgx_filings` | Weekdays at 6:00 AM | `0 6 * * 1-5` |
| `idx_stock_split` | Monthly 1st at 7:00 AM | `0 7 1 * *` |
| `sgx_company_report` | Monthly 16th at 7:00 AM | `0 7 16 * *` |
| `index_daily_data` | Monthly 1st at 8:00 AM | `0 8 1 * *` |
| `idx_news` | Monthly 1st at 9:00 AM | `0 9 1 * *` |
| `sgx_manual_input` | Every 6 months (Jan, Jul) 1st at 10:00 AM | `0 10 1 1,7 *` |
| `idx_company_profile` | Monthly 29th at 11:00 AM | `0 11 29 * *` |
| `idx_filings` | Daily at 9:15 AM | `15 9 * * *` |

### Other Workflows

| Workflow | Schedule | Purpose |
|----------|----------|---------|
| `daily-summary.yml` | Manual dispatch | Sends daily summary email (disabled by default) |
| `check-api.yml` | Monthly 15th at 8:00 AM | Checks Periwatch API health |
| `fetch-sheet.yml` | Daily at 12:00 AM | Triggers Google Sheets fetch |

### Manual Trigger

All validation workflows support `workflow_dispatch` for manual runs:

```yaml
workflow_dispatch:
  inputs:
    validation_type:
      options:
        - 'all'
        - 'annual'
        - 'quarterly'
        - 'daily'
        - 'dividend'
        - 'alltime'
        - 'filings'
        - 'stocksplit'
        - 'daily_completeness'
        - 'index_daily'
        - 'news'
        - 'sgx_company_report'
        - 'sgx_manual_input'
        - 'company_profile'
        - 'sgx_filings'
```

---

## RPC Functions Validation

The system validates 16 Supabase RPC functions for data freshness and availability.

### RPC Validation Architecture

RPC validation results are stored separately from regular table validations:
- **Table name format**: `rpc_functions` (for bulk validation) or `rpc_{function_name}` (for individual validation)
- **Dedicated endpoints**: `/rpc-results` for fetching RPC-only results
- **Database filtering**: Uses `LIKE 'rpc%'` pattern matching at database level for efficient querying

### RPC Functions List

| Function | Validation Rule | Severity |
|----------|-----------------|----------|
| `get_idx_mcap_data_1m` | Last date ≥ yesterday (vs latest idx_daily_data) | error |
| `get_indices_price_changes` | Latest date = latest idx_daily_data date | error |
| `get_top_mcap_gainers(1)` | `last_close_price` matches latest `idx_daily_data` close price | error |
| `get_top_mcap_losers(5)` | `last_close_price` matches latest `idx_daily_data` close price | error |
| `get_top_gainers(2,true)` | `latest_close_date` in JSONB response ≥ yesterday | error |
| `get_top_losers(2,true)` | `latest_close_date` in JSONB response ≥ yesterday | error |
| `get_peers_and_idx_valuation_summary('banks')` | Returns data (not empty) | error |
| `get_idx_peers_growth_and_forecasts('financial','banks')` | Returns data (not empty) | error |
| `get_news_per_dimensions_by_ticker_subsector('BBCA.JK','banks')` | Returns data (not empty) | warning |
| `get_idx_yield_ttm` | Returns data (not empty) | error |
| `get_companies_loan_quality` | Latest year = current_year - 1 | error |
| `get_idx_resilience` | Returns data (not empty) | error |
| `get_companies_state_owned` | Returns data (not empty) | error |
| `get_upcoming_dividends_and_splits` | All dates ≥ yesterday | error |
| `get_idx_most_traded(1,5)` | Date = latest idx_daily_data date | error |
| `get_idx_volume(1)` | Date = latest idx_daily_data date | error |

### RPC Response Structure

Some RPC functions return JSONB data with period keys:

```json
// get_top_gainers / get_top_losers response structure
{
  "1d": [{"symbol": "BBCA.JK", "latest_close_date": "2025-11-27", ...}],
  "7d": [...],
  "14d": [...],
  "30d": [...]
}
```

The validator extracts items from all period keys and checks the `latest_close_date` field.

### RPC Validation for Market Cap Functions

`get_top_mcap_gainers` and `get_top_mcap_losers` use a different validation approach:
- These functions don't have date columns
- Instead, the validator compares `last_close_price` with the latest close price from `idx_daily_data`
- An anomaly is flagged if prices don't match for any symbol

**API Endpoints:**
- `POST /api/validation/run/rpc-functions` - Validate all 16 functions
- `POST /api/validation/run/rpc-functions/{function_name}` - Validate specific function

---

## Local Development Setup

### Backend Setup

```cmd
REM 1. Clone repository
git clone https://github.com/supertypeai/sectors_guard_validator.git
cd sectors_guard_validator

REM 2. Create virtual environment
python -m venv venv
venv\Scripts\activate

REM 3. Install dependencies
pip install -r requirements.txt

REM 4. Create .env file (copy from .env.example)
copy .env.example .env
REM Edit .env with your credentials

REM 5. Run development server
uvicorn app.main:app --host 0.0.0.0 --port 8000 --reload

REM 6. Access API docs at http://localhost:8000/docs
```

### Frontend Setup

```cmd
REM 1. Clone repository
git clone https://github.com/supertypeai/sectors_guard.git
cd sectors_guard

REM 2. Install dependencies
npm install

REM 3. Create .env file
echo REACT_APP_API_URL=http://localhost:8000 > .env

REM 4. Run development server
npm start

REM 5. Access dashboard at http://localhost:3000
```

### Running Validations Locally

```cmd
REM Run specific table validation
curl -X POST "http://localhost:8000/api/validation/run/idx_daily_data" ^
  -H "Authorization: Bearer YOUR_TOKEN"

REM Run all validations
curl -X POST "http://localhost:8000/api/validation/run-all" ^
  -H "Authorization: Bearer YOUR_TOKEN"

REM Run RPC validation (all functions)
curl -X POST "http://localhost:8000/api/validation/run/rpc-functions" ^
  -H "Authorization: Bearer YOUR_TOKEN"

REM Run single RPC validation
curl -X POST "http://localhost:8000/api/validation/run/rpc-functions/get_top_gainers" ^
  -H "Authorization: Bearer YOUR_TOKEN"

REM Get RPC validation results
curl "http://localhost:8000/api/dashboard/rpc-results?limit=10"
```

### Frontend API Service

The frontend uses `api.js` service with dedicated methods for RPC validation:

```javascript
// Regular table results (excludes RPC)
validationAPI.getResults(tableName, limit)

// RPC-only results (new endpoints)
validationAPI.getRPCResults(functionName, limit)
validationAPI.getRPCResultsByFunction(functionName, limit)

// Run RPC validations
validationAPI.runSingleRPCValidation(functionName)
validationAPI.runAllRPCValidation()
```

### Debug Scripts

The backend includes debug scripts for testing individual validations:

```cmd
python debug_daily_validation.py
python debug_annual_validation.py
python debug_quarterly_validation.py
python debug_dividend_validation.py
python debug_filings_validation.py
python debug_stocksplit_validation.py
python debug_alltime_validation.py
```

---

## Environment Variables

### Backend (.env)

| Variable | Required | Description |
|----------|----------|-------------|
| `SUPABASE_URL` | Yes | Supabase project URL |
| `SUPABASE_KEY` | Yes | Supabase anon/service key |
| `DB_PASSWORD` | Yes | Database password (for SQLAlchemy connection) |
| `BACKEND_API_TOKEN` | Yes | Bearer token for API authentication |
| `JWT_SECRET` | Yes | JWT signing secret |
| `AWS_ACCESS_KEY_ID` | Yes | AWS access key for SES email |
| `AWS_SECRET_ACCESS_KEY` | Yes | AWS secret key for SES email |
| `AWS_REGION` | No | AWS region (default: us-east-1) |
| `DEFAULT_FROM_EMAIL` | Yes | Sender email address for notifications |
| `DEFAULT_EMAIL_RECIPIENTS` | No | Comma-separated default email recipients |

### Frontend (.env)

| Variable | Required | Description |
|----------|----------|-------------|
| `REACT_APP_API_URL` | Yes | Backend API URL |

### GitHub Actions Secrets

| Secret | Used In | Description |
|--------|---------|-------------|
| `SUPABASE_URL` | data-validation.yml | Supabase URL |
| `SUPABASE_KEY` | data-validation.yml | Supabase key |
| `JWT_SECRET` | data-validation.yml | JWT secret |
| `PASSWORD` | data-validation.yml | DB password |
| `BACKEND_API_TOKEN` | data-validation.yml | API auth token |
| `AWS_ACCESS_KEY_ID` | data-validation.yml | AWS access key for SES |
| `AWS_SECRET_ACCESS_KEY` | data-validation.yml | AWS secret key for SES |
| `DEFAULT_FROM_EMAIL` | data-validation.yml | Sender email address |
| `DEFAULT_EMAIL_RECIPIENTS` | data-validation.yml | Email recipients |
| `FLY_API_TOKEN` | deploy.yml | Fly.io deployment token |
| `BACKEND_TRIGGER_URL` | fetch-sheet.yml | Backend trigger URL |

---

## Deployment

### Backend (Fly.io)

**App Name**: `sectors-guard-validator`
**Region**: Singapore (sin)
**Internal Port**: 8080

```cmd
REM Deploy to Fly.io
fly deploy

REM View logs
fly logs

REM SSH into instance
fly ssh console
```

**fly.toml highlights:**
```toml
app = "sectors-guard-validator"
primary_region = "sin"

[env]
PORT = "8080"

[[services]]
internal_port = 8080
protocol = "tcp"

[[services.http_checks]]
path = "/health"
method = "GET"
interval = "15s"
timeout = "2s"
```

### Frontend (Vercel)

Deployed automatically on push to main branch.

**vercel.json:**
```json
{
  "rewrites": [
    { "source": "/(.*)", "destination": "/" }
  ]
}
```

### Docker (Alternative)

```cmd
REM Build image
docker build -t sectors-guard-validator .

REM Run container
docker run -p 8080:8080 --env-file .env sectors-guard-validator
```

---

## Quick Reference

### Common API Calls

```bash
# Health check
curl https://sectors-guard-validator.fly.dev/health

# Get available tables
curl https://sectors-guard-validator.fly.dev/api/validation/tables

# Run validation (with auth)
curl -X POST "https://sectors-guard-validator.fly.dev/api/validation/run/idx_daily_data" \
  -H "Authorization: Bearer $TOKEN"

# Get dashboard stats
curl https://sectors-guard-validator.fly.dev/api/dashboard/stats

# Get validation results (regular tables only, excludes RPC)
curl "https://sectors-guard-validator.fly.dev/api/dashboard/results?table_name=idx_daily_data&limit=5"

# Get RPC validation results
curl "https://sectors-guard-validator.fly.dev/api/dashboard/rpc-results?limit=10"

# Run all RPC validations
curl -X POST "https://sectors-guard-validator.fly.dev/api/validation/run/rpc-functions" \
  -H "Authorization: Bearer $TOKEN"

# Run single RPC validation
curl -X POST "https://sectors-guard-validator.fly.dev/api/validation/run/rpc-functions/get_top_gainers" \
  -H "Authorization: Bearer $TOKEN"
```

### Anomaly Severity Levels

| Severity | Description | Stored in DB | Email Alert |
|----------|-------------|--------------|-------------|
| `error` | Critical issue requiring attention | Yes | Yes |
| `warning` | Potential issue for review | No | No |
| `info` | Informational only | No | No |

### Status Values

| Status | Meaning |
|--------|---------|
| `success` | Validation completed with no errors |
| `error` | Validation found error-level anomalies |
| `warning` | Validation found warnings only |

---