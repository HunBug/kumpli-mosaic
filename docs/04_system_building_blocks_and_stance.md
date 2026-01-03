Personal Memory Processing & Sandbox System
System Building Blocks and Stance (Core vs Plug-in vs Later)

CORE-GRADE (we commit; this becomes part of the system’s spine)

1. Kubernetes distribution (runtime substrate): k3s
   What it does: lightweight Kubernetes for homelab / LAN clusters; runs your core services + workers + UI in a standardized way.
   Why core-grade: you explicitly want to learn Kubernetes and build “above” it. It becomes the common deployment and scaling layer.
   Pros: one mental model for everything; multi-node + GPU worker placement later without redesign.
   Cons: ops overhead; you’ll need discipline to keep it small.
   License: Apache-2.0. ([GitHub][1])

2. Workflow orchestration (durable runs): Temporal (+ .NET SDK)
   What it does: durable long-running workflows with retries, resumability, state, and clean failure semantics. Perfect for multi-day archive processing and “partial results now”.
   Why core-grade: it matches your core contract needs (runs, lineage, progress, partial visibility, cancel safely).
   Pros: strong durability; .NET SDK fits your C#-core plan.
   Cons: it’s infrastructure (Temporal service + backing DB).
   License: Temporal .NET SDK is MIT; Temporal is described as MIT in their materials. ([GitHub][2])

3. Source-of-truth database: PostgreSQL
   What it does: canonical catalog of Assets, Runs, Artifacts, lineage, status/coverage, human labels, and cross-links.
   Why core-grade: this is your “truth layer.” Everything else (search, dashboards, vector stores) can be rebuilt from it.
   Pros: reliable, mature, flexible.
   Cons: schema discipline matters or it becomes a junk drawer.
   License: PostgreSQL License (permissive). ([PostgreSQL][3])

4. Event bus for streaming progress + decoupling: NATS
   What it does: pub/sub + request/reply for events like “artifact written”, “progress update”, “new assets discovered”, “queue this batch.”
   Why core-grade: enables partial results and UI feedback without tight coupling.
   Pros: fast, simple, low ops.
   Cons: you must define message schemas and versioning (but that’s good discipline).
   License: Apache-2.0. ([NATS.io][4])

5. Observability baseline: Prometheus + OpenTelemetry Collector
   What it does: metrics (Prometheus) + unified logs/traces/metrics pipelines (OTel collector).
   Why core-grade: your “don’t melt my RAM/disk/CPU” and “recover from failures” requirement needs visibility from day one.
   Pros: standard ecosystem; works with anything.
   Cons: requires deciding what to measure.
   Licenses: Apache-2.0 (Prometheus, OTel collector). ([Prometheus][5])

6. Ingress + TLS automation (LAN-first, future-proof): ingress-nginx + cert-manager
   What it does: stable routing to your APIs/UI inside the LAN; cert-manager automates TLS issuance/rotation (even for internal CA patterns).
   Why core-grade: makes UI/API access sane and consistent as you add modules.
   Pros: standard K8s pattern; reduces “random ports everywhere.”
   Cons: extra components; keep configuration minimal.
   Licenses: Apache-2.0. ([GitHub][6])

7. Derived artifact store interface (not a specific product): “Artifact Storage Abstraction”
   What it does: a core interface so artifacts can live in:

* plain filesystem paths (first milestone), or
* an S3-compatible object API later
  Why core-grade: you want to avoid migration pain; swapping storage should not rewrite the system.
  Hard constraint (document this in bold): Originals stay on your existing filesystem, mounted read-only, never moved, never rearranged, never duplicated.

(Notice: no specific storage product is core-grade. The abstraction is.)

---

PLUG-IN (replaceable; can be swapped without rewriting the core truth)

A) GitOps deployment tooling: Argo CD
What it does: declarative “cluster state = git repo”; reproducible installs and rollbacks.
Why plug-in (not core): extremely useful, but the system can function without it.
Pros: excellent for learning Kubernetes properly; infrastructure becomes documentation.
Cons: another “platform tool” to maintain.
License: Apache-2.0. ([GitHub][7])

B) Event-driven autoscaling: KEDA
What it does: scale workers based on queues/events; can scale-to-zero GPU workers.
Why plug-in: great once you have event patterns; unnecessary day one.
Pros: aligns with resource safety.
Cons: adds complexity; needs good signals.
License: Apache-2.0. ([GitHub][8])

C) Kubernetes-native workflow engines (alternative/adjacent): Argo Workflows, Tekton
What they do: orchestrate container jobs as K8s resources (CRDs).
Why plug-in: you’re committing to Temporal as the workflow brain; these remain optional for specific styles of pipelines.
Pros: very “K8s native.”
Cons: can pull you into CRD/DAG land you said you don’t want as the core model.
Licenses: Apache-2.0. ([Argo Project][9])

