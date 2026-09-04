# Postmortem: Order API outage after cache TTL change

> This is a worked example. All content is fictional.

| Item | Details |
| --- | --- |
| Status | Final |
| Author | @bob |
| Reviewers | @alice, @dave, @erin |
| Date of incident | 2026-06-18 |
| Severity | Sev1 |
| Impact duration | 47 min |
| Detection | Monitoring alert (OrderApiFastBurn) |
| Responders | @bob (on-call), @alice, @carol |
| Related | incident channel `#inc-2026-06-18-orders`, Issue #518, alert OrderApiFastBurn |

## Summary

A configuration change reduced the product-catalog cache TTL from 300 s to 3 s by mistake.
Cache hit rate fell from 97% to 12%, the catalog database saturated its connection pool,
and the Order API returned 5xx for 62% of requests for 47 minutes. Checkout was unavailable
for most users during that time. Rolling back the configuration restored service.

## Impact

| Item | Details |
| --- | --- |
| Affected users | All storefront and mobile customers attempting checkout |
| What they experienced | "Something went wrong" on the checkout page; order status page slow or failing |
| Count / share | ~41,000 failed requests, 62% of Order API traffic in the window. ~2,300 checkouts abandoned (estimate from funnel data) |
| Error budget consumed | 47 min of a 40 min 19 s budget. Budget for the 28-day window is now negative; budget policy is in effect |
| Data loss or inconsistency | None. Failed requests did not create partial orders (verified by reconciliation job) |
| Financial impact | Estimated ¥3.1M in delayed or lost orders. About 60% were recovered within 24 h |
| External communication | Status page updated at 14:27; apology banner shown until 18:00 |

## Timeline

All times JST, 2026-06-18.

| Time | Event | Who / what |
| --- | --- | --- |
| 13:52 | Config PR #512 merged: intended to change catalog cache TTL from 300 s to 30 s | @carol |
| 13:58 | Config deployed to production by the config pipeline | pipeline |
| 14:01 | Cache hit rate drops from 97% to 12%. Catalog DB connections reach pool limit (200) | — |
| 14:03 | Order API error rate crosses 30% | — |
| 14:06 | OrderApiFastBurn pages on-call | alerting |
| 14:08 | On-call acknowledges, opens incident channel, checks deploys: no application deploy in the last hour | @bob |
| 14:15 | Catalog DB dashboard shows connection saturation. Hypothesis: DB problem. DBA paged | @bob |
| 14:22 | DBA finds query volume 8× normal, all cache-miss shaped queries. Asks whether cache changed | @alice |
| 14:24 | Config pipeline log shows deploy at 13:58. PR #512 diff shows `ttl: 3` instead of `ttl: 30` | @bob |
| 14:27 | Status page updated | @carol |
| 14:31 | Config reverted via pipeline. Rollback takes 6 min because the config pipeline rebuilds the full bundle | @bob |
| 14:37 | Cache hit rate recovers to 90%+. Error rate falls below 1% | — |
| 14:48 | Error rate at baseline for 10 min. Incident closed | @bob |

### Key durations

| Metric | Duration | Target | Why the gap |
| --- | --- | --- | --- |
| Occurrence to detection | 5 min | 5 min | Met. Fast-burn alert worked as designed |
| Detection to response start | 2 min | 5 min | Met |
| Response start to recovery | 29 min | 15 min | 16 min spent on the DB hypothesis before checking config deploys. Config deploys were not on the incident dashboard |

## Causes

### Trigger

The catalog cache TTL was set to 3 seconds instead of 30 seconds in PR #512.

### Root causes

- The config file takes TTL as a bare integer with no unit and no validation. `3` and `30` and `300`
  are all accepted. Nothing in review or CI checks the value against a sane range.
- The config pipeline deploys to production directly on merge. There is no staging soak and no
  canary for configuration, although application deploys have both.
