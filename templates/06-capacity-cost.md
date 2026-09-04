# Capacity / Cost Doc: <target>

<!--
Use for capacity planning, scaling design, and cost optimization.
The core is the demand forecast (section 2) and what hits its limit first (section 4).
-->

| Item | Content |
| --- | --- |
| Status | Draft / In Review / Approved / Implemented |
| Author / Reviewers / Approver | @<github-id> / @<github-id> / @<github-id> |
| Created / Last updated | YYYY-MM-DD / YYYY-MM-DD |
| Target system | |
| Planning period | YYYY-MM to YYYY-MM |
| Current monthly cost | |
| Monthly cost after change (estimate) | |

## TL;DR

## 1. Background and purpose

<!--
State up front which motivation you are writing for. Mixing them blurs the decision criteria.
- Capacity is short, or expected to run short (capacity)
- We are paying too much (cost)
-->

## 2. Demand forecast

### Current actuals

| Metric | Normal | Peak | Peak conditions | Measurement period |
| --- | --- | --- | --- | --- |
| Requests (rps) | | | | |
| Concurrent connections | | | | |
| Data volume | | | | |
| Job / batch volume | | | | |

### Forecast

| Metric | Now | +3 months | +6 months | +12 months | Basis for forecast |
| --- | --- | --- | --- | --- | --- |
| | | | | | |

<!--
Always write the basis: business plan figures, past growth rate, or planned initiatives.
Without it, when the forecast misses you will not know what to revisit.
-->

### How the forecast could miss

| Direction | Possible situation | What to do then |
| --- | --- | --- |
| Higher than expected | | |
| Lower than expected | | |

## 3. Current resources and utilization

| Resource | Spec | Count | Average utilization | Peak utilization | Monthly cost |
| --- | --- | --- | --- | --- | --- |
| | | | | | |

## 4. Limits (what saturates first)

<!--
The limit of the whole system is set by its narrowest part. Before CPU, it is often
connection limits, IOPS, bandwidth, external API rate limits,
pool sizes, or file descriptors that saturate.
-->

| # | Constraint | Current | Limit | Headroom | Expected to hit | Response |
| --- | --- | --- | --- | --- | --- | --- |
| 1 | | | | | | |

### Load test results

| Test | Conditions | Result | Bottleneck | Date |
| --- | --- | --- | --- | --- |
| | | | | |

<!-- If not run, write "Not run". Do not mix estimates with measured values. -->

## 5. Scaling design

| Item | Content |
| --- | --- |
| Scaling method (horizontal / vertical) | |
| Automated or not | |
| Scaling metric and threshold | |
| Min / max instance count | |
| Time to scale out | |
| Handling sudden spikes (warm-up, pre-scaling) | |
| Parts that cannot scale | |

### Preparing for expected peaks

| Event | When | Expected multiplier | Preparation | Decision deadline |
| --- | --- | --- | --- | --- |
| | | | | |

## 6. Cost breakdown

| Item | Current (monthly) | After change (monthly) | Difference | Share | Notes |
| --- | --- | --- | --- | --- | --- |
| Compute | | | | | |
| Database | | | | | |
| Storage | | | | | |
| Network / transfer | | | | | |
| Monitoring / logging | | | | | |
| External services | | | | | |
| Total | | | | 100% | |

## 7. Measures and their effect

<!--
Put the savings next to what they cost (availability, performance, operational load, effort).
Cost reduction always trades something away, so write down what you give up.
-->

| # | Measure | Expected savings (monthly) | What you give up | Effort | Risk | Decision |
| --- | --- | --- | --- | --- | --- | --- |
| 1 | | | | | | |

### Measures not adopted and why

| Measure | Expected savings | Reason not adopted |
| --- | --- | --- |
| | | |

## 8. Implementation plan

| Stage | Content | Due | Owner | How to verify the effect |
| --- | --- | --- | --- | --- |
| | | | | |

## 9. Monitoring and review

| Item | Content |
| --- | --- |
| Monitoring of utilization and remaining headroom | |
| Leading-indicator alert (notice before hitting the limit) | |
| Detection of abnormal cost increases | |
| How often forecast is compared against actuals | |
| When to review this Doc | |

## 10. Risks

| Risk | Impact | Mitigation |
| --- | --- | --- |
| | | |

## 11. References

- [AWS Well-Architected: Cost Optimization](https://docs.aws.amazon.com/wellarchitected/latest/cost-optimization-pillar/welcome.html)
