..# Personal Memory Processing & Sandbox System

## Glossary & Core Contracts (One-Page Reference)

This document defines the **shared language and invariants** of the system.
If these terms stay stable, the system stays understandable and extensible.

No implementation details. No tech stack assumptions.
Only meaning, intent, and boundaries.

---

## 1. Core Concepts (Glossary)

### Asset

An **Asset** represents one original, immutable piece of media.

Examples:

* A photo file
* A video file
* An audio recording
* A text or document file

Key properties:

* Read-only, never modified
* Identified by a canonical Asset ID
* Linked to one or more file paths over time
* Has intrinsic metadata (hash, size, EXIF, duration, codecs, timestamps)

Mental model:

> *“This is something that existed before the system and must never be harmed.”*

---

### Artifact

An **Artifact** is any **derived output** produced from one or more Assets.

Examples:

* Thumbnail
* Video keyframe
* Face bounding boxes
* Face embedding vectors
* Transcripts
* OCR text
* Scene labels
* Quality scores
* Cluster assignments
* Enhanced or cropped preview images

Key properties:

* Immutable once written
* Always linked to:

  * the Asset(s) it came from
  * the Run that produced it
* May be:

  * small structured data (stored in DB)
  * large binary data (stored in blob storage)
* May be persistent or ephemeral (TTL-based)

Mental model:

> *“This is knowledge extracted from an asset.”*

---

### Run

A **Run** represents **one execution of a pipeline or experiment**.

Examples:

* “Face detection v2 on all photos (sandbox)”
* “ASR transcription on Stockholm videos”
* “CLIP embedding benchmark on sample set A”

Key properties:

* Has a unique Run ID
* Defines:

  * tools involved
  * versions
  * parameters
  * execution policy (sandbox vs production)
* Can be:

  * running
  * paused
  * cancelled
  * completed
  * partially completed
* Produces many Artifacts over time

Mental model:

> *“This is a scientific experiment with traceable conditions.”*

---

### Coverage / Progress

**Coverage** describes how much of a Run has been executed.

Key properties:

* Tracked per unit of work (asset, frame batch, segment)
* Includes:

  * status (pending / running / done / failed)
  * timestamps
  * errors (if any)
* Enables:

  * partial visibility of results
  * early stopping
  * reruns of failed subsets

Mental model:

> *“How far did we get, and what exactly has been processed?”*

---

### Collection

A **Collection** is a named set of Assets.

Types:

* Dynamic: defined by a query (e.g., “photos from 2018 with faces”)
* Static: manually curated list

Uses:

* Albums
* Sandbox datasets
* Benchmark datasets
* Export packages

Key properties:

* Does not duplicate media
* Acts as an input scope for Runs

Mental model:

> *“A lens through which I look at my archive.”*

---

### Lineage

**Lineage** describes *why* an Artifact exists.

For every Artifact, lineage records:

* Which Run created it
* Which tool/module was used
* Tool version
* Parameters
* Time of execution

Mental model:

> *“Explain yourself.”*

Lineage is non-negotiable.

---

### Media View (e.g. FrameSource, AudioSource)

A **Media View** is a logical representation of derived media used as input to tools.

Examples:

* Frames extracted from a video
* Audio track extracted from a video
* Scene-based keyframes

Views can be:

* Precomputed (materialized artifact)
* Generated on demand (with parameters)
* Generated and cached depending on policy

Mental model:

> *“This is how a tool sees the asset.”*

---

### Policy

A **Policy** defines execution behavior for a Run.

Minimum policies:

* Sandbox
* Production

Policy influences:

* Sampling rate
* Resolution
* Persistence of intermediates
* Retention / TTL
* Resource usage expectations

Mental model:

> *“How careful and permanent should this run be?”*

---

## 2. Core Contracts (System Invariants)

These are **rules that must always hold**, regardless of future features.

### Contract 1: Originals Are Sacred

* Assets are never modified.
* All outputs are Artifacts.
* No tool may write to the archive.

---

### Contract 2: Artifacts Are First-Class

* Artifacts are stored immediately when produced.
* Partial results are valid results.
* Artifacts do not disappear silently.

---

### Contract 3: Lineage Is Mandatory

* Every Artifact must explain:

  * where it came from
  * how it was created
* “Magic results” are not acceptable.

---

### Contract 4: Incremental Visibility

* Long-running Runs must expose progress and partial outputs.
* Users can inspect, search, and act on incomplete Runs.
* Cancellation must preserve already-written Artifacts.

---

### Contract 5: Replaceability Over Finality

* No Artifact is “the truth forever”.
* New Runs can supersede old results.
* Old results remain inspectable and comparable.

---

### Contract 6: Humans Trump Automation

* Manual labels and corrections are authoritative.
* Automated tools must not silently overwrite human input.
* Conflicts must be explicit and inspectable.

---

### Contract 7: Experiments Are Safe

* Sandbox Runs must not pollute production data unintentionally.
* Ephemeral intermediates are allowed but controlled.
* Storage and resources are protected by policy.

---

## 3. Mental Model Summary (TL;DR)

* **Assets** are memories.
* **Artifacts** are interpretations.
* **Runs** are experiments.
* **Collections** are perspectives.
* **Lineage** is honesty.
* **Policies** define seriousness.
* **Early results** are a feature, not a bug.
