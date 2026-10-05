# Confirm Replacement Acceptance Task Specification

## Basic Information

- **Task ID:** T7
- **Task name:** Confirm Replacement Acceptance
- **Task type:** Act
- **Task owner:** OCOB Peer Advisor lead

## 1. Task Description

This task gets a real person to agree to take the shift. Accepting a shift is the advisor's own choice, so ShiftMatch only offers and records. The system sends the offer to every advisor on the candidate list at the same time and then waits 30 minutes. The first recorded acceptance wins the shift, and the other candidates are told it has been filled. If no one accepts within 30 minutes, the case goes to the lead.

## 2. Inputs

### Input 1

- **Input name:** Replacement Candidate List
- **Contents and format:** Ranked list of up to three advisors (name, email), plus the request ID, the requester, and the shift details.
- **Source:** T6: Identify Eligible Replacements

### Input 2

- **Input name:** Candidate Responses
- **Contents and format:** One response per candidate who replies: request ID, candidate email, "Accept" or "Decline", and a timestamp.
- **Source:** Candidate Peer Advisors (accept/decline links in the offer email, recorded in the Coverage Offers tab)

- **If a required input is missing or invalid:** If the candidate list is empty or missing shift details, record status "No offer sent" and send the case to T9: Resolve Scheduling Exception. A response that does not match the request ID is ignored.

## 3. Outputs

### Output 1

- **Output name:** Confirmed Coverage Assignment
- **Contents and format:** Structured record with the request ID, the shift, the requester email, the replacement's email, the acceptance timestamp, and status "Accepted".
- **Next task or recipient:** T8: Publish Schedule Update
- **Complete when:** One "Accept" response is recorded within the 30-minute window, and the offer is closed to everyone else.

### Output 2

- **Output name:** No Acceptance Report
- **Contents and format:** The request ID, the shift, the candidates contacted, each candidate's response (declined or none), and the time the window closed.
- **Next task or recipient:** T9: Resolve Scheduling Exception
- **Complete when:** The 30-minute window has ended with no acceptance.

## 4. Planned Tools

### Tool 1

- **Tool name:** `send_coverage_offers`
- **Input:** Replacement Candidate List
- **Output:** None to the workflow (offer emails with accept/decline links tied to the request ID)
- **Implementation Route:** Web API calls (Gmail API)
- **Integration approach:** MCP integration (Gmail connector)
- **Role in this task:** Sends one offer to each candidate at the same time and starts the 30-minute window. When the shift is filled, it sends a "shift filled" note to the candidates who did not take it.
- **Task timeout:** Human response deadline of 30 minutes after the offers are sent; 2 minutes for sending
- **Maximum retries:** 1
- **Retry only when:** The send for a specific candidate returns no message ID. Wait 10 seconds before retrying. Before resending, check the Coverage Offers log for that request ID and candidate. If it is uncertain whether the offer went out, do not resend; continue with the candidates who were confirmed as sent.
- **On timeout, exhausted retries, or an error that cannot be retried:** If no offer could be sent, record status "No offer sent" and send the case to T9: Resolve Scheduling Exception.

### Tool 2

- **Tool name:** `record_offer_response`
- **Input:** Candidate Responses
- **Output:** Confirmed Coverage Assignment; No Acceptance Report
- **Implementation Route:** Web API calls (Google Sheets API, write to the Coverage Offers tab)
- **Integration approach:** MCP integration (Google Drive connector)
- **Role in this task:** Records each response. It accepts only the first "Accept" for a request ID, locks the offer so that later acceptances are refused, and produces the assignment or, when the window ends, the no-acceptance report.
- **Task timeout:** Human response deadline of 30 minutes after the offers are sent
- **Maximum retries:** 1
- **Retry only when:** A write returns HTTP 429 or 5xx. Wait 5 seconds before retrying. Read the offer's lock status first, so an acceptance is never recorded twice.
- **On timeout, exhausted retries, or an error that cannot be retried:** On window expiry, produce the No Acceptance Report for T9. If the lock state cannot be confirmed, record status "Acceptance uncertain" and send the case to T9 rather than assume coverage.
