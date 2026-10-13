# NextPlan Command Inbox

**Status: LIVE — production write/read acceptance verified on 14 October 2026.**

This deliberately PUBLIC repository carries explicit ChatGPT Plus commands to NextPlan without a Chrome listener or a paid Railway instance. [LifeOS GitHub Actions](https://github.com/changxinjiresearch/LifeOS/actions/workflows/nextplan-issue-inbox.yml) scans Issues on an approximately five-minute schedule (GitHub may delay runs). The app keeps reading the same canonical LifeOS `state.json`.

## Submit an operation

Create a GitHub Issue authored by `changxinjiresearch` with a title starting **`[NextPlan]`** and a JSON-only body, for example:

```json
{"version":3,"action":"update_project","project_id":"research-neuri","changes":{"status":"completed"}}
```

Exact IDs are required. Supported commands:

| Action | Required fields |
|---|---|
| `create_project` | `project_id`, `name`; optional `category`, `priority`, `next_action` |
| `update_project` | `project_id`, `changes` with status, priority, progress, or next_action |
| `create_task` | `project_id`, `name`; optional `task_id` |
| `update_task` | `project_id`, `task_id`, `changes` with status, name, deadline, or prep_days |
| `complete_task` | `project_id`, `task_id` |
| `update_milestone` | `project_id`, `milestone_id`, `status` |
| `set_deadline` | `project_id`, `title`, `date` (YYYY-MM-DD); optional deadline_id, task_id |
| `remove_deadline` | `project_id`, `deadline_id` |
| `request_delete_task` | `project_id`, `task_id`, `expected_name` (must match existing task) |
| `request_delete_project` | `project_id`, `expected_name` (must match existing project) |
| `confirm_delete` | second, separate Issue: `project_id`, `request_issue`, `expected_name`, `confirmation: "CONFIRM DELETE N"`; supply task_id for task deletion |

Deleting requires two separate Issues in order, plus the user's explicit confirmation before creating the second. A request Issue receives `pending_confirmation` and changes no project data. The confirmation Issue must match the request and target identity. This is a two-step application-level confirmation, **not** independent authentication or protection against compromise of the GitHub owner account.

**PUBLIC means PUBLIC. Never submit passwords, API keys, private letters, personal or patient data, or chat history.**

## Receipts

GitHub Issue creation proves only **submitted**. Processing is confirmed by a persisted receipt at:

`https://github.com/changxinjiresearch/LifeOS/blob/main/events/receipts/issue-N.json`

- `verified`: actual state builder finished and postcondition matched at execution time
- `pending_confirmation`: deletion request staged; no delete executed yet
- `confirmed`: original deletion request matched a separate `verified` confirmation Issue; request is closed in the operation ledger
- `rejected`: invalid command, missing target, wrong date, mismatch, etc.
- receipt absent: unverified or not processed; check Actions history

The `verified` result is a historical execution record; a later valid operation may intentionally change the same field.

## Acceptance

- [Transport smoke #2](https://github.com/changxinjiresearch/nextplan-command-inbox/issues/2) — verified.
- [Production sandbox Issues #3–#13](https://github.com/changxinjiresearch/nextplan-command-inbox/issues/3) — create, edit, task, date, two-step task/project deletion and deliberate rejection all matched expected receipts.
- After cleanup, the sandbox project and test deadline were absent from canonical state; nine pre-existing projects remained, with each successful QA event recorded once. Requests #9 and #11 have since been reconciled to `confirmed` by verification Issues #10 and #12.
- GitHub CI includes 100 consecutive commands, deduplication, malformed inputs, simulated failure-and-recovery and no-op poll checks.

## Limitations

ChatGPT Plus may not automatically invoke GitHub for every ordinary message. GitHub schedules can be late or disabled by quota/settings; there is no guaranteed seconds-level delivery. Do not say NextPlan has updated unless the Issue exists **and** its receipt was independently verified. The GitHub-based backend is event-sourced in `state.json`; Railway and the separate SQLite v3 service are not operating as the authority.
