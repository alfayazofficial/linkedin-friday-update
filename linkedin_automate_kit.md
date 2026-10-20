# LinkedIn Friday Update: Operating Playbook

Companion to `n8n_friday_status_report.json`. Covers setup, hardening, what to post, how to convert engagement into conversations, and how to measure it.

Contents: 1. Setup, 2. Troubleshooting, 3. Post anatomy and formats, 4. Example post, 5. Comment replies, 6. DM scripts, 7. Connection notes, 8. Weekly routine, 9. 30-day plan, 10. Metrics, 11. Rules of the road.

---

## 1. Setup

The workflow runs every Friday at 4 PM, pulls your done and open Jira tickets, and creates two Gmail drafts: a three-line manager update and a LinkedIn post with a review header.

1. **Import.** In n8n: Workflows > Import from File > `n8n_friday_status_report.json`.
2. **Jira.** Open `Jira Done` and `Jira Open`, select or create a Jira Software Cloud credential. Scope to a project by editing the JQL, for example add `AND project = ABC`.
3. **LLM credential.** Create an HTTP Header Auth credential: name `x-api-key`, value is your Anthropic API key. Select it in `AI Write Update`. The key is stored by n8n and never appears in the workflow JSON.
4. **Gmail.** Open both Gmail nodes and select a Gmail OAuth2 credential. Set `YOUR_MANAGER@example.com` on the manager draft and your own address on the LinkedIn draft.
5. **Model.** Set in the JSON body of `AI Write Update` (`claude-sonnet-4-5`). Change it there if you prefer another.
6. **Schedule.** Cron `0 16 * * 5` uses the n8n instance timezone. Set the timezone under Settings or in the workflow settings.
7. **Dry run.** Execute Workflow manually, then read both drafts before you activate.

### Customising the tone

All voice and structure rules live in the prompt string inside `Build Prompt`. Useful edits:

- Change the audience line (for example "CTOs and platform leads").
- Add a banned-words list.
- Add a sentence about your domain so the post reads as yours.

## 2. Troubleshooting

| Symptom | Likely cause | Fix |
|---|---|---|
| Draft says `(none)` everywhere | JQL matched nothing | Check the JQL in Jira's issue search; confirm `currentUser()` resolves to the right account |
| Manager draft is raw JSON | Model returned malformed JSON | Fallback fired; check the review header, rerun, or lower the temperature by adding `temperature: 0.3` to the request body |
| 401 from the LLM node | Wrong header name or key | Header name must be exactly `x-api-key` |
| Draft appears an hour early or late | Timezone | Set the n8n instance timezone |
| Blocked line is always "Nothing blocked." | Your team uses other words | Edit `KEEP` in `Build Prompt` |

## 3. Post anatomy and formats

The generated post follows a structure that senior readers skim well:

1. **Hook, one line.** A concrete outcome, not an announcement. "We cut deploy time from 40 minutes to 9" beats "Excited to share".
2. **What shipped.** Three bullets, each a result, not an activity.
3. **What I learned.** Two sentences of real insight. This is the part that earns trust with experienced readers.
4. **Next week.** One sentence. Shows direction.
5. **Closing question.** Specific enough that practitioners want to answer.

### Rotating formats (keeps the feed from becoming repetitive)

| Week | Format | Aim |
|---|---|---|
| 1 | Shipped and learned (default) | Establish the habit |
| 2 | Decision log: a trade-off you made and why | Show judgement |
| 3 | Mistake and fix: what broke, how you found it | Build credibility |
| 4 | Metric story: one number and what moved it | Attract operators and buyers |

To switch formats, edit the `linkedin_post` structure line in `Build Prompt`.

## 4. Example post (illustrative)

> We cut our release cycle from two weeks to three days this quarter.
>
> What shipped this week:
> - Automated rollback on failed health checks
> - Staging parity for the payments service
> - A runbook the whole team can follow at 2 AM
>
> What I learned: the speed gain did not come from faster pipelines. It came from removing the approval step nobody could explain. Every manual gate should have an owner who can say what it protects.
>
> Next week: load testing the new rollback path.
>
> Which gate in your release process would you delete if you could?

Replace specifics with your own. Never publish numbers you cannot defend in an interview.

## 5. Comment replies (same day)

Keep replies short and specific. Do not paste templates into the comment.

