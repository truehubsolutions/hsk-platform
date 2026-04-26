# HSK Platform — Integrated Business Operations System

A full-stack business operations platform built for a recycled materials trading client. Replaced a spreadsheet-only operation with a unified system covering bookkeeping, payroll, expense tracking, leave management, and day-to-day operational workflows.

---

## Background

The client was running a growing recycled materials trading business entirely on manual spreadsheets. No unified records, no automated payroll, no visibility into financials. This platform was built from scratch to give the team accurate books and operational systems that run without constant manual intervention.

---

## Architecture

```
┌─────────────────────────────────────────────┐
│              Web Application                │
│         (AppSheet — internal ops UI)        │
└──────────────────┬──────────────────────────┘
                   │
┌──────────────────▼──────────────────────────┐
│               Backend API                   │
│         (FastAPI — Python 3.11)             │
│  • Payroll engine                           │
│  • Expense categorisation                   │
│  • Leave management                         │
│  • Reporting endpoints                      │
└─────────┬───────────────────┬───────────────┘
          │                   │
┌─────────▼──────┐   ┌────────▼────────────────┐
│  Google Sheets │   │       Xero API           │
│  (operational  │   │  (source-of-truth books) │
│   data store)  │   │                          │
└────────────────┘   └─────────────────────────┘
```

---

## Modules

### Bookkeeping (Xero)
- Full Xero setup from scratch — chart of accounts, bank feeds, reconciliation rules
- Automated bank reconciliation via Xero API
- Monthly P&L, balance sheet, and aged receivables reports
- Multi-currency support for cross-border supplier payments

### Payroll System
- Custom payroll engine built in Python (FastAPI)
- Handles SSS, PhilHealth, Pag-IBIG, and withholding tax computations
- Payslip generation (PDF) triggered per pay period
- Payroll data pushed to Xero as journal entries

### Expense Tracking
- Employees submit expenses via AppSheet form
- Auto-categorised against chart of accounts
- Approval workflow (manager → finance) before Xero posting
- Monthly expense summary report

### Leave Management
- Leave requests and balances tracked in Google Sheets (AppSheet front-end)
- Leave types: Vacation Leave, Sick Leave, Emergency Leave
- Auto-deduction from payroll for unpaid leave
- Leave summary visible to managers in real time

### Internal Web Application
- AppSheet app used by the team for daily operations
- Dashboard: sales orders, delivery schedules, supplier records
- Connected to Google Sheets as the operational data layer
- Role-based access (management vs. staff views)

---

## Tech Stack

| Layer | Tool |
|---|---|
| Backend API | FastAPI (Python 3.11) |
| Internal App | AppSheet |
| Operational Data | Google Sheets + Apps Script |
| Bookkeeping | Xero (via REST API) |
| Automation | Google Apps Script + n8n |
| Hosting | Hetzner VPS (Ubuntu + Docker) |

---

## Key Outcomes

- Xero bookkeeping set up from scratch; clean monthly close achieved from month one
- Payroll processing time reduced from 2+ days (manual spreadsheet) to under 30 minutes
- Expense approval cycle reduced from ad-hoc to a structured 48-hour workflow
- Leave balances visible in real time — eliminated end-of-year leave disputes
- Internal app gave the ops team a single place for daily records instead of scattered files

---

## Status

Active — currently in retainer engagement. Ongoing bookkeeping, system maintenance, and feature additions as the business grows.

---

*Built and maintained by [True Hub Solutions](https://truehubsolutions.com)*
