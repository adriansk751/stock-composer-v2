# Base44 Dev Environment

## Project Overview
Streamlit-based Inventory Forecast System. Calculates purchasing requirements from sales velocity and stock levels. Integrates with SmartBill (stock + invoices), easySales (orders), Google Sheets (incoming inventory), PostgreSQL (forecast history), and SMTP (email alerts).

## Stack
- **Framework:** Streamlit (Python 3.11)
- **Package manager:** uv (lockfile: `uv.lock`, manifest: `pyproject.toml`)
- **Database:** PostgreSQL 16 (local compose service)
- **Entry point:** `app.py` (Streamlit app)

## Running the app
```
docker compose -f docker-compose.base44.yml up -d
```
- App is served on **port 3000** (mapped to internal Streamlit port 8501)
- Health check: `GET /_stcore/health`
- PostgreSQL runs as a companion service with auto-created `forecast` database

## Architecture notes
- All external integrations are **optional** — the app boots and shows a setup page without any credentials. Missing credentials produce warnings, not errors.
- PostgreSQL is optional too: if `PGHOST`/`PGDATABASE`/`PGUSER` are set (they are, via compose), the app initializes tables on first forecast run and enables the History & Trends tab.
- The `.streamlit/config.toml` already disables CORS and XSRF protection and binds to 0.0.0.0.
- There is a second page at `pages/` (Multi Warehouse Comparison) accessible via the sidebar.

## External service credentials (all optional)
Provided by the user via the Base44 secrets system (delivered to `/run/base44/app.env`):
- `SMARTBILL_EMAIL`, `SMARTBILL_TOKEN`, `SMARTBILL_CIF` — SmartBill API (stock + invoice data)
- `EASYSALES_TOKEN` — easySales API (order/sales data)
- `GOOGLE_CREDENTIALS_JSON` — Google service account JSON string for Sheets API
- `GOOGLE_SHEET_ID`, `GOOGLE_WORKSHEET_NAME` — default Google Sheets config
- `SMTP_SERVER`, `SMTP_PORT`, `SMTP_USER`, `SMTP_PASSWORD` — email alerts

## Verifying the app works
1. `docker compose -f docker-compose.base44.yml ps` — both services should be healthy
2. `curl -s http://localhost:3000/_stcore/health` — should return 200
3. Open the preview — should show the "📦 Inventory Forecast System" page with sidebar config
