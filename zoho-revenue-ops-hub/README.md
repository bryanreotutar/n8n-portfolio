# Zoho Revenue Ops Hub

**66 nodes** — connects a website, Zoho CRM, Zoho Books and Zoho Flow, with AI lead qualification and an AI website chatbot.

## Lanes

| Lane | Trigger | What it does |
|---|---|---|
| ① Website → CRM | Webhook (header-secret auth) | Server-side validation with a honeypot, lead upsert, AI qualification with a rules-based fallback, routing through a Deluge function, nurture event to Zoho Flow |
| ② Closed Won → Books | Webhook from a Deluge workflow | Re-reads the deal, finds or creates the Books customer, checks for an existing `CRM-DEAL-<id>` invoice, creates a one-off invoice or a recurring profile, writes invoice details back to the deal |
| ③ Overdue reminders | Weekdays 9:05 Sydney | Cadence of 1/7/14/21/30 days then weekly, with a Monday catch-up for weekend days; AI writes wording only; 21+ days creates a CRM escalation task; daily AR digest |
| ④ Payment → CRM | Webhook from a Books Deluge function | Splits one payment across the invoices it settled, maps each back to its deal through the reference, updates billing status |
| ⑤ Website chatbot | n8n Chat Trigger | AI agent with window memory and two tools: an invoice-status check (verified server-side in Deluge) and CRM lead creation |
| ⑥ Errors | Error Trigger | Formats the failure and posts it to Zoho Flow |

## Design decisions

- **Webhooks carry IDs only.** Deal and payment webhooks send just an ID; the workflow re-reads the record from Zoho, so a forged call can't inject an amount or customer.
- **Idempotent invoicing.** Every invoice gets a `CRM-DEAL-<id>` reference that is looked up before creation, so a retried webhook can't double-bill.
- **AI never writes facts into customer emails.** Reminder text from the model is scanned for URLs, currency amounts, dates, currency codes and day counts. If any appear, or the JSON doesn't parse, a pre-written template for that tone is used instead. Invoice number, balance, due date and pay link are inserted by code.
- **Prompt-injection hygiene.** Web form text is passed to the model as data with an explicit instruction never to follow it.

## Setup

1. Import `workflow.json`.
2. Create credentials: **Zoho OAuth2** (CRM nodes), generic **OAuth2** named "Zoho Books", **OpenAI** (pointed at Groq's OpenAI-compatible endpoint in this build), **Header Auth** named `X-Webhook-Secret` for the three webhooks.
3. Replace placeholders in every Config node: `__ZOHO_FLOW_WEBHOOK_URL__`, `__CRM_FN_ROUTE_LEAD_URL__`, `__BOOKS_ORG_ID__`, `__BOOKS_ITEM_ID__`. In the chatbot lane: `__WEBSITE_ORIGIN__`, `__COMPANY_NAME__`, `__ACCOUNTS_EMAIL__`, `__SALES_EMAIL__`, `__CRM_FN_INVOICE_STATUS_URL__`.

The Config nodes stop the run with a clear message if a placeholder is still unset. The PHP website form and the Deluge scripts referenced in the sticky notes are not included in this repo.

**APIs:** Zoho CRM REST v8 · Zoho Books v3 · Zoho Flow webhooks · Deluge functions · Groq (OpenAI-compatible)
