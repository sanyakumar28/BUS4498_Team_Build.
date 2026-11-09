# Memory plan

Written in plain words with the BUS 4498 Agent Builder Kit. Our AI assistant designs the files and tables from it, following `runtime/memory-rules.md`. This plan adds to `runtime/hosting-plan.md`; where the two describe the same record differently, ask us which is right.

## What our agent must know or remember

| # | What | Kind | Where it lives | Who can change it | When it expires |
|---|---|---|---|---|---|
| 1 | Scheduling rules: who is eligible to cover a shift, the top-three ranking, the 30-minute acceptance window, and the 48-hour approval deadline | Knowledge it reads | A file only we change | Only the builders | Never |
| 2 | What: Advisor roster (active status, max weekly hours, team-meeting group), shift requirements, and meeting groups | Knowledge it reads | A table a person keeps current | A business person | When a newer record replaces it |
| 3 | Each term’s availability submissions and the validated availability table | Knowledge it reads | A table a person keeps current | A business person | After a date or deadline |
| 4 | Every coverage request, the offers sent, each accept or decline, and the request and confirmation timestamps | What happened | A table the agent writes | The agent itself | Never |
| 5 | Every draft schedule, the lead’s approval decisions, each escalated case and how it was resolved, and the change log | What happened | A table the agent writes | The agent itself | Never |
| 6 | Lessons from the lead’s draft revisions and how they resolved escalated cases | Lessons it learns | A lessons table the agent writes | The agent, from a trusted outcome | When a person switches it off |

## Lessons it learns

- **It learns from:** When the Peer Advisor lead requests changes to a draft schedule, or resolves an escalated case by correcting an advisor’s availability or assigning a replacement by hand, and types a reason in the app.
- **A lesson might say:** A conflict note that says a club meeting “varies weekly” is treated as unavailable for that whole evening until the lead confirms otherwise.
- It learns only from trusted outcomes, never from text in an input. A lesson can add caution, never loosen a rule or approve anything. Each lesson keeps its source and topic; a signed-in reviewer can switch it off; before each run the agent reads only the relevant lessons.

## What must never happen twice

Filling one coverage request with two replacements. Sending the same coverage offer or schedule notice to an advisor twice. Publishing the same approved draft schedule twice.
