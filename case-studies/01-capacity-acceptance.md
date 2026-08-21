---
layout: default
title: "Capacity acceptance under a fixed hardware envelope"
---

# Capacity acceptance under a fixed hardware envelope

> Written from work at a former employer. The company, product and customers are
> unnamed, capacity figures are rounded or omitted, and no employer code appears
> here. I can go deeper under NDA in conversation.

*Demonstrates: capacity engineering, resource governance, measurement
methodology, root-cause attribution.*

A controlled three-arm experiment - same cohort, same build, same hour, only
the server log level varied - showed a single DEBUG logger inflating p95
signalling latency from about one second to over two. That number was on its
way to being filed as a platform defect. It was our own logging.

I ran that experiment months into the programme. It is the cheapest thing I
did and it invalidated the most work, which is why everything below is
organised around proving the instrument before it is allowed to accuse the
platform.

## Context

A mission-critical push-to-talk platform - 44 microservices on a four-node
on-premise Kubernetes cluster, roughly 28 cores and 100 GiB allocatable. A
customer milestone required signing numeric acceptance criteria: call setup
latency, message delivery, emergency call reach, under a cohort of a few
thousand concurrent sessions.

The hardware was fixed. Not "expensive to grow" - fixed. No node was ever going
to be added.

## Problem

Two things were wrong, and the second was worse.

The obvious problem was that some criteria failed. The real problem was that
nobody could say *why* a criterion failed. Runs disagreed with each other. A
slow number could mean the platform was too slow, or the injector fleet was the
bottleneck, or a service was starved of CPU, or the measurement itself was
wrong - and we had no way to tell those apart. Two defects had already been
filed against application code that turned out to be CPU-starved by its own
cgroup limit.

Without attribution, every result was an opinion, and tuning was guesswork that
consumed the envelope it was trying to fit inside.

## Approach

I treated the envelope as a budget with no overdraft, and the measurement as
something that had to prove itself before it could accuse anything else.

**Zero-sum resource accounting.** Any increase to a request, limit or replica
count had to name the donor workload it took from, and state per-node request
and limit sums before and after. This converted "give this service more CPU"
from an opinion into an arithmetic claim someone could check.

**Effect proof before a limit raise.** Raising a limit on a pod that was not
measurably throttled relieves nothing. So a raise had to carry evidence of
actual CFS throttling - otherwise it was void.

**Five admissible exits for a performance knee.** A knee was not closed until it
resolved as exactly one of: reallocation inside the envelope (naming a donor),
escalation because the envelope is insufficient, a named server-side
bottleneck, an injector-side bottleneck, or a reduction in what the service consumes.
"It's slow" was not an exit.

**Measure the instrument first.** The log-level experiment above made measuring
at WARN a rule, and the rule generalised: I measured the CPU cost of each
metrics endpoint per call and budgeted scrape cadence against it, so profiling
could not perturb what it was profiling.

### Alternatives I rejected

**Buying headroom.** Unavailable by constraint, which is what forced the
discipline. In hindsight the constraint improved the engineering.

**Trusting the convenient metrics.** `kubectl top` read 72.6% on a container
that was at 98.8% of its cgroup limit with over 56,000 throttle events. A
Spring-exposed CPU gauge normalised by four JVM-visible cores against a
two-core limit and under-read by half. A thread-pool utilisation metric reading 52% described a pool with 102 of its
103 threads parked at 2.5 concurrent requests. I read from cgroup counters instead and
differenced throttling over the run window rather than reading a cumulative
ratio - a running cluster's counters carry history from before the run started.

**Averaging percentiles across injectors.** Mathematically wrong, and it
mattered: taking the maximum per-injector p99 gave a materially lower number
than the true pooled value. I emitted per-event Prometheus `le`-bucket
histograms so percentiles merged correctly across injectors, and validated the
synchronised-burst barrier at 0.0 ms release skew.

## Outcome

- **97 measurement reports**, written to a self-imposed template:
  conditions, evidence, attribution, reproduction steps, honest caveats.
- **A machine-linted acceptance ledger** mapping client requirements to
  measurable criteria to verification rows, with a linter enforcing ten
  hygiene rules so the document could not rot.
- **A Go load-injection harness** of 21 scenario packages, with per-scenario latency
  targets derived from 3GPP TS 22.179, driving a multi-host injector fleet.
- Performance knees were attributed to a specific resource, layer or defect
  **without re-running them**, because every run captured CPU against cgroup
  limit, throttling, memory events, GC and thread state as a matter of course.
- At least one false platform defect was prevented, and several real ones were
  filed with source-level evidence instead of a screenshot.

The honest limit: this was measured on internal stands, not in production under
real customer traffic.

## What I would do differently

**Enforce the methodology in code, not prose.** The rules lived in documents
that I followed. When I checked, only 51 of the 97 reports carried the explicit
attribution section the rule required - nothing mechanically failed a report
that omitted it. The ledger linter proved the pattern works; I should have
extended it to the reports on day one.

**Run the observer-effect experiment first, not months in.** Every measurement
taken before it is suspect, and I had to go back and reclassify results. The
cheapest experiment in the whole programme was the one that invalidated the
most work, and I ran it late.

**Say "SLO" out loud.** The ledger is functionally a set of service level
objectives with error budgets and named owners. Calling it an acceptance ledger
made it harder for anyone outside the project to recognise what it was.
