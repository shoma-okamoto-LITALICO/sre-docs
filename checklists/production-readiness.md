# Production readiness checklist

<!--
Run this checklist before shipping a new service or component to production,
or before making a large change to an existing system.
Use it as an appendix to a Design Doc, or as a standalone PR.

How to use:
- Fill in each item as "Done / Not done / N/A". Shipping with items left as
  "Not done" is allowed, but record who accepted it
- For N/A items, write a one-line reason. Do not leave them blank
- Based on the Launch Coordination Checklist and the
  Production Readiness Review (PRR) from the SRE book
-->

| Item | Details |
| --- | --- |
| Target | |
| Planned launch date | YYYY-MM-DD |
| Filled in by | @<github-id> |
| Reviewed by | @<github-id> |
| Related Design Doc | |

## 1. Design and review

| # | Item | Status | Notes |
| --- | --- | --- | --- |
| 1.1 | Design Doc is Approved | | |
| 1.2 | Alternatives considered are recorded | | |
| 1.3 | Security review completed | | |
| 1.4 | Agreement reached with dependent teams | | |
| 1.5 | Single points of failure identified and the accepted scope decided | | |

## 2. Reliability

| # | Item | Status | Notes |
| --- | --- | --- | --- |
| 2.1 | SLIs / SLOs defined | | |
| 2.2 | Failure modes and their impact identified | | |
| 2.3 | Behavior when dependencies fail decided (timeouts, retries, degraded mode) | | |
| 2.4 | Retries do not amplify load (backoff, caps) | | |
| 2.5 | Redundant configuration in place (or the reason for no redundancy is recorded) | | |
| 2.6 | RTO / RPO decided | | |
| 2.7 | Backups are taken | | |
| 2.8 | Restore tested and succeeded | | |

<!-- If 2.8 is "Not done", treat it as almost the same as having no backup. -->

## 3. Monitoring and operations

| # | Item | Status | Notes |
| --- | --- | --- | --- |
| 3.1 | Metrics are collected | | |
| 3.2 | Logs are collected and retention is decided | | |
| 3.3 | Dashboard exists | | |
| 3.4 | Alerts configured, and notifications reach a real person or on-call rotation | | |
| 3.5 | Alert firing verified in practice | | |
| 3.6 | Runbook exists (first response is written) | | |
| 3.7 | On-call engineers know the target system | | |
| 3.8 | Only alerts that are worth acting on in the middle of the night page someone | | |

## 4. Performance and capacity

| # | Item | Status | Notes |
| --- | --- | --- | --- |
| 4.1 | Expected load defined, with its basis | | |
| 4.2 | Load test performed | | |
| 4.3 | First constraint to saturate is identified | | |
| 4.4 | Scaling method decided | | |
| 4.5 | Limits (rate limits, quotas) are known and have headroom | | |

## 5. Security

| # | Item | Status | Notes |
| --- | --- | --- | --- |
| 5.1 | Authentication and authorization verified to work as designed | | |
| 5.2 | Permissions are least-privilege | | |
| 5.3 | No secrets in code, logs, or images | | |
| 5.4 | Data encrypted in transit and at rest | | |
| 5.5 | Exposure is as intended (nothing unintentionally public) | | |
| 5.6 | Audit logs are retained | | |
| 5.7 | Handling of personal and confidential data follows policy | | |

## 6. Deployment and rollback

| # | Item | Status | Notes |
| --- | --- | --- | --- |
| 6.1 | Deployment is automated (if manual, a procedure exists) | | |
| 6.2 | Rollback procedure exists and has been tested | | |
| 6.3 | On-call can run the rollback alone | | |
| 6.4 | Staged rollout plan exists | | |
| 6.5 | Post-launch checks and pass criteria decided | | |

## 7. Cost and contracts

| # | Item | Status | Notes |
| --- | --- | --- | --- |
| 7.1 | Monthly cost estimate exists | | |
| 7.2 | Unexpected cost increases can be detected | | |
| 7.3 | Contracts and terms of service of external services reviewed | | |

## 8. Communication

| # | Item | Status | Notes |
| --- | --- | --- | --- |
| 8.1 | Stakeholders informed of the launch schedule | | |
| 8.2 | Users notified if they are affected | | |
| 8.3 | Launch-day staffing and communication channels decided | | |

## Items shipped as Not done

<!-- If any item stays "Not done", copy it here and record who accepted it. -->

| # | Item | Reason not done | Workaround | Due | Accepted by |
| --- | --- | --- | --- | --- | --- |
| | | | | | |

## References

- [Reliable Product Launches at Scale (SRE book)](https://sre.google/sre-book/reliable-product-launches/)
- [Evolving SRE Engagement Model (SRE book)](https://sre.google/sre-book/evolving-sre-engagement-model/)
