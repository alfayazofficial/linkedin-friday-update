# LinkedIn Friday Update

An automated weekly-reporting pipeline for engineering and product leaders. Every Friday it turns your Jira activity into two drafts:

1. a three-line status update for your manager, and
2. a publish-ready LinkedIn post, with a confidentiality check, aimed at senior engineers, founders and hiring managers.

Nothing is sent or posted automatically. Both outputs land as Gmail drafts, so a human stays in the loop.

## Why this exists

A consistent weekly public update does two jobs: it keeps your stakeholders informed, and it builds a visible track record of shipping. Most people skip it because writing it takes 20 to 30 minutes of recall and editing. This pipeline removes the recall step and leaves you the edit.

## Architecture

```
Every Friday 4 PM (cron)
        |
        +--> Jira: done this week -----+
        |                              |
        +--> Jira: open / in flight ---+--> Merge
                                              |
                                      Build Prompt (code)
                                       - normalise tickets
                                       - detect blocked / waiting
                                       - compute week metrics
                                              |
                                      LLM call (HTTP, retry x3)
                                       - returns strict JSON
                                              |
                                  Parse And Review Checks (code)
                                   - JSON parse with fallback
                                   - length check
                                   - confidentiality flags
                                              |
                          +-------------------+-------------------+
                          |                                       |
              Gmail draft to manager                 Gmail draft to you
              (3-line status)                        (LinkedIn post + review header)
```

## What's inside

| File | Purpose |
|---|---|
| `n8n_friday_status_report.json` | Importable n8n workflow, 9 nodes. |
| `linkedin_automate_kit.md` | Operating playbook: setup, hardening, post formats, comment and DM scripts, connection notes for different audiences, 30-day plan, metrics. |

## Design decisions

- **Human in the loop.** Output is a draft, never a send. Public posting from internal ticket data is a confidentiality risk, so review is a feature.
- **Confidentiality by construction.** The prompt forbids ticket keys, customer names and internal project names in the public post, and the model must return its own list of risky phrases. The parse step surfaces them at the top of the draft.
- **Structured output.** The model is asked for strict JSON. The parse step strips code fences and falls back to raw text if parsing fails, so a bad response produces a visible warning instead of a silent failure.
- **Credentials out of the file.** The API key lives in an n8n Header Auth credential, not in the workflow JSON, so the export is safe to share.
- **Resilience.** The LLM call retries three times with a 5 second gap and a 60 second timeout. Jira nodes use `alwaysOutputData`, so a quiet week still produces a draft.
- **Deterministic metrics.** Counts (shipped, in flight, blocked, high priority open) are computed in code and passed to the model. The model is told not to invent numbers.

## Quick start

1. In n8n, choose Workflows > Import from File and select `n8n_friday_status_report.json`.
2. Add credentials: Jira Software Cloud, Gmail OAuth2, and an HTTP Header Auth credential named `x-api-key` holding your Anthropic API key.
3. Replace `YOUR_MANAGER@example.com` and `YOU@example.com` in the two Gmail nodes.
4. Run once manually and read both drafts before activating.

Full instructions, troubleshooting and the posting playbook are in `linkedin_automate_kit.md`.

## Configuration

| Setting | Where | Default |
|---|---|---|
| Schedule | `Every Friday 4 PM` | `0 16 * * 5` (n8n instance timezone) |
| Jira scope | `Jira Done`, `Jira Open` JQL | assigned to you; done in last 7 days / not done |
| Blocked keywords | `Build Prompt` (`KEEP`) | `blocked`, `waiting` |
| Model | `AI Write Update` JSON body | `claude-sonnet-4-5` |
| Post length target | `Build Prompt` prompt text | 900 to 1300 characters |

## Limitations

- The workflow has not been run end to end against a live Jira and Gmail account. Test before relying on it.
- Comment-to-DM is manual by design. LinkedIn does not permit automated comment reading or auto-DMs, and automating them risks the account.
- Output quality depends on ticket quality. Vague ticket titles produce vague updates.

## Roadmap

- Pull merged pull requests from GitHub as a second source.
- Add a Slack digest node for team channels.
- Track post performance in a sheet and feed top-performing formats back into the prompt.
