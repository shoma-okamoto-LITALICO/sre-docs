# ADR-0002: Set application log retention to 30 days and archive a further 90 days to infrequent-access storage

> This is a worked example. All content is fictional.

| Item | Details |
| --- | --- |
| Status | Accepted |
| Decision date | 2026-06-03 |
| Decided by | @dave |
| Stakeholders | @alice, @bob, data platform team |
| Related | Issue #412, ADR-0001 |

## Context

Application logs are currently retained for 400 days in the searchable logging platform, at a cost of
¥680,000/month. Log-related costs account for 23% of the whole platform and have grown 1.7x in the last 6 months.

Looking at actual usage, 96% of search queries targeted the last 7 days and 99.4% the last 30 days.
Searches over logs older than 30 days happen 2–4 times a month, all for audit requests and
long-term trend analysis.

Meanwhile, the internal information management policy requires 1-year retention of access logs. Legal confirmed
that this requirement applies only to access logs, and application logs are out of scope.

## Decision

Shorten the searchable retention of application logs to 30 days. Compress and archive days 31–120 to
infrequent-access storage, and ingest them for search when needed. Delete anything older than 120 days.

Access logs, which the policy covers, are out of scope for this decision and remain retained for 400 days as before.

## Rationale

- 99.4% of searches fall within 30 days, so the shorter retention has little impact on day-to-day work
- Archiving to infrequent-access storage keeps logs from day 31 onward, which satisfies the audit-request requirement
- Monthly cost is expected to drop from ¥680,000 to ¥210,000. The savings pay back the implementation effort for archive and restore (estimated 5 person-days) within 1 month

## Alternatives

| Option | Summary | Reason not adopted |
| --- | --- | --- |
| Keep 400-day retention and sample to reduce volume | Retain only a fixed share of logs | Logs for a specific request may be missing during an incident investigation. Whether the investigation succeeds would depend on luck |
| Move to a cheaper logging product | Replace the platform itself | Migration cost is high, and existing dashboards and alerts need to be rebuilt. Reconsider after the retention review is done |
| Do nothing | Keep the status quo | At the current growth rate, monthly cost reaches ¥1,150,000 in 12 months |

## Consequences

### Benefits

- Log-related monthly cost drops from ¥680,000 to ¥210,000
- Less data to search, so search response times get shorter

### Costs and accepted constraints

- Viewing logs older than 31 days requires ingesting them from the archive. Takes about 30 minutes
- Investigating an incident that continues past 30 days needs one extra step
- The archive and restore mechanism itself becomes one more thing to operate. Breakage goes unnoticed in normal use,
  so run a restore test every quarter

### Follow-up work

- Implement the archive process and write the restore procedure into the Runbook (Issue #412)
- Change the retention setting on the logging platform
- Review the 3 existing dashboards that reference logs older than 30 days
- Register the restore test as a routine task

## Conditions for revisiting

- If searches over logs older than 30 days exceed 10 per month, reconsider the retention period
- If restoring from the archive keeps taking more than 1 hour, revisit the archive method
- If the information management policy is revised to include application logs in its retention scope

## References

- [Documenting Architecture Decisions](https://cognitect.com/blog/2011/11/15/documenting-architecture-decisions)
