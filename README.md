# linkedin-friday-update

An n8n workflow that drafts a weekly status update from Jira, plus a kit for turning a LinkedIn "Friday update" post into conversations.

## What's inside

- `n8n_friday_status_report.json`: importable n8n workflow. Every Friday at 4 PM it pulls your done and open Jira tickets, asks Claude for a 3-line update (Done / Next / Blocked), and saves it as a Gmail draft. Nothing sends automatically.
- `linkedin_automate_kit.md`: comment reply, saved DM, setup steps, and four connection request notes (post engagers, recruiters, fellow PMs, cold), each under 200 characters before filling placeholders.

## Setup

1. In n8n, choose Workflows > Import from File and select `n8n_friday_status_report.json`.
2. Add Jira Software Cloud and Gmail OAuth2 credentials.
3. Replace `YOURKEY` (Anthropic API key) and `YOUR_MANAGER@example.com`.
4. Run it once manually and check the Gmail draft before activating.

Full steps are in `linkedin_automate_kit.md`. Do not commit a real API key.

## Notes

- The Claude model is `claude-sonnet-4-5`. Change it in the Claude node's JSON body.
- Comment-to-DM is manual by design: LinkedIn does not allow automated comment reading or auto-DMs.
- The workflow has not been run end to end. Test before relying on it.

