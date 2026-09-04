# Worked examples

These show how far a template needs to be filled in before it is ready for review.
All content is fictional and does not describe any real system.

Examples 0001 and 0003–0005 describe the same fictional order service, so you can follow one system
from SLO definition, through a design decision and a migration, to an incident and the runbook it improved.

| Example | Template used | What to look at |
| --- | --- | --- |
| [0001-mysql-8-upgrade.md](0001-mysql-8-upgrade.md) | [03-migration](../templates/03-migration.md) | Parallel run and cutover decision points; how to write the point of no return |
| [0002-adr-log-retention.md](0002-adr-log-retention.md) | [07-adr](../templates/07-adr.md) | The level of detail for a one-page decision record |
| [0003-slo-order-api.md](0003-slo-order-api.md) | [02-observability-slo](../templates/02-observability-slo.md) | Deriving SLIs from user journeys; justifying each SLO target; burn-rate alerts replacing cause-based alerts |
| [0004-postmortem-order-api-cache-ttl.md](0004-postmortem-order-api-cache-ttl.md) | [08-postmortem](../templates/08-postmortem.md) | Root causes that go past "someone typed 3 instead of 30"; action items with owners; rejected actions and why |
| [0005-runbook-order-api-high-error-rate.md](0005-runbook-order-api-high-error-rate.md) | [10-runbook](../templates/10-runbook.md) | First-five-minutes checks, state-changing steps marked, a review log fed by the postmortem |
