---
layout: default
title: "Skills"
---

# Skills

Everything below is backed by commits, files or public repositories; where the
evidence is a private employer repo, the claim is stated at the scope the
evidence supports. Three grades are used honestly: **built** (designed or owned
it), **used** (worked with it substantially in production), **familiar**
(touched it, shallow — listed so the strong claims stay credible).

## The arc that ties it together

I inherited a docker-compose platform orchestrated by Rancher 1.6 Cattle, became
the maintainer of its Ansible installer and of its compose stack as it grew to
47 services, and carried the platform to Kubernetes: first a full
vanilla-manifest study of the platform, then a Helm chart, then a 15-role K3s
installer and an ArgoCD GitOps delivery chain that I wired into every service's
release pipeline. At my departure the cutover was
live on internal environments and the first customer bundles; the legacy
installer was still mine to fix, and I shipped fixes to both generations to the
end.

## Kubernetes and platform

- **Kubernetes (K3s, Rancher)** — built: production clusters stood up by my
  installer; scheduling design (taints, tolerations, topology spread,
  priority classes), StatefulSet ordered rollouts with canary partitions,
  host-network workloads for RTP media, NetworkPolicies, node lifecycle and
  scaling runbooks. Used: descheduler tuned for post-recovery rebalancing.
- **Helm** — built: the production umbrella chart deploying 44 first-party
  services, a helper library that turns chart-wide changes into single edits,
  a hand-maintained values schema, 18 tagged releases. A small public chart of mine predates it.
- **OpenShift** — used: operated a telecom customer's deployment for a year
  (`oc`-driven config, Redis Sentinel wiring, a bulk image-roller with dry-run
  and rollout waits). To investigate an inter-cluster networking problem I
  installed two multi-node clusters in my own lab, mirroring the customer's
  topology, and worked the joining layer there: Multus macvlan and bridge
  attachments, MachineConfig static routes, and an OVN-Kubernetes hybrid-overlay
  spanning both site ranges.
- **Rancher 1.6 Cattle** — used, to end-of-life: drove its REST API from
  Ansible, maintained the 47-service compose template, forced cgroups v1 on
  Debian 12 and rebuilt the Cattle server and agent images from a fork, to keep
  it alive while the replacement was built.
- **ArgoCD / GitOps** — built: the two-file-per-environment values contract, a
  multi-source Application combining an OCI chart with git values and
  vault-injected secrets, and the CI step that writes image tags into the
  delivery repo (~900 automated commits to date).
- **Kubespray** — used: wrote the inventory and bring-up runbook for a
  seven-node staging cluster in February 2025 — three-node control plane and
  etcd quorum, containerd, Calico, IPVS, secret encryption at rest, MetalLB
  layer 2, Rancher pinned to a labelled and tainted node — and committed both
  to the platform repository the following month.
- **kubeadm, CNI, containerd** — familiar, personal lab: a three-node
  Vagrant/VirtualBox cluster built from scratch on Rocky Linux 9.

## Infrastructure as code

- **Ansible** — built the K3s installer and maintained the one it replaced, a
  colleague-led installer derived from a public Rancher HA playbook set. The
  current one: 15 roles, a preflight of ~20 assertions that fails before
  anything is changed, vault-encrypted secrets, and a deployment summary that
  hands the operator URLs, credentials and next steps. The legacy one: sole
  author of its HA data-tier role (Galera + GARB + HAProxy + RabbitMQ + Redis
  Sentinel) and a generator that emits versioned,
  self-contained upgrade playbooks. The installer ships documented: 14
  operator documents — day-2 operations, upgrade, backup and DR, scaling,
  certificate rotation, customer onboarding — written in-repo and published
  by a CI job to Confluence and into every customer bundle, so neither the
  wiki nor the shipped docs drift from the code. Two small public roles and a
  published collection exist as linkable samples.
- **Terraform** — built, on personal work, and larger than one project:
  seven authored roots in my public coursework repository provisioning
  multi-AZ networking, compute, load balancing and IAM bindings on Yandex
  Cloud; AWS EC2 exercises through `hashicorp/aws` and the official
  `terraform-aws-modules/ec2-instance/aws` module; and a separate exercise
  keeping remote state in Terraform Cloud.
- **Rex (Perl)** — used, inherited: took over a separate legacy product's
  deployment automation, fixed its bugs and carried it from Debian 10 to
  Debian 12. Mine within it: Percona XtraDB Cluster, MongoDB and keepalived.
  I did not write it.
- **Vagrant** — used: the installer's local multi-VM test rig and the lab.

