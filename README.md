# NextPlan Command Inbox

**Status: LIVE — GitHub Actions transport verified on 13 October 2026.**

This repository is the deliberately **public** command inbox for NextPlan. ChatGPT Plus can submit an Issue using the user's existing GitHub connection. A GitHub Actions workflow in [LifeOS](https://github.com/changxinjiresearch/LifeOS) periodically processes authorized commands and updates the existing canonical NextPlan project data.

No Chrome message listener, self-hosted API or paid Railway service is required for this transport.

## How to send a command

Create an Issue with a title beginning with `[NextPlan]`. Its body must contain one JSON object. Example:

```json
{"version":3,"action":"update_project","project_id":"research-neuri","changes":{"status":"completed"}}
```

Supported actions:

- `update_project`: status (planned, active, waiting, blocked, completed), priority (1–5), or next_action (short text)
- `create_project`: project_id, name, optional category, next_action and priority
- `create_task`: project_id and name; optional task_id
- `complete_task`: project_id and task_id
- `update_milestone`: project_id, milestone_id and status

To avoid accidents, destructive deletes are **not** accepted through public Issues. Commands must use exact project IDs, not guessed names.

**Never put access tokens, passwords, private conversations or sensitive personal data into Issue titles/bodies. All Issues in this repository are public.**

## Timing and receipts

[LifeOS GitHub Actions](https://github.com/changxinjiresearch/LifeOS/actions/workflows/nextplan-issue-inbox.yml) checks for commands on a five-minute schedule, with delays possible. An Issue being created means **submitted**, not applied.

After processing Issue number `N`, the durable result is saved at:

`https://github.com/changxinjiresearch/LifeOS/blob/main/events/receipts/issue-N.json`

The `status` must say `verified` before reporting success. `rejected` contains an error reason. No receipt means it has not been confirmed yet.

[Issue #2](https://github.com/changxinjiresearch/nextplan-command-inbox/issues/2) was the first successful end-to-end delivery test; its [verified receipt](https://github.com/changxinjiresearch/LifeOS/blob/main/events/receipts/issue-2.json) was checked against the actual canonical state.

## Reliability

GitHub Issue numbers map to deterministic event IDs. Previously processed Issues cannot apply a second event. The workflow uses GitHub's built-in repository write token, rebuilds canonical state with the existing engine, verifies each postcondition, and commits changes with receipts in one Git commit. Git conflicts cause a clean re-run against the latest main branch.

Limitations: ChatGPT Plus does not guarantee tool invocation for every normal message; GitHub schedule may run later than planned; this does not promise zero downtime. Always check a real Issue link and the final receipt.
