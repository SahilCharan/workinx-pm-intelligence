# WorkinX PM Intelligence

Single source of truth for the WorkinX PM Morning Brief / EOD reports.
Claude reads ClickUp, emits a compact JSON blob, and `render.py` turns it into a
branded, filterable HTML report that is posted to each PM's Slack channel.

> SOP this tool follows: **https://externalsop.workinxdigital.com**

## Pipeline
```
ClickUp ──▶ Claude (routines/*-prompt-template.md) ──▶ compact JSON
                                                          │
                                                render.py │ (zero tokens)
                                                          ▼
                                                   template.html  ──▶ Slack (upload_slack.sh)
```

## Files
| File | Purpose |
|------|---------|
| `template.html` | The report UI. Renders client-side from an embedded JSON blob. |
| `render.py` | JSON → HTML. Injects `date_iso` + field aliases so nothing renders blank. |
| `schema_example.json` | Reference payload + render smoke-test fixture. |
| `routines/morning-prompt-template.md` | Morning Brief prompt (fetch, flag, draft). |
| `routines/eod-prompt-template.md` | EOD Report prompt. |
| `routines/kavitha.json`, `routines/muskan.json` | Per-PM list_id + Slack channel. |
| `run.sh` | Render + upload to Slack for one PM. |
| `upload_slack.sh` | Slack 3-step external file upload. |

## Run locally
```bash
# render the sample to see the UI
python render.py schema_example.json report.html

# full pipeline (needs SLACK_BOT_TOKEN + SLACK_KAVITHA_CHANNEL)
bash run.sh kavitha morning kavitha_morning.json
```

## What the report shows
- **📅 Next 48 Hours calendar** — every due task bucketed into Overdue / Today / Tomorrow / Day After. Click any item to jump to its card. This is the "what do I do today + next 2 days" view.
- **Numbered task cards** — each card carries a `#N` badge that renumbers live when you filter, so "31 escalations" reads 1…31 with no manual counting. Every filter button shows a live count, e.g. `🚨 Escalations (31)`.
- **Full last client comment** — passed verbatim from ClickUp (never truncated).
- **48h follow-up** — when the PM/agency was the last to comment >48h ago and the client is silent, the task is flagged `FOLLOWUP_DUE` and a ready-to-send nudge is drafted (SOP follow-up rule).
- **Days in status** — `days_stale` shown per card (red >7d, amber >3d).
- **Suggested client reply / PM action / Post-to-ClickUp** inline.

## Data contract (per task)
`id, name ("Brand | Deliverable"), status, due_date, due_iso, assignee, days_stale, url, flags[], last_comment (full), reply_draft, followup_draft, action`

Flags: `ESCALATION, FOLLOWUP_DUE, CLIENT_WAITING, CLIENT_REPLIED, SEND_NOW, SLA_BREACH, OVERDUE, DUE_SOON, DUE_TOMORROW, BLOCKED, URGENT, NEEDS_PM_REVIEW, WIN, PROTOCOL_BREACH, STATUS_MISMATCH, INCOMPLETE`.
