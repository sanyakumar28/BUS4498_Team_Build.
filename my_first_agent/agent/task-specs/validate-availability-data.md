# Validate Availability Data Task Specification

## Basic Information

- **Task ID:** T2
- **Task name:** Validate Availability Data
- **Task type:** Sense
- **Task owner:** OCOB Peer Advisor lead

## 1. Task Description

This task makes sure the schedule is built from complete, usable availability. First, a rule-based check compares the submissions against the active roster and flags any advisor who has not submitted, any duplicate submissions (the latest one is kept), and any grid with unmarked time blocks. Next, a language-model operation reads each free-text conflict note, such as a class, a job, or a club meeting, and converts it into specific unavailable time blocks with a confidence label. Notes it cannot convert with high confidence are flagged rather than guessed. The output is one structured availability table that T3 and T6 can rely on.

## 2. Inputs

### Input 1

- **Input name:** Raw Availability Data
- **Contents and format:** Structured table of submission rows: advisor name, email, timestamp, 30-minute availability grid, and free-text conflict notes.
- **Source:** T1: Retrieve Availability Submissions

### Input 2

- **Input name:** Active Roster List
- **Contents and format:** Structured list of active advisors with name, email, maximum weekly hours, and team-meeting group.
- **Source:** T1: Retrieve Availability Submissions

### Input 3

- **Input name:** Corrected Availability Data
- **Contents and format:** Lead-entered corrections: advisor email, corrected time blocks or conflict interpretation, and the lead's name. This input is optional and appears only on runs that resume after an exception.
- **Source:** T9: Resolve Scheduling Exception

- **If a required input is missing or invalid:** If either required input is missing, record status "Validation not started" and send the case to T9: Resolve Scheduling Exception.

## 3. Outputs

### Output 1

- **Output name:** Validated Availability Table
- **Contents and format:** Structured table with one row per active advisor and one column per 30-minute block, each marked available or unavailable, plus maximum weekly hours and team-meeting group. It is saved to the Validated Availability tab of the ShiftMatch Scheduling Workbook.
- **Next task or recipient:** T3: Generate Draft Schedule; T6: Identify Eligible Replacements (reads the saved tab)
- **Complete when:** Every active advisor has exactly one row, no blocks are unmarked, and every conflict note is either converted with high confidence or resolved by a lead correction.

### Output 2

- **Output name:** Validation Report
- **Contents and format:** List of issues found: missing submitters, duplicates removed, unmarked blocks, and low-confidence conflict notes with the original text. Includes a pass or fail status.
- **Next task or recipient:** T9: Resolve Scheduling Exception if the status is fail; otherwise stored with the run record.
- **Complete when:** Every check has run and the status is recorded.

## 4. Planned Tools

### Tool 1

- **Tool name:** `check_submission_completeness`
- **Input:** Raw Availability Data; Active Roster List; Corrected Availability Data (if present)
- **Output:** Validation Report
- **Implementation Route:** Functions/scripts (Python)
- **Integration approach:** Direct integration
- **Role in this task:** Matches submissions to roster emails, keeps the latest duplicate, applies lead corrections, and flags missing submitters and unmarked blocks.
- **Task timeout:** 5 minutes for the whole task
- **Maximum retries:** 0
- **Retry only when:** Not applicable. The check is deterministic, so a retry would give the same result.
- **On timeout, exhausted retries, or an error that cannot be retried:** Record status "Validation failed" with the error, and send the Validation Report to T9: Resolve Scheduling Exception.

### Tool 2

- **Tool name:** `interpret_conflict_notes`
- **Input:** Raw Availability Data (free-text conflict notes field)
- **Output:** Validated Availability Table; Validation Report (low-confidence notes)
- **Implementation Route:** Web API calls (language-model API)
- **Integration approach:** Direct integration
- **Role in this task:** Converts each conflict note into day, start time, and end time blocks and labels each conversion high or low confidence. Only high-confidence blocks are applied to the table.
- **Task timeout:** 5 minutes for the whole task, shared with Tool 1
- **Maximum retries:** 1
- **Retry only when:** The API times out or returns a server error. Wait 15 seconds before retrying. The call only returns text and changes no records, so a retry cannot create duplicates.
- **On timeout, exhausted retries, or an error that cannot be retried:** Mark the affected notes "uninterpreted", set the Validation Report status to fail, and send it to T9: Resolve Scheduling Exception.

### Tool 3

- **Tool name:** `save_validated_availability`
- **Input:** Validated Availability Table
- **Output:** Validated Availability Table (saved)
- **Implementation Route:** Web API calls (Google Sheets API, write to the Validated Availability tab only)
- **Integration approach:** MCP integration (Google Drive connector)
- **Role in this task:** Replaces the contents of the Validated Availability tab with the validated table for this term. It runs only when the Validation Report status is pass.
- **Task timeout:** 5 minutes for the whole task, shared with Tools 1 and 2
- **Maximum retries:** 1
- **Retry only when:** The API returns HTTP 429 or 5xx. Wait 10 seconds before retrying. The write replaces the whole tab, so repeating it gives the same result and creates no duplicates.
- **On timeout, exhausted retries, or an error that cannot be retried:** Record status "Save failed" and send the case to T9: Resolve Scheduling Exception. Do not pass the table to T3.
