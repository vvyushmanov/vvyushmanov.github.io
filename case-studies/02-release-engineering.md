---
layout: default
title: "One chart, 44 workloads, 18 releases in 14 weeks"
---

# One chart, 44 workloads, 18 releases in 14 weeks

> Written from work at a former employer. The company, product and customers are
> unnamed, capacity figures are rounded or omitted, and no employer code appears
> here. I can go deeper under NDA in conversation.

*Demonstrates: release engineering, CI/CD pipeline design, Helm at scale,
supply-chain provenance.*

## Context

One Helm chart deployed an entire mission-critical communications platform: 44
workloads across three language runtimes, rendering 185 Kubernetes objects
(StatefulSets with ordered rollout and canary partitions, a host-network
DaemonSet, Jobs, CronJobs, NetworkPolicies, PriorityClasses, PodDisruptionBudgets),
plus four vendored subcharts for the data layer, one of them an in-house
patched fork of an upstream operator chart.

It shipped to customer-owned hardware. There was no rolling back a bad release
by clicking something in a console, and no hotfixing a cluster I could not
reach.

I was the only engineer on it: the chart's commit history is single-author
but for one version bump.

## Problem

Two failure modes, both cheap to create and expensive to discover.

**Chart-wide changes were 44-file mechanical edits.** Adding a probe, changing a
log level default, altering how replicas are derived - each meant touching every
service template, with a per-file chance of divergence that nothing would catch
until a customer hit it.

**Nothing verified that a rendered manifest was internally consistent.** Helm
will happily render a pod that mounts a Secret key which does not exist. That
error surfaces at apply time on someone else's cluster, or worse, as a pod that
starts and misbehaves.

## Approach

**A template helper library instead of copy-paste.** Fifty named templates,
each encoding an explicit precedence chain rather than a
literal - per-service value, then a global default, then a hard fallback.
Adoption is the evidence it worked: the most-used helper has 114 call sites.
Chart-wide changes became single-helper edits.

**A post-render validator wired in as a release gate.** A Python tool that
renders the chart and cross-checks what Helm cannot: that every
`configMapKeyRef` and `secretKeyRef` resolves to a key the chart actually
creates. The interesting part is that some keys are created imperatively inside
`kubectl create` commands in Job containers, so the validator parses those keys
out of shell heredocs in the command text. It also asserts the HA placement
contract, including a negative check that no non-quorum workload tolerates the
arbiter node's taint.

**A release pipeline that fails closed.** Tag-driven: a regex validates the
release tag; unrelated project tags are soft-skipped rather than failing the
build; the pipeline **hard-fails if the chart's declared version disagrees with
the git tag**; the validator runs in strict HA mode; then an OCI push. The
downstream bundle repository is auto-bumped only when a version comparison
proves the version actually increased. A pre-push hook blocks development
version suffixes from reaching the main branch.

### Alternatives I rejected

**Splitting into 44 subcharts.** It is the textbook answer for a platform this
size, and it would have replaced one versioning problem with 44. The helper
library achieved the same de-duplication without giving every service its own
release cadence to keep in step.

**Round-tripping the values file through a YAML library** for the automated
tag-pinning tool. A round-trip discards comments, and in this values file - six
thousand lines, over a thousand comments - the comments *are* the customer's
documentation. I wrote an indentation-aware line scanner instead - uglier, and
correct.

## Outcome

- **18 tagged releases in 14 weeks** through the gate chain, with no placement
  regression published.
- A hand-maintained JSON Schema catching malformed values before render.
- Generated documentation enforced by a pre-commit hook that degrades to a
  warning rather than blocking a commit.

The pipeline work that actually consumed the time was not the design. It was
three defects that only appear in real CI: containers leaving root-owned files
in the workspace so the next checkout failed; a Docker plugin tokenising an
argument string without a shell, so quoted arguments arrived split; and a
transfer client hanging forever rather than erroring when it ran as a UID with
no password entry. None of these are in any tutorial.

## What I would do differently

**Test the infrastructure code the way I demanded the platform be tested.** This
is the sharpest criticism of the work and I agree with it. There were no chart
unit tests, no golden-file render snapshots, and the pipeline was tag-triggered
only - a pull request ran nothing. I wrote a rule set demanding measurement
rigour for the platform and did not hold my own release path to it. Golden-file
renders for a handful of representative templates would have been a weekend.

**Reach for policy-as-code sooner.** I enforced placement invariants in Python
because I already had the validator open. Kyverno or OPA was the right layer,
and the pipeline should have produced a signed artifact and an SBOM.
