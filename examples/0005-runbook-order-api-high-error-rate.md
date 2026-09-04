# Runbook: Order API / high error rate (OrderApiFastBurn, OrderApiSlowBurn)

> This is a worked example. All content is fictional.

| Item | Details |
| --- | --- |
| Service | Order API (`orders-api`) |
| Owning team / on-call rotation | Commerce platform team / `commerce-oncall` |
| Alerts that route here | OrderApiFastBurn, OrderApiSlowBurn |
| Severity of those alerts | Page |
| Last reviewed | 2026-06-20 by @bob |
| Last tested (drill or real use) | 2026-06-18 (incident), 2026-07-04 (drill) |
| Design Doc | docs/0003-order-api-resilience.md (fictional) |
| Dashboard | Order API incident dashboard |
| Logs | Log backend, saved query `orders-api errors` |
| Escalation | see section 6 |

## 1. What this alert means

The Order API is returning 5xx (or 429) for enough requests that, at the current rate, the
28-day error budget would be gone within hours (FastBurn) or days (SlowBurn). Users see checkout
failing or order status pages erroring. The alert measures the success-rate SLI at the load
balancer, so instances that fail to start count as errors even though the application logs show nothing.

It does not mean the database is down. Since the 2026-06-18 incident, the most common cause is a
change somewhere upstream of the Order API, not a failure in it.

- User-visible impact: checkout errors, order status page failures
- What the alert measures: non-5xx responses / all responses on `/v1/orders*` paths, at the LB
- Typical causes, most frequent first:
  1. A change was just deployed: application, configuration, or feature flag
  2. A dependency is slow or down: catalog service, payment provider, catalog DB
  3. One or a few unhealthy instances behind the LB

## 2. Before you start

| Need | Details |
| --- | --- |
| Access required | `commerce-oncall` role (grants read on dashboards and logs, deploy rollback, instance drain) |
| Where to run commands | Ops bastion `ops-01`, or the `orders-api` GitHub Actions workflows |
| Tools that must be installed | `kubectl` with the production context, `ordersctl` (installed on the bastion) |
| Safe to run during business hours? | Read checks: yes. Rollback and drain: yes, they are the standard mitigation. Database commands: no, page the DBA |

Commands marked **CHANGES STATE** alter production. Everything else is read-only.

## 3. First five minutes

| # | Check | How | Normal | If abnormal, go to |
| --- | --- | --- | --- | --- |
| 1 | Is it still happening? | Incident dashboard, "Error rate" panel | < 0.1% | continue |
| 2 | Did any change deploy in the last 60 min? | Incident dashboard, "Recent changes" panel (application, config, feature flags) | none | 4.1 |
| 3 | Are dependencies healthy? | "Dependencies" panel: catalog p99, payment provider status, catalog DB connections | catalog p99 < 200 ms, payment green, DB connections < 150/200 | 4.2 |
| 4 | Is it all instances or a few? | "Errors by instance" panel | evenly spread | 4.3 |
| 5 | Is the error 5xx or 429? | "Errors by status" panel | mostly 5xx | 429 means load shedding is active; see 4.2 |

### Declare an incident if

- Error rate above 5% for more than 5 minutes
- Any sign of duplicate or partial orders in the reconciliation panel
- Payment provider is also degraded (customers may be charged without an order)

To declare: run `/incident declare orders` in Slack. It creates the channel and pages the incident commander.

## 4. Procedures

### 4.1 A change was just deployed

Symptoms that point here: error rate rose sharply within 10 minutes of a change in the "Recent changes" panel.

Steps:

1. Identify the change and its type.
   ```sh
   ordersctl changes --since 60m
   ```
   Expected: a list with type (`app`, `config`, `flag`), time, and author.
2. Roll it back. Pick the matching command. **CHANGES STATE**
   - Application:
     ```sh
     ordersctl deploy rollback --to previous
     ```
     Expected: `rollout restarted`, completes in ~3 min.
   - Configuration:
     ```sh
     ordersctl config revert --to previous
     ```
     Expected: `config applied`, completes in ~1 min (targeted revert, added 2026-07). If this fails, the full pipeline revert takes ~6 min.
   - Feature flag:
     ```sh
     ordersctl flag set <flag-name> --to previous
     ```
     Expected: immediate.
3. Verify:
   ```sh
   ordersctl slo status --window 5m
   ```
   Expected: error rate back below 1% within 5 minutes of the rollback completing.
4. Post in the incident channel: what was rolled back, and ask the author not to re-apply until the postmortem.

Rollback of this procedure: re-apply the change with `ordersctl <type> apply --to <version>`. Do this only if the rollback made things worse.

### 4.2 A dependency is slow or down

Symptoms that point here: no recent change; catalog p99 > 500 ms, or payment provider status is not green, or catalog DB connections at the limit.

Steps:

1. Confirm which dependency.
   ```sh
   ordersctl deps status
   ```
   Expected: each dependency with latency, error rate, and circuit-breaker state.
