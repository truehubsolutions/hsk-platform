# Data Flow — HSK Platform

This document maps how data moves through the system end-to-end.

---

## Payroll Flow

```
1. HR triggers payroll run via AppSheet
2. AppSheet calls POST /payroll/run
3. FastAPI fetches employee records from Google Sheets
4. Engine computes: gross pay, SSS, PhilHealth, Pag-IBIG, withholding tax, net pay
5. Payslip PDFs generated (one per employee)
6. PDFs stored on cloud storage, signed URLs returned
7. Payroll journal entry posted to Xero
8. Summary report emailed to management
```

---

## Expense Flow

```
1. Employee submits expense via AppSheet form
2. FastAPI logs to Google Sheets with status: PENDING
3. Manager notified via email
4. Manager approves/rejects in AppSheet
5. On approval: expense posted to Xero as supplier bill
6. Google Sheets status updated: APPROVED
7. Included in next payroll run if reimbursable
```

---

## Leave Flow

```
1. Employee submits leave request via AppSheet
2. FastAPI checks leave balance in Google Sheets
3. If sufficient: status set to PENDING, manager notified
4. Manager approves/rejects
5. On approval: leave balance decremented in Google Sheets
6. If unpaid leave: flagged for payroll deduction in next run
```

---

## Bookkeeping Sync

```
Xero bank feeds → auto-matched against invoices
Unmatched items → flagged for manual review
Monthly close → P&L + Balance Sheet pulled from Xero API → formatted report generated
```
