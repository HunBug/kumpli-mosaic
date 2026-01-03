Workflow / durable orchestration (runs pipelines, retries, progress, partial results)

* Temporal (you already picked it): durable workflows, retries, heartbeats/progress, long-running jobs without losing state; .NET SDK is a real advantage. Pros: “I can stop mid-way and resume” is basically its whole thing; great for your “store results early” requirement. Cons: infrastructure-y (server + persistence), more moving parts than “a script runner.” Temporal is MIT licensed. ([Temporal][1])
* Prefect OSS: super friendly “turn scripts into flows” vibe; nice UI, retries, caching, task state. Pros: fastest time-to-value for Python-heavy pipelines. Cons: more Python-first (fine for modules, less ideal as the *core* if you want C# to be the brain). ([Prefect][2])
* Dagster OSS: strong on “data assets” and observability; clean dev experience. Pros: great lineage/metadata thinking. Cons: can feel more “data-platform” than “personal sandbox,” and its asset model may push structure you don’t want. (Core is Apache-2.0.) ([GitHub][3])

Distributed compute / worker execution (GPU workers, multi-node, “run this over there”)

* Ray: distributed tasks + actors + data; very common for ML batch processing. Pros: easy scale-out to GPU nodes; good for “fan out over 4TB.” Cons: becomes a mini-platform; you’ll want to keep it behind your core so it doesn’t take over your architecture.
* Celery: classic distributed task queue. Pros: simple mental model. Cons: less ergonomic for “lineage + artifacts + experiments,” unless you build a bunch around it.
* Kubernetes / “lighter K8s”: k3s is the usual lightweight choice for a homelab; Nomad is simpler and pleasant if you want “scheduler vibes without full K8s.” Pros: clean multi-node story. Cons: operational overhead; might be overkill if Temporal + a few worker services already cover you.

Media plumbing (read-only archive, extraction, thumbnails, frame sampling)

* FFmpeg (+ ffprobe): the backbone for video/audio decode, frame extraction, transcoding. Pros: everything supports it; perfect for “frames are a derived artifact.” Cons: licensing complexity if you ever distribute binaries; FFmpeg is LGPL with optional GPL parts depending how it’s built/used. ([FFmpeg][4])
* MediaInfo: fast technical metadata (codec, duration, streams). Pros: lightweight, great for indexing. Cons: not a processor, just an informer. BSD-2-Clause. ([GitHub][5])
* ExifTool: best-in-class EXIF/IPTC/XMP metadata read/write. Pros: insanely comprehensive; great for ingest/index. Cons: you’ll need to design how you treat “truth” (files vs DB). Packaged under Artistic-Perl OR GPL-3.0-or-later (so: check your comfort level). ([Arch Linux][6])
* PySceneDetect: scene/shot detection for smarter video sampling. Pros: reduces “process every frame” pain; makes “preprocessing node” very real. Cons: another param-surface to manage (thresholds, detectors). BSD-3-Clause. ([SceneDetect][7])
* OpenCV: general CV glue (image transforms, classical detection, etc.). Pros: universal; integrates with everything. Cons: easy to accidentally do too much “in OpenCV.” OpenCV 4.5+ is Apache-2.0. ([OpenCV][8])

Search + query index (your “main place where I can search/query”)

* OpenSearch: full-text search + filtering + aggregations; scalable and well-known. Pros: “Google-like” keyword search + facets; great for logs/metrics too. Cons: heavier ops footprint than a small embedded search engine. Apache-2.0. ([OpenSearch][9])
* Meilisearch: very developer-friendly search. Pros: quick to get “pleasant search UX.” Cons: licensing has changed over time via dual licensing / enterprise edition discussions—worth re-checking before committing long-term. ([Meilisearch][10])
* Typesense: fast, simple, great UX. Pros: lovely for personal projects. Cons: GPL-3.0 (some people avoid GPL for “core infrastructure” dependencies). ([GitHub][11])

Vector search (for CLIP/embeddings: “find photos like this vibe”)

* pgvector (Postgres extension): store embeddings *in your main DB*. Pros: simplest architecture if Postgres is your source-of-truth; good enough for a lot of personal-scale vector search. Cons: not as feature-rich as dedicated vector DBs at huge scale. pgvector is under the PostgreSQL License. ([EDB][12])
* Qdrant: dedicated vector DB, fast, filters, good APIs. Pros: very clean “vectors + payload filters” story. Cons: yet another service to run. Apache-2.0. ([GitHub][13])
* Milvus / Weaviate: bigger platforms. Pros: very capable. Cons: more operational weight than you likely need at first (but fine if you want “professional overkill”). Milvus is Apache-2.0; Weaviate is BSD-3-Clause. ([GitHub][14])

Annotation / human-in-the-loop (“name faces”, labels, QA, training sets)

* Label Studio: flexible annotation UI for images/video/text/audio; exports datasets; can integrate ML-assisted prelabels. Pros: perfect “attachable UI tool” node. Cons: you’ll still need your sync/connector layer to your core DB. Apache-2.0. ([GitHub][15])
* CVAT: very strong for computer vision annotation (boxes, polygons, tracking). Pros: great for serious CV labeling. Cons: heavier than Label Studio; more “team tool” vibes.

Photo library tools you can steal ideas from (or use as a temporary UI)

* Immich: self-hosted Google Photos-like; does faces/objects/CLIP search and a solid browsing UI. Pros: you can use it as a “reference implementation” for UX, or even as a *front-end* while your core grows. Cons: it becomes tempting to rely on its internal DB/assumptions, so treat it as replaceable. ([GitHub][16])
* PhotoPrism / LibrePhotos / PhotoStructure: similar category—useful to evaluate expected features and what metadata you’ll want.

Speech/audio (for “memories”: voice notes, videos, diarization)

* Whisper: local ASR baseline; MIT license. Pros: works offline, huge ecosystem. Cons: accuracy/hallucination caveats; you’ll want confidence signals + “keep original audio + provenance” anyway. ([GitHub][17])
* faster-whisper: faster/lower-memory inference via CTranslate2; MIT. Pros: friendlier for big batch jobs. Cons: still Whisper-family tradeoffs. ([GitHub][18])
* WhisperX: alignment + diarization hooks. Pros: “who spoke when” is huge for home videos. Cons: diarization often relies on gated Hugging Face models/tokens (operational friction). ([GitHub][19])

OCR

* Tesseract: classic OCR engine; Apache-2.0. Pros: easy to run locally; stable. Cons: weaker on “weird layouts” unless you build more around it. ([GitHub][20])
* PaddleOCR: very strong modern OCR toolkit; Apache-2.0. Pros: great quality, lots of models. Cons: more ML stack weight. ([GitHub][21])
* EasyOCR: popular, but watch licenses per distribution/model; don’t assume it’s always permissive just because it’s common. ([GitHub][22])

Face recognition (big gotcha: model licenses)

* InsightFace (framework): code is MIT, commercial-friendly; BUT many pretrained model weights / training-data-derived models can have non-commercial restrictions. Pros: strong quality; widely used. Cons: you must track licenses for *model artifacts* separately from code (this matters even for “personal use” if you ever share outputs/tools). ([GitHub][23])

Messaging / events (glue between core, workers, UI)

* NATS: lightweight, fast pub/sub; great for “events: new asset indexed, job progress, artifact ready.” Pros: simple and performant; Apache-2.0. Cons: you still need your own conventions (schemas, topics). ([GitHub][24])

Storage abstraction (derived artifacts, thumbnails, embeddings dumps)

* MinIO: S3-compatible object store. Pros: great interface for “derived artifacts live here.” Cons: licensing/feature packaging has been contentious; MinIO is dual-licensed AGPLv3 + commercial, and some management features shifted between editions. ([MinIO][25])
  (For a single-user homelab, plain filesystem + content-addressed paths can also be enough—object store can come later.)

Monitoring / “don’t eat my RAM/disk”

* Prometheus: metrics scraping + alerting; Apache-2.0. Pros: standard; pairs with everything. Cons: you’ll need to define what to measure. ([Prometheus][26])
* Grafana: dashboards; but AGPLv3 now. Pros: best dashboards. Cons: AGPL may or may not matter for you personally, but it’s a “license you should consciously choose.” ([Grafana Labs][27])

[1]: https://temporal.io/about?utm_source=chatgpt.com "About Temporal | Cloud-Oriented Durable Workflow ..."
[2]: https://www.prefect.io/prefect/open-source?utm_source=chatgpt.com "Prefect Open Source - Python Workflow Orchestration"
[3]: https://github.com/dagster-io/dagster?utm_source=chatgpt.com "dagster-io/dagster: An orchestration platform for the ..."
[4]: https://www.ffmpeg.org/legal.html?utm_source=chatgpt.com "FFmpeg License and Legal Considerations"
[5]: https://github.com/MediaArea/MediaInfo/blob/master/LICENSE?utm_source=chatgpt.com "MediaInfo/LICENSE at master"
[6]: https://archlinux.org/packages/extra/any/perl-image-exiftool/?utm_source=chatgpt.com "perl-image-exiftool 13.44-1 (any)"
[7]: https://www.scenedetect.com/?utm_source=chatgpt.com "PySceneDetect: Home"
[8]: https://opencv.org/license/?utm_source=chatgpt.com "License"
[9]: https://opensearch.org/?utm_source=chatgpt.com "OpenSearch: Home"
[10]: https://www.meilisearch.com/blog/enterprise-license?utm_source=chatgpt.com "Introducing the Meilisearch Enterprise Edition license"
[11]: https://github.com/typesense/typesense/blob/v30/LICENSE.txt?utm_source=chatgpt.com "typesense/LICENSE.txt at v30"
[12]: https://www.enterprisedb.com/docs/pg_extensions/pgvector/?utm_source=chatgpt.com "EDB Docs - pgvector"
[13]: https://github.com/qdrant/qdrant/blob/master/LICENSE?utm_source=chatgpt.com "qdrant/LICENSE at master"
[14]: https://github.com/milvus-io/milvus?utm_source=chatgpt.com "Milvus is a high-performance, cloud-native vector database ..."
[15]: https://github.com/HumanSignal/label-studio?utm_source=chatgpt.com "Label Studio is a multi-type data labeling and annotation ..."
[16]: https://github.com/immich-app/immich?utm_source=chatgpt.com "immich-app/immich: High performance self-hosted photo ..."
[17]: https://github.com/openai/whisper?utm_source=chatgpt.com "openai/whisper: Robust Speech Recognition via Large- ..."
[18]: https://github.com/SYSTRAN/faster-whisper/blob/master/LICENSE?utm_source=chatgpt.com "faster-whisper/LICENSE at master"
[19]: https://github.com/m-bain/whisperX?utm_source=chatgpt.com "m-bain/whisperX"
[20]: https://github.com/tesseract-ocr/tesseract?utm_source=chatgpt.com "Tesseract Open Source OCR Engine (main repository)"
[21]: https://github.com/PaddlePaddle/PaddleOCR?utm_source=chatgpt.com "PaddlePaddle/PaddleOCR"
[22]: https://github.com/gjwgit/easyocr/blob/master/LICENSE?utm_source=chatgpt.com "easyocr/LICENSE at master"
[23]: https://github.com/deepinsight/insightface?utm_source=chatgpt.com "deepinsight/insightface: State-of-the-art 2D and 3D Face ..."
[24]: https://github.com/nats-io/nats-server?utm_source=chatgpt.com "nats-io/nats-server"
[25]: https://www.min.io/commercial-license?utm_source=chatgpt.com "MinIO Open Source"
[26]: https://prometheus.io/?utm_source=chatgpt.com "Prometheus - Monitoring system & time series database"
[27]: https://grafana.com/licensing/?utm_source=chatgpt.com "Licensing | Grafana Labs"
