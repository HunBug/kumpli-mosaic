Yep — here’s a reorganized “system + orchestration first” list that you can drop into the kickoff doc. I’ll treat Temporal + Kubernetes as the baseline direction, keep search/index explicitly “plug-in / optional,” and keep the storage story aligned with your hard constraint: **never move/rearrange/copy the original archive**.

SYSTEM / ORCHESTRATION LAYER (the stuff that makes everything else reliable)

Kubernetes as the platform (your “system bus”)

* What it solves: packaging, scheduling, health checks, restart policies, isolation, multi-node, GPU worker placement, rolling updates. It becomes the “universal runtime” for your C# core + worker services + UI.
* Pros: you can scale from “one box” to “a few nodes” without redesigning; makes your system modular by default.
* Cons: learning + ops overhead; you’ll want to be strict about “keep it small” (few namespaces, Helm charts, GitOps).
* Practical flavor: start with a lightweight distro like **k3s** for homelab/edge use (Apache-2.0). ([GitHub][1])

GitOps deployment + “everything reproducible”

* Argo CD: declarative sync from a Git repo to your cluster (Apache-2.0). This is the easiest way to keep your setup reproducible and avoid snowflake clusters. ([GitHub][2])
* Pros: rollbacks are “git revert”; your infra becomes documentation.
* Cons: can be overkill early; but if you *want* to learn Kubernetes properly, it’s the right kind of overkill.

Ingress / routing (so your UI + APIs are reachable on your LAN)

* ingress-nginx (Apache-2.0) or NGINX Ingress (Apache-2.0): routes traffic to your services. ([GitHub][3])
* Traefik (MIT): very nice DX, great for homelab, dynamic config. ([GitHub][4])
* Pros: standard pattern; easy TLS later.
* Cons: another piece to learn; keep it to one controller.

Certificates / TLS automation

* cert-manager (Apache-2.0): certificate issuance/renewal inside the cluster. Even if you’re “LAN-only,” it simplifies internal TLS and local CA patterns. ([cert-manager][5])

Autoscaling + “don’t melt my box”

* KEDA: event-driven autoscaling, including scale-to-zero for workers (great for GPU workers you don’t want burning power). ([KEDA][6])
* Pros: matches your “safe resource handling” requirement.
* Cons: only worth it once you have queues/events; otherwise static replicas are simpler.

WORKFLOW ORCHESTRATION (how you run pipelines durably, with partial results)

Baseline decision: Temporal as the durable workflow engine

* What it solves: durable long-running workflows, retries, resumability, state, timeouts, “progress heartbeats,” and clean failure semantics for multi-day runs.
* Why it fits your needs: your “store results early / see partial outputs / stop if wrong” aligns with Temporal’s model (activities can write artifacts as they go; workflow state tracks progress and lineage).
* Pros: .NET SDK + strong guarantees. ([Temporal][7])
* Cons: it is infrastructure (Temporal server + DB). But that’s exactly what buys you reliability.

Kubernetes-native workflow alternatives (optional, but worth knowing)

* Argo Workflows (Apache-2.0): a Kubernetes-native workflow engine (CRDs) for orchestrating container jobs. ([GitHub][8])

  * Pros: “K8s-first” and very natural if your pipelines are mostly container steps.
  * Cons: you may end up re-implementing some of Temporal’s durability semantics at the app layer (depending on needs).
* Tekton Pipelines (Apache-2.0): very good for CI/CD-ish pipelines; can be used for data jobs too. ([GitHub][9])

  * Pros: strong standardization in K8s land.
  * Cons: can feel like “CI tools repurposed” rather than “personal compute fabric.”

Event triggers (turn “new media arrived” into “run workflows”)

* Argo Events (Apache-2.0): event-driven triggers for Kubernetes objects/workflows. ([GitHub][10])
* How it fits: you can use it as a trigger layer even if Temporal is the workflow brain (e.g., “S3 artifact dropped”, “cron schedule”, “webhook from your scanner”).

MESSAGING / EVENT BUS (connect core ↔ workers ↔ UI; progress streaming)

NATS (Apache-2.0)

* What it solves: lightweight pub/sub + request/reply messaging for events like “asset indexed,” “artifact ready,” “progress update,” “queue work for GPU node.”
* Pros: simple, fast, low-ops; perfect homelab messaging backbone. ([GitHub][11])
* Cons: you still define your own schemas + conventions (but you want that anyway).

Other queues (only if you need them)

* RabbitMQ (MPL 2.0): very mature queueing semantics. ([rabbitmq.com][12])
* Kafka (Apache-2.0): powerful but heavy; usually unnecessary for a single-user LAN system unless you *want* it. ([kafka.apache.org][13])
* Redis: note licensing changes (dual source-available RSALv2/SSPLv1 for newer releases), so I’d avoid it as a “core dependency” unless you pin versions intentionally. ([Redis][14])

OBSERVABILITY / SAFETY RAILS (your “F operational requirement”)

Metrics + alerts

* Prometheus (Apache-2.0): standard metrics collection/alerting. ([GitHub][15])
* Grafana: great dashboards, but it’s AGPLv3 now (fine for personal use; just be aware). ([GitHub][16])

Tracing/log plumbing

* OpenTelemetry Collector (Apache-2.0): vendor-neutral telemetry pipelines; useful if you want one consistent “job run timeline” story across C#, Python, etc. ([GitHub][17])

