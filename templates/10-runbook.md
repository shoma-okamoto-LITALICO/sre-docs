# Runbook: <service> / <alert or situation>

<!--
A runbook is read by someone who was woken up, may not know this system, and has to act
in the next ten minutes. Optimize for that reader: short steps, exact commands, expected
output, and a clear line for when to stop and escalate.

One runbook per alert or per recurring operational situation. If a runbook needs more than
one page of triage before any action, the alert is probably too vague; fix the alert.
Design rationale does not belong here. Link the Design Doc instead.
-->

| Item | Details |
| --- | --- |
| Service | |
| Owning team / on-call rotation | |
| Alerts that route here | <alert name>, <alert name> |
| Severity of those alerts | Page / Ticket |
| Last reviewed | YYYY-MM-DD by @<github-id> |
| Last tested (drill or real use) | YYYY-MM-DD |
| Design Doc | |
| Dashboard | |
| Logs | |
| Escalation | see section 6 |

## 1. What this alert means

<!--
Two or three sentences. What is broken from the user's point of view, and what the alert
condition actually measures. Say what it does not mean, if that is a common confusion.
-->

- User-visible impact:
- What the alert measures:
- Typical causes, most frequent first:
  1.
  2.
  3.

## 2. Before you start

| Need | Details |
| --- | --- |
| Access required | |
| Where to run commands | |
| Tools that must be installed | |
| Safe to run during business hours? | Yes / No, because |

<!-- If any command here changes state, say so next to it. Reads and writes must be distinguishable at a glance. -->

## 3. First five minutes

<!--
The minimum set of checks that tells you which procedure to follow. Each check has an
exact command or link, and what "normal" looks like, so the reader can tell abnormal.
-->

| # | Check | How | Normal | If abnormal, go to |
| --- | --- | --- | --- | --- |
| 1 | Is it still happening? | dashboard link | error rate < 0.1% | continue |
| 2 | Did a deploy just happen? | deploy log link | no deploy in last 60 min | 4.1 |
| 3 | Are dependencies healthy? | status page / dashboard link | all green | 4.2 |
| 4 | Is it one host or all? | | spread evenly | 4.3 |

### Declare an incident if

<!-- Objective conditions. The on-call should not have to judge whether "it is bad enough". -->

- Error rate above <n>% for more than <n> minutes
- Any data loss or corruption suspected
- More than one service affected

## 4. Procedures

<!--
One subsection per cause. Each step: the exact command, the expected output, and how to
verify it worked. Mark state-changing steps. End every procedure with a verification step
and the rollback for what was just done.
-->

### 4.1 <Cause: e.g. bad deploy>

Symptoms that point here:

Steps:

1. <read-only check>
   ```sh
   <command>
   ```
   Expected:
2. <state-changing action> (CHANGES STATE)
   ```sh
   <command>
   ```
   Expected:
3. Verify:
   ```sh
   <command>
   ```
   Expected: error rate back below <n>% within <n> minutes

Rollback of this procedure:

### 4.2 <Cause: e.g. dependency down>

Symptoms that point here:

Steps:

1.

Rollback of this procedure:

### 4.3 <Cause: e.g. single unhealthy host>

Symptoms that point here:

Steps:

1.

Rollback of this procedure:

## 5. If none of the above helps

<!-- The mitigation that buys time regardless of cause, and the point at which to stop debugging and escalate. -->

- Mitigation that reduces user impact without knowing the cause (e.g. shed load, fail over, enable degraded mode):
- Stop debugging and escalate when:
  - <n> minutes have passed without a working hypothesis
  - The mitigation above did not reduce impact

## 6. Escalation

| Situation | Escalate to | How | Expected response time |
| --- | --- | --- | --- |
| Need a second pair of hands | secondary on-call | | |
| Suspected data loss | | | |
| Dependency owned by another team | | | |
| Vendor or managed-service issue | vendor support, case priority | | |

## 7. Communication

| When | Where | What to say |
| --- | --- | --- |
| Incident declared | incident channel | impact, what is known, next update time |
| Every <n> minutes | incident channel | status, actions taken, next update time |
| Resolved | incident channel, stakeholders | impact window, resolution, postmortem owner |

<!-- Keep a message template here if the team has one, so nobody writes from scratch at 3 a.m. -->

## 8. Known issues and gotchas

<!-- Things that have bitten people before: misleading metrics, commands that hang, false positives. -->

-

## 9. After the incident

- [ ] Alert resolved and acknowledged
- [ ] Timeline notes saved (raw, unedited) to the incident channel or ticket
- [ ] Postmortem opened if the incident met the criteria in section 3 (use templates/08-postmortem.md)
- [ ] This runbook updated with anything that was missing or wrong

## 10. Review log

<!-- A runbook that nobody has tested in a year is a guess. Record tests and reviews. -->

| Date | Reviewed / tested by | What changed |
| --- | --- | --- |
| | | |

## References

- [Effective Troubleshooting (SRE book)](https://sre.google/sre-book/effective-troubleshooting/)
- [Being On-Call (SRE book)](https://sre.google/sre-book/being-on-call/)
- [PagerDuty Incident Response documentation](https://response.pagerduty.com/)
