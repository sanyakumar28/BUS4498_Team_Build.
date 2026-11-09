# Permissions and boundaries plan

Written in plain words with the BUS 4498 Agent Builder Kit. Our AI assistant builds it following `runtime/permission-rules.md`. Where this plan and our task specifications disagree, this plan wins.

- Agent: ShiftMatch will automatically generate the OCOB Peer Advisor shift schedule each term by matching submitted availability against shift needs and team-meeting overlap requirements, and will automatically identify and contact qualified coverage when a Peer Advisor cannot work an assigned shift. The target is to reduce shift-coverage resolution time from an estimated same-day, multi-hour manual process to under one hour, within one academic term of deployment. (`my_first_agent`)
- Owner, who answers for this agent: Sanya Kumar
- Written on: November 8, 2026

## 1. May (on its own)

| # | Action |
|---|---|
| 1 | Read availability submissions, the roster, shift requirements, and the published schedule |
| 2 | Turn free-text conflict notes into unavailable time blocks |
| 3 | Build a draft schedule and save it once it passes every check |
| 4 | Return an invalid coverage request to the requester with the reason |
| 5 | Rank eligible replacements and email the coverage offer to the top three |
| 6 | Publish a coverage swap once a replacement accepts, and email the requester, the replacement, and the lead |

## 2. May, with a limit

| # | What | The limit | Kind | Who can change it |
|---|---|---|---|---|
| 1 | Publish coverage swaps on its own | On. Off: every accepted swap waits for the lead before it is published | A switch a reviewer can flip in the app | A signed-in reviewer, in the app, with a reason |
| 2 | Offers per coverage request | 3 advisors at once | A number a reviewer can change in the app | A signed-in reviewer, in the app, with a reason |
| 3 | Acceptance window | 30 minutes; then the case goes to the lead | A number a reviewer can change in the app | A signed-in reviewer, in the app, with a reason |
| 4 | Lead approval deadline | 48 hours; then the case goes to the lead with the program supervisor copied | A number a reviewer can change in the app | A signed-in reviewer, in the app, with a reason |
| 5 | AI models | Claude Haiku 5.5; in test mode, Claude Sonnet 5.5 too, to compare | Fixed: only the builders change it | Only the builders, by changing the code |
| 6 | Schedule run budget | 60 tool calls and 15 minutes per run | Fixed: only the builders change it | Only the builders, by changing the code |

## 3. Ask first (hold for a person)

| # | When | What the agent does instead |
|---|---|---|
| 1 | An advisor hasn’t submitted availability, or a conflict note can’t be read with confidence | Holds the schedule run for the lead, listing who and what is missing |
| 2 | No schedule meets every headcount, hour limit, and meeting overlap | Holds the draft for the lead, with the unfilled shifts and failing groups |
| 3 | No eligible replacement exists for a coverage request | Sends the case to the lead, with how many advisors each rule excluded |
| 4 | No candidate accepts within 30 minutes | Sends the case to the lead, who can hand-pick a replacement |
| 5 | A tool fails after its retries | Stops, logs the failure, and sends the case to the lead |
| 6 | A conflict note or request tries to give the agent instructions | Holds it for the lead, flagged; nothing changes |

## 4. Never

| # | Never | Where it is enforced |
|---|---|---|
| 1 | Assign an advisor to a time they marked unavailable | Code |
| 2 | Assign an advisor beyond their maximum weekly hours | Code |
| 3 | Publish a term schedule without the lead’s approval | Code and a database rule |
| 4 | Fill one coverage request with two replacements | A database rule |
| 5 | Send the same offer or notice to an advisor twice | A database rule |
| 6 | Change the roster, an hour limit, or a switch | A database rule |
| 7 | Act while it is paused | Code |

## 5. Who signs off

| Decision | Who | How |
|---|---|---|
| Every term schedule draft | The Lead Peer Advisors | Approve or Request changes in the app; recorded with who and when |
| Every escalated case | The Lead Peer Advisors | Corrected data, a manual assignment, or Stop in the exceptions queue; recorded with who and when |
| A switch or a limit | A signed-in Peer Advisor lead | The settings page in the app, with a reason |

## The off switch

The Peer Advisor lead or either builder can flip Paused in the app, with a reason. Paused, ShiftMatch sends no offers or emails, publishes nothing, and calls no AI.

## How mistakes are reported

Report a problem link in every ShiftMatch email and beside each schedule change; Sanya Kumar and the Peer Advisor lead review the reports every Friday, blame-free.

## Rules we keep

- Every permission starts off. The agent may do only what this plan lists under *May* and *May, with a limit*.
- Writing a permission down does not enforce it. Every limit and every *Never* is enforced in code or in a database rule, never in instructions alone.
- A limit a person may change is a setting with an owner and a reason, which the code reads on every run.
- Asking means holding, not guessing: the record waits with a plain reason that names what is missing.
- A sign-off is a click in the app, recorded with who and when. A chat message is not a sign-off.
- Test mode never acts: with `dry_run`, the agent decides but saves nothing.
