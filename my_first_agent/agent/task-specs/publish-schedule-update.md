# Publish Schedule Update Task Specification

## Basic Information

- **Task ID:** T8
- **Task name:** Publish Schedule Update
- **Task type:** Act
- **Task owner:** OCOB Peer Advisor lead

## 1. Task Description

This task makes approved changes official and tells the people they affect. It handles three kinds of input:

- **Approved term schedule:** The task copies the whole approved draft to the Published Schedule tab and emails every active advisor.
- **Confirmed coverage swap:** The task replaces the requester with the replacement on that one shift and emails the requester, the replacement, and the lead.
- **Lead's manual assignment:** The task applies it the same way as a coverage swap.

Each update is logged with its source ID, which is used to check that it was not already applied.

## 2. Inputs

### Input 1

- **Input name:** Approval Decision
- **Contents and format:** Draft ID, the decision "Approve", the lead's name, and a timestamp. The draft itself is read from the Draft Schedule tab.
- **Source:** T4: Approve Draft Schedule

### Input 2

- **Input name:** Confirmed Coverage Assignment
- **Contents and format:** Request ID, the shift, the requester email, the replacement email, and the acceptance timestamp.
- **Source:** T7: Confirm Replacement Acceptance

### Input 3

- **Input name:** Manual Assignment
- **Contents and format:** Case ID, the shift, the advisor or advisors to assign or remove, and the lead's name.
- **Source:** T9: Resolve Scheduling Exception

- **If a required input is missing or invalid:** Only one input arrives per run. If it is missing required fields, or if the draft ID does not match the saved draft, nothing is published. The task records status "Publish blocked" and sends the case to T9: Resolve Scheduling Exception.

## 3. Outputs

### Output 1

- **Output name:** Updated Published Schedule
- **Contents and format:** The Published Schedule tab showing the new assignments, plus a change-log row with the source ID, the shifts changed, and a timestamp.
- **Next task or recipient:** Published Schedule tab of the ShiftMatch Scheduling Workbook (read later by T5 and T6)
- **Complete when:** The tab shows the change and the change-log row exists for the source ID.

### Output 2

- **Output name:** Schedule Notifications
- **Contents and format:** Emails stating what changed. For a full schedule, every active advisor receives the schedule link. For a swap or manual change, the requester, the replacement, and the lead receive the shift details and the request or case ID.
- **Next task or recipient:** Affected Peer Advisors and the OCOB Peer Advisor lead (the run ends successfully)
- **Complete when:** Every intended recipient has a logged message ID.

## 4. Planned Tools

### Tool 1

- **Tool name:** `publish_schedule_update`
- **Input:** Approval Decision, Confirmed Coverage Assignment, or Manual Assignment
- **Output:** Updated Published Schedule
- **Implementation Route:** Web API calls (Google Sheets API, write to the Published Schedule and Change Log tabs)
- **Integration approach:** MCP integration (Google Drive connector)
- **Role in this task:** Applies the change to the Published Schedule tab and adds a change-log row. For a swap, it first confirms that the requester is still assigned to the shift.
- **Task timeout:** 3 minutes for the whole task
- **Maximum retries:** 1
- **Retry only when:** The write returns HTTP 429 or 5xx. Wait 10 seconds before retrying. Before retrying, check the Change Log for the source ID. If the change is already logged, do not write it again.
- **On timeout, exhausted retries, or an error that cannot be retried:** Record status "Publish failed" with the source ID and send the case to T9: Resolve Scheduling Exception. Do not send notifications for a change that is not confirmed as written.

### Tool 2

- **Tool name:** `send_schedule_notifications`
- **Input:** Updated Published Schedule
- **Output:** Schedule Notifications
- **Implementation Route:** Web API calls (Gmail API)
- **Integration approach:** MCP integration (Gmail connector)
- **Role in this task:** Sends the change emails after the publish succeeds and logs a message ID for each recipient.
- **Task timeout:** 3 minutes for the whole task, shared with Tool 1
- **Maximum retries:** 1
- **Retry only when:** A send returns no message ID. Wait 10 seconds before retrying, and retry only for recipients without a logged message ID. If it is uncertain whether an email was sent, do not resend.
- **On timeout, exhausted retries, or an error that cannot be retried:** Record status "Notification incomplete" with the recipients who were missed, and send the case to T9: Resolve Scheduling Exception so the lead can notify them directly. The schedule change itself stays published.
