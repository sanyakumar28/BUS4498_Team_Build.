# Approve Draft Schedule Task Specification

## Basic Information

- **Task ID:** T4
- **Task name:** Approve Draft Schedule
- **Task type:** Decide
- **Task owner:** OCOB Peer Advisor lead

## 1. Task Description

This is a human-review task. The lead checks the draft schedule before anyone sees it, because the lead is accountable for fairness and for details the system cannot know, such as an advisor's informal requests. The lead uses their own judgment to choose either "Approve" or "Request changes". If they request changes, they write notes explaining what to change. Nothing is published without an approval.

## 2. Inputs

### Input 1

- **Input name:** Draft Schedule
- **Contents and format:** Draft ID and a link to the Draft Schedule tab, which lists every shift (day, time) with its assigned advisors.
- **Source:** T3: Generate Draft Schedule

### Input 2

- **Input name:** Draft Evidence Summary
- **Contents and format:** Check results from T3: headcount for each shift, each advisor's hours compared with their limit, meeting-group overlap, and any revision requests that could not be met.
- **Source:** T3: Generate Draft Schedule

- **If a required input is missing or invalid:** If the draft link does not open or the evidence summary is missing, the lead does not approve. The case is sent to T9: Resolve Scheduling Exception with status "Draft incomplete".

## 3. Outputs

### Output 1

- **Output name:** Approval Decision
- **Contents and format:** Structured record with the draft ID, the decision ("Approve" or "Request changes"), the lead's name, and a timestamp.
- **Next task or recipient:** T8: Publish Schedule Update if approved; T3: Generate Draft Schedule if changes are requested
- **Complete when:** The decision is recorded in the Approvals tab of the ShiftMatch Scheduling Workbook.

### Output 2

- **Output name:** Lead Revision Notes
- **Contents and format:** Written description of the requested changes, linked to the draft ID. Required when the decision is "Request changes".
- **Next task or recipient:** T3: Generate Draft Schedule
- **Complete when:** The notes are recorded with the decision and are not blank.

## 4. Planned Tools

### Tool 1

- **Tool name:** `send_approval_request`
- **Input:** Draft Schedule; Draft Evidence Summary
- **Output:** None to the workflow (an email to the lead with the draft link, the evidence summary, and a link to the decision form)
- **Implementation Route:** Web API calls (Gmail API)
- **Integration approach:** MCP integration (Gmail connector)
- **Role in this task:** Hands the draft to the lead for review and sends one reminder 24 hours later if no decision has been recorded.
- **Task timeout:** Human response deadline of 48 hours after the request email is sent
- **Maximum retries:** Not applicable — manual task.
- **Retry only when:** Not applicable
- **On timeout, exhausted retries, or an error that cannot be retried:** If the email cannot be sent, or no decision arrives within 48 hours, record status "Approval overdue" and send the case to T9: Resolve Scheduling Exception, copying the Peer Advisor program supervisor. The draft is never treated as approved.

### Tool 2

- **Tool name:** `record_approval_decision`
- **Input:** The lead's form response (decision and notes)
- **Output:** Approval Decision; Lead Revision Notes
- **Implementation Route:** Web API calls (Google Sheets API, write to the Approvals tab)
- **Integration approach:** MCP integration (Google Drive connector)
- **Role in this task:** Saves the lead's decision and notes, rejects a "Request changes" response with blank notes, and routes the result to T8 or T3.
- **Task timeout:** Human response deadline of 48 hours, shared with Tool 1
- **Maximum retries:** Not applicable — manual task.
- **Retry only when:** Not applicable
- **On timeout, exhausted retries, or an error that cannot be retried:** Record status "Approval overdue" or "Decision not saved", and send the case to T9: Resolve Scheduling Exception.
