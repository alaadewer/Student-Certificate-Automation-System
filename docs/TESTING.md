# Testing Guide

Import `data/certificates_demo.xlsx` into Google Sheets first (see the README, step 1). To receive emails during tests, replace `@example.com` with your own address in the rows or form inputs you run.

## Lane 1: Form (Event-driven)

Submit each row of `data/form_test_inputs.csv` to the form's **Test URL**.

| # | Test | Input | Expected result |
|---|------|-------|-----------------|
| 1 | Valid new request | Mona Adel, Data Analysis, 88 | Certificate issued; new row `issued`, `template_version=v2` |
| 2 | Duplicate | Ahmed Hassan, AI Automation, 92 | Skip Duplicate: no certificate, no new row |
| 3 | Duplicate, different casing and spaces | `  Ahmed.Hassan@Example.com ` | Same `cert_id` after cleanup, so Skip Duplicate |
| 4 | Same person, other course | Ahmed Hassan, Digital Marketing, 79 | Different `cert_id`, new certificate |
| 5 | Score below passing mark | Sherif Mounir, 50 | Row `rejected`, error `Score below passing mark (60)` |
| 6 | Re-submit a stuck request | Hana Tarek, AI Automation, 81 | Row was `pending`, so issuing completes and it becomes `issued` |
| 7 | Invalid email | `reem.sayed.example.com` | The form's email field may block it; if it passes, logged as `rejected` |

## Lanes 2 to 4

Open the workflow and use **Execute workflow** on the named trigger.

| Lane | Trigger | Expected result | Count |
|------|---------|-----------------|-------|
| 2: Catch-up | Schedule Nightly 23:00 | Issues Hana, Bassem, Salma, Mostafa | 4 |
| 3: Backfill | Manual Backfill Trigger | Regenerates `issued` rows with `template_version` ≠ v2; no email, PDF in Drive | 6 |
| 3: Backfill (repeat) | Manual Backfill Trigger | Finds nothing (rows are now v2) | 0 |
| 4: Weekly report | Schedule Sunday 09:00 | Excel of certificates issued in the last 7 days | 6* |
| Rejected rows | none | Stay `rejected`; ignored by Catch-up and Backfill | 2 |
| Already v2 | Backfill | Ahmed Hassan, Nour Samir, Alaa Demo are not regenerated | 3 |

\* The weekly count is a rolling 7-day window and the sample dates are fixed. The value 6 is calculated for **Sunday 2026-11-15 09:00 (Africa/Cairo)**. At any other time, edit a few `issued_at` values to recent dates and count the `issued` rows from the last 7 days.

## Verifying a run

- **Sheet:** `status`, `issued_at`, `template_version`, `regenerated_at`, and `error` update per row.
- **Drive:** one `Certificate_<cert_id>.pdf` per certificate in the archive folder.
- **Inbox:** the certificate PDF (Lanes 1 and 2) or the weekly XLSX.
- **n8n Executions:** check the lane's execution for the failing node if a row stays `pending` or turns `failed`.

## Reset

Re-import `certificates_demo.xlsx` as a new sheet, or edit rows back to their original `status` and `template_version`.
