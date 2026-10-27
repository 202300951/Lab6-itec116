# ITEC 116 Activity 6 — E-Commerce to BI System

A runnable JavaScript prototype aligned with the Activity 6 rubric. It uses Node.js built-in modules only; no `npm install` is required.

## Requirements
- Node.js 18 or newer

## Run
1. Extract the ZIP.
2. Open a terminal in the project folder.
3. Run `npm start` (or `node server.js`).
4. Open http://localhost:3000

Default demo API token: `itec116-demo-token`

For a custom token:
- Windows PowerShell: `$env:BI_API_TOKEN="your-secret-token"; node server.js`
- macOS/Linux: `BI_API_TOKEN="your-secret-token" node server.js`

## How to demo
1. Use **Track A — Manually Enter an Order** to enter a customer ID, transaction ID (optional), date/time, status, and one or more product line items.
2. Click **Save Order to BI Staging**. The system calculates the order total and submits the transaction to the authenticated BI ingestion endpoint.
3. Click **Run ETL Pipeline** to clean and transform raw JSON into the fact/dimension model.
4. Review KPI cards and the recent transaction table.
5. Click **Export JSON Data** to download the raw and transformed data.

## Rubric mapping
- **Track A — Transactional Event Code:** `/api/demo-order` generates a transaction with `transaction_id`, `timestamp`, `entity_id`, `monetary_value`, `status`, and `line_items`.
- **Track B — BI Ingestion/Staging:** `POST /api/transactions` requires `Authorization: Bearer <token>`, validates schema, rejects duplicate IDs, and logs accepted raw transactions in `data/raw-transactions.jsonl`.
- **Track C — ETL/Dimensional Model:** `POST /api/etl/run` normalizes dates to ISO-8601, trims identifiers/names, validates numeric fields, maps status codes to labels, and writes `Fact_Sales`, `Dim_Customer`, `Dim_Product`, and `Dim_Time` structures to `data/bi-mart.json`. Optional `unit_cost` enables net margin calculation; if absent, margin is `null`, not guessed.
- **Track D — QA/Governance:** Dashboard reports ingestion error rate, row reconciliation accuracy, and payload total match rate. ETL response reports input/transformed counts and rejected/duplicate counts.

## API examples

Create a transaction with the authenticated staging endpoint:

```bash
curl -X POST http://localhost:3000/api/transactions \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer itec116-demo-token" \
  -d '{
    "transaction_id": "ORD-1001",
    "timestamp": "2026-10-27T10:30:00.000Z",
    "entity_id": "C-1001",
    "monetary_value": 998,
    "status": "completed",
    "line_items": [
      {"product_id":"P-1","name":"Keyboard","quantity":2,"unit_price":499,"unit_cost":300}
    ]
  }'
```

Endpoints:
- `GET /api/health` — health check
- `POST /api/transactions` — authenticated BI ingestion
- `POST /api/etl/run` — run ETL
- `GET /api/dashboard` — dashboard metrics and dimensions
- `GET /api/export` — export raw and transformed data as JSON

## Notes / limitations
- This is a classroom prototype with local JSON-file persistence, not a production commerce system or secure enterprise warehouse.
- The dashboard manual-entry form uses the configured demo token to submit to the authenticated ingestion endpoint. For classroom use, change the default token before sharing the app.
- The raw JSONL log is append-only; the ETL mart is rebuilt from raw records each run.
- Ingestion error rate reflects invalid records present in the raw log. Requests rejected before logging are returned as HTTP errors and do not enter the raw log.
- Row reconciliation compares expected transaction line-item count with fact row count. Duplicate transaction IDs are prevented at the ingestion endpoint and skipped during ETL as an extra safeguard.
- Use only synthetic data; do not store real customer payment details.
