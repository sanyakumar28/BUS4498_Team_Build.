# Hosting plan

Written in plain words with the BUS 4498 Agent Builder Kit. Our AI assistant turns it into a GitHub Pages website and a Supabase backend, following `runtime/hosting-rules.md`. Where this plan and our task specifications disagree, this plan wins.

## Our project

- GitHub repository: https://github.com/sanyakumar28/BUS4498_Team_Build.
- Our agent’s folder: `my_first_agent` (our plans are in `my_first_agent/runtime/`)
- Supabase project: https://gvkksdwgsquelrxckmfx.supabase.co
- Owner of the GitHub repository: Sanya Kumar. Jacob Young is a collaborator on it.
- Owner of the Supabase project: Jacob Young. Sanya Kumar is a Developer in the Supabase organization.
- We say “I’m working on it” and “pushed” in: iMessages. One of us works at a time.

## What our agent does

ShiftMatch will automatically generate the OCOB Peer Advisor shift schedule each term by matching submitted availability against shift needs and team-meeting overlap requirements, and will automatically identify and contact qualified coverage when a Peer Advisor cannot work an assigned shift. The target is to reduce shift-coverage resolution time from an estimated same-day, multi-hour manual process to under one hour, within one academic term of deployment.

## Our tasks

Use the tasks in our task specifications: `my_first_agent/agent/workflow-of-tasks.md` and `my_first_agent/agent/task-specs/`.

## Who can see or change what

Each Peer Advisor sees the published schedule, their own availability submission, and their own coverage requests and offers. They can change only their own availability (before the deadline) and accept or decline offers sent to them. Only the Peer Advisor lead sees every submission, every draft, and the exceptions queue, and only the lead approves a term schedule, enters a manual fix, or changes the roster, shift requirements, or the agent’s switches.

## External tools and applications

| # | Tool or application | What the agent uses it for | What it does with it | Most sensitive data it touches | How it connects |
|---|---|---|---|---|---|
| 1 | Google Sheets | Reads availability submissions, the advisor roster, shift requirements, and the published schedule | Reads from it | Confidential | A sign-in (OAuth) |
| 2 | Google Sheets | Writes validated availability, draft schedules, the published schedule, coverage offers, approvals, exceptions, and the change log | Writes to it | Confidential | A sign-in (OAuth) |
| 3 | Gmail | Sends coverage offers, returned-request notices, approval requests, exception notices, and schedule-change emails to Peer Advisors and the lead | Sends with it | Internal | A sign-in (OAuth) |

## Files people upload

None.
