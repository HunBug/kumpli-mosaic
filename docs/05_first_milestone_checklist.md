# 05 — First Milestone Checklist (Action Plan + Validation)

## Milestone name

**M1: Trustable Core Loop**

## Goal (one sentence)

Prove the end-to-end system loop works: **catalog → run → partial artifacts → query/browse → lineage → cancel/resume**, with originals remaining read-only and safe.

## Definition of “done”

This milestone is complete when you can:

* scan/catalog a real subset of the archive safely
* start a sandbox run
* see partial results while it’s still running
* browse/query results and inspect lineage
* cancel a run without losing already-produced artifacts
* diagnose failures and rerun only what failed

No “fancy AI” required yet — just one or two simple artifact types to validate the system.

---

## Phase 0 — Pre-flight (decisions and boundaries)

1. Confirm the non-negotiables (write them in the repo)

* Originals are read-only, never modified.
* Artifacts are stored separately from originals.
* Lineage is mandatory for every artifact.
* Partial results must be visible.
* Runs must be cancelable/resumable.

Success check:

* These principles exist in docs and you agree they won’t be compromised “temporarily.”

2. Choose the smallest realistic test slice of the archive
   Pick a subset that is big enough to be real, small enough to iterate fast:

* e.g. 5k photos + 50 videos from mixed years, mixed devices, mixed folders

Success check:

* You can point to a concrete subset path/query and say “this is M1 scope.”

3. Decide what artifacts M1 will produce
   Pick **two** artifact types maximum (to keep it tight):
   Recommended:

* **image thumbnail** (fast, binary artifact)
* **technical metadata** (EXIF + video stream info) as structured artifact

Optional swap:

* If you prefer: “video keyframe” instead of thumbnail

Success check:

* A written list: “M1 artifacts = X, Y.”

---

## Phase 1 — Build the “Core Loop Vocabulary” into the system

4. Define the canonical entities (conceptually, not schema-level)

* Asset
* Artifact
* Run
* Coverage/Progress
* Collection
* Lineage
* Policy (Sandbox vs Production)

Success check:

* A single page exists (Doc 03 already) and you can describe each entity in one sentence.

5. Decide how “partial results” are represented
   High-level rules:

* Artifacts can be written per-asset (or per-segment) at any time.
* Coverage/progress is tracked independently of completion.

Success check:

* You can answer: “If the run is cancelled midway, what data remains and how do we see it?”

---

## Phase 2 — Operational skeleton (minimal, but real)

6. Stand up the minimum platform components (M1 only)
   You’re allowed to do this “ugly but reproducible.”
   Required:

* Workflow engine running
* Core API/service running
* DB for truth store running
* Artifact store location writable
* One worker execution path running (local or cluster)

Success check:

* “Hello world” run can be initiated and observed (even if it does nothing meaningful yet).

Notes:

* It’s fine if this is a single-node K8s at first.
* It’s fine if the first worker is a single process.

7. Establish your safety rails early
   High-level controls to include from day one:

* Do not exceed disk threshold (stop artifact creation gracefully)
* Concurrency limits (don’t melt CPU/RAM)
* Logging for job failures with enough info to rerun

Success check:

* You can simulate “disk almost full” and see the system stop safely (even if it’s manual at first).

---

## Phase 3 — Catalog subset and prove “originals are sacred”

8. Implement the catalog pass for M1 subset
   The catalog pass should:

* discover assets in the chosen slice
* compute stable identity (hash or equivalent)
* store path + basic facts (type, timestamps, size)
* record “read-only source” assumption

Success check:

* Catalog results exist, and you can list assets without touching the originals.

9. Verify the read-only guarantee
   Run a “red team” check:

* confirm no writes occurred in the archive mount
* confirm no hidden files created
* confirm no metadata writes back to media files

Success check:

* You can confidently say “the system cannot accidentally write to originals.”

---

## Phase 4 — Runs, collections, and sandbox policy

10. Create collections (saved scopes)
    At minimum:

* A “M1_test_slice” collection
* A “tiny_sample” collection (like 50–200 assets) for fast iteration

Success check:

* You can run the same pipeline against different collections without editing code.

11. Define sandbox run policy behavior (in practice)
    Sandbox should:

* allow partial runs
* allow cancellation
* optionally store intermediates with short retention (even if “retention” is manual at first)
* prioritize fast feedback

Success check:

* You can initiate a sandbox run and see results appear as it progresses.

---

## Phase 5 — First real pipeline: thumbnail + metadata

12. Create the M1 pipeline template
    Pipeline intent (conceptual):

* For each asset:

  * extract basic technical metadata
  * produce thumbnail (images) and/or keyframe (videos)

Success check:

* A single “Run” corresponds to a specific pipeline version + params, and this is recorded.

13. Ensure per-asset chunking
    Rule:

* Every asset (or small batch) is its own unit of work so results appear early.

Success check:

* You can see artifacts being written continuously while the run is in progress.

14. Prove lineage for every artifact
    Minimum lineage fields:

* run id
* tool id/name
* tool version
* parameters
* timestamp
* input asset id(s)

Success check:

* Pick any thumbnail and explain exactly how it was produced.

---

## Phase 6 — Minimal browsing + debug UI (or CLI)

15. Implement minimum “inspectability”
    You need at least one interface (CLI is fine) that can:

* search/list assets by basic filters
* open an asset detail view (shows artifacts + lineage)
* show run progress and failures
* cancel a run

Success check:

* You can debug a wrong thumbnail or missing artifact without digging into raw logs on disk.

---

## Phase 7 — Failure and recovery drills (the most important test)

16. Failure simulation checklist
    Run these tests deliberately:

A) Worker crash mid-run

* Kill the worker process / pod.
  Expected:
* run continues (after retry) or pauses in a known state
* already written artifacts remain
* coverage shows what is done vs pending vs failed

B) Cancel a run mid-way
Expected:

* run stops
* partial results remain visible/searchable
* coverage reflects partial completion

C) Out-of-space / disk threshold
Expected:

* new artifact writes stop safely
* run marks remaining items as paused/failed with clear reason

D) Bad tool parameters
Expected:

* you detect it early via partial results
* you cancel and rerun with different params
* both runs remain comparable by lineage

Success check:

* You can perform each drill and the system behaves predictably.

---

## Phase 8 — Benchmark seed (optional but recommended for M1)

17. Add a “benchmark run record” pattern
    Even if metrics are minimal, record:

* start/end time
* counts processed
* counts failed
* average time per asset
* resource notes (rough)

Success check:

* You can compare two runs and answer “which was faster / more reliable?”

---

## Exit Criteria Summary (quick pass/fail)

M1 passes if all are true:

* Cataloging does not modify originals.
* A collection can be defined and reused.
* A run can be started in sandbox mode.
* Artifacts appear incrementally during a run.
* Every artifact has lineage.
* You can browse/query results (CLI or minimal UI).
* You can cancel a run and keep partial artifacts.
* You can crash a worker and recover.
* Failures are visible and rerunnable without rerunning everything.

---

## Notes: choices you are allowed to postpone

Do not block M1 on:

* face recognition
* embeddings/vector search
* OCR / ASR
* annotation UI
* multi-node scheduling sophistication
* perfect storage/retention automation

M1 is about trust and loop closure, not intelligence.


