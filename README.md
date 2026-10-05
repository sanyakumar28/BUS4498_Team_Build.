# About the Agentic System

**BUS 4498 Team Build**

> **Problem to be solved**: OCOB Peer Advisors submit their weekly availability through a manual spreadsheet, and a team lead has to manually cross-reference 21+ individual availability grids (marked in 30-minute increments, cluttered with free-text class/work/club conflicts) to build a shift schedule by hand — while also making sure teammates who need to meet together have overlapping availability. Because there are no RTOs, any Peer Advisor who can't work a shift has to informally find their own coverage, typically through group messaging with no guaranteed response time. This manual process is slow and error-prone, and delays or gaps in coverage directly reduce the number of appointment slots available to OCOB students. Currently, schedule-building and coverage-finding rely entirely on manual coordination, with no formal turnaround-time standard — our rough baseline estimate is that finding shift coverage can take anywhere from several hours up to a full day, sometimes not resolving until the day of the shift.

## Team Charter

### Team Name

Goated

### Team Members

- Sanya Kumar
- Jacob Young

### System Name

ShiftMatch

### System Goal

ShiftMatch will automatically generate the OCOB Peer Advisor shift schedule each term by matching submitted availability against shift needs and team-meeting overlap requirements, and will automatically identify and contact qualified coverage when a Peer Advisor cannot work an assigned shift. The target is to reduce shift-coverage resolution time from an estimated same-day, multi-hour manual process to under one hour, within one academic term of deployment.

**Boundaries**

- **Scope:** ShiftMatch covers only OCOB Peer Advisor shift scheduling and shift coverage. It does not handle student appointment booking, payroll or timesheets, hiring, or scheduling for other campus programs.
- **Human approval:** ShiftMatch does not publish a term schedule until the Peer Advisor lead approves it. The lead remains accountable for the final schedule and for every case the system escalates.
- **Assignment limits:** ShiftMatch never assigns a Peer Advisor to a time they marked unavailable, beyond their maximum weekly hours, or to a shift they have not accepted. Coverage is only offered; an advisor must accept it.
- **Escalation:** When required information is missing, no qualified replacement exists, no one accepts within the response window, or a tool fails, ShiftMatch stops and hands the case to the lead instead of guessing.
- **Data:** ShiftMatch uses only the availability submissions, advisor roster, and schedule data needed for scheduling. It sends messages only to Peer Advisors and the lead, about their own shifts.

### Who Is Better Off When This Works?

OCOB Peer Advisor leads spend far less time manually building schedules, Peer Advisors reduce time chasing down shift coverage, and OCOB students benefit from more reliable, consistently staffed advising appointments.