## CI/CD and release engineering

- **Jenkins** — built: 18 declarative pipelines from scratch and 92 modified
  across 8 repositories — a 10-stage multi-customer installer pipeline with
  per-customer failure isolation, tag-driven Helm and Ansible release trains
  that fail closed on a version/tag disagreement, an Electron cross-build in a
  date-pinned Docker image, and the shared-library step that bridges every
  service build into the GitOps repo.
- **GitLab CI** — built: a tag-driven release pipeline that cross-builds Windows
  under Wine, renders an NSIS installer from a template, builds a Linux AppImage
  inside a container image I published, and cuts a GitLab Release with
  per-platform assets. A macOS DMG job on a dedicated runner ran from December
  2023 until I disabled it in May 2024. Ran across 35 tagged versions. A public
  snapshot of it is on my GitHub.
- **Docker / Compose** — used daily across both platform generations; images,
  multi-stage builds, a purpose-built pipeline builder image.
- **Release discipline** — built: machine-checked Ansible-to-Helm
  compatibility matrix with a hand-rolled SemVer range matcher, fail-closed
  ship allowlists, provenance manifests with content hashes, changelogs kept
  release over release.
- **Integrations** — built: CI notifications into MS Teams, per-customer Jira
  comments posted by the bundle pipeline (build goes UNSTABLE if the comment
  fails), a docs-to-Confluence sync pipeline, GitLab tag-push webhooks into
  Jenkins, and cross-cluster alert forwarding into a central Alertmanager.

## Observability

- **Prometheus and Grafana** — built, from scratch, docker-compose era:
  Prometheus, Grafana, Alertmanager, cAdvisor and Node Exporter, provisioned as
  code with eight alert rules of my own; five of the seven dashboards are
  community imports, the SIP one substantially rewritten. Later, on Kubernetes:
  remote-write cardinality control (dropping histogram buckets at the edge with
  the loss stated), an agent-mode revert with the cost documented, TLS
  certificate-expiry probing via blackbox-exporter.
- **Logging (EFK, Fluentd, Fluent Bit)** — used, on a stack a colleague built
  before I joined. Mine within it: the aggregator buffer moved from memory to
  disk after data loss under load, the overflow action set to drop the oldest
  chunk, an index template installed at startup, Elasticsearch tuning, the
  image migration to the internal registry, and the TLS reverse proxy fronting
  Kibana.
- **JVM observability** — used deeply: Micrometer/actuator metrics consumed by
  a profiling stack I wrote — GC and thread-state capture, cgroup-counter CPU
  accounting, CFS throttling differenced over a run window, and a documented
  list of metrics deliberately not collected because they mislead.

## Data layer

- **MySQL (Percona, Galera/XtraDB)** — built the legacy HA data tier (Galera
  cluster with an arbitrator, HAProxy balancing, health checks with response
  caching) and ran the Kubernetes-era Percona operator deployment; wrote the
  promotion guard that re-asserts the preferred primary because the
  orchestrator expires candidates hourly; pre-upgrade dumps wired into the
  upgrade path. Query-digest analysis when a stall demanded it.
- **Redis (Sentinel)** — built: a watchdog sidecar in the legacy stack; four
  fixes to a public Kubernetes Redis operator's sentinel handling, one merged
  upstream.
- **RabbitMQ** — used: quorum topology, memory-watermark tuning verified under
  load, health checks, monitoring dashboards.
- **MongoDB** — used: added replica-set-with-arbiter deployment to a legacy
  product's installer.
- **MinIO / S3** — built the object-storage roles in both installers, and four
  bash harnesses that failure-test a distributed deployment: node kills,
  erasure-coding limits against an arbiter disk, two-site replication, and
  service-account policy enforcement per site.
- **PostgreSQL** — familiar: querying and application-side configuration.
- **Liquibase** — familiar: operational handling of stuck migration locks.

## Networking, VoIP and security

- **SIP / RTP** — a career-long thread: Tier I/II VoIP support, then SIP load
  scenarios (SIPp) against OpenSIPS nodes I helped configure, then packet
  analysis (Wireshark, tcpdump, sngrep, opusrtp) used in production
  root-cause work on a 3GPP MCPTT platform, then a data race found and fixed
  in an open-source Go SIP library.
- **TURN/STUN (coturn)** — used: high-availability TURN configuration and
  health checks for host-network media; in the Helm chart the TURN deployment
  is mine, including an auto-discovery DaemonSet that harvests each node's
  external address.
