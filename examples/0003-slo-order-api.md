# Observability / SLO Doc: Order API

> This is a worked example. All content is fictional.

| Item | Details |
| --- | --- |
| Status | Approved |
| Author / Reviewers / Approver | @alice / @bob, @erin / @dave |
| Created / Last updated | 2026-04-02 / 2026-05-11 |
| Target service | Order API (`orders-api`) |
| SLO scope | production only |
| Review cadence | quarterly |

## TL;DR

Define two SLOs for the Order API, availability 99.9% and latency 99% under 400 ms over 28 days,
measured at the load balancer. Replace the current CPU and 5xx-count alerts with burn-rate alerts
on these SLOs. Current measurements meet both targets with room to spare.

## 1. Service and users

| Item | Details |
| --- | --- |
| Role of the service | Creates, reads and cancels orders for the storefront and the mobile app |
| Direct users | Storefront web (BFF), mobile app, customer-support console |
| Services that depend on it | Inventory service (order events), billing batch (nightly) |
| Business criticality | Revenue path. An outage stops checkout |

### Critical user journeys (CUJ)

| # | Journey | Path | Priority |
| --- | --- | --- | --- |
| 1 | Place an order | `POST /v1/orders` | High |
| 2 | View order status | `GET /v1/orders/{id}` | High |
| 3 | Cancel an order | `POST /v1/orders/{id}/cancel` | Medium |
| 4 | Support looks up order history | `GET /v1/customers/{id}/orders` | Low |

Journeys 1 and 2 drive the SLOs. Journey 3 is low volume (under 1% of requests) and rides on the same
SLIs. Journey 4 is internal and gets a ticket-level alert only.

## 2. SLIs

| # | SLI name | Type | Definition (numerator / denominator) | Measurement point | Data source |
| --- | --- | --- | --- | --- | --- |
| 1 | Order API success rate | Availability | Responses that are not 5xx / all responses to CUJ 1–3 paths | Load balancer | LB access logs → metrics pipeline |
| 2 | Order API latency | Latency | Responses under 400 ms / all successful responses to CUJ 1–3 paths | Load balancer | LB access logs → metrics pipeline |
| 3 | Order event freshness | Freshness | Order events consumed by inventory within 60 s / all order events | Inventory consumer | Consumer lag metric |

Measured at the load balancer rather than in the application, so that instances that fail to start or
time out are counted as failures. Client-side measurement was considered and rejected: the mobile app
reports with a delay of hours and cannot drive alerts.

### Exclusions

- Health-check requests (`GET /healthz`) — not user traffic
- Requests blocked by the WAF (403) — attack traffic, not users
- 4xx other than 429 — client errors. 429 is counted as a failure because it means we shed load

## 3. SLOs

| SLI | SLO | Window | Current measurement | Basis for the target |
| --- | --- | --- | --- | --- |
| Order API success rate | 99.9% | 28-day rolling | 99.96% (last 90 days) | Payment provider SLO is 99.95%; we cannot promise more than our dependencies. 99.9% gives 43 min/28 d of budget, enough for one bad deploy |
| Order API latency | 99% under 400 ms | 28-day rolling | 99.4% | Checkout UX research: abandonment rises above 500 ms end to end; 400 ms at the LB leaves 100 ms for the BFF |
| Order event freshness | 99.5% within 60 s | 28-day rolling | 99.8% | Inventory reservation tolerates 60 s of lag before oversell risk |

Targets are set below current measurements on purpose. Tightening comes after two quarters of meeting them.

### Error budget

| Item | Details |
| --- | --- |
| Allowed downtime per 28 days | 40 min 19 s (availability), 1% of requests over 400 ms (latency) |
| Where to check consumption | SLO dashboard, "Error budget remaining" panel |
| Budget policy | If remaining budget goes below 0: feature deploys to `orders-api` stop, on-call lead decides what ships. Reliability fixes and security patches are exempt. Policy owner: @dave |

## 4. Alerts

