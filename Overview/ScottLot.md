
## High-Level Design — Goodyear Website (`goodyear_website/`)

### 1. What it is
A monolithic **Flask** web app (single ~5,800-line file, `website.py`) that acts as an internal operations portal sitting between the warehouse (**Logiwa WMS**), freight forwarder (**Flexport**), and ERP (**NetSuite**), plus MySQL databases used for reporting/workflow state. It's deployed via **Gunicorn** (`wsgi.py` exposes `app`) behind an **Nginx** reverse proxy (`ProxyFix` middleware is configured for `X-Forwarded-*` headers).

### 2. Core libraries/stack

| Concern | Library |
|---|---|
| Web framework | `Flask` (Jinja2 templates in `html_templates/`, static assets in `static/`) |
| WSGI/proxy | `werkzeug` (`ProxyFix`, `generate_password_hash`/`check_password_hash`) |
| Rate limiting | `Flask-Limiter` (in-memory storage, per-route limits e.g. login 10/min, password reset 5/min) |
| DB access | `mysql-connector-python` (raw SQL, no ORM) — 4 separate MySQL schemas |
| HTTP calls out | `requests` (Flexport, Logiwa, NetSuite) |
| ERP auth | `oauthlib.oauth1.Client` (HMAC-SHA256 signed requests to NetSuite RESTlets) |
| Config/secrets | `python-dotenv` — loads ~11 separate `.env.*` files at startup (per integration: Logiwa, Flask key, reporting DB, playground DB, workflow DB, chargeback DB, Flexport, NetSuite, outbound shipping DB, AWS, MS/IT) |
| Email | `smtplib` / `email.mime` for password-reset emails via Outlook SMTP |
| Docs/labels | `reportlab`, `python-barcode`, `Pillow` (used by the `scripts/` subprocess jobs for BOL/pallet/carton labels) |
| Metrics | `prometheus_client` + `psutil` exposed at `/metrics` (`metrics.py`) |
| Misc | `uuid`, `re`, `secrets`, `subprocess`, `logging` |

### 3. Application structure
- **`website.py`** – single Flask app: routes, auth, business logic, DB queries, external API calls, decorators — all in one file.
- **`scripts/`** – standalone Python scripts (BOL generator, pallet/carton label generators, replenishment jobs, PO updates, retries) invoked via `subprocess.run([python_path, script_path, ...])` rather than imported — isolates long-running/label-generation work from the web process, with timeouts (e.g. 300s) and stdout/stderr captured into logs.
- **`html_templates/`** – Jinja2 templates (auth, admin, shipment details/review, tariff review, BOL/pick-list templates, NetSuite response, error pages).
- **`static/`** – images (signature, letterhead) served with signed/obscure resource URLs (`/api/resource/auth-sig-b8f3k9`) rather than direct static paths — a minor obfuscation/anti-scraping measure.
- **`metrics.py`** – separate module for Prometheus gauges (load, memory, disk, CPU), imported into `website.py` and exposed at `/metrics` (rate-limiter exempt).

### 4. Data layer — 4 distinct MySQL "databases" (all via `mysql.connector.connect`, opened/closed per-request, no pooling)

| Config var | Purpose |
|---|---|
| `DB_CONFIG` (`logiwa_reporting`) | User credentials/auth, outbound shipping review |
| `LABEL_DB_CONFIG` (`logiwa_workflow_process`) | Carton-label processing queue |
| `VENDOR_CHARGEBACK_DB_CONFIG` | Flexport packing/shipment data, tariff rates, inbound receiving, NetSuite submission status — the biggest consumer |
| `OUTBOUND_SHIPPING_DB_CONFIG` | Outbound shipment documents |