- **TLS lifecycle** — built: certificate rotation automated in both
  generations (preparation, pre-flight checks, post-deploy verification), and
  a PKCS#12 keystore preflight that turns three classes of bad customer
  keystore into named build-time errors.
- **Reverse proxies** — used: NGINX (TLS, WebSocket upgrade) and HAProxy.
- **Secrets** — used: ansible-vault throughout, per-installation generated
  credentials, vault-sourced values kept out of the GitOps repo.

## Languages

- **Go** — the load-injection harness behind capacity acceptance: 21 scenario
  packages, a cross-injector barrier, histogram-correct percentile merging.
  Also operator reconcile-loop fixes and SDK protocol work. Tests included: table tests,
  contract tests, the race detector used in anger.
- **Python** — the chart's manifest validator, release tooling, ledger
  linter, analysis scripts, and a public GUI utility with a cross-platform CI
  build.
- **Bash** — installers, capture tooling, operational scripts across every
  repo; maintained a large legacy build dispatcher I did not write.
- **TypeScript / React / Electron** — a complete internal desktop tool built
  solo: main/renderer process isolation, a three-tier test suite, shipped in
  tagged releases.
- **Java / Kotlin** — diagnosis-led production fixes in Spring Boot services
  (a whole-service deadlock read out of thread dumps; a replica-local state
  bug blocking horizontal scaling), and a QA-era test framework.
- **Groovy** — the Jenkins pipelines above. **Jinja2** — both installers'
  templating. **SQL** — schema init, digest analysis. **HCL** — the diploma.

## Reliability practice

- Capacity acceptance run as an engineering programme: fixed hardware
  envelope, zero-sum resource accounting, five admissible exits for a
  performance knee, KPIs measured only at WARN after proving DEBUG logging
  doubled the observed p95.
- 97 measurement reports; a machine-linted acceptance ledger that functions
  as an SLO document; ~110 defects filed with source-level evidence.
- Incident practice: production RCAs from thread dumps, packet captures and
  log correlation; a post-mortem written after a Rancher control-plane
  incident took management of every service offline.

## Earlier: QA, then test infrastructure

Five years before platform work, and the bridge into it. A Kotlin API/SSE test
framework I founded and colleagues adopted after a handover session (JUnit 5,
RestAssured, Allure, an SSE client written by hand); a separate Java UI suite on
Selenide that I contributed to rather than owned; manual testing across mobile,
desktop and web; VoIP support to Tier II; and SIP/HTTP traffic analysis.

The bridge was infrastructure for tests rather than tests themselves: a Jenkins
pipeline that gates every merge request — running the suite on each push,
writing the pass/fail state back to the request and posting the report link into
it; a listener that pulls the browser-session recording of a failed UI test into
the report, replacing a manual download step; the migration of that shared suite
from Java 8 to 17; and an Appium pilot on a dedicated agent that fetched
each new build over FTPS. Pipelines for tests came first, pipelines for
everything came after. Later, from the platform side, I moved the company's
test-case corpus off TestRail into Allure TestOps — the custom-field mapping
written by hand, the upload wired into both suites' pipelines. This is also why
I measure before I claim: I tested systems for a living before I built them.

## Public artifacts

- A Kubernetes operator fix merged upstream (1.4k-star project), linked on the
  landing page.
- The diploma project (Terraform, kubespray, qbec/jsonnet, monitoring), a
  small Helm chart, two Ansible roles and a collection, and the GUI utility
  with its release pipeline — all under my GitHub.
- `concert-tracker`, the most recent thing I have built end to end and the only
  one of these that is not infrastructure: a self-hosted app that tracks
  upcoming concerts for a set of artists, giving date, city, venue and map
  position, with cities inside 35 km grouped into one metro area. Artists come
  from a list you curate or, optionally, from a Last.fm account. Its scraper is
  an Electron desktop agent rather than a server job, because concerts-metal put
  Cloudflare Turnstile in front of the listings and a persistent browser profile
  was the only thing that held the clearance. 146 commits over Oct 2025 to
  Jun 2026, all mine. Linked on the landing page.

## What I would fix first

Both of these are already the *what I would do differently* sections of the case
studies, which is the honest place for them.

**Policy as code.** The placement invariants that keep quorum services off the
same node live in a Python validator because I already had the validator open.
Kyverno or OPA Gatekeeper is where they belong — in the API server, protecting
everything, rather than only what my pipeline renders.

**Infrastructure code tested the way I demanded the platform be tested.**
Molecule runs against my own public Ansible roles under a pipeline that executes
it; the production installers and the umbrella chart never got the same
treatment, and golden-file render snapshots for a handful of representative
templates would have been a weekend's work.