DERIVED ARTIFACT STORAGE (thumbnails, embeddings, extracted frames, previews, model outputs)
Key constraint you gave: originals stay exactly where they are, mounted read-only.

The clean pattern is:

* Originals: your existing filesystem mount, read-only, never reorganized.
* Derived artifacts: a separate writable store (filesystem directory, object store, or PV), fully managed by your system.

S3-compatible object store (for derived artifacts only)

* MinIO: S3-compatible, high performance; AGPLv3 + commercial dual-licensing (fine for personal; just be aware). ([GitHub][18])
* Why S3 helps even locally: it becomes a “virtual API layer” for artifacts without touching your originals. Your core DB stores URIs like `s3://artifacts/...` while originals remain `file:///mnt/archive/...`.
* Overkill note: you can start with “just the filesystem” for artifacts and add S3 later; your core should abstract it either way.

Kubernetes persistent storage (for the cluster itself)

* Longhorn (Apache-2.0): distributed block storage for Kubernetes PVs; good if you eventually spread across nodes. ([GitHub][19])
* Rook/Ceph (Apache-2.0): very powerful (file/block/object), but heavy ops; “professional overkill.” ([GitHub][20])

SEARCH / INDEX (explicitly plug-in, not part of “core truth”)
Totally aligned with your direction: treat search as replaceable. The core truth stays in your DB + artifact store; search engines are “materialized views.”

When you want to experiment:

* OpenSearch (Apache-2.0): full-text + filtering + aggregations (heavy but solid). ([GitHub][21])
* Vector DB options: keep these as plugins too (you can test multiple without rewriting the core). (I can list candidates later when you want to pick an embeddings strategy.)

HUMAN-IN-THE-LOOP UI TOOLS (annotation, labeling, “name faces,” etc.)
These are perfect “attachable modules,” not core.

* Label Studio (Apache-2.0): flexible annotation UI across modalities; good for your “sync node with 3rd party results/DB” plan. ([GitHub][22])

A tiny “don’t forget” note for your doc (because it’s central to your question about frames/preprocessing)

* Treat “frame extraction / scene detection / sampling” as its own reusable *preprocessing artifact pipeline*, not something hardcoded into every video model pipeline.
* But allow “inline preprocessing” too for quick sandbox runs.
  Temporal + Kubernetes makes both patterns easy: one workflow can call a “precompute frames” activity (shared artifacts), or do it ad-hoc inside a detector run. The key is: artifacts are content-addressed + lineage-tracked either way.


[1]: https://github.com/k3s-io/k3s?utm_source=chatgpt.com "k3s-io/k3s: Lightweight Kubernetes"
[2]: https://github.com/argoproj/argo-cd?utm_source=chatgpt.com "argoproj/argo-cd: Declarative Continuous Deployment for ..."
[3]: https://github.com/kubernetes/ingress-nginx?utm_source=chatgpt.com "Ingress NGINX Controller for Kubernetes"
[4]: https://github.com/traefik/traefik/blob/master/LICENSE.md?utm_source=chatgpt.com "traefik/LICENSE.md at master"
[5]: https://cert-manager.io/?utm_source=chatgpt.com "cert-manager"
[6]: https://keda.sh/?utm_source=chatgpt.com "KEDA | Kubernetes Event-driven Autoscaling"
[7]: https://docs.temporal.io/develop/dotnet?utm_source=chatgpt.com "Net SDK developer guide"
[8]: https://github.com/argoproj/argo-workflows?utm_source=chatgpt.com "argoproj/argo-workflows: Workflow Engine for Kubernetes"
[9]: https://github.com/tektoncd/pipeline?utm_source=chatgpt.com "tektoncd/pipeline: A cloud-native Pipeline resource."
[10]: https://github.com/argoproj/argo-events?utm_source=chatgpt.com "argoproj/argo-events: Event-driven Automation Framework ..."
[11]: https://github.com/nats-io/nats-server?utm_source=chatgpt.com "nats-io/nats-server"
[12]: https://www.rabbitmq.com/?utm_source=chatgpt.com "RabbitMQ: One broker to queue them all | RabbitMQ"
[13]: https://kafka.apache.org/?utm_source=chatgpt.com "Apache Kafka"
[14]: https://redis.io/legal/licenses/?utm_source=chatgpt.com "Licenses"
[15]: https://github.com/prometheus/prometheus?utm_source=chatgpt.com "The Prometheus monitoring system and time series ..."
[16]: https://github.com/grafana/grafana/blob/main/LICENSE?utm_source=chatgpt.com "grafana/LICENSE at main"
[17]: https://github.com/open-telemetry/opentelemetry-collector?utm_source=chatgpt.com "open-telemetry/opentelemetry-collector"
[18]: https://github.com/minio/minio?utm_source=chatgpt.com "MinIO is a high-performance, S3 compatible object store, ..."
[19]: https://github.com/longhorn?utm_source=chatgpt.com "Longhorn.io"
[20]: https://github.com/rook/rook?utm_source=chatgpt.com "rook/rook: Storage Orchestration for Kubernetes"
[21]: https://github.com/rabbitmq/rabbitmq-server/blob/main/LICENSE-MPL-RabbitMQ?utm_source=chatgpt.com "rabbitmq-server/LICENSE-MPL-RabbitMQ at main"
[22]: https://github.com/kedacore/keda?utm_source=chatgpt.com "KEDA is a Kubernetes-based Event Driven Autoscaling ..."
