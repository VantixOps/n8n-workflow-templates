# Send deduplicated n8n error alerts by email with cooldown and fallback

An n8n *error workflow*: when any workflow that points to it fails, you get one
clear email. Repeats of the same error are held back for a cooldown window, and
the next email says how many were held back.

## What you get

| Situation | Result |
|---|---|
| First occurrence of an error | Email with workflow, node, error, execution ID, link to the execution |
| Same error again within the cooldown | No email, run ends in **Skip (cooldown active)** |
| Different error | Email immediately |
| Alert text cannot be built (e.g. code execution unavailable) | Plain-text **fallback alert**, built from expressions only |
| The alert email itself fails | Cooldown is released, run is marked as failed (**Report send failure**) so the next error is tried again |

"Same error" means: same workflow ID, node, error type and message pattern
(digits and whitespace are normalised, so `timeout after 3012 ms` and
`timeout after 2950 ms` count as one problem).

## Requirements

- n8n with the Code node available (self-hosted or Cloud).
- An SMTP account for sending email. No other services.

## Credentials

| Node | Credential type |
|---|---|
| Send alert email | SMTP |
| Send fallback alert | SMTP (can be the same) |

The template contains no credentials. n8n asks you to select them after import.

## Installation

1. Import `workflow.json` (*Workflows > Import from file*).
2. Open both email nodes and select your SMTP credential.
3. Open **Settings** and set:
   - `alertTo` – recipient, e.g. `ops@your-company.example`
   - `alertFrom` – sender, e.g. `n8n alerts <alerts@your-company.example>`
   - `cooldownMinutes` – default `60`
4. Activate the workflow. Deduplication state is only stored for real error
   runs, not for manual test runs in the editor.
5. In every workflow you want to watch: *Workflow settings > Error workflow* >
   select this workflow.

Do not set an error workflow on this workflow itself (that would loop).

## Example input (what the Error Trigger delivers)

Synthetic example, shortened:

```json
{
  "execution": {
    "id": "231",
    "url": "https://your-n8n.example.com/workflow/AbC123/executions/231",
    "error": { "name": "Error", "message": "Synthetic failure A run 1792496943943", "node": { "name": "Fail on purpose" } },
    "lastNodeExecuted": "Fail on purpose",
    "mode": "webhook"
  },
  "workflow": { "id": "AbC123", "name": "Probe that fails" }
}
```

## Example output

Output of **Build alert** (sent because it is the first occurrence):

```json
{
  "sendNow": true,
  "subject": "[n8n] Probe that fails failed at Fail on purpose",
  "fingerprint": "AbC123 | Fail on purpose | Error | Synthetic failure A run #",
  "heldBack": 0
}
```

Email text part:

```
Workflow: Probe that fails (AbC123)
Type: Node error
Node: Fail on purpose
Error: Error: Synthetic failure A run 1792496943943
Execution: 231
Time (UTC): 2026-10-20T11:49:03.943Z
Execution URL: https://your-n8n.example.com/workflow/AbC123/executions/231
```

## Limits

- The error message is included in the email (escaped). If your workflows put
  sensitive data into error messages, it will appear in the alert.
- Deduplication state lives in the workflow's static data. Two errors arriving at
  exactly the same moment can both be sent (no locking in n8n static data).
- If SMTP is down, no email can arrive; the failed run stays visible in the
  execution list of this workflow.
- Trigger/activation errors are formatted, but were not reproduced in the
  author's tests.

## Customising

Replace the two email nodes with Slack, Microsoft Teams or Telegram nodes and map
`subject` and `text`. Keep the error output of the main alert node connected to
**Allow next alert**.
