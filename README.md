# SRE Docs

Templates for the documents an SRE / DevOps / infrastructure team writes: design docs, runbooks,
postmortems, SLO definitions, ADRs, and the checklists that gate a launch or a handover.

Infrastructure changes rarely fail on *what* was built. They fail on *why it was shaped that way*.
A diagram and a configuration diff leave no record of the alternatives that were weighed, and no answer
to the question that matters at 3 a.m.: who rolls this back, and how. These templates exist to put that
reasoning in writing before implementation, get it reviewed, and keep it readable months later.

## Choosing a template

| What you are writing | Template |
| --- | --- |
| Anything else. The general-purpose starting point | [templates/00-base-design-doc.md](templates/00-base-design-doc.md) |
| Topology changes, new resources, re-architecture | [templates/01-infrastructure-change.md](templates/01-infrastructure-change.md) |
| SLIs / SLOs, monitoring, alerting, dashboards | [templates/02-observability-slo.md](templates/02-observability-slo.md) |
| Migrations — cloud to cloud, database, version upgrades, cutovers (design and cutover plan, not the command-level runbook) | [templates/03-migration.md](templates/03-migration.md) |
| CI/CD pipelines, deployment strategy, release process | [templates/04-cicd-release.md](templates/04-cicd-release.md) |
| Access control, network boundaries, secret management | [templates/05-security-access.md](templates/05-security-access.md) |
| Capacity planning, cost optimization | [templates/06-capacity-cost.md](templates/06-capacity-cost.md) |
| A single technical decision, recorded lightly (one page) | [templates/07-adr.md](templates/07-adr.md) |
| Incident retrospectives | [templates/08-postmortem.md](templates/08-postmortem.md) |
| What the on-call does when an alert fires | [templates/10-runbook.md](templates/10-runbook.md) |
| Handing operations of a system, platform, or SaaS to another team and enabling them to run it | [templates/09-operations-handover.md](templates/09-operations-handover.md) |
| Pre-production review | [checklists/production-readiness.md](checklists/production-readiness.md) |
| Final gate before handing over operations | [checklists/operational-readiness.md](checklists/operational-readiness.md) |

The handover template and the operational-readiness checklist are not alternatives. The template is
the plan — who takes what, how the receiving team gets there, and when the handing-over team steps
back. The checklist is the gate you run at the end of that plan. For a small handover where the
receiving team already runs something similar and every task is procedural, skip the template and
run the checklist alone.

Design docs, runbooks and postmortems form a loop. The design doc says what should happen and
names the failure modes. The runbook says what the on-call does when one of them happens. The
postmortem records what actually happened and feeds fixes back into both. Link them to each other.

Worked examples live in [examples/](examples/).

## Usage

```sh
cp templates/01-infrastructure-change.md docs/0007-multi-region-failover.md
```

- Name files `docs/<4-digit sequence>-<kebab-case-title>.md`.
- Sequence numbers are unique across the repository. Never reuse a number, and never fill a gap.
- The `<!-- -->` blocks are authoring instructions. Delete them once the section is written.
- Do not delete a section you cannot fill. Write "N/A" and one line saying why. An empty section and
  "we did not consider this" are indistinguishable to a reviewer, and the difference matters.

## Status and lifecycle

A Design Doc is a proposal when written and an agreement once approved. Move the status field in the
header table as it progresses.

| Status | Meaning | Next step |
| --- | --- | --- |
| Draft | Being written. Shareable, but not agreed | Show it to one or two people as soon as the skeleton exists |
| In Review | Under review. Open a PR and collect comments | Consolidate unsettled points under Open Questions |
| Approved | Agreed. Implementation may start | Link back to this doc from the implementation PR |
| Implemented | Shipped to production | Reconcile the doc with what was actually built |
| Superseded | Replaced by a later doc | Link forward to the successor at the top |
| Abandoned | Decided against | Record why it was dropped. Do not delete the doc |

Review happens in the PR. Keep design arguments in the doc's PR so the implementation PR only has to
answer one question: does this match the approved doc? That keeps code review short.

### Length

Google's convention is 10–20 pages for a substantial design and 1–3 pages for a mini design doc.
You are not expected to fill every section deeply. Write at length where judgment is contested, and
briefly where the answer is obvious.

### When not to write one

- The design has no real choices in it — you are following an established procedure.
- The decision is a two-way door: getting it wrong costs minutes to undo.
- A diagram and the code already say everything you would write.

The inverse is worth stating too: a one-way door — a change that needs a migration or downtime to
reverse — is worth a doc even when it looks small.