2. If the catalog circuit breaker is `closed` while catalog p99 > 500 ms, open it manually. This serves stale product data and hides product images, but checkout works. **CHANGES STATE**
   ```sh
   ordersctl breaker open catalog
   ```
   Expected: `breaker catalog: open`. Error rate should fall within 2 minutes.
3. If the payment provider is degraded: nothing to fix on our side. Confirm on their status page, post the link in the incident channel, and enable the checkout notice banner. **CHANGES STATE**
   ```sh
   ordersctl flag set checkout_payment_notice --to on
   ```
4. If the catalog DB is at its connection limit: page the DBA (section 6). Do not run database commands yourself.
5. Verify:
   ```sh
   ordersctl slo status --window 5m
   ```
   Expected: error rate below 1%, or 429s only (load shedding working as intended).

Rollback of this procedure: `ordersctl breaker close catalog` once catalog p99 is under 200 ms for 10 minutes; `ordersctl flag set checkout_payment_notice --to off` when the provider recovers.

### 4.3 A few unhealthy instances

Symptoms that point here: errors concentrated on one or two instances in the "Errors by instance" panel.

Steps:

1. List instance health.
   ```sh
   kubectl -n orders get pods -o wide
   ```
   Expected: all `Running`, `READY 1/1`. Look for `CrashLoopBackOff` or `0/1`.
2. Drain the unhealthy instance. The LB stops routing to it within 30 s. **CHANGES STATE**
   ```sh
   kubectl -n orders delete pod <pod-name>
   ```
   Expected: pod terminates, a replacement starts within 1 min.
3. Verify:
   ```sh
   ordersctl slo status --window 5m
   ```
   Expected: error rate below 1%.
4. If the replacement is also unhealthy, this is not an instance problem. Go back to section 3 and re-check for changes.

Rollback of this procedure: none needed; the deleted pod is replaced automatically.

## 5. If none of the above helps

- Mitigation regardless of cause: enable degraded checkout, which skips product enrichment and inventory pre-check. Orders are validated asynchronously and customers are emailed if anything fails. **CHANGES STATE**
  ```sh
  ordersctl flag set checkout_degraded_mode --to on
  ```
- Stop debugging and escalate when:
  - 15 minutes have passed since acknowledgement without a working hypothesis
  - Degraded mode did not bring the error rate under 5%

## 6. Escalation

| Situation | Escalate to | How | Expected response time |
| --- | --- | --- | --- |
| Need a second pair of hands | Secondary on-call | `/oncall page commerce-secondary` | 5 min |
| Suspected duplicate or partial orders | Incident commander + @alice (data owner) | `/incident declare orders` if not yet declared, then page | 10 min |
| Catalog DB at connection limit | DBA (@alice, business hours; otherwise platform on-call) | `/oncall page dba` | 15 min |
| Payment provider degraded > 30 min | Vendor support, priority P1, account ID in the team vault | Support portal | 30 min per contract |

## 7. Communication

| When | Where | What to say |
| --- | --- | --- |
| Incident declared | `#inc-<date>-orders` and `#status-commerce` | Impact, what is known, next update time |
| Every 15 minutes | Same | Status, actions taken, next update time |
| Status page | statuspage, component "Checkout" | Use the "Degraded performance" template; do not speculate about cause |
| Resolved | Same channels, stakeholders list | Impact window, resolution, postmortem owner and date |

Message template for the first update:

> Checkout is currently failing for some customers. We are investigating. Next update at HH:MM.

## 8. Known issues and gotchas

- The "Recent changes" panel lags by up to 2 minutes. If the error rate jumped in the last 2 minutes, re-check it.
- `ordersctl config revert` needs the previous config bundle to exist in the artifact store. Bundles older than 30 days are pruned; for those, use the pipeline revert (6 min).
- Deleting more than two pods at once trips the pod disruption budget and the command hangs. Delete one at a time.
- During the 2026-06-18 incident, the DB dashboard looked like the cause but was a symptom. If the DB is saturated, ask what changed upstream before paging the DBA.

## 9. After the incident

- [ ] Alert resolved and acknowledged
- [ ] Timeline notes saved (raw, unedited) to the incident channel
- [ ] Postmortem opened if the incident was declared (use templates/08-postmortem.md); owner named in the resolution message
- [ ] This runbook updated with anything that was missing or wrong

## 10. Review log

| Date | Reviewed / tested by | What changed |
| --- | --- | --- |
| 2026-03-10 | @bob | Initial version |
| 2026-06-20 | @bob | Added config and feature flags to first checks; added 4.2 circuit breaker step; documented rollback times (postmortem 0004, items 3 and 7) |
| 2026-07-04 | @erin, @bob | Quarterly drill. 4.3 step 2 clarified after the PDB hang was hit |

## References

- Postmortem: examples/0004-postmortem-order-api-cache-ttl.md
- SLO definition: examples/0003-slo-order-api.md
- [Effective Troubleshooting (SRE book)](https://sre.google/sre-book/effective-troubleshooting/)
