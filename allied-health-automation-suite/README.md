# Allied Health Automation Suite

**83 nodes · demo build** — automation for an Australian multidisciplinary allied health practice (psychology, exercise physiology, OT). The Halaxy practice-management steps return **sample data** so the suite runs without a Halaxy account; Groq, Outlook, Excel and Xero steps call the real APIs.

## Lanes

| Lane | Trigger | What it does |
|---|---|---|
| 00 · API capability check | Manual | Reads Halaxy's FHIR CapabilityStatement and lists which resources allow read/search/create/update, before anything is built on them |
| 01 · Emailed referral → Halaxy | Outlook poll (or the demo trigger) | Cleans the email, tokenises identifiers, has the LLM extract structured fields, validates them with plain rules, matches or creates the patient, attaches the referral, and routes anything unclear to a staff review queue |
| 02 · Halaxy invoices → Xero | Hourly, business days | Maps FHIR invoices to Xero, finds or creates the contact, checks the `HLX-<number>` reference before creating a **DRAFT** invoice, logs every result |
| 03 · Fill cancelled appointments | Weekdays 4:30pm | Finds today's cancellations that haven't rebooked, has the LLM draft a re-engagement message from minimal data, and holds everything behind a human approval form |
| 99 · Monitoring | Error Trigger + daily 8am | Scrubbed error alerts (no client data), plus a daily count of failed executions via the n8n API |

## Design decisions

- **The LLM never sees identifiers.** Phones, emails, Medicare and NDIS numbers are replaced with tokens such as `[PHONE_1]` in a code node; the model returns token names and the real values are mapped back locally.
- **AI output is never written unchecked.** Extracted fields go through deterministic validation (required fields, date formats, allowed service values, confidence threshold) before any Halaxy call.
- **Risk language goes to a human the same day.** Referrals mentioning self-harm or urgent contact are flagged `urgent` and skip automation entirely.
- **Patient matching is scored.** Surname + DOB search, then given name, mobile and email are scored; only a single strong match is auto-linked, and possible duplicates go to staff.
- **Duplicate-safe invoicing.** Xero is checked for the invoice reference before creating, an `Idempotency-Key` header is sent, and calls are batched to stay under Xero's rate limit.
- **Human approval gate.** Outreach drafts are sent to the practice manager with a form link (not a plain link, so email scanners can't auto-approve). No reply in 18 hours means nothing is sent.

## Running the demo

1. Import `workflow.json`.
2. Create credentials: **Microsoft Outlook OAuth2**, **Microsoft Excel OAuth2**, **Header Auth** for Groq (`Authorization: Bearer <key>`), generic **OAuth2** for Xero, **n8n API**.
3. Replace placeholders: `__YOUR_EMAIL__`, `__EXCEL_WORKBOOK_ID__`, `__OUTLOOK_PROCESSED_FOLDER_ID__`, `__OUTLOOK_NEEDS_REVIEW_FOLDER_ID__`.
4. Choose **Demo: Run With Sample Referrals** from the Execute workflow menu. It feeds in five fictional referrals: new client, existing client, possible duplicate, incomplete, and urgent risk.

To go live, replace each node marked *DEMO – sample Halaxy data* with an HTTP Request node to the Halaxy API.

**APIs:** Halaxy FHIR (sample) · Microsoft Graph (Outlook, Excel) · Groq · Xero Accounting · n8n API