## Prior art these templates draw on

The skeleton follows Google's Design Doc structure — Context and scope → Goals / Non-goals →
The actual design → Alternatives considered → Cross-cutting concerns — with the reliability,
operability, security, performance and cost concerns from SRE practice and the Well-Architected
frameworks layered in as the cross-cutting section.

### Writing design docs and RFCs

- [Design Docs at Google](https://www.industrialempathy.com/posts/design-docs-at-google/) — Malte Ubl. The structure these templates are built on, plus the doc lifecycle and when *not* to write one
- [Software Engineering at Google, Chapter 10: Documentation](https://abseil.io/resources/swe-book/html/ch10.html) — treating docs like code: reviewed, owned, and kept fresh
- [Chromium: Design Documents](https://www.chromium.org/developers/design-documents/) — a large corpus of real, public design docs
- [Oxide: RFD 1, Requests for Discussion](https://oxide.computer/blog/rfd-1-requests-for-discussion) — managing document state in the repository itself. [The live RFD](https://rfd.shared.oxide.computer/rfd/0001)
- [Kubernetes Enhancement Proposals (KEP)](https://github.com/kubernetes/enhancements) — strong examples of Goals / Non-goals, risks and mitigations, and graduation criteria
- [Rust RFCs](https://github.com/rust-lang/rfcs) — makes Alternatives and Unresolved questions mandatory sections
- [Scaling Engineering Teams via Writing Things Down](https://blog.pragmaticengineer.com/scaling-engineering-teams-via-writing-things-down-rfcs/) — how RFCs worked at Uber, including the downsides
- [Working Backwards](https://www.allthingsdistributed.com/2006/11/working_backwards.html) — Amazon's PR/FAQ: start from the outcome and reason backwards
- [Google Engineering Practices: Code Review](https://google.github.io/eng-practices/review/) — standards for review depth and turnaround that transfer to doc review

### SRE: reliability, SLOs, postmortems

- [Site Reliability Engineering (the SRE book)](https://sre.google/sre-book/table-of-contents/)
- [The Site Reliability Workbook](https://sre.google/workbook/table-of-contents/)
- [Service Level Objectives](https://sre.google/sre-book/service-level-objectives/) — SLI, SLO, and error budgets
- [Implementing SLOs](https://sre.google/workbook/implementing-slos/) — how to actually pick an SLI
- [Postmortem Culture: Learning from Failure](https://sre.google/sre-book/postmortem-culture/) — blameless postmortems
- [Example Postmortem](https://sre.google/sre-book/example-postmortem/) — a filled-in example
- [Reliable Product Launches at Scale](https://sre.google/sre-book/reliable-product-launches/) — the Launch Coordination checklist behind `checklists/production-readiness.md`
- [Evolving SRE Engagement Model](https://sre.google/sre-book/evolving-sre-engagement-model/) — where the Production Readiness Review (PRR) fits

### Architecture frameworks

- [Google Cloud Architecture Framework](https://cloud.google.com/architecture/framework) — [Reliability](https://cloud.google.com/architecture/framework/reliability) / [Operational excellence](https://cloud.google.com/architecture/framework/operational-excellence)
- [AWS Well-Architected Framework](https://docs.aws.amazon.com/wellarchitected/latest/framework/welcome.html) — the six pillars. [Operational Excellence pillar](https://docs.aws.amazon.com/wellarchitected/latest/operational-excellence-pillar/welcome.html)
- [Azure Well-Architected Framework](https://learn.microsoft.com/en-us/azure/well-architected/)
- [Disaster recovery planning guide](https://cloud.google.com/architecture/dr-scenarios-planning-guide) — how to set RTO and RPO

### On-call and runbooks

- [Effective Troubleshooting](https://sre.google/sre-book/effective-troubleshooting/) — the diagnose / test / treat loop a runbook should support
- [Being On-Call](https://sre.google/sre-book/being-on-call/) — what the person reading a runbook is going through
- [PagerDuty Incident Response documentation](https://response.pagerduty.com/) — a complete, public incident-response process, including communication templates

### Architecture Decision Records

- [Documenting Architecture Decisions](https://cognitect.com/blog/2011/11/15/documenting-architecture-decisions) — Michael Nygard's original ADR post
- [Architecture decision records (Google Cloud)](https://cloud.google.com/architecture/architecture-decision-records)
- [adr.github.io](https://adr.github.io/) / [a collection of ADR templates](https://github.com/joelparkerhenderson/architecture-decision-record)

## License

[MIT](LICENSE). Copy the templates into your own repositories and change whatever does not fit.
