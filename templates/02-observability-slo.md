# Observability / SLO Doc: <service name>

<!--
Use for creating or revising SLIs / SLOs and for designing monitoring and alerting.
For choosing SLIs, see Implementing SLOs in the SRE Workbook.
-->

| Item | Details |
| --- | --- |
| Status | Draft / In Review / Approved |
| Author / Reviewers / Approver | @<github-id> / @<github-id> / @<github-id> |
| Created / Last updated | YYYY-MM-DD / YYYY-MM-DD |
| Target service | |
| SLO scope | production only / all environments |
| Review cadence | Quarterly / Semi-annually |

## TL;DR

## 1. Target service and users

| Item | Details |
| --- | --- |
| Role of the service | |
| Direct users | |
| Services that depend on it | |
| Business criticality | |

### Critical user journeys (CUJ)

<!--
List the paths that users cannot do without.
Derive SLIs from these. Do not start from technical metrics (CPU utilization, etc.).
-->

| # | Journey | Path | Criticality |
| --- | --- | --- | --- |
| 1 | | | High |

## 2. SLI

<!--
Define each SLI as a ratio of good events to valid events.
State the measurement point (client side / load balancer / application). The value differs by location.
-->

| # | SLI name | Type | Definition (numerator / denominator) | Measurement point | Data source |
| --- | --- | --- | --- | --- | --- |
| 1 | <target API> success rate | Availability | Non-5xx responses / all responses | Load balancer | |
| 2 | <target API> response time | Latency | Responses within 300ms / all responses | Load balancer | |
| 3 | <target data> freshness | Freshness | Records reflected within <n> minutes of update / all updates | | |

### Exclusions

<!-- Events excluded from measurement and why. Examples: health checks, bots, 4xx. -->

-

## 3. SLO

| SLI | SLO | Measurement window | Current measured value | Rationale |
| --- | --- | --- | --- | --- |
| <target API> success rate | 99.9% | 28-day rolling | | |
| <target API> response time | 99% within 300ms | 28-day rolling | | |

<!--
Always write the rationale. "99.9% is the norm" is not a rationale.
Past measured values, the level users can tolerate, and the product of dependency SLOs are the inputs.
Start slightly looser than the current measured value and tighten once it is met; this works better.
-->

### Error budget

| Item | Details |
| --- | --- |
| Allowed downtime per 28 days | |
| Where to check budget consumption | |
| Handling of fast consumption (budget policy) | |

<!--
Example budget policy: when the remaining budget drops below 0, stop feature releases and
prioritize reliability work. Write down who makes that call.
-->

## 4. Alerts

<!--
Page only for alerts that need a human to act right now.
Route everything else to a ticket or a dashboard.
Multi-window burn rate alerts (rate of error budget consumption) are recommended.
-->

| # | Alert name | Condition | Severity | Destination | Runbook | Expected detection time |
| --- | --- | --- | --- | --- | --- | --- |
| 1 | | Burn rate 14.4x sustained for 1 hour | Page | On-call | | |
| 2 | | Burn rate 6x sustained for 6 hours | Page | On-call | | |
| 3 | | Burn rate 1x sustained for 3 days | Ticket | Team | | |

### False positives and missed detections

| Alert | Situations that can cause false positives | Failures it cannot detect |
| --- | --- | --- |
| | | |

## 5. Metrics, logs, and traces

| Type | What is collected | Collection method | Store | Retention | Monthly cost |
| --- | --- | --- | --- | --- | --- |
| Metrics | | | | | |
| Logs | | | | | |
| Traces | | | | | |

- High-cardinality labels (drivers of cost growth):
- How it is guaranteed that logs contain no personal data or secrets:

## 6. Dashboards

| Dashboard | Purpose | Audience | Main panels | Link |
| --- | --- | --- | --- | --- |
| Service overview | Steady-state status | Whole team | SLO attainment, remaining error budget | |
| Incident response | First-line triage | On-call | Error breakdown, dependency status | |

## 7. Gaps versus today

| Needed | Current | Action | Owner | Due |
| --- | --- | --- | --- | --- |
| | Not measured | | | |

## 8. Open questions

| # | Question | Status |
| --- | --- | --- |
| 1 | | Open |

## 9. References

- [Implementing SLOs (SRE Workbook)](https://sre.google/workbook/implementing-slos/)
- [Alerting on SLOs (SRE Workbook)](https://sre.google/workbook/alerting-on-slos/)
