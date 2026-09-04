# Postmortem: <incident>

<!--
Write this as a blameless postmortem.
The goal is to build a mechanism so the same incident does not recur, not to identify individual fault.
Do not put a person's name as the cause. Not "X made a mistake" but
"the procedure could not prevent the mistake".
For the standard, see Postmortem Culture in the SRE book; for a worked example, see Example Postmortem.
-->

| Item | Content |
| --- | --- |
| Status | Draft / In Review / Final |
| Author | @<github-id> |
| Reviewers | @<github-id> |
| Date of incident | YYYY-MM-DD |
| Severity | Sev1 / Sev2 / Sev3 |
| Duration of impact | <n> h <n> min |
| Detected by | Monitoring alert / User report / Developer noticed |
| Responders | |
| Related | Incident channel, Issue #, alerts |

## Summary

<!--
5 lines or fewer. What happened, who was affected and how, and how it was stopped.
Management and other teams read only this.
-->

## Impact

| Item | Content |
| --- | --- |
| Affected users | |
| Nature of impact | |
| Affected count / percentage | |
| Error budget consumed | |
| Data loss or inconsistency | Yes / No (details if yes) |
| Financial impact | |
| External communication | Done / Not done |

## Timeline

<!--
Use JST for all times. Delays in detection and slow triage
become improvement items as-is, so include the "noticed at" times too.
-->

| Time | Event | Who / what |
| --- | --- | --- |
| HH:MM | Change applied (trigger introduced) | |
| HH:MM | Incident begins | |
| HH:MM | Detection | |
| HH:MM | First response begins | |
| HH:MM | Cause identified | |
| HH:MM | Recovery action taken | |
| HH:MM | Recovery confirmed | |
| HH:MM | Incident resolved | |

### Key durations

| Metric | Duration | Target | Reason for gap |
| --- | --- | --- | --- |
| Start to detection | | | |
| Detection to first response | | | |
| First response to recovery | | | |

## Causes

### Trigger

### Root causes

<!--
Do not stop at the surface. Do not end with "the config was wrong";
go down to why it could be wrong and why it was not caught before applying.
When several factors overlap, describe how they overlap.
-->

-
-
-

### Contributing factors

<!-- Factors that let it spread (missing safeguards, missing monitoring, gaps in procedures). -->

-

## What went well

<!--
This section identifies mechanisms worth keeping. Do not skip it.
What is listed here will work in the next incident too.
-->

-

## What went badly

-

## Where we got lucky

<!-- Points where chance saved us. Find what would not save us if the same situation recurs. -->

-

## Action items

<!--
"Be more careful" and "notify everyone" are not action items.
Prevent by mechanism; if you cannot prevent, detect early and restore quickly.
Tie every item to an owner, a due date, and an Issue. Items without them do not get done.
-->

| # | Category | Action | Owner | Due | Issue | Status |
| --- | --- | --- | --- | --- | --- | --- |
| 1 | Prevent | | | | # | Not started |
| 2 | Detect | | | | # | |
| 3 | Mitigate / recover | | | | # | |
| 4 | Process / docs | | | | # | |

### Options not adopted

| Option | Reason not adopted |
| --- | --- |
| | |

## Lessons learned

<!-- Write any insight that applies to other teams too. Also note who to share it with. -->

## References

- [Postmortem Culture: Learning from Failure](https://sre.google/sre-book/postmortem-culture/)
- [Example Postmortem](https://sre.google/sre-book/example-postmortem/)
