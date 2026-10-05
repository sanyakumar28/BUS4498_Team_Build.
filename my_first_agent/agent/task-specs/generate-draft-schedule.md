# Generate Draft Schedule Task Specification

```yaml
# BASIC INFORMATION
task_id: "T3"
task_name: "Generate Draft Schedule"
task_owner: "OCOB Peer Advisor lead"
```

## 1. Task Goal

- **Objective:** Produce a draft term shift schedule that meets four conditions. Every shift is staffed with its required number of Peer Advisors, each advisor is assigned only during their validated available time, no advisor exceeds their maximum weekly hours, and every team-meeting group keeps the required shared free time. The draft goes to the lead for approval.

## 2. Inbound Inputs

### Input 1

- **Input name:** Validated Availability Table
- **What it contains:** One row per active advisor and one column per 30-minute block, marked available or unavailable, plus maximum weekly hours and team-meeting group.
- **Source:** T2: Validate Availability Data (Validated Availability tab of the ShiftMatch Scheduling Workbook)

### Input 2

- **Input name:** Shift Requirements
- **What it contains:** One row per shift: day, start time, end time, and required headcount. Also includes one row per team-meeting group with its member emails and the minimum weekly overlap required, in hours.
- **Source:** Shift Requirements and Meeting Groups tabs of the ShiftMatch Scheduling Workbook (maintained by the lead)

### Input 3

- **Input name:** Lead Revision Notes
- **What it contains:** Optional. The lead's written requested changes to a previous draft, such as "move Alex off Friday mornings", plus the ID of the earlier draft.
- **Source:** T4: Approve Draft Schedule (only on revision runs)

## 3. Tool Permissions and Boundaries

### Task-Wide Limits

- **Total task timeout:** 15 minutes per run, including all tool calls, retries, and waiting
- **Maximum tool calls:** 60 per run, including retries

### Tool 1

- **Tool name:** `read_shift_requirements`
- **Tool type:** API request (Google Sheets API, read-only)
- **Supports these permitted subtasks:** Assign Shift Candidates; Verify Draft Schedule
- **Allowed use:** Read the Shift Requirements and Meeting Groups tabs of the ShiftMatch Scheduling Workbook.
- **Prohibited use:** Editing any tab, or reading any file other than the Scheduling Workbook.
- **Approval required:** None within the allowed use
- **Timeout per call:** 30 seconds
- **Maximum retries per call:** 2
- **Retry conditions and failure response:** Retry on HTTP 429 or 5xx after a 10-second wait. The tool is read-only, so retries cannot create duplicates. If the call still fails, stop and hand off to T9: Resolve Scheduling Exception with status "Requirements unavailable".

### Tool 2

- **Tool name:** `assign_shift_candidates`
- **Tool type:** Python script (constraint-matching function)
- **Supports these permitted subtasks:** Assign Shift Candidates; Fill Understaffed Shifts; Rebalance Advisor Hours; Apply Lead Revisions
- **Allowed use:** Take the availability table, the shift requirements, and any fixed or excluded assignments the agent supplies. Return a candidate assignment set and a list of shifts still short of headcount. It works in memory only.
- **Prohibited use:** Assigning an advisor to a block marked unavailable, assigning an inactive advisor, or writing to any data source.
- **Approval required:** None within the allowed use
- **Timeout per call:** 60 seconds
- **Maximum retries per call:** 1
- **Retry conditions and failure response:** Retry once only on a runtime error, not when the result is simply infeasible. The tool changes no records, so a retry cannot create duplicates. If it fails again, hand off to T9 with status "Scheduling tool failed".

### Tool 3

- **Tool name:** `check_meeting_overlap`
- **Tool type:** Python script
- **Supports these permitted subtasks:** Repair Meeting Overlap; Verify Draft Schedule
- **Allowed use:** Compare a candidate assignment set with the Meeting Groups requirements and return each group's shared free hours and any groups below the minimum.
- **Prohibited use:** Changing assignments, or writing to any data source.
- **Approval required:** None within the allowed use
- **Timeout per call:** 30 seconds
- **Maximum retries per call:** 1
- **Retry conditions and failure response:** Retry once on a runtime error. If it fails again, hand off to T9 with status "Overlap check failed".

### Tool 4

- **Tool name:** `check_hour_limits`
- **Tool type:** Python script
- **Supports these permitted subtasks:** Rebalance Advisor Hours; Verify Draft Schedule
- **Allowed use:** Total each advisor's assigned hours in a candidate assignment set and return anyone over their maximum weekly hours.
- **Prohibited use:** Changing assignments or hour limits, or writing to any data source.
- **Approval required:** None within the allowed use
- **Timeout per call:** 30 seconds
- **Maximum retries per call:** 1
- **Retry conditions and failure response:** Retry once on a runtime error. If it fails again, hand off to T9 with status "Hour check failed".

### Tool 5

