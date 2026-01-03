# Personal Memory Processing & Sandbox System

## High-Level Technical Architecture & Early Decisions

### 1. Architectural intent

This system is designed around a simple principle:
the core is a reliable “metadata spine + orchestration brain”, and everything else is replaceable modules.

We want durability, lineage, incremental results, and easy experimentation without building a full platform. We accept some infrastructure (Temporal + DB) to avoid reinventing failure-handling, retries, and long-running workflows.

---

### 2. High-level component map

A. Read-only media archive (source of truth)
A large archive (photos/videos/audio/text) is mounted read-only on the server. Originals are never modified. Workers may access the archive via mount or read-only network share.

B. Core services (C#)
The core is a set of services (may run as one process initially) responsible for:

* Cataloging assets (identity, hashes, technical metadata)
* Managing runs, lineage, and artifacts
* Providing query/search APIs
* Managing collections (saved queries / album manifests / sandbox datasets)
* Coordinating with the workflow engine (Temporal) for pipeline execution
* Enforcing policies (resource caps, retention, sandbox vs production behavior)

C. Workflow engine (Temporal)
Temporal orchestrates long-running pipelines with:

* durable workflow state
* activity retries, timeouts, heartbeats
* cancellation/pause/resume behavior
* fan-out/fan-in patterns across large sets (millions of assets, potentially)

Temporal is not the truth store for results; it orchestrates work. Results live in the core database/blob store.

D. Workers (any language)
Workers execute activities: Python scripts, containers, or native tools. They:

* pull assigned work (via Temporal activity execution)
* read media (mount/share or streaming)
* emit partial and final artifacts early and often
* write logs/metrics/errors for debugging and benchmarking

E. Storage
Two layers:

* A relational DB as the “truth store” (assets, runs, artifacts, lineage, status, collections)
* A blob store for artifact payloads (thumbnails, embeddings arrays, keyframes, transcripts, debug outputs)

F. UI + CLI
Two first-class interaction methods:

* CLI for power users and automation
* Minimal Web UI for browsing/searching results, inspecting lineage, and reviewing/labeling

UI is designed as attachable panels, so experimental tools can be added without redesign.

---

### 3. Core data model (conceptual)

These entities are intentionally stable and form the backbone of extensibility:

Asset
Represents one original file. Stores immutable identity (hash), canonical ID, and technical metadata. Also stores provenance (path history) without assuming paths are stable forever.

Run
Represents one pipeline execution (a workflow instance) with a defined toolchain and parameters. Run is what you can pause/resume/cancel and benchmark. Run also defines “policy” (sandbox vs production).

Artifact
A derived output produced from an asset (or from a group of assets). Examples: thumbnail, transcript, embedding vector, face boxes, face embeddings, cluster membership, OCR text, scene tags, quality scores.

Lineage
Every artifact is linked to the run that produced it, including tool ID, tool version, and parameters. This answers: “why is this tagged like that?”

Coverage / Progress
For every run, store processing status per unit of work (asset or segment). This powers partial visibility and early debugging: you can see and use results while the run is still executing.

Collection
A saved query or a curated set of assets used for albums, sandbox samples, or benchmark suites. Collections may be dynamic (query-based) or static (explicit list).

---

### 4. Pipeline model without a full DAG engine

Pipelines are modeled as “workflow templates” that schedule jobs in predictable chunks, not as a generic DAG graph.

There are three supported pipeline shapes:

1. Per-asset workflows (most common)
   Example: “for each asset: extract metadata → generate thumbnail → compute embedding”.

2. Linear multi-step pipelines
   Example: “video: extract audio → transcribe → diarize → index transcript”.

3. Batch/aggregate workflows
   Example: face clustering across a collection; duplicate detection; “best-of” selection.

This covers most real needs while keeping the system understandable and debuggable.

If branching/complex DAG is needed later, it can be layered on, but the initial design avoids Airflow-style complexity.

---

### 5. Early-results requirement: incremental artifacts, not end-of-run dumps

A core requirement is that long runs produce usable output continuously.

Therefore:

* Work is chunked (per asset, per segment, or per batch), never “one job for everything”
* Activities write artifacts as soon as they’re produced
* Run coverage/progress is updated continuously
* UI and query endpoints show partial results immediately
* The user can cancel a run when early results indicate wrong parameters or tool behavior

Temporal supports this well because activities can heartbeat progress, and workflows can be cancelled at any time. The core DB shows the truth as artifacts arrive.

---

### 6. Preprocessing design: “views” and reusable intermediates

Video (and sometimes audio) processing needs flexible preprocessing (frames, scenes, proxies). The system supports both approaches:

Approach A: separated reusable preprocessing
Frame extraction, scene detection, audio extraction, etc., are first-class pipeline steps producing intermediate artifacts that multiple pipelines can reuse.

Approach B: inline/just-in-time preprocessing
Some tools may extract frames internally with their own parameters. This remains supported for simplicity and sandbox runs.

To unify both, the system uses the idea of a “media view” input contract.

Example: FrameSource
A downstream tool receives a FrameSource rather than “a video file”:

* materialized frames artifact
* generated frames with params (fps, resolution, sampling)
* generated + cached (persisted for reuse depending on run policy)

This avoids hardcoding “video face detection must do frame extraction itself” while still allowing it when convenient.

---

### 7. Run policies: sandbox vs production (a very useful early decision)

Every run has a policy that influences persistence and resource use.

Sandbox policy

* faster sampling (subset of assets, lower resolution, fewer frames)
* ephemeral intermediates allowed (TTL retention)
* quick feedback and early cancellation expected
* outputs are still stored with lineage, but may have shorter retention

Production policy

* full coverage
* stable parameters and tool versions recorded
* intermediates persisted when it helps future reuse
* stronger guarantees around completeness

This one knob makes experimentation safe without polluting storage.

---

### 8. Worker execution model

Workers are language-agnostic.

A worker is defined by:

* capabilities (cpu/gpu, memory class)
* supported tool types (image, video, audio, text)
* max concurrency
* local access mode (archive mounted vs network streaming vs offline bundle)

Workers do not need to know system internals. They only need:

* access to media
* ability to produce artifacts + metadata
* ability to emit logs and progress

Temporal activities are the “execution interface”. The core remains the artifact truth store.

---

### 9. Storage decisions (early, but intentionally minimal)

Truth DB
A relational DB stores assets, runs, artifacts, lineage, collections, and progress. This is the “brain”.

Blob store
Artifacts that are large or binary are stored outside the DB (filesystem or object store). The DB stores pointers and hashes. Content addressing is preferred to deduplicate repeated outputs.

Embeddings / semantic search
Keep it flexible. The initial design assumes embeddings are first-class artifacts and can later be indexed in a vector index. The exact indexing mechanism can be chosen after the core loop is proven.

---

### 10. Minimal UI/CLI scope for the first usable milestone

The first “working system” needs to prove the end-to-end loop:
catalog → run pipeline → see partial results → query → inspect lineage.

Minimum CLI features:

* scan/import archive
* define collection from query
* start a run (sandbox or production)
* watch status/progress
* fetch/export artifacts for a collection
* list failed units and rerun

Minimum UI features:

* search/browse assets (date/type filters initially)
* asset detail page showing derived artifacts
* run dashboard showing progress + failures + cancel button
* lineage view: “this artifact came from this run/tool/params”

This UI is for debugging and trust-building first, not polish.

---

### 11. Early trade-offs and “not decided yet” list

Deliberate early trade-offs:

* Use Temporal to avoid reinventing durability, retries, and cancellation for long workflows.
* Keep pipelines simple and composable rather than building a generic DAG platform.
* Store results early and continuously, even when runs are incomplete.

Intentionally deferred decisions:

* Which specific ML models to adopt first (faces, embeddings, OCR, ASR)
* Whether to use a dedicated vector database vs pgvector vs later migration
* Whether blob storage is filesystem-first or object-store-first
* How advanced the UI becomes (plugin architecture vs single app)
* Whether to implement fine-tuning/training workflows (likely later)

These remain open so the core can be validated with minimal risk.

---

### 12. First milestone definition (to keep the project “results-first”)

A milestone is considered achieved when:

* The archive is cataloged (IDs, hashes, metadata)
* A simple pipeline can run in sandbox mode on a collection
* Results appear incrementally during execution
* The user can browse/search those results
* Lineage for each artifact is visible
* Runs can be cancelled, and failures are inspectable

This proves the system is a usable laboratory, not just scaffolding.
