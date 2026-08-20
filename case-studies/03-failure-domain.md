---
layout: default
title: "Making a failure domain enforceable"
---

# Making a failure domain enforceable

> Written from work at a former employer. The company, product and customers are
> unnamed, capacity figures are rounded or omitted, and no employer code appears
> here. I can go deeper under NDA in conversation.

*Demonstrates: distributed-systems reasoning, Kubernetes scheduling internals,
storage operations, failure-mode analysis.*

## Context

The high-availability deployment ran quorum-based stateful services — a MySQL
cluster with an orchestrator, Redis with Sentinel, a message broker, a
service-discovery tier — across three nodes plus an arbiter that existed to
break ties, not to carry load.

Customers installed this themselves, on their own hardware, in one to three
datacentre rooms.

## Problem

High availability that is not enforced is theatre.

Every one of these services survives one node failing *only if* its replicas are
actually on different nodes. Kubernetes will not guarantee that unless you tell
it to, and the default way of telling it to — a soft topology-spread preference
— degrades silently. Under scheduling pressure it co-locates replicas and
reports success. The cluster looks healthy, the dashboard is green, and the
failure domain has quietly collapsed to one node.

The mirror-image risk was the arbiter. It has no capacity to spare. A data-plane
pod scheduled onto it consumes the resources the tie-breakers need, and the
first node failure then takes the quorum with it.

## Approach

**Make the absence of a permission the policy.** I tainted the arbiter and gave
the toleration to nothing except genuine quorum members — the third database
replica, the orchestrator, Sentinel, service discovery. Every other workload in
the chart is kept off the arbiter by *not* being told it may go there. There is
no list to maintain and no rule to forget; a new service is excluded by default,
which is the correct default.

**Enforce it at three layers, because one is a suggestion.**

1. *Render time.* The chart refuses to template at all if quorum sizing is
   invalid — fewer than three database replicas in HA, an orchestrator below
   three, a broker or cache below two.
2. *Scheduling.* The topology-spread helper defaults to a hard host constraint.
   A replica that cannot find its own node goes `Pending` and stays there. That
   was a deliberate choice: a loud failure at install time beats a silent
   co-location discovered during an outage.
3. *CI.* The manifest validator asserts the negative — that no non-quorum
   workload carries the arbiter toleration. Positive checks catch what you
   forgot to add; the negative check catches what someone else added.

**Verify against a running cluster, not only against the templates.** I wrote a
live-cluster audit play that asserts the same invariants against a running
system, gated behind Ansible's `never` tag so it cannot run as part of an
install and block one.

### The trap that justified all of it

Storage replicas are zone-aware, and whether the arbiter joins the storage pool
is derived from the inventory's zone topology rather than declared per install.
Getting there surfaced a genuinely subtle failure: the storage layer defers
tolerance changes while volumes are attached. A node therefore "joins" the
storage pool, reports itself joined, and silently never hosts a replica — the
exact class of bug that HA theatre is made of. I added a two-stage convergence
guard that waits for the engine image to be deployed on every participating node
before proceeding.

### Alternatives I rejected

**Soft topology spread.** It is the friendlier default and it is why this
problem exists. I would rather an operator call me because a pod is `Pending`
than have a customer discover during a node failure that both replicas were on
the failed node.

**A one-shot promotion rule** for restoring the preferred database primary. I
wrote one, and it stopped working after an hour. Reading the orchestrator's
source explained why: it expires promotion candidates older than sixty minutes.
The rule was not wrong, it was being reclaimed. I replaced it with a CronJob
that reasserts it.

## Outcome

- Node-failure recovery went from **over seven minutes to roughly 105 seconds**,
  by tuning three settings as one causal chain — the controller's node-monitor
  grace period, the per-service unreachable toleration, and the storage layer's
  pod-deletion policy. I documented the timeline and, more usefully, the
  constraint the change does *not* remove.
- The failure domain became something a reviewer could check rather than
  something I asserted.

Honest limit: this was validated by deliberately failing nodes on internal
stands. I never watched it survive an unplanned production outage.

## What I would do differently

**Express the invariants in a policy engine.** Kyverno or OPA Gatekeeper is the
right home for "no non-quorum workload tolerates the arbiter taint". I put it in
Python because the validator already existed, which meant the rule only ran
where my pipeline ran.

**Test the recovery path continuously, not once.** The 105-second figure was
measured after the tuning and then trusted. A periodic automated node-kill would
have told me whether it stayed true as the platform grew.
