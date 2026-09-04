# Migration Doc: MySQL 8.0 upgrade for the order service database

> This is a worked example. All content is fictional.

| Item | Details |
| --- | --- |
| Status | Approved |
| Author / Reviewers / Approver | @alice / @bob, @carol / @dave |
| Created / Last updated | 2026-07-14 / 2026-08-20 |
| Source → Target | Managed MySQL 5.7 → 8.0 |
| Migration approach | Parallel run (replication) + big-bang cutover |
| Expected downtime | 8-minute write freeze (reads continue) |
| Planned cutover | 2026-09-14 02:00 JST |
| Deadline and reason | 2026-11-30 (end of extended support for 5.7; cannot be moved) |

## TL;DR

Upgrade the order service database from MySQL 5.7 to 8.0. Build an 8.0 replica and let it catch up,
then freeze writes for 8 minutes overnight and switch the connection target. Keep the old environment for 2 weeks before decommissioning it.

## 1. Background and deadline

Extended support for the current 5.7 ends on 2026-11-30, after which no security fixes are provided.
The vendor sets this deadline, and it is not negotiable.

November is the peak season for orders, so the effective deadline is the end of October. To keep two
fallback dates, the cutover is planned for mid-September.

## 2. Goals and non-goals

### Goals

- Complete the migration to 8.0 by the end of November
- Keep the write freeze under 15 minutes
- Stay able to roll back to the old environment within 30 minutes of cutover

### Non-goals

- Changing the instance class (it widens the scope of performance testing; consider it separately after the migration)
- Cleaning up unused tables (doing it alongside the migration complicates diff verification)
- Standardizing the character set on utf8mb4 (the 8.0 default changes, but converting existing tables is a separate doc)

## 3. Inventory of migration targets

| # | Target | Type | Size | Approach | Order | Owner | Status |
| --- | --- | --- | --- | --- | --- | --- | --- |
| 1 | orders schema | Data | 420 GB / 810 million rows | Replication | 1 | @alice | Done |
| 2 | 2 read-only replicas | Infrastructure | — | Create new | 2 | @alice | Done |
| 3 | Connection targets of 6 aggregation batch jobs | Config | — | Config change | 3 | @bob | Done |
| 4 | BI tool connection target | Config | — | Config change | 4 | @carol | Not started |
| 5 | 12 DB users and grants | Permissions | — | Recreate manually | 1 | @alice | Done |

### Not migrating

| Target | Reason | Handling in source |
| --- | --- | --- |
| 3 archive_2019 tables | Zero reads for 2 years. Excluded from the migration | Take a dump and keep it; delete after migration |

### Dependencies

| Depends on the migration target | Form of dependency | Change required | Owning team |
| --- | --- | --- | --- |
| Order API | Hostname in environment variables | Yes (restart at cutover) | Our team |
| 6 aggregation batch jobs | Hostname in config files | Yes | Our team |
| BI tool | Connection details registered directly in the admin console | Yes (manual) | Data platform team |
| Inventory service | Via the order API only | No | Inventory team |

Searched all repositories for places where connection details are written and confirmed they are limited to the 4 above.
Direct connections from the inventory service to the DB were removed in 2025.

## 4. Choosing the migration approach

| Approach | Summary | Downtime | Ease of rollback | Implementation cost | Decision |
| --- | --- | --- | --- | --- | --- |
| In-place upgrade | Upgrade the same instance to 8.0 | 40–60 min | Low (restore from snapshot, over 1 hour) | Low | Rejected |
| Parallel run + big-bang cutover | Build an 8.0 replica and switch the connection target | 8 min | High (just switch the target back) | Medium | Adopted |
| Parallel run + dual writes | Write to both, move reads over gradually | None | High | High | Rejected |

### Chosen approach and reason

The deciding factor is rollback speed. Recovering from a failed in-place upgrade takes over an hour,
which would miss the early-morning batch jobs even in the overnight window.

Dual writes bring downtime to zero, but they require a dual-write path in the application,
and that code is thrown away after the migration. The product side agreed that an 8-minute write freeze
overnight is acceptable, so we did not adopt it.

## 5. Migration plan

| Phase | Details | Period | Criteria to proceed | Owner |
| --- | --- | --- | --- | --- |
| 1. Preparation | Compatibility check on 8.0, fix incompatibilities | 7/15–8/8 | All tests pass on 8.0 | @alice |
| 2. Build new environment | Create the 8.0 instance and replicas | 8/11–8/15 | Build complete | @alice |
| 3. Data sync | Start replication from 5.7 to 8.0 | 8/18 | Lag stable within 5 seconds | @alice |
| 4. Verification | Consistency checks, 2 dry runs in staging | 8/19–9/5 | 2 consecutive successful dry runs | @bob |
| 5. Cutover | Production cutover | 9/14 02:00 | — | @alice |
| 6. Monitoring | 2-week monitoring period | 9/14–9/28 | No SLO violations | Everyone |
| 7. Decommission old environment | Delete 5.7 | 9/29 | Monitoring period complete | @alice |

## 6. Data migration and consistency

| Item | Details |
| --- | --- |
| Sync method | Continuous replication with 5.7 as the source |
| Writes during migration | Write to 5.7, replicate to 8.0 (keep lag within 5 seconds) |
| Character set and type differences | The default collation changes, so existing tables have been changed to specify it explicitly |
| Measured migration time | Initial sync 6 hours 20 minutes (measured 8/18) |