- **Tool name:** `save_draft_schedule`
- **Tool type:** API request (Google Sheets API, write to the Draft Schedule tab only)
- **Supports these permitted subtasks:** Verify Draft Schedule
- **Allowed use:** Write the verified draft, its draft ID, and the run timestamp to the Draft Schedule tab of the ShiftMatch Scheduling Workbook. The write replaces the earlier draft for the same term.
- **Prohibited use:** Writing to the Published Schedule tab, sending messages to advisors, or saving a draft that has not passed Verify Draft Schedule.
- **Approval required:** None within the allowed use. Publishing requires lead approval in T4.
- **Timeout per call:** 30 seconds
- **Maximum retries per call:** 1
- **Retry conditions and failure response:** Before retrying, read the tab to check whether the draft ID is already saved. Retry only if it is not, to avoid duplicates. If the outcome is still uncertain or the retry fails, hand off to T9 with status "Draft save failed".

## 4. How the Agent Should Reason

### Permitted Subtask 1

- **Subtask name:** Assign Shift Candidates
- **Subtask description:** Builds a first full assignment set from the availability table and the shift requirements. It produces candidate assignments and a list of understaffed shifts.
- **Subtask boundary:** Requires both the Validated Availability Table and the Shift Requirements. May use only advisors and blocks marked available.
- **Retry limits:** 1

### Permitted Subtask 2

- **Subtask name:** Fill Understaffed Shifts
- **Subtask description:** For each shift below headcount, looks for advisors who are available for the whole shift and have spare hours, and may move an advisor from a fully staffed shift if that frees them. It produces an updated assignment set and any shifts that are still short.
- **Subtask boundary:** May not assign anyone outside their available blocks or above their hour limit. May not lower a shift's required headcount.
- **Retry limits:** 3

### Permitted Subtask 3

- **Subtask name:** Repair Meeting Overlap
- **Subtask description:** When `check_meeting_overlap` reports that a group is below its minimum, reassigns that group's members' shifts to create shared free time. It produces an updated assignment set and the overlap for each group.
- **Subtask boundary:** Changes may not leave any shift understaffed or put any advisor above their hour limit.
- **Retry limits:** 3

### Permitted Subtask 4

- **Subtask name:** Rebalance Advisor Hours
- **Subtask description:** When `check_hour_limits` reports an advisor over their maximum, moves that advisor's extra shifts to available advisors with spare hours. It produces an updated assignment set.
- **Subtask boundary:** May not break shift headcount or meeting overlap requirements.
- **Retry limits:** 2

### Permitted Subtask 5

- **Subtask name:** Apply Lead Revisions
- **Subtask description:** On revision runs, turns the Lead Revision Notes into fixed or excluded assignments and re-runs assignment around them. It produces an updated assignment set and a note on any request that could not be met.
- **Subtask boundary:** Only for runs that include Lead Revision Notes. May not override availability or hour limits to satisfy a note. Unmet requests are reported, not forced.
- **Retry limits:** 1

### Permitted Subtask 6

- **Subtask name:** Verify Draft Schedule
- **Subtask description:** Runs the headcount, availability, hour-limit, and meeting-overlap checks on the final assignment set. If every check passes, saves the draft with `save_draft_schedule`.
- **Subtask boundary:** The draft may be saved only if every check passes. This subtask may not publish the draft or notify advisors.
- **Retry limits:** 1

- **Decision guidance:** After each subtask, use its findings to select the permitted subtask most likely to resolve the most important remaining uncertainty. For example, if shifts remain short, choose Fill Understaffed Shifts. If a meeting group falls below its minimum, choose Repair Meeting Overlap. If an advisor is over hours, choose Rebalance Advisor Hours. Do not follow a fixed sequence. If no permitted subtask can make useful progress, stop and hand the case to a person.

## 5. When to Stop or Hand Off to a Human

- **Stop successfully when:** Verify Draft Schedule confirms all of the following, and the draft is saved in the Draft Schedule tab with a draft ID:
  - Every shift meets its required headcount.
  - Every assignment falls within the advisor's available blocks.
  - No advisor exceeds their maximum weekly hours.
  - Every meeting group meets its minimum overlap.
- **Hand off early when:** Any of the following happens:
  - A shift still cannot be staffed after Fill Understaffed Shifts reaches its retry limit.
  - A meeting group still cannot reach its minimum overlap after Repair Meeting Overlap reaches its retry limit.
  - Two consecutive subtasks produce no improvement.
  - A required input is missing.
  - A tool fails after its retries.
  - The 15-minute timeout or the 60-call limit is reached.
- **Hand off to:** T9: Resolve Scheduling Exception (OCOB Peer Advisor lead)

Stop at the first applicable budget limit or handoff condition. While awaiting review, take no further autonomous action.

## 6. Outbound Deliverable

- **Status:** Completed or escalated to human.
- **Result or recommendation:** The draft schedule (draft ID and a link to the Draft Schedule tab) listing every shift and its assigned advisors. If escalated before reaching a supported result, write undetermined.
- **Evidence summary:** The check results: headcount met for each shift, each advisor's assigned hours compared with their limit, and each meeting group's overlap hours. For escalated cases, the unfilled shifts or failing groups and the reason.
- **Subtasks performed:** Permitted subtasks completed, including repeated attempts.
- **Unresolved issues:** Remaining uncertainties, such as lead revision requests that could not be met. Use none only if no unresolved issue remains.
- **Handoff note:** The reason for stopping, any unresolved questions, and what the lead needs to decide, such as which shift to leave short or whom to ask for more availability. Write "Not applicable" for a completed task.
- **Next task or recipient:** T4: Approve Draft Schedule. Unresolved cases go to T9: Resolve Scheduling Exception.
