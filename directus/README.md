# Divination Directus Backend

This project expects its own Directus instance for divination report and consult operations.

## Local Start

1. Copy `directus/.env.example` to `directus/.env`.
2. Replace `DIRECTUS_KEY`, `DIRECTUS_SECRET`, and admin credentials.
3. Run `docker compose --env-file .env up -d` from this `directus` folder.
4. Set the frontend env to `VITE_DIRECTUS_URL=http://127.0.0.1:8056`.

## Operations Workflow

Use the backend as a report, share, and consult workspace:

1. New activity: user spins the wheel, shares a card, or submits a deep consult request.
2. Qualification: operator reviews topic, contact info, channel, consent, and consult intent.
3. Follow-up: assign a consultant, set next action, and record interpretation notes.
4. Conversion: qualified consults move to `booked` or `in_progress`.
5. Closeout: records are closed, rejected, or archived with a reason.

## Collections

Create these collections in Directus Studio:

- `divination_reports`: wheel/spin report records.
- `divination_consults`: deep consult lead records.
- `divination_shares`: share card generation records.

## Shared Fields

Recommended fields for all three collections:

| Field | Type | Notes |
| --- | --- | --- |
| `name` | string | User nickname or display name. |
| `phone` | string | PII when present; restrict visibility. |
| `source_page` | string | Expected value: `divination`. |
| `action_id` | string | `spin`, `consult`, or `share`. |
| `context` | JSON | Frontend state such as active tab or current result. |
| `consent_accepted` | boolean | Required before consult follow-up. |
| `submitted_at` | datetime | Client submission time. |
| `status` | string | Pipeline enum below. Default `new`. |
| `owner` | string | Consultant or operator. |
| `priority` | string | `low`, `normal`, `high`, `urgent`. |
| `next_action` | string | Next follow-up step. |
| `notes` | text | Internal interpretation or follow-up notes. |

## Domain Fields

- `divination_reports`: `fortune_title`, `score`, `focus`, `result_payload`, `spin_seed`.
- `divination_consults`: `topic`, `preferred_channel`, `consultant_name`, `consult_summary`.
- `divination_shares`: `channel`, `share_nickname`, `share_card_url`, `shared_at`.

## Status Pipeline

Use one consistent enum across collections:

- `new`: just submitted, not reviewed.
- `contacted`: first contact attempted.
- `qualified`: valid topic and contact intent.
- `booked`: consult session scheduled.
- `in_progress`: interpretation or follow-up is ongoing.
- `closed`: handled successfully.
- `rejected`: invalid, duplicate, or not suitable.
- `archived`: kept for reporting only.

## Dashboard Views

Create Directus Insights or saved filters for:

- Today's spins, shares, and consults.
- Consults grouped by `topic`, `status`, and `owner`.
- Share channel performance grouped by `channel`.
- High-score reports that later converted to consults.
- Open consults requiring next action.

## Permissions

- Public role: allow user registration and login only.
- Authenticated role: create records in the three business collections.
- Consultant role: read assigned consults and update notes/status.
- Operator role: read and update status, owner, priority, next action, and notes.
- Admin role: manage users, records, files, roles, permissions, and Insights.

## Local Mock Admin

The repository also includes a development-only Directus-compatible mock server. Run it from the workspace root:

```bash
node scripts/start-local-directus.mjs
```

Open `http://127.0.0.1:8056` to view the local divination operations dashboard. The mock supports record creation, status updates, list metadata, and JSON export at `/admin/export`.
