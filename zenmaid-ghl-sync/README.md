# ZenMaid → GoHighLevel Sync

**26 nodes · learning project** — keeps GoHighLevel in step with a cleaning business's ZenMaid bookings. ZenMaid has no public API or webhooks, so a Zapier step forwards each event to this workflow and n8n does the GHL work through the official API v2.

## What each event does in GHL

| ZenMaid event | GHL result |
|---|---|
| `customer_created` | Upsert contact, tag `zenmaid-customer` |
| `booking_created` / `one_time_service_created` | Tags `zenmaid-customer` `zenmaid-booked`, opportunity → *Booked in ZenMaid* (won) |
| `recurring_service_created` | Tag `recurring-client`, opportunity → *Recurring Client* (won) |
| `recurring_service_cancelled` | Tag `recurring-cancelled` (fires a GHL win-back workflow) |
| `appointment_status_updated` (Completed) | Tag `job-completed` (fires a GHL review-request workflow), log row |
| `invoice_paid` | Log row for the weekly revenue report |

## Design decisions

- **Shared-secret check** on the `x-tcs-secret` header; anything else gets a 401.
- **Plan first, then act.** One code node turns the event into a plan of GHL changes with no API calls, so **dry-run mode** can show every call it would make.
- **Pipeline and stage matched by name**, so nobody pastes stage IDs. The workflow moves the contact's open opportunity or creates one if none exists.
- **Retries** on every GHL call and an Error Trigger lane that emails the failure.

## Setup

1. Import `workflow.json`, open **Config**, keep `dryRun: true`, and run **Test with sample events**. The *Preview (dry run)* node lists every GHL call.
2. In GHL: Settings → Private Integrations → create a token with contacts, opportunities and calendars scopes.
3. Create an n8n **Header Auth** credential: name `Authorization`, value `Bearer <token>`, and select it on every GHL node.
4. Fill `locationId`, `sharedSecret` and optionally `sheetId` in Config; set `__YOUR_EMAIL__` in *Format Error Alert*; then set `dryRun: false`.

**APIs:** GoHighLevel API v2 (LeadConnector) · Google Sheets · Gmail · Zapier webhooks
