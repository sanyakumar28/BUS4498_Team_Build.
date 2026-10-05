# Identify Eligible Replacements Task Specification

## Basic Information

- **Task ID:** T6
- **Task name:** Identify Eligible Replacements
- **Task type:** Reason
- **Task owner:** OCOB Peer Advisor lead

## 1. Task Description

This task finds who can cover the requested shift so that offers can be sent quickly. It applies fixed rules. An eligible advisor must be:

- Active.
- Not the requester.
- Marked available for every 30-minute block of the shift in the Validated Availability Table.
- Not already assigned to an overlapping shift.
- Still within their maximum weekly hours once the shift is added.

Eligible advisors are ranked by the fewest assigned hours that week, with ties broken alphabetically. The top three become the offer list.

## 2. Inputs

### Input 1

- **Input name:** Validated Coverage Request
- **Contents and format:** Structured record with the request ID, requester email, shift date, start time, and end time.
- **Source:** T5: Validate Coverage Request

### Input 2

- **Input name:** Validated Availability Table
- **Contents and format:** One row per active advisor, marked available or unavailable for each 30-minute block, plus maximum weekly hours.
- **Source:** Validated Availability tab of the ShiftMatch Scheduling Workbook (written by T2: Validate Availability Data)

### Input 3

- **Input name:** Published Schedule
- **Contents and format:** One row per shift with date, times, and assigned advisor emails.
- **Source:** Published Schedule tab of the ShiftMatch Scheduling Workbook (written by T8: Publish Schedule Update)

- **If a required input is missing or invalid:** If any input cannot be read, record status "Inputs unavailable" and send the case to T9: Resolve Scheduling Exception.

## 3. Outputs

### Output 1

- **Output name:** Replacement Candidate List
- **Contents and format:** Ranked list of up to three advisors with name, email, and hours that week, plus the request ID and shift details.
- **Next task or recipient:** T7: Confirm Replacement Acceptance
- **Complete when:** The list contains at least one advisor who meets every eligibility rule.

### Output 2

- **Output name:** No Eligible Replacement Report
- **Contents and format:** The request ID, the shift details, and the count of advisors excluded by each rule (unavailable, already scheduled, over hours).
- **Next task or recipient:** T9: Resolve Scheduling Exception
- **Complete when:** The report is produced because no advisor met every rule.

## 4. Planned Tools

### Tool 1

- **Tool name:** `read_published_schedule`
- **Input:** Published Schedule
- **Output:** Published Schedule (rows for that week)
- **Implementation Route:** Web API calls (Google Sheets API, read-only)
- **Integration approach:** MCP integration (Google Drive connector)
- **Role in this task:** Reads the week's assignments to find overlapping shifts and each advisor's current hours. This is the same tool used in T5.
- **Task timeout:** 1 minute for the whole task
- **Maximum retries:** 2
- **Retry only when:** The API returns HTTP 429 or 5xx. Wait 5 seconds before each retry. The tool is read-only.
- **On timeout, exhausted retries, or an error that cannot be retried:** Record status "Inputs unavailable" and send the case to T9: Resolve Scheduling Exception.

### Tool 2

- **Tool name:** `identify_eligible_replacements`
- **Input:** Validated Coverage Request; Validated Availability Table; Published Schedule
- **Output:** Replacement Candidate List; No Eligible Replacement Report
- **Implementation Route:** Functions/scripts (Python), plus a read-only Google Sheets API call for the Validated Availability tab
- **Integration approach:** Direct integration
- **Role in this task:** Applies the eligibility rules, ranks the eligible advisors, and returns the top three or the no-replacement report. It changes no records.
- **Task timeout:** 1 minute for the whole task, shared with Tool 1
- **Maximum retries:** 1
- **Retry only when:** The availability read returns HTTP 429 or 5xx. Wait 5 seconds before retrying. The tool is read-only.
- **On timeout, exhausted retries, or an error that cannot be retried:** Record status "Matching failed" and send the case to T9: Resolve Scheduling Exception.