D) Search engines (explicitly not core truth): OpenSearch (example candidate)
What it does: full-text search, filtering, aggregations; good for “materialized views” of metadata.
Why plug-in: your truth is Postgres + artifacts; search indexes are disposable.
Pros: powerful queries + facets.
Cons: heavier ops footprint.
License: Apache-2.0. ([OpenSearch][10])

E) Annotation UI: Label Studio
What it does: human labeling/annotation across media types; exports datasets; integrates with ML assistance.
Why plug-in: you want “attachable UIs” that sync into your DB; this is exactly that.
Pros: flexible, mature.
Cons: you still implement the sync/connector conventions.
License: Apache-2.0. ([GitHub][11])

F) Vector storage (if/when you want): pgvector (inside Postgres)
What it does: store embeddings and run similarity search inside your canonical DB.
Why plug-in: you may later try Qdrant/Milvus/etc; keep embeddings interface abstract.
Pros: simplest architecture; good enough for personal-scale.
Cons: dedicated vector DBs can outperform at large scale.
License: PostgreSQL License. ([EDB][12])

G) S3 layer for derived artifacts (optional implementation): MinIO
What it does: S3-compatible API for artifacts without touching your originals.
Why plug-in: your abstraction should allow filesystem-only first; object-store later.
Pros: standard API; nice for distributed workers.
Cons: MinIO is AGPLv3 (and dual-licensed commercially), so you should consciously accept that license even for personal use. ([MinIO][13])

---

LATER / OPTIONAL “PROFESSIONAL OVERKILL” (only if the system earns it)

1. Kubernetes storage fabric across nodes: Longhorn or Rook/Ceph
   What it solves: replicated persistent storage across nodes (useful if artifacts must survive node loss and you distribute storage).
   Why later: it increases ops complexity a lot; you can start with a single-node artifact volume.
   Longhorn: Apache-2.0. ([GitHub][14])
   Rook/Ceph: Apache-2.0, very powerful, heavier ops. ([Rook][15])

2. Bare-metal load balancer: MetalLB
   What it solves: getting “LoadBalancer services” on a homelab cluster.
   Why later: only needed depending on how you expose services on your LAN.
   License: Apache-2.0. ([MetalLB][16])

3. Advanced CNI / networking: Cilium
   What it solves: powerful networking + policy + observability.
   Why later: it’s awesome, but unnecessary until you need its features.
   License: user-space components Apache-2.0 (with some eBPF templates dual-licensed). ([GitHub][17])

[1]: https://github.com/k3s-io/k3s?utm_source=chatgpt.com "k3s-io/k3s: Lightweight Kubernetes"
[2]: https://github.com/temporalio/sdk-dotnet?utm_source=chatgpt.com "temporalio/sdk-dotnet: Temporal .NET SDK"
[3]: https://www.postgresql.org/about/licence/?utm_source=chatgpt.com "PostgreSQL: License"
[4]: https://nats.io/about/?utm_source=chatgpt.com "About NATS"
[5]: https://prometheus.io/?utm_source=chatgpt.com "Prometheus - Monitoring system & time series database"
[6]: https://github.com/kubernetes/ingress-nginx?utm_source=chatgpt.com "Ingress NGINX Controller for Kubernetes"
[7]: https://github.com/argoproj/argo-cd?utm_source=chatgpt.com "argoproj/argo-cd: Declarative Continuous Deployment for ..."
[8]: https://github.com/kedacore/keda?utm_source=chatgpt.com "KEDA is a Kubernetes-based Event Driven Autoscaling ..."
[9]: https://argoproj.github.io/workflows/?utm_source=chatgpt.com "Argo Workflows"
[10]: https://opensearch.org/?utm_source=chatgpt.com "OpenSearch: Home"
[11]: https://github.com/HumanSignal/label-studio?utm_source=chatgpt.com "Label Studio is a multi-type data labeling and annotation ..."
[12]: https://www.enterprisedb.com/docs/pg_extensions/pgvector/?utm_source=chatgpt.com "EDB Docs - pgvector"
[13]: https://www.min.io/commercial-license?utm_source=chatgpt.com "MinIO Open Source"
[14]: https://github.com/longhorn/longhorn?utm_source=chatgpt.com "longhorn/longhorn: Cloud-Native distributed storage built ..."
[15]: https://rook.io/?utm_source=chatgpt.com "Rook Ceph"
[16]: https://metallb.io/?utm_source=chatgpt.com "MetalLB :: MetalLB, bare metal load-balancer for Kubernetes"
[17]: https://github.com/cilium/cilium?utm_source=chatgpt.com "cilium/cilium: eBPF-based Networking, Security, and ..."