| # | Alert name | Condition | Severity | Notify | Runbook | Expected detection time |
| --- | --- | --- | --- | --- | --- | --- |
| 1 | OrderApiFastBurn | Burn rate ≥ 14.4× for 1 h AND ≥ 14.4× for 5 min | Page | On-call | examples/0005-runbook-order-api-high-error-rate.md | ~5 min at 100% failure |
| 2 | OrderApiSlowBurn | Burn rate ≥ 6× for 6 h AND ≥ 6× for 30 min | Page | On-call | same | ~30 min |
| 3 | OrderApiBudgetDrift | Burn rate ≥ 1× for 3 d AND ≥ 1× for 6 h | Ticket | Team channel | none, review in weekly ops meeting | hours |
| 4 | OrderEventLag | Freshness burn rate ≥ 6× for 1 h | Page | On-call | inventory team runbook | ~10 min |

Alerts 1–3 replace `OrdersApiCpuHigh` and `OrdersApi5xxCount` on 2026-05-18. The CPU alert paged
23 times in Q1 with zero user impact.

### False positives and missed detections

| Alert | Can fire falsely when | Cannot detect |
| --- | --- | --- |
| OrderApiFastBurn | A load test runs against production without the exclusion header | Failures that return 200 with an error body (none known today; contract tests cover this) |
| OrderApiSlowBurn | A single retail campaign shifts the traffic mix toward slow endpoints | Degradation confined to one customer segment under 1% of traffic |
| OrderEventLag | Inventory consumer is intentionally paused for a deploy (add silence to deploy script) | Events lost before publish. Covered by a separate reconciliation job |

## 5. Metrics, logs, traces

| Type | What is collected | How | Storage | Retention | Monthly cost |
| --- | --- | --- | --- | --- | --- |
| Metrics | LB access log derived counters (status, latency histogram by path template) | Log → metrics pipeline | Metrics backend | 15 months | ¥38,000 |
| Logs | Application JSON logs, LB access logs | Agent shipping | Log backend | 30 days searchable (see ADR-0002) | ¥210,000 (shared) |
| Traces | 5% head sampling, 100% of errors | SDK | Trace backend | 7 days | ¥24,000 |

- High-cardinality labels to avoid: `order_id`, `customer_id` on metrics. Use path templates, not raw paths.
- Personal data in logs: request bodies are not logged. Email addresses in error messages are masked by the log library (verified in test `LogMaskingTest`).

## 6. Dashboards

| Dashboard | Purpose | Audience | Main panels | Link |
| --- | --- | --- | --- | --- |
| Order API SLO | Day-to-day health | Whole team, product | SLO attainment, error budget remaining, burn rate | (link) |
| Order API incident | First triage | On-call | Error breakdown by status and path, latency by path, dependency status, recent deploys | (link) |
| Order events | Consumer health | Inventory team, on-call | Consumer lag, publish rate | (link) |

## 7. Gaps versus today

| Needed | Today | Action | Owner | Due |
| --- | --- | --- | --- | --- |
| Latency histogram per path template | Only p50/p99 across all paths | Add path-template label in the log → metrics pipeline | @alice | 2026-05-09 (done) |
| Load-test exclusion | Load tests count as user traffic | Add `X-Load-Test: 1` header handling at the LB | @bob | 2026-05-16 |
| Deploy-time silence for OrderEventLag | None | Add silence step to inventory deploy script | inventory team | 2026-05-30 |

## 8. Open questions

| # | Question | Status |
| --- | --- | --- |
| 1 | Should cancel (CUJ 3) get its own SLO once volume grows past 5% of requests? | Open, revisit Q3 |
| 2 | Storefront BFF wants an end-to-end SLO. Who owns it? | Open, product to decide |

## 9. References

- [Implementing SLOs (SRE Workbook)](https://sre.google/workbook/implementing-slos/)
- [Alerting on SLOs (SRE Workbook)](https://sre.google/workbook/alerting-on-slos/)
- ADR-0002: log retention (examples/0002-adr-log-retention.md)
