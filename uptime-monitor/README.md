# Monitor website uptime with confirmed outage and recovery emails

Checks a list of websites, APIs or health endpoints every 5 minutes. A single
failed check stays quiet; an outage is emailed only after it is confirmed. While
it lasts you get reminders, and one email when the target is back.

## What you get

| Situation | Result |
|---|---|
| Target up (HTTP 2xx/3xx and, if set, keyword found) | Nothing |
| First failed check | Nothing (blip protection) |
| `failuresBeforeAlert` failed checks in a row (default 2) | **DOWN** email |
| Still down after `reminderMinutes` (default 60) | **STILL DOWN** reminder |
| Back up after an alert | **RECOVERED** email |
| HTTP 429 | Reported as "rate limiting the monitor", no retry loop |
| Timeout, DNS or connection error | Counted as down ("No response: …"), the run itself stays green |
| Invalid URL in Settings | Counted as down ("Invalid URL in Settings") |

## Requirements

- n8n (self-hosted or Cloud). Targets must be reachable from your n8n instance.
- An SMTP account.

## Credentials

| Node | Credential type |
|---|---|
| Send alert email | SMTP |

The HTTP check uses no credential. Do not put tokens into target URLs.

## Installation

1. Import `workflow.json`.
2. Select your SMTP credential in **Send alert email**.
3. Open **Settings** and set `alertTo`, `alertFrom`, `failuresBeforeAlert`,
   `reminderMinutes` (0 = no reminders), `timeoutSeconds` and `targets`.
4. Activate the workflow. State between runs is only kept for active
   (production) runs, not for manual test runs.

## Example input (`targets` in Settings)

```json
[
  { "name": "Company website", "url": "https://www.example.com/", "expectedKeyword": "Example Domain" },
  { "name": "Public API health", "url": "https://api.example.com/health", "expectedKeyword": "ok" }
]
```

`expectedKeyword` is optional (empty string = only the HTTP status counts).

## Example output

Output of **Evaluate checks** on the run that confirms an outage:

```json
{
  "alertNeeded": true,
  "subject": "Uptime: 1 down - Public API health",
  "checked": 2,
  "up": 1,
  "results": [
    { "name": "Company website", "url": "https://www.example.com/", "up": true, "detail": "HTTP 200" },
    { "name": "Public API health", "url": "https://api.example.com/health", "up": false, "detail": "HTTP 503" }
  ]
}
```

Email text: `DOWN: Public API health (https://api.example.com/health) - HTTP 503`

## Limits

- Checks run from your n8n server only; a problem between your users and the
  target that does not affect your server is not seen.
- The subject of a reminder reads "1 down"; the body says STILL DOWN.
- An empty `targets` list ends the run without checks and without an email.
- Two entries with the same name and URL share one state.
- Interval below 1 minute is not recommended; respect the target's rate limits.

## Customising

Change the interval in **Every 5 minutes**, or replace the email node with a chat
node using `subject` and `text`.