- Generic: `Thanks {Name}! Sending you the Friday update template in a DM now.`
- Peer with a real point: `Good point on {detail}. We ran into the same thing with {area}. How did you handle {specific follow-up}?`
- Senior or hiring-side commenter: `Thanks {Name}. Happy to go deeper on {topic} if useful, I'll message you.`

Reply within a few hours when possible; early replies tend to extend a post's reach.

## 6. DM scripts (manual, about 20 seconds each)

LinkedIn does not allow automated comment reading or auto-DMs and bots risk your account. Save each as a text snippet and fill in the name.

**Template share**

> Hi {Name}, thanks for the comment on my Friday update post. Here's the 3-line template I use:
>
> Done: one sentence on what shipped.
> Next: one sentence on next week's top priority.
> Blocked: one sentence on blockers, or "Nothing blocked."
>
> If you want, I can share the n8n workflow that drafts it from Jira every Friday at 4 PM. Just say the word.

**Workflow share (after they say yes)**

> Here you go: {link}. It drafts two things from Jira each Friday: a manager update and a post, and nothing sends without your review. Happy to walk through how it's built if that's useful.

**Conversation to call (for founders and engineering leaders)**

> {Name}, I enjoyed your point on {topic}. I build automation for teams that report across Jira and similar tools, and I'd like to compare notes on what is working for you. Open to a 20 minute call next week?

Send the third only to people who have engaged twice or whose work clearly overlaps with yours.

## 7. Connection request notes

Free accounts limit notes to 200 characters. Replace every `{...}` with one specific detail, then count characters.

**Post engagers (166 before filling)**

> Hi {Name}, thanks for engaging with my Friday update post, especially your point on {specific detail}. I'd like to stay connected and swap notes on how you run yours.

**Recruiters (150 before filling)**

> Hi {Name}, I'm a {role} with {X} years in {domain}. I saw you hire for {team/role type} and would like to connect in case something relevant opens up.

**Fellow PMs and engineers (152 before filling)**

> Hi {Name}, I liked your take on {specific post or product}. I'm a PM working on {area} and always keen to compare how other teams handle status updates.

**Founders and engineering leaders (about 170 before filling)**

> Hi {Name}, I followed your write-up on {specific topic}. I build workflow automation for engineering teams and would value connecting to exchange ideas on {area}.

**Big-tech engineers and managers (about 160 before filling)**

> Hi {Name}, your talk or post on {specific topic} stuck with me, especially {detail}. I work on {area} and would like to follow your work. Happy to connect.

**Cold (141 before filling)**

> Hi {Name}, I noticed {specific detail: their launch, talk or article}. I work on {area} and think we'd have useful overlap. Happy to connect.

## 8. Weekly routine (about 25 minutes)

| When | Task | Time |
|---|---|---|
| Fri 4:00 PM | Workflow creates drafts | 0 |
| Fri 4:05 PM | Read the manager draft, send | 3 min |
| Fri 4:10 PM | Read the LinkedIn draft, resolve every flag, add one personal detail, post | 10 min |
| Fri to Sun | Reply to every comment, send DMs | 10 min |
| Sat | Send 5 connection requests to engagers and target accounts | 5 min |

## 9. 30-day plan

- **Week 1.** Run the default format. Goal: start a streak of four consecutive weeks. Connect with everyone who engages.
- **Week 2.** Decision log. Start commenting thoughtfully on 3 posts per day from people you want to know.
- **Week 3.** Mistake and fix. Send the first conversation-to-call DMs to people who engaged twice.
- **Week 4.** Metric story. Review the numbers below and keep the format that earned the most conversations, not the most likes.

## 10. Metrics worth tracking

Track weekly in a sheet. Conversations matter more than impressions.

| Metric | Why it matters | Healthy signal |
|---|---|---|
| Comments from people you want to know | Quality of audience | Rising week over week |
| Profile views | Interest beyond the feed | Spike within 48 hours of posting |
| Inbound DMs and connection requests | Pipeline | At least 2 a week by week 4 |
| Calls booked | The actual goal | At least 1 a month |
| Post-to-reply rate | Whether your closing question works | Above 1 comment per 1,000 impressions |

## 11. Rules of the road

- Never publish customer names, internal ticket keys, revenue, unreleased features, or anything under NDA. Check the flags at the top of the draft every time.
- Do not claim outcomes you cannot back up.
- Read every draft in full. You are accountable for what posts under your name.
- Do not automate comments, DMs or connection requests. The account is the asset.
- Stay within your employer's social media policy.
- The workflow JSON contains no secrets by default. Keep it that way: never commit a real API key.