- The catalog database has no protection against a cache-miss storm: no connection limit per
  client, no request coalescing, no circuit breaker in the Order API when catalog lookups slow down.

### Contributing factors

- Config deploys were not shown on the incident dashboard, only application deploys. The
  on-call's first check ("did a deploy just happen?") returned "no", which sent triage toward the DB.
- The runbook for high error rate did not mention configuration changes as a cause.
- Config rollback takes 6 minutes because the pipeline rebuilds everything. A targeted revert
  would take under 1 minute.

## What went well

- The fast-burn alert detected the problem within 5 minutes with no false positives before or after.
- The incident channel and status page process were followed without prompting.
- The DBA's observation ("these are all cache-miss queries") was the pivot that found the cause.
- Order creation is transactional; no partial orders were created despite the failures.

## What went badly

- 16 minutes were lost on a wrong hypothesis because the dashboard hid the relevant change.
- The reviewer of PR #512 approved a one-character diff in under a minute. The PR had no
  description of the expected effect.
- Nobody on the call knew how long a config rollback would take, so nobody considered a faster
  manual override.

## Where we got lucky

- The incident happened at 14:00 on a weekday. At peak (20:00–22:00) the database might have
  failed entirely rather than saturating, which would have taken longer to recover.
- The DBA happened to be available within 7 minutes. There is no DBA on-call rotation.

## Action items

| # | Type | Action | Owner | Due | Issue | Status |
| --- | --- | --- | --- | --- | --- | --- |
| 1 | Prevent | Config schema validation in CI: TTL fields require a unit suffix (`30s`) and a range (10s–1h) | @carol | 2026-07-02 | #519 | Done |
| 2 | Prevent | Config pipeline gets a staging soak (10 min) and a 10% canary before full production | @alice | 2026-07-16 | #520 | In progress |
| 3 | Detect | Add config deploys to the incident dashboard's "recent changes" panel and to the runbook's first checks | @bob | 2026-06-25 | #521 | Done |
| 4 | Mitigate / recover | Circuit breaker in Order API for catalog lookups: serve stale cache and degrade product details when catalog p99 > 500 ms | @erin | 2026-07-30 | #522 | In progress |
| 5 | Mitigate / recover | Per-client connection limit on the catalog DB (Order API capped at 120 of 200) | @alice | 2026-07-09 | #523 | Done |
| 6 | Mitigate / recover | Targeted config revert path in the pipeline (< 1 min) | @carol | 2026-07-23 | #524 | Not started |
| 7 | Process / docs | Runbook 0005 updated: config changes added to first checks, config rollback time documented | @bob | 2026-06-20 | #525 | Done |
| 8 | Process / docs | PR template for the config repo requires "expected effect" and "how to verify" fields | @carol | 2026-06-27 | #526 | Done |

### Rejected action items

| Proposal | Why rejected |
| --- | --- |
| Require two approvals on all config PRs | Slows every change and would not have caught this one: the reviewer read `3` as `30`. Validation (item 1) addresses the actual failure |
| Move catalog cache TTL to a hard-coded constant | Removes a legitimate operational knob. Validation with a range is enough |
| Establish a DBA on-call rotation | Only one DBA on staff. Instead, item 5 and item 4 reduce the need for DBA involvement in this class of incident |

## Lessons learned

- "Did a deploy just happen?" must include every kind of deploy the system accepts. We had
  been treating configuration as not-a-deploy. Shared with the platform team; the same gap exists
  for feature-flag changes and is being fixed there.
- A unit-less integer in a config file is a latent incident. The config schema validator (item 1)
  is being generalized for other repositories.

## References

- [Postmortem Culture: Learning from Failure](https://sre.google/sre-book/postmortem-culture/)
- [Example Postmortem](https://sre.google/sre-book/example-postmortem/)
- SLO definition: examples/0003-slo-order-api.md
- Runbook: examples/0005-runbook-order-api-high-error-rate.md