### 5. Auth & security
- Session-based login (`session["logged_in"]`, `session["role"]`), password hashing via Werkzeug, password-reset tokens with 24h expiry emailed via SMTP.
- Role-based decorators stacked on routes: `login_required`, `admin_required`, `accounting_or_admin_required`, `rf_supervisor_or_admin_required`, `operations_supervisor_or_admin_required`, `operations_accounting_supervisor_or_admin_required` (roles: user/supervisor/accounting/RF_user/operations/admin).
- `before_request` hook: IP blocklist check (`BLOCKED_IPS`/`logs/blocked_ips.txt`), auto-block after repeated malicious requests (`THREAT_THRESHOLD=10` in 24h), 5-hour idle session timeout.
- `after_request` hook adds `X-Robots-Tag`, `X-Frame-Options: DENY`, `X-Content-Type-Options: nosniff`.
- Decoy routes for bots: `/.git`, `/favicon.ico`, `/robots.txt` return controlled responses; global exception handlers (`internal_error`, `handle_exception`).

### 6. Key API flow patterns
Two repeating patterns across the ~50 routes (grep of `@app.route`):

**A. Fire-and-forget label/BOL generation** (`/api/bill-of-lading`, `/api/labels/pallet`, `/api/labels/carton`, `/api/retry-fulfillment`, `/api/master-bill-of-lading`, `/api/dtc-replenishment-pick-list`, `/api/update-open-purchase-orders`):
```
Browser → Flask route → subprocess.run(scripts/*.py, order_id) → parses stdout/return code → JSON response
```
These scripts themselves call the Logiwa REST API (`myapi.logiwa.com/v3.1/...`) using a bearer token from `.env.logiwa_production`.

**B. Shipment review / cross-system data aggregation** (`/inbound-shipments`, `/shipment-details/<id>`, `/inbound-receiving-review`, `/outbound-shipping-review`, `/tariff-review`, `/shipment-documents`):
```
Browser → Flask page route (renders template)
        → Flask /api/... route
             → MySQL query (VENDOR_CHARGEBACK / OUTBOUND_SHIPPING db)
             → fetch_flexport_data() → Flexport API (shipment + booking + PO line items), with 401 → auto-reload token from .env and retry once
             → merge DB + Flexport data → JSON to frontend
```

**C. NetSuite submission** (`/api/submit-to-netsuite/<id>`, the most complex flow, ~650 lines):
```
Frontend sends edited header + line items (user-editable freight/tariff overrides)
  → validate against DB shipment record
  → fetch_flexport_data() for origin/destination + accepted freight quote (source of truth for country fields — DB/UI values are explicitly overridden)
  → calculate_freight_allocation() / build_container_payload() (groups line items by container)
  → call_netsuite_restlet(): oauthlib HMAC-SHA256 signs request → POST to NetSuite RESTlet URL
  → aggregate_container_responses() / parse_netsuite_error() → update_shipment_netsuite_status() in DB
  → JSON result → frontend, with a dedicated /shipment-details/<id>/netsuite-response page to view raw responses
```

### 7. Notable design characteristics
- **Monolith, not modular**: everything (routes, SQL, business logic, external API clients) lives in one file — no blueprints, no service layer, no ORM.
- **Subprocess isolation for heavy jobs**: label/BOL generation and retries run as separate OS processes rather than in-request, likely to isolate crashes/long-running work from the web worker and reuse existing standalone scripts.
- **Flexport treated as source of truth** for country/geo data even when duplicated in local DB or user-edited UI fields — data-integrity guardrails are hardcoded (e.g. NetSuite submission always re-derives origin country from Flexport, ignoring any UI-submitted value).
- **Defense-in-depth for a low-traffic internal tool**: bot-scan counters, IP auto-block, decoy routes, obfuscated static resource URLs — suggests the app has been targeted by scanners despite being internal-facing.
- **Config sprawl**: 11 separate `.env.*` files loaded at import time, one per integration/DB — makes secrets rotation and env management explicit per-integration but couples startup to filesystem layout (`/home/ubuntu/logiwa_flask_app/.env.*` hardcoded paths).

Let me know if you'd like a deeper dive into any specific flow (e.g. the NetSuite freight-allocation math, the label-generation scripts, or the tariff-review upload/download flow).