---
layout: default
title: "Decision records"
---

# Decision records

> Written from work at a former employer. The company, product and customers
> are unnamed, capacity figures are rounded or omitted, and no employer code
> appears here. I can go deeper under NDA in conversation.

Eight decisions from three years of running a mission-critical communications
platform on on-premise Kubernetes. Each follows the same shape: context,
decision, alternatives considered, consequences. These are the trade-offs I
would defend in an interview.

---

## 1. Replicas go Pending rather than co-locate

**Context.** Quorum services (MySQL, Redis Sentinel, the message broker,
service discovery) survive a node failure only if their replicas actually sit
on different nodes. Kubernetes' soft topology spread degrades silently under
scheduling pressure: it co-locates replicas and reports success.

**Decision.** The chart's spread helper defaults to a hard host constraint.
A replica that cannot find its own node stays Pending, visibly.

**Alternatives considered.** Soft spread (the friendly default) — rejected
because the failure it permits is discovered during an outage, which is the
most expensive possible time. Descheduler-only correction — rejected as the
primary mechanism because it repairs after the fact and proves nothing at
install time; I run it as a supplement for post-recovery rebalancing.

**Consequences.** Install-time failures became loud and cheap: an operator
calls about a Pending pod instead of a customer calling about a lost quorum.
The cost is real — under-provisioned clusters cannot quietly limp — and
deliberate.

## 2. The arbiter's placement policy is the absence of a toleration

**Context.** The HA topology includes an arbiter node that exists to break
ties. It has no capacity to spare; one data-plane pod scheduled onto it can
starve the tie-breakers exactly when they are needed.

**Decision.** Taint the arbiter and give the toleration to nothing except
genuine quorum members. Every other workload — including every future one —
is excluded by default, because exclusion requires no list, no rule and no
memory.

**Alternatives considered.** A node-selector allowlist on the arbiter —
rejected: it must be maintained, and a new service is included in the failure
domain by forgetting it. Admission policy (Kyverno/OPA) — the right long-term
layer; at the time I enforced the same invariant in three places I already
owned: render-time failures on invalid quorum sizing, the hard spread above,
and a CI check asserting the negative — that no non-quorum workload carries
the toleration.

**Consequences.** The failure domain became checkable rather than asserted.
The honest gap: the invariant lives in my pipeline instead of the API server,
so it protects only what my pipeline renders.

## 3. Give every node a memory eviction signal before the kernel takes over

**Context.** The Kubernetes distribution we ship configures disk-only hard
eviction and reserves no memory, and because the kubelet replaces rather than
merges that setting, nodes ran with no memory eviction signal at all and
allocatable equal to capacity. Memory exhaustion was resolved by the kernel
OOM killer, which is entitled to kill the container runtime instead of a pod.

**Decision.** Render an explicit kubelet reservation on every node: memory
reserved for the system and for Kubernetes itself, soft eviction with a grace
window long enough for a service to release its database migration lock, and
a hard floor beneath it.

**Alternatives considered.** Leaving it to the distribution's defaults —
rejected after watching the OOM killer choose the runtime. Sizing
reservations per-node-role — deferred; a uniform backstop was defensible and
shippable.

**Consequences.** Node allocatable shrank by a stated amount on every node,
which on a full cluster is not free — the changelog entry tells operators to
verify headroom before applying. Memory pressure now terminates a pod through
the API, with its shutdown window honoured, instead of terminating the node's
ability to run pods.

## 4. A CronJob that re-asserts, not a fix that assumes it sticks

**Context.** After a database failover, the cluster orchestrator would not
promote the preferred primary back once it recovered. I set the promotion
rule; it worked; an hour later it did not.

**Decision.** Read the orchestrator's source. It expires promotion candidates
older than sixty minutes, so any one-shot rule is silently reclaimed. The fix
became a CronJob that re-asserts the rule on a cadence inside the expiry
window.

**Alternatives considered.** Patching the orchestrator to make the rule
durable — rejected: carrying a fork of a data-path component for a
convenience rule is a bad trade. Documenting a manual re-assert — rejected as
a runbook step that fails exactly when nobody is looking.

**Consequences.** The recovery behaviour became permanent and boring. The
general lesson I kept: when a fix stops working on a timer, the system has an
expiry you have not read about yet.

## 5. An indentation-aware line scanner instead of a YAML round-trip

**Context.** Release tooling needs to rewrite image tags inside a
six-thousand-line values file that carries over a thousand documentation
comments — comments that are the customer-facing reference for every
parameter.

**Decision.** The rewriter is a line scanner that tracks indentation and
touches only the target line. It refuses the obvious implementation — parse,
modify, dump — on purpose.

**Alternatives considered.** A YAML round-trip — rejected because standard
dumpers discard comments, and a comment-preserving library still re-flows
formatting; either way the diff stops being reviewable and the documentation
degrades with every release. Moving tags out of the values file entirely —
the cleaner architecture, but it changes the operator interface for every
customer and was not worth coupling to a tooling change.

**Consequences.** Uglier code, smaller diffs. Every release since has
modified exactly the lines it meant to, and the file's comments have survived
untouched.

## 6. Prometheus server mode, not agent mode, on the edge clusters

**Context.** Internal clusters forward metrics to a central Prometheus.
Agent mode looked ideal: scrape locally, stream centrally, keep nothing.
But agent mode is streaming-only — it does not evaluate recording rules, so
none of the derived series the standard dashboards filter on ever existed.
Imported dashboards rendered No data.

**Decision.** Run full server mode with minimal retention: two hours, a
1 GiB cap, ephemeral storage, long-term history living centrally via the same
remote-write. Rules and alerts evaluate in-cluster.

**Alternatives considered.** Recreating the recording rules centrally —
rejected: it duplicates the entire rule set per cluster and drifts. Rebuilding
every dashboard against raw series — rejected: fighting the upstream
ecosystem forever to save one pod's memory.

**Consequences.** Roughly half a gigabyte more RAM on the monitoring pod,
stated in the commit and within headroom. Dashboards and alerts work as the
ecosystem intends, and the two-hour local window doubles as an in-cluster
debugging buffer.

## 7. Drop histogram buckets at the remote-write boundary, and say what is lost

**Context.** The central Prometheus was buckling — compaction p95 near four
minutes, CPU spikes feeding a cascade into the logging store. Measurement
showed control-plane histogram buckets dominated series cardinality.

**Decision.** Drop the offending bucket series at the edge with a
write-relabel rule. Keep every histogram's sum and count flowing, so mean
latency and request rate remain queryable centrally. Accept, and write down,
that central-side quantiles for those metrics are gone; the edge cluster's
two-hour retention can still compute them during a debugging session.

**Alternatives considered.** Scaling the central server — treats the symptom
and raises the floor forever. Longer scrape intervals — degrades every metric
to save a few. Dropping whole metrics — loses the mean and the rate, which
answer most questions the quantiles would have been asked.

**Consequences.** About forty percent of the central head-series load gone,
measured before and after, per-series counts recorded in the config comment.
The loss is explicit in the file, so nobody rediscovers it as a mystery.

## 8. KPIs are measured at WARN, and the instrument is measured first

**Context.** Latency acceptance numbers drifted between runs with no code
change. A controlled three-arm experiment — same cohort, same build, same
hour, only the server log level varied — showed DEBUG logging on two services
roughly doubling the client-observed p95 and manufacturing a KPI breach that
would otherwise have been filed as a platform defect.

**Decision.** Acceptance KPIs are valid only when measured at WARN. The rule
extends to the instruments themselves: each metrics endpoint's per-call CPU
cost was measured, and scrape cadence budgeted so profiling stays a
fraction of a percent of the resource envelope it observes.

**Alternatives considered.** Trusting the numbers and filing the defect —
this nearly happened, and it is why the rule exists. Measuring at DEBUG "for
more detail" and correcting afterwards — rejected: there is no defensible
correction factor for an instrument that changes the system's timing.

**Consequences.** Earlier DEBUG-tainted results were reclassified rather than
quietly kept. Every subsequent number carries its log level as part of its
conditions, and the cheapest experiment of the programme is the one I now run
first.
