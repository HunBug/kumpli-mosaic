# Roadmap (Longer Path Without Early Lock-In)

This roadmap describes the likely evolution of **Kumpli Mosaic** from a trustworthy core loop (M1) into a genuinely useful “memory intelligence lab.”
It avoids committing to specific models or tools early. Each milestone keeps the system usable and offers multiple “choose your next adventure” options.

## Guiding strategy

* Build **core reliability** first, then add “intelligence.”
* Prefer features that become **daily-usable** quickly.
* Every milestone should produce something you can browse, trust, and enjoy.
* Keep all big subsystems replaceable (especially search/vector/ML tools).

---

## Milestone 1 — Trustable Core Loop (done by checklist)

Outcome: the system is a reliable laboratory and catalog.

You can already use it for:

* browsing your archive via the system’s own catalog
* generating thumbnails/keyframes
* running sandbox runs and seeing partial results
* understanding lineage and debugging processing

This is where the project stops being “idea” and becomes “machine.”

---

## Milestone 2 — Usability Baseline: “I can actually browse and export”

Goal: make the system pleasant enough to use day-to-day, even before AI magic.

What becomes possible:

* A timeline view that loads fast (from DB + thumbnails)
* Saved collections and exports (“give me this album as a package”)
* Basic quality filters (blurry, duplicates) even if simple heuristics

Key additions (conceptual):

* Better browsing/search UX (still minimal, but comfortable)
* Export manifests: zip, folder copy, or “playlist” style export
* Basic “asset health”: missing files, moved paths, mismatch detection

You can stop here and already get value:

* fast browsing, consistent organization, easy “collect and export” workflows

---

## Milestone 3 — The First “Wow”: choose one intelligence pillar

Pick **one** pillar first. Any of these unlocks real “Google Photos feeling.”

### Option A: Faces & identities (high emotional value, very “memories”)

Outcome:

* Face detection → embeddings → clustering → naming UI
* Search: “Boo”, “Maa”, “Pupi”, “friends”, etc.
* Timeline per person/cat

What this unlocks:

* automatic albums per person
* “all photos of X over 15 years”
* “faces in videos” later

### Option B: Words inside videos (high practical value)

Outcome:

* Audio extraction → speech-to-text → searchable transcripts
* Optional: diarization (“who spoke when”)

What this unlocks:

* “find that conversation” across years of videos
* searchable home recordings
* “kitchen talk” retrieval by words and topics

### Option C: Semantic search / vibe retrieval (your long-term magic)

Outcome:

* Embeddings for images/video keyframes (and optionally text/audio)
* Search by meaning: “cosy”, “rainy street”, “cat on blanket”
* Similarity: “find moments like this photo”

What this unlocks:

* “mood albums”
* “find similar moments”
* aesthetic-based discovery

Recommendation (non-binding):

* Faces first if you want emotionally grounded utility.
* Vibe retrieval first if you want discovery and play.
* Transcripts first if you want “search my life like a knowledge base.”

---

## Milestone 4 — Human-in-the-loop workflows: “correct, train, refine”

Goal: make it easy to improve results over time.

Outcomes:

* Review pipelines: approve/deny, merge/split clusters, fix labels
* Manual corrections stored as authoritative truth
* Reprocessing workflows that respect human truth

This milestone is where the system becomes *yours*:

* not “what the model thinks,” but “what we know.”

Optional expansions:

* Annotation UI integration (Label Studio or similar) via a sync connector
* “active learning”: show uncertain cases first

---

## Milestone 5 — Quality & Curation: “frame-worthy” and “best of”

Goal: make the system help you curate and preserve beauty, not only search.

Outcomes:

* Quality metrics as artifacts (sharpness, noise, exposure, face quality)
* “Print candidates” smart collection
* Auto-cropping/enhancement as derived artifacts (never touching originals)

Useful features:

* “best of day” / “best of trip” selection
* burst grouping + best frame
* duplicate/near-duplicate consolidation

This is where you start making:

* “wall-worthy picks”
* yearly curated albums
* photo sets for printing

---

## Milestone 6 — Video intelligence: scenes, highlights, summaries

Goal: make videos as searchable and useful as photos.

Outcomes:

* Scene detection + keyframes become standard “views”
* Highlight reel generation from:

  * faces + smiles/laughter peaks (later)
  * motion + audio peaks
  * quality/aesthetic peaks

Optional “fun artifacts”:

* 30–90 second monthly highlight
* “trip recap” reels
* “cosy kitchen” montage

This milestone usually comes after M3 because it benefits from:

* faces, audio transcripts, or embeddings

---

## Milestone 7 — Automation & “Surprise me” mode

Goal: turn the system into a companion, not just a tool.

Outcomes:

* Scheduled runs (“nightly ingest”, “weekly surprise album”)
* Automatic resurfacing:

  * “On this day”
  * “forgotten gems”
  * “cosy moments”
* Notifications are optional (you can keep it passive)

Important constraint:

* surprises must remain explainable (lineage) and controllable

This is where the system starts feeling like:

* “it has a memory”
* “it sees patterns I forgot”

---

## Milestone 8 — Advanced lab mode (optional, only if you want it)

These are powerful but not required.

Examples:

* model benchmarking harness becomes richer (metrics, confusion sets)
* fine-tuning pipelines (only if you truly need them)
* multi-node storage fabric (Longhorn/Ceph) if you distribute artifacts heavily
* offline worker sync workflows become polished

This is “professional overkill,” and that’s okay — only after the system earns it.

---

## “Use it early” checkpoints (so you feel progress)

You can get real use at:

* After M1: catalog + thumbnails + runs + lineage (trust)
* After M2: comfortable browsing + exports (daily utility)
* After M3: first intelligence pillar (magic)
* After M5: curation + print-worthy picks (lasting value)
* After M7: surprise mode (companion feeling)

---

## Roadmap flexibility rules (to avoid lock-in)

* Search is a plugin: try multiple, keep truth in the DB.
* Embeddings are artifacts: change models, keep lineage.
* Pipelines are replaceable: new run supersedes old results.
* UI panels are attachable: you can add a face reviewer without rebuilding the app.
* Always preserve originals, always preserve explainability.

