# Vadim Iushmanov (Yushmanov)

**Platform Engineer / DevOps Engineer**: Kubernetes, CI/CD, and the reliability
work that keeps a platform standing after it ships.

## Skills

- **CI/CD & release**: Jenkins · GitLab CI · Helm · ArgoCD (GitOps) · Docker · OCI registries
- **Observability & SLOs**: Prometheus · Grafana · Alertmanager · EFK / Fluentd · service level objectives · capacity planning
- **Kubernetes & containers**: Kubernetes · K3s · Rancher · OpenShift · StatefulSets · taints and tolerations · topology spread
- **Infrastructure as code**: Terraform · Ansible · Python · Go · Bash · Linux (Debian, RHEL)
- **Stateful services & storage**: Percona MySQL · Galera · Redis Sentinel · RabbitMQ · MinIO · Longhorn · CSI
- **Networking & real-time media**: SIP · RTP · SIPp · OpenSIPS · Wireshark · packet capture

[SKILLS.md](SKILLS.md) grades every one of these - built, used or familiar - so
a strong claim stays legible as a strong claim.

## Verifiable without taking my word for it

- **[OT-CONTAINER-KIT/redis-operator #1684](https://github.com/OT-CONTAINER-KIT/redis-operator/pull/1684)**:
  a 1.4k-star Kubernetes operator's Helm chart did not template a field its
  own CRD already supported, forcing users into a post-install `kubectl patch`
  that races the reconciler. I filed
  [the issue](https://github.com/OT-CONTAINER-KIT/redis-operator/issues/1683),
  sent the fix, a maintainer merged it.
- **[devops-netology](https://github.com/vvyushmanov/devops-netology)**: 239 of
  239 commits mine. Seven Terraform roots provisioning multi-AZ networking,
  compute, load balancing and IAM bindings, remote state in Terraform Cloud, AWS
  EC2 through the official modules, then kubespray, qbec/jsonnet and a
  monitoring stack above them.
- **[concert-tracker](https://github.com/vvyushmanov/concert-tracker)**: the most
  recent thing I built end to end: a three-stage Dockerfile dropping to a
  non-root user, compose gated on a database healthcheck, 16 schema migrations.
  When Cloudflare Turnstile killed the scraper, the crawl moved into an Electron
  agent holding a persistent browser profile.
- **EF SET English Certificate - 86/100 (C2).**

## Case studies

1. [Capacity acceptance under a fixed hardware envelope](case-studies/01-capacity-acceptance.md):
   the observability tooling was manufacturing the KPI breach: one DEBUG
   logger roughly doubled p95 signalling latency. Resource governance where
   every increase names a donor.
2. [One chart, 44 workloads, 18 releases in 14 weeks](case-studies/02-release-engineering.md):
   release engineering for software installed on hardware I could not reach,
   behind a gate chain that fails closed.
3. [Making a failure domain enforceable](case-studies/03-failure-domain.md):
   node-failure recovery from over 7 minutes to about 105 seconds, and quorum
   placement enforced at three layers rather than asserted.

## Decisions, with the alternative rejected

Eight of them in [decision-records.md](decision-records.md) - context, decision,
alternatives, consequences. Among them: why a replica goes `Pending` rather than
co-locate with the one it exists to survive; why a promotion rule that stopped
working after exactly one hour became a CronJob rather than a patched fork; and
what is lost by dropping histogram buckets at the remote-write boundary, written
into the config so nobody rediscovers it as a mystery.

## What the work required

Running Kubernetes without a managed control plane means owning the primitives
directly: CSI storage and replica placement, quorum placement across failure
domains, etcd survival on node loss, in-place upgrades on a system that cannot
go down. A managed control plane hides that work; it does not remove it, which
is why the reasoning carries onto a platform someone else operates.

I came to platform engineering from the test side, which is why the capacity
programme validates its own instruments before it accuses the platform.

## The constraint

The source is a former employer's property and is not published; the case
studies are architecture and outcomes write-ups. Company, product and customers
are unnamed, capacity figures rounded or omitted, no employer code anywhere.

## Contact

[vadim.yushmanov@gmail.com](mailto:vadim.yushmanov@gmail.com) ·
[LinkedIn](https://www.linkedin.com/in/vadim-iushmanov) ·
[GitHub](https://github.com/vvyushmanov)