Of the 8.0 incompatibilities, 3 actually required fixes: 1 column named with `rank`, now a reserved word,
and 2 queries that relied on implicit `GROUP BY` sorting. All were fixed on 8/8.

### Consistency verification

| Check | Method | Pass criteria | Timing |
| --- | --- | --- | --- |
| Row counts | Compare row counts of all 47 tables | Exact match | Right before cutover |
| Content sampling | Checksum comparison of 10,000 random orders from the last 30 days | Zero differences | Right before cutover |
| Business view | Run the previous day's sales aggregation in both environments and compare | Amounts match exactly | Right before cutover |
| Replication lag | Check lag in seconds | 0 seconds | After write freeze |

### If inconsistencies are found

Abort the cutover, resume writes, and keep running on 5.7. Retry on a fallback date (9/21 or 9/28).

## 7. Cutover procedure

### Preparation

| # | Task | Owner | Due | Verification | Done |
| --- | --- | --- | --- | --- | --- |
| 1 | Dry run in staging with production-equivalent data | @bob | 9/5 | Within 8 minutes following the procedure | [x] |
| 2 | Demonstrate the rollback procedure | @bob | 9/5 | Back on the old environment within 5 minutes | [x] |
| 3 | Notify product and CS | @carol | 9/7 | Notified | [x] |
| 4 | Confirm communication channels for the day | @alice | 9/13 | Call connects | [ ] |
| 5 | Take a snapshot right before cutover | @alice | 9/14 01:30 | Snapshot complete | [ ] |

### Timeline on the day

| Time | Task | Owner | Duration | Verification |
| --- | --- | --- | --- | --- |
| 01:30 | Confirm staffing, take snapshot, open monitoring dashboards | Everyone | 20m | Snapshot complete |
| 02:00 | Show maintenance page, freeze writes | @bob | 2m | Write APIs return 503 |
| 02:02 | Wait for replication lag to reach 0 | @alice | 1m | Lag 0 seconds |
| 02:03 | Stop replication, promote 8.0 | @alice | 1m | Writable |
| 02:04 | Consistency verification (row counts, checksums, sales aggregation) | @bob | 3m | All match |
| 02:07 | Switch the application connection target and restart | @alice | 2m | Connectivity check |
| 02:09 | Connectivity check, place a test order | @bob | 3m | Order can be created |
| 02:12 | Remove maintenance page | @bob | 1m | Normal display |
| 02:15–03:00 | Monitor (error rate, latency, slow queries) | Everyone | 45m | Same level as normal |
| 03:00 | Stand down | Everyone | — | — |

### Decision points

| Time | Decision | Decided by | Condition to proceed | Action on abort |
| --- | --- | --- | --- | --- |
| 02:02 | Does replication lag reach 0 | @alice | 0 within 3 minutes | Resume writes, move to a fallback date |
| 02:07 | Does consistency verification pass | @bob | All 3 checks match | Resume writes, move to a fallback date |
| 02:12 | Does the connectivity check pass | @alice | Test order succeeds | Run the rollback procedure |
| 03:00 | Move to the monitoring period | @dave | Error rate and p99 at normal levels | Run the rollback procedure |

## 8. Rollback

| Item | Details |
| --- | --- |
| Rollback conditions | Connectivity check fails, error rate exceeds 1%, or p99 more than doubles |
| Procedure | Freeze writes → switch the application connection target back to 5.7 and restart → connectivity check → reopen |
| Time required | 5 minutes (measured in dry runs) |
| Point of no return and its time | After promoting 8.0 at 02:03, writes that land on 8.0 are not reflected in 5.7 |
| If problems appear after that point | Reverting to 5.7 would lose orders placed after cutover. Extract the delta since 02:03 from the binary log and apply it to 5.7 before reverting. The procedure is in the Runbook. Overnight orders run 3–8 per 10 minutes, which is few enough to correct by hand |
| Handling of delta data written to the target | Extract and apply manually as above |

## 9. Risks

| Risk | Likelihood | Impact | Mitigation | Detection |
| --- | --- | --- | --- | --- |
| Query plans change on 8.0 and specific queries slow down | Medium | High | Compared query plans of the top 50 queries on production-equivalent data (2 differences, resolved by adding indexes) | Slow query log, p99 |
| Missed connection target changes | Medium | Medium | Set 5.7 to read-only 30 minutes after cutover so missed writes fail and are detected | Error log |
| Write freeze exceeds 8 minutes | Low | Medium | Two dry runs measured 7 min 10 s and 7 min 40 s. A 15-minute maintenance window is reserved | Timekeeping on the day |
| Initial sync does not finish by the deadline | Low | Medium | Measured on 8/18 (6 hours 20 minutes) | — |

## 10. Decommissioning the old environment

| Item | Details |
| --- | --- |
| Decommission criteria | No SLO violations during the 2-week monitoring period, and no rollback was needed |
| Retention period | Keep 5.7 running for 2 weeks after cutover (9/14–9/28) |
| Planned decommission date | 2026-09-29 |
| What to decommission | 1 5.7 instance, 2 read replicas, 8 monitors for 5.7, and config referencing the old connection details. Delete the archive_2019 dump in 2026-12 or later |
| Cost savings | ¥420,000/month (incurred twice during the parallel run) |

## 11. Open questions

| # | Question | Decided by | Due | Status |
| --- | --- | --- | --- | --- |
| 1 | Change the BI tool connection target on cutover day or the next business day | @carol | 9/10 | Open |
| 2 | When to standardize the collation of existing tables | @dave | Separate doc after migration | Open |
