# Timesheet Compliance Hub

**36 nodes** — checks staff timesheets in Xero Practice Manager across AU, PH and IN, chases missing time, and gives management daily and weekly visibility. Runs on built-in demo data by default.

## Lanes

| Lane | Trigger | What it does |
|---|---|---|
| ① Check | Weekdays 8:47am, Fri 3:43pm, or dashboard webhook | Loads XPM staff and time entries (XML → JSON) or demo data, classifies every staff-day as complete, short, missing, leave or holiday against each person's roster and country holidays, and checks entries for missing or vague notes, billable internal time and duplicates |
| AI note review | — | Claude labels vague notes on billable time (notes become invoice narratives); it never rewrites them |
| ② Reminders | — | Level 1 friendly → level 2 direct → level 3 cc'd to the manager. Claude writes the wording; code adds dates, hours and the XPM link. Sent via Microsoft Graph, with static data so a re-run never double-sends |
| ③ Visibility | — | Daily digest with a compliance grid, Friday summary with hours, billable % and busiest jobs, and a live HTML dashboard at `GET /webhook/timesheet-status` |
| ⑤ Errors | Error Trigger | Email alert through Microsoft Graph |

## Design decisions

- **AI never writes a number.** Reminder text containing any digit or link is replaced with a template. The digest summary is rejected if it contains a number that isn't in the computed facts.
- **Shadow mode.** In live mode, reminders first go to a manager for a week of review before reaching staff.
- **Untrusted input.** Staff-typed notes are passed to Claude as data with an instruction never to follow them.

## Setup

1. Import `workflow.json` and open **Config** (the only node to edit).
2. Keep `dataSource: 'demo'` and set `demoInbox` to your email; every message goes there.
3. Credentials: **Microsoft Outlook OAuth2**, **Anthropic**, **Basic Auth** (dashboard webhook); for live data, a generic **OAuth2** credential for Xero with the `practicemanager` scope.
4. For live XPM data: set `xeroTenantId`, the staff roster and holiday lists, then `dataSource: 'xpm'`.

**APIs:** Xero Practice Manager v3.1 · Microsoft Graph (Outlook sendMail, optional Teams) · Anthropic Messages API
