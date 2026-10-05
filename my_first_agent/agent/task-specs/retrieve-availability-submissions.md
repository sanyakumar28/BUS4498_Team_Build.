# Retrieve Availability Submissions Task Specification

## Basic Information

- **Task ID:** T1
- **Task name:** Retrieve Availability Submissions
- **Task type:** Retrieve
- **Task owner:** OCOB Peer Advisor lead

## 1. Task Description

This task starts a schedule run. It collects every Peer Advisor availability submission for the term and the current advisor roster so that T2 can check completeness and interpret the data. Retrieval follows a fixed rule. The task reads all response rows in the availability spreadsheet whose term matches the schedule request, plus every active advisor on the roster. It returns them unchanged and does not interpret or edit them.

## 2. Inputs

### Input 1

- **Input name:** Schedule Request
- **Contents and format:** Structured record with the term name, the availability submission deadline, and the requesting lead's name and email.
- **Source:** OCOB Peer Advisor lead (workflow trigger)

### Input 2

- **Input name:** Availability Submissions Sheet
- **Contents and format:** Spreadsheet table with one row per submission: advisor name, advisor email, term, submission timestamp, a weekly grid in 30-minute increments marked available or unavailable, and a free-text conflict notes field.
- **Source:** Availability Submissions Sheet (Google Sheet filled by the Peer Advisor availability form)

### Input 3

- **Input name:** Advisor Roster
- **Contents and format:** Spreadsheet table with one row per active Peer Advisor: name, email, status (active or inactive), maximum weekly hours, and team-meeting group.
- **Source:** Roster tab of the ShiftMatch Scheduling Workbook (Google Sheet maintained by the lead)

- **If a required input is missing or invalid:** If the term is missing from the Schedule Request, the sheet or roster cannot be opened, or no submissions exist for the term, the task records status "Retrieval failed" with the reason and sends the case to T9: Resolve Scheduling Exception.

## 3. Outputs

### Output 1

- **Output name:** Raw Availability Data
- **Contents and format:** Structured table containing every submission row for the term, unchanged, plus a retrieval timestamp and a row count.
- **Next task or recipient:** T2: Validate Availability Data
- **Complete when:** The table contains every term row in the sheet at retrieval time, and the row count matches the sheet.

### Output 2

- **Output name:** Active Roster List
- **Contents and format:** Structured list of active advisors with name, email, maximum weekly hours, and team-meeting group.
- **Next task or recipient:** T2: Validate Availability Data
- **Complete when:** Every roster row with status "active" is included.

## 4. Planned Tools

### Tool 1

- **Tool name:** `retrieve_availability_submissions`
- **Input:** Schedule Request; Availability Submissions Sheet
- **Output:** Raw Availability Data
- **Implementation Route:** Web API calls (Google Sheets API, read-only)
- **Integration approach:** MCP integration (Google Drive connector)
- **Role in this task:** Reads all rows for the requested term and returns them as a table with the retrieval timestamp and row count. It does not change the sheet.
- **Task timeout:** 2 minutes
- **Maximum retries:** 2
- **Retry only when:** The API returns a rate-limit or temporary server error (HTTP 429 or 5xx). Wait 10 seconds before each retry. The tool only reads data, so retries cannot create duplicates.
- **On timeout, exhausted retries, or an error that cannot be retried:** Record status "Retrieval failed" with the error message and the time, and send the case to T9: Resolve Scheduling Exception. Do not pass partial data to T2.

### Tool 2

- **Tool name:** `retrieve_advisor_roster`
- **Input:** Advisor Roster
- **Output:** Active Roster List
- **Implementation Route:** Web API calls (Google Sheets API, read-only)
- **Integration approach:** MCP integration (Google Drive connector)
- **Role in this task:** Reads the Roster tab and returns the advisors whose status is active.
- **Task timeout:** 2 minutes, shared with Tool 1
- **Maximum retries:** 2
- **Retry only when:** The API returns HTTP 429 or 5xx. Wait 10 seconds before each retry. The tool is read-only, so retries cannot create duplicates.
- **On timeout, exhausted retries, or an error that cannot be retried:** Record status "Retrieval failed" and send the case to T9: Resolve Scheduling Exception.
