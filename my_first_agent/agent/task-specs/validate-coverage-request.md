# Validate Coverage Request Task Specification

## Basic Information

- **Task ID:** T5
- **Task name:** Validate Coverage Request
- **Task type:** Verify
- **Task owner:** OCOB Peer Advisor lead

## 1. Task Description

This task starts a coverage run. It confirms that a coverage request is real before ShiftMatch contacts anyone. It applies fixed rules against the published schedule:

- The shift exists.
- The requester is assigned to that shift.
- The shift has not started.
- No other open request exists for the same requester and shift.

A request that passes moves to T6. A request that fails is returned to the requester with the reason.

## 2. Inputs

### Input 1

- **Input name:** Coverage Request
- **Contents and format:** Form response with a request ID, the requester's name and email, the shift date and start time, an optional reason, and a submission timestamp.
- **Source:** Peer Advisor (ShiftMatch coverage request form)

### Input 2

- **Input name:** Published Schedule
- **Contents and format:** Spreadsheet table with one row per shift: date, start time, end time, and assigned advisor emails.
- **Source:** Published Schedule tab of the ShiftMatch Scheduling Workbook (written by T8: Publish Schedule Update)

- **If a required input is missing or invalid:** If the form is missing the shift date or time, the request is returned to the requester as invalid. If the Published Schedule cannot be read, the case goes to T9: Resolve Scheduling Exception.

## 3. Outputs

### Output 1

- **Output name:** Validated Coverage Request
- **Contents and format:** Structured record with the request ID, requester email, shift date, start time, end time, status "Valid", and a validation timestamp.
- **Next task or recipient:** T6: Identify Eligible Replacements
- **Complete when:** All four rule checks pass and the record is logged in the Coverage Requests tab.

### Output 2

- **Output name:** Request Return Notice
- **Contents and format:** Email to the requester with the request ID, status "Returned", and the failed rule in plain language.
- **Next task or recipient:** Requesting Peer Advisor (the run then ends)
- **Complete when:** The email is sent and logged with the request ID.

## 4. Planned Tools

### Tool 1

- **Tool name:** `read_published_schedule`
- **Input:** Published Schedule
- **Output:** Published Schedule (rows used in the checks)
- **Implementation Route:** Web API calls (Google Sheets API, read-only)
- **Integration approach:** MCP integration (Google Drive connector)
- **Role in this task:** Reads the shift rows for the requested date.
- **Task timeout:** 1 minute for the whole task
- **Maximum retries:** 2
- **Retry only when:** The API returns HTTP 429 or 5xx. Wait 5 seconds before each retry. The tool is read-only.
- **On timeout, exhausted retries, or an error that cannot be retried:** Record status "Schedule unavailable" and send the case to T9: Resolve Scheduling Exception.

### Tool 2

- **Tool name:** `validate_coverage_request`
- **Input:** Coverage Request; Published Schedule
- **Output:** Validated Coverage Request
- **Implementation Route:** Functions/scripts (Python)
- **Integration approach:** Direct integration
- **Role in this task:** Applies the four rule checks and returns "Valid" or "Returned" with the failed rule.
- **Task timeout:** 1 minute for the whole task, shared with the other tools
- **Maximum retries:** 0
- **Retry only when:** Not applicable. The check is deterministic.
- **On timeout, exhausted retries, or an error that cannot be retried:** Record status "Validation failed" and send the case to T9: Resolve Scheduling Exception.

### Tool 3

- **Tool name:** `send_request_return_notice`
- **Input:** Coverage Request (when returned)
- **Output:** Request Return Notice
- **Implementation Route:** Web API calls (Gmail API)
- **Integration approach:** MCP integration (Gmail connector)
- **Role in this task:** Emails the requester the reason the request was returned.
- **Task timeout:** 1 minute for the whole task, shared with the other tools
- **Maximum retries:** 1
- **Retry only when:** The send call fails with no message ID. Wait 10 seconds before retrying. Every email carries the request ID, and the log is checked before resending to avoid duplicates. If it is uncertain whether the email was sent, do not resend.
- **On timeout, exhausted retries, or an error that cannot be retried:** Record status "Notice not sent" and send the case to T9: Resolve Scheduling Exception so the lead can tell the requester directly.
