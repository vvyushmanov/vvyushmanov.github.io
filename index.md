---
layout: default
title: "Vadim Iushmanov (Yushmanov)"
---

# Vadim Iushmanov (Yushmanov)

**Platform Engineer / DevOps Engineer** — Kubernetes on bare metal, and the
CI/CD, storage and reliability work that keeps it standing.

## Verifiable without taking my word for it

- **[OT-CONTAINER-KIT/redis-operator #1684](https://github.com/OT-CONTAINER-KIT/redis-operator/pull/1684)**
  — a 1.4k-star Kubernetes operator's Helm chart did not template a field its
  own CRD already supported, forcing users into a post-install
  `kubectl patch` that races the reconciler. I filed
  [the issue](https://github.com/OT-CONTAINER-KIT/redis-operator/issues/1683),
  sent the fix, and a maintainer merged it.
- **[devops-netology](https://github.com/vvyushmanov/devops-netology)** — 239 of
  239 commits mine. Seven Terraform roots provisioning multi-AZ networking,
  compute and load balancing, with remote state in Terraform Cloud, then
  kubespray, qbec/jsonnet and a monitoring stack above them.
- **[concert-tracker](https://github.com/vvyushmanov/concert-tracker)** — the most
  recent thing I built end to end: a three-stage Dockerfile dropping to a
  non-root user, compose gated on a database healthcheck, 16 schema migrations.
  Cloudflare Turnstile killed the server-side scraper, so the crawl moved into
  an Electron agent holding a persistent browser profile.
- **EF SET English Certificate — 86/100 (C2).**

## Case studies

1. [Capacity acceptance under a fixed hardware envelope](case-studies/01-capacity-acceptance.md)
   — the observability tooling was manufacturing the KPI breach: one DEBUG
   logger roughly doubled p95 signalling latency. Resource governance where
   every increase names a donor.
2. [One chart, 44 workloads, 18 releases in 14 weeks](case-studies/02-release-engineering.md)
   — release engineering for software installed on hardware I could not reach,
   behind a gate chain that fails closed.
3. [Making a failure domain enforceable](case-studies/03-failure-domain.md)
   — node-failure recovery from over 7 minutes to about 105 seconds, and quorum
   placement enforced at three layers rather than asserted.

[decision-records.md](decision-records.md) holds eight decisions, each with the
alternative rejected. [SKILLS.md](SKILLS.md) is the graded tool
inventory, including what I have not used.

## What bare metal made me own

Running Kubernetes without a managed control plane means owning what one hides:
CSI storage and replica placement, quorum placement across failure domains, etcd
survival on node loss, in-place upgrades on a system that cannot go down. I came
to platform engineering from the test side, which is why the capacity programme
validates its own instruments before it accuses the platform.

## The constraint

The source is a former employer's property and is not published; the case
studies are architecture and outcomes write-ups. The company, product and
customers are unnamed, capacity figures are rounded or omitted, and no employer
code appears here.

## Contact

[vadim.yushmanov@gmail.com](mailto:vadim.yushmanov@gmail.com) ·
[LinkedIn](https://www.linkedin.com/in/vadim-iushmanov-62b631148) ·
[GitHub](https://github.com/vvyushmanov)
