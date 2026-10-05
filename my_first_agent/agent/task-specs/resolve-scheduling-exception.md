# Resolve Scheduling Exception Task Specification

## Basic Information

- **Task ID:** T9
- **Task name:** Resolve Scheduling Exception
- **Task type:** Decide
- **Task owner:** OCOB Peer Advisor lead

## 1. Task Description

This is the human-review task for every case that ShiftMatch cannot finish on its own:

- Missing or unreadable availability
- No feasible schedule
- No eligible replacement
- No acceptance within the window
- An overdue approval
- A tool failure

The lead uses judgment and direct contact with advisors to resolve the case, then chooses one of three outcomes:

- **Corrected availability data:** The workflow resumes at T2: Validate Availability Data.
- **Manual assignment:** The workflow resumes at T8: Publish Schedule Update.
- **Cannot resolve in system:** The run stops and the lead handles the case outside ShiftMatch.

## 2. Inputs

### Input 1

- **Input name:** Exception Case
- **Contents and format:** Structured record with the case ID, the source task ID and name, the status code (for example, "No eligible replacement"), the request or draft ID, the time of escalation, and the run type (schedule or coverage).
- **Source:** T1–T8 (whichever task escalated)

### Input 2

- **Input name:** Exception Evidence
- **Contents and format:** The supporting report from the escalating task, such as the Validation Report, the unfilled shifts and failing groups from T3, the No Eligible Replacement Report, the No Acceptance Report, or an error message.
- **Source:** The escalating task (T1–T8)

- **If a required input is missing or invalid:** If the evidence is missing, the lead still receives the case. The lead looks at the source task's run log in the ShiftMatch Scheduling Workbook before deciding.

## 3. Outputs

### Output 1

- **Output name:** Exception Resolution
- **Contents and format:** Structured record with the case ID, the outcome ("Corrected availability data", "Manual assignment", or "Cannot resolve in system"), the lead's notes, the lead's name, and a timestamp.
- **Next task or recipient:** Exceptions tab of the ShiftMatch Scheduling Workbook (the run record)
- **Complete when:** An outcome is recorded for the case ID.

### Output 2

- **Output name:** Corrected Availability Data
- **Contents and format:** Advisor email, the corrected time blocks or conflict interpretation, and the lead's name. Produced only when that outcome is chosen.
- **Next task or recipient:** T2: Validate Availability Data
- **Complete when:** The corrections are saved and T2 is restarted for the term.

### Output 3

- **Output name:** Manual Assignment
- **Contents and format:** The case ID, the shift, the advisor or advisors to assign or remove, and the lead's name. Produced only when that outcome is chosen.
- **Next task or recipient:** T8: Publish Schedule Update
- **Complete when:** The assignment is saved and passed to T8.

## 4. Planned Tools

### Tool 1

- **Tool name:** `send_exception_notice`
- **Input:** Exception Case; Exception Evidence
- **Output:** None to the workflow (an email to the lead with the case summary, the evidence, and a link to the resolution form; the case is also logged in the Exceptions tab)
- **Implementation Route:** Web API calls (Gmail API and Google Sheets API)
- **Integration approach:** MCP integration (Gmail and Google Drive connectors)
- **Role in this task:** Hands the case to the lead and adds it to the open-exceptions list.
- **Task timeout:** Human response deadline of 30 minutes after the notice for coverage cases, and 48 hours for schedule cases
- **Maximum retries:** Not applicable — manual task.
- **Retry only when:** Not applicable
- **On timeout, exhausted retries, or an error that cannot be retried:** If the notice cannot be sent or the deadline passes, mark the case "Overdue" in the Exceptions tab and email the Peer Advisor program supervisor. The original request stays unresolved and is not marked complete.

### Tool 2

- **Tool name:** `record_exception_resolution`
- **Input:** The lead's resolution form response
- **Output:** Exception Resolution; Corrected Availability Data; Manual Assignment
- **Implementation Route:** Web API calls (Google Sheets API, write to the Exceptions tab)
- **Integration approach:** MCP integration (Google Drive connector)
- **Role in this task:** Saves the lead's outcome and routes the run: corrected data to T2, a manual assignment to T8, or the case closed as handled offline.
- **Task timeout:** Same human response deadline as Tool 1
- **Maximum retries:** Not applicable — manual task.
- **Retry only when:** Not applicable
- **On timeout, exhausted retries, or an error that cannot be retried:** If the resolution cannot be saved, mark the case "Resolution not saved" and email the lead to re-enter it. Do not route the run onward until the save is confirmed.
