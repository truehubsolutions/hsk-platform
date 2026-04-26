# API Reference — HSK Platform Backend

The FastAPI backend exposes a set of internal REST endpoints consumed by the AppSheet front-end and automated scripts. All endpoints require an `Authorization: Bearer <token>` header.

---

## Base URL

```
https://<internal-host>/api/v1
```

---

## Payroll

### `POST /payroll/run`

Triggers a payroll computation for the specified pay period.

**Request body:**
```json
{
  "period_start": "2026-04-01",
  "period_end": "2026-04-15",
  "employees": ["all"] // or list of employee IDs
}
```

**Response:**
```json
{
  "period": "2026-04-01 to 2026-04-15",
  "processed": 12,
  "total_gross": 185000.00,
  "total_net": 158420.50,
  "payslips_generated": 12,
  "xero_journal_posted": true
}
```

### `GET /payroll/payslip/{employee_id}/{period}`

Returns a signed URL to the generated payslip PDF for the given employee and period.

---

## Expenses

### `POST /expenses/submit`

Submits a new expense for approval.

**Request body:**
```json
{
  "employee_id": "EMP-001",
  "amount": 1500.00,
  "currency": "PHP",
  "category": "transport",
  "description": "Grab to supplier site",
  "date": "2026-04-22",
  "receipt_url": "https://storage.example.com/receipts/abc123.jpg"
}
```

### `PATCH /expenses/{expense_id}/approve`

Manager approval endpoint. Posts the expense to Xero as a bill on approval.

### `GET /expenses/summary`

Returns monthly expense totals grouped by category.

---

## Leave

### `POST /leave/request`

**Request body:**
```json
{
  "employee_id": "EMP-001",
  "leave_type": "vacation",
  "start_date": "2026-05-01",
  "end_date": "2026-05-03",
  "reason": "Family event"
}
```

### `GET /leave/balance/{employee_id}`

Returns remaining leave credits per type for the given employee.

---

## Data Flow

```
AppSheet Form Submit
  → FastAPI endpoint
  → Validate + compute
  → Write to Google Sheets (operational log)
  → Push to Xero API (financial record)
  → Return response to AppSheet
```

---

## Xero Integration

All financial records (payroll journals, expense bills, invoice reconciliation) are pushed to Xero via the Xero API using OAuth 2.0. The API client is initialised with a refresh token stored securely in environment variables — never in source code.

Key Xero operations used:
- `POST /api.xro/2.0/Invoices` — supplier bills for approved expenses
- `POST /api.xro/2.0/ManualJournals` — payroll journal entries
- `GET /api.xro/2.0/BankTransactions` — bank feed for reconciliation
