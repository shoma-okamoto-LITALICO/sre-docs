# Operational readiness checklist

<!--
Run this checklist when handing what you built over to routine operations, or when the operations owner changes.
For pre-launch checks, use checklists/production-readiness.md.
This one confirms that the system keeps running without the people who built it.
The handover process itself (who receives what, and how they become self-sufficient)
goes in templates/09-operations-handover.md; run this checklist as the final gate.
-->

| Item | Details |
| --- | --- |
| Target | |
| Handing-over team → Receiving team | @<github-id> → <team name> |
| Handover date | YYYY-MM-DD |
| Related Design Doc | |

## 1. Ownership

| # | Item | Status | Notes |
| --- | --- | --- | --- |
| 1.1 | Owning team decided | | |
| 1.2 | Scope of alerts to handle decided | | |
| 1.3 | Coverage hours (business hours / 24x7) decided | | |
| 1.4 | Events the receiving team cannot handle, and the escalation path for them, decided | | |

## 2. Documentation

| # | Item | Status | Location |
| --- | --- | --- | --- |
| 2.1 | Architecture diagram is current | | |
| 2.2 | Design Doc matches the actual implementation | | |
| 2.3 | Runbook exists | | |
| 2.4 | Procedures for routine tasks exist | | |
| 2.5 | Incident contact list exists (including external vendors) | | |

## 3. Runbook coverage

<!-- For each expected alert, check that a response is written. -->

| Alert / event | First response written | Tested in practice | Notes |
| --- | --- | --- | --- |
| | | | |

| # | Item | Status | Notes |
| --- | --- | --- | --- |
| 3.1 | Procedures to start, stop, and restart the service exist | | |
| 3.2 | Rollback procedure exists | | |
| 3.3 | Restore procedure exists | | |
| 3.4 | Degraded-mode procedure exists (if applicable) | | |
| 3.5 | Who to contact when unsure is written down | | |

## 4. Access and permissions

| # | Item | Status | Notes |
| --- | --- | --- | --- |
| 4.1 | Receiving team has the required permissions | | |
| 4.2 | No dependency on personal accounts of the handing-over team | | |
| 4.3 | Read access to monitoring and logs granted | | |
| 4.4 | How to look up secrets is shared (not the values themselves) | | |
| 4.5 | Admin rights for external services transferred to the team | | |

## 5. Routine tasks

| Task | Frequency | Time required | Automated | Procedure | Owner |
| --- | --- | --- | --- | --- | --- |
| | | | Done / Not done | | |

- Total effort for non-automated tasks (per month):
- Tasks with deadlines (certificate renewals, licenses, EOL handling):

| Tasks with deadlines | Deadline | Reminder mechanism |
| --- | --- | --- |
| | | |

<!-- A task with a deadline and no reminder mechanism will be forgotten. -->

## 6. Handover execution

| # | Item | Status | Date |
| --- | --- | --- | --- |
| 6.1 | Explained the architecture and design | | |
| 6.2 | Receiving team ran the procedures by following the Runbook | | |
| 6.3 | Ran an incident response drill together | | |
| 6.4 | Set a shadowing period (duration: ) | | |
| 6.5 | Decided the post-handover contact for questions and its end date | | |

## 7. Open items

| # | Item | Impact | Handling by receiving team | Due |
| --- | --- | --- | --- | --- |
| 1 | | | Fix / Accept | |

## References

- [The Site Reliability Workbook: Incident Response](https://sre.google/workbook/incident-response/)
