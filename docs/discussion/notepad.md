# Discussion Notes

## 2026-09-24 — Repository name and initial scope

The project will cover two connected stages:

1. Document preparation: process, validate, normalize, and structure source documents.
2. Knowledge ingestion: chunk and enrich the structured content, create embeddings, and index it for search or retrieval.

Names considered:

- `document-ingestion-pipeline`: concise, but it does not clearly communicate the later knowledge-building stage.
- `document-processing-pipeline`: describes document preparation but not knowledge ingestion.
- `knowledge-ingestion-pipeline`: describes the destination but understates document preparation.
- `document-to-knowledge-pipeline`: describes the complete journey from source documents to searchable knowledge.

Decisions:

- Repository name: `document-to-knowledge-pipeline`.
- GitHub owner: `deaxparadox`.
- Visibility: public.
- The current local folder remains named `injection-pipeline` and will be connected to the GitHub repository.
- Initial branch: `main`.
- The root `AGENTS.md` file remains local and will not be tracked by Git.
- Discussion progress and confirmed decisions will be recorded in this file.
- The local folder will be initialized as a Git repository and the setup will be recorded in an initial commit.

Verified starting state:

- GitHub CLI 2.101.0 is authenticated to GitHub as `deaxparadox` over HTTPS.
- Git 2.43.0 is installed.
- The current folder is not yet a Git repository.
- `deaxparadox/document-to-knowledge-pipeline` does not currently exist on GitHub.

## 2026-09-24 — Learning domain and first scope

The project is intended to be a learning project, so its scope should expose the important problems in document processing and knowledge ingestion rather than only produce a small demonstration.

Domains considered:

- SEC company filings: public filings with stable identities, narrative sections, tables, reporting metadata, and machine-readable Inline XBRL facts. The SEC provides public submission and XBRL data APIs without API keys.
- RBI regulations: public circulars and master directions that introduce PDF processing, effective dates, amendments, superseded documents, and relationships between regulations.
- Scanned or synthetic financial documents: useful later for OCR and confidence-based validation, but unnecessarily difficult as the first source.

Decision:

- Start with SEC filings.
- The first vertical slice will use one company's 10-K filing.
- It will preserve the filing identity and source, extract document sections and Inline XBRL facts, validate the structured result, and support retrieval with citations back to the filing.
- RBI regulatory documents are a possible later extension after the SEC workflow is understood.
- Scanned documents and OCR are deferred to a later stage.

Reasoning:

- SEC filings provide both human-readable content and machine-readable financial facts, allowing the project to learn extraction and validation against an authoritative structure.
- Starting with one filing keeps the first slice observable while still exercising the full path from source acquisition to cited retrieval.
- Starting with RBI documents would require solving source collection, PDF variation, regulatory versioning, and retrieval quality at the same time.

Still to decide before a spec can be written:

- The first company and filing.
- Whether "latest 10-K" is selected dynamically or pinned to a specific accession number for repeatable development.
- The exact questions or retrieval tasks used to judge success.
- The canonical structured-document model.
- The implementation language, libraries, storage, embedding provider, and retrieval approach.

Primary references reviewed:

- SEC EDGAR APIs: https://www.sec.gov/search-filings/edgar-application-programming-interfaces
- SEC Inline XBRL: https://www.sec.gov/data-research/structured-data/inline-xbrl
- RBI notifications: https://www.rbi.org.in/Scripts/NotificationUser.aspx/searchnew/BS_ViewMasterCirculardetails.aspx

## 2026-09-24 — Dataset source and reproducibility

Decision:

- The official SEC EDGAR system is the authoritative dataset source; the project will not depend on a third-party repackaged dataset.
- Version 1 will pin a specific filing accession number rather than dynamically changing when a newer filing appears.
- The source snapshot will be described by a committed manifest containing source URLs, SEC identifiers, retrieval timestamps, content types, byte sizes, and cryptographic hashes.
- Large raw filing artifacts will be downloaded into ignored local storage rather than committed to Git.
- The ingestion client will use a declared `User-Agent`, cache downloaded artifacts, and respect the SEC's current fair-access limit of 10 requests per second.

For the proposed PayPal 2025 10-K, the source set consists of the SEC filing index, primary Inline XBRL HTML document, extracted XBRL instance, complete raw submission, company submissions JSON, and company facts JSON. PayPal and accession `0001633917-26-000024` remain proposed until explicitly approved as the first filing.

Primary references reviewed:

- SEC access and fair-use guidance: https://www.sec.gov/search-filings/edgar-search-assistance/accessing-edgar-data
- SEC EDGAR APIs: https://www.sec.gov/search-filings/edgar-application-programming-interfaces
- Proposed PayPal filing index: https://www.sec.gov/Archives/edgar/data/1633917/000163391726000024/0001633917-26-000024-index.htm

## 2026-09-24 — First filing and operational requirements

Decisions:

- The first document is PayPal's 2025 Form 10-K, pinned to SEC accession `0001633917-26-000024`.
- Monitoring, observability, and traceability are core system requirements from the first runnable vertical slice.
- The pipeline will initially remain one well-instrumented application. It will not be split into microservices merely to demonstrate distributed tracing.
- OpenTelemetry's vendor-neutral model will be used for correlated traces, metrics, and structured logs. The telemetry backend remains undecided.
- Trace context will follow the W3C Trace Context standard.
- Persistent data lineage will use the OpenLineage concepts of jobs, runs, input datasets, output datasets, and data-quality assertions. Whether the implementation needs the OpenLineage package or only compatible internal contracts remains undecided.
- Because observability and lineage are cross-cutting design choices, an ADR will be required before their implementation spec.

Required end-to-end lineage:

`SEC filing -> downloaded bytes -> parsed document -> validated structure -> sections and financial facts -> chunks -> embeddings/index entries -> retrieval result -> cited answer`

Minimum correlation and provenance:

- Every execution has a pipeline run ID and an OpenTelemetry trace ID.
- Every stage emits correlated traces, metrics, and structured logs.
- Every source artifact records SEC identifiers, URL, retrieval time, size, content type, and SHA-256 hash.
- Every derived artifact records its parent artifacts, processing stage, code version, configuration version, and parser version.
- Every validation records its assertion, expected and observed values, severity, and outcome.
- Every chunk identifies its structured section and precise source location.
- Every retrieval records query identity, chunk identities, retrieval scores, and timing.
- Every generated answer records the retrieved evidence, citations, and relevant model/version information.
- Failures and fallbacks are explicit and observable; stages may not silently skip work.

Initial monitoring coverage:

- SEC request success, latency, retries, and rate-limit responses.
- Pipeline runs completed, failed, or stuck at each stage.
- Parsing warnings and extracted section, table, and fact counts.
- Validation pass and failure counts.
- Chunk, embedding, and indexing counts and failures.
- Indexing delay and retrieval latency.
- Empty retrieval results and citation coverage.
- Numeric-answer agreement with filing-specific XBRL facts.

Primary references reviewed:

- OpenTelemetry signals: https://opentelemetry.io/docs/concepts/signals/
- W3C Trace Context: https://www.w3.org/TR/trace-context/
- OpenLineage object model: https://openlineage.io/docs/spec/object-model/
- OpenLineage data-quality assertions: https://openlineage.io/docs/spec/facets/dataset-facets/data_quality_assertions/

## 2026-09-24 — Source formats and canonical JSON

Decision:

- Source acquisition is hybrid rather than JSON-only.
- SEC submission and company-facts APIs provide JSON discovery metadata and standardized financial facts.
- Complete filing content is acquired from EDGAR as original Inline XBRL/HTML, XBRL/XML, raw submission text, and related filing artifacts.
- Original SEC bytes are retained and hashed as immutable evidence.
- The pipeline produces its own canonical JSON representation containing document identity, sections, tables, financial facts, validation results, and lineage.
- The generated canonical JSON does not replace the source artifacts; every derived object must link back to exact source evidence.
- The acquisition boundary must expose SEC access failures. A read-only verification request from the development environment received SEC's automated-traffic rejection, reinforcing the need for a genuine configurable contact identity, compliant request pacing, caching, and observable `403` handling.

Primary references reviewed:

- SEC EDGAR APIs: https://www.sec.gov/search-filings/edgar-application-programming-interfaces
- Accessing EDGAR data: https://www.sec.gov/search-filings/edgar-search-assistance/accessing-edgar-data

## 2026-09-24 — Service and database direction

Decisions:

- PostgreSQL will be the relational database.
- Django is the only authority for relational schema changes through its models and migrations.
- SQLAlchemy may read and write application data from the FastAPI processing service, but it may not create, alter, or drop database objects.
- The intended model generator is `sqlacodegen`. It will run as a reproducible development/CI generation step after Django migrations are applied to a temporary database, not as runtime schema reflection.
- Generated SQLAlchemy mappings will be treated as generated code and kept separate from manually written repositories and domain logic.
- FastAPI will be exposed to authenticated clients rather than being an internal-only service.
- Django remains the authentication and account authority. The exact mechanism by which FastAPI verifies Django-issued identity and authorization remains to be decided.
- Heavy pipeline work will not run inside FastAPI request handlers. FastAPI will accept work and return a run identity; separate worker processes will execute queued stages.
- The API and workers may share the same pipeline-service code and container image while running as separate processes.
- A real message queue will be used so acknowledgements, retries, backpressure, and failure handling can be learned and observed. The broker and worker framework remain to be selected after delivery requirements are specified.
- Database-to-queue transitions require a transactional outbox, and consumers must be idempotent because queued delivery is expected to be at least once.
- Original and derived document artifacts use an artifact-storage abstraction. The initial local implementation uses the filesystem on a persistent Docker named volume; PostgreSQL stores artifact identities, relative keys, metadata, hashes, state, relationships, and lineage. Remote object storage is deferred.
- Deterministic acquisition, parsing, validation, chunking, and indexing are kept separate from optional agentic enrichment and answer generation.

Verified local toolchain:

- Docker 29.5.2 and Docker Compose 5.1.3 are installed.
- Python 3.11.6 is available through the Python launcher.
- The default `python` executable points to Python 3.13.0a5, an alpha build, so container and development commands must pin an explicitly selected stable Python version.
- Node.js 22.18.0, npm 11.19.1, and pnpm 11.25.0 are installed.
- No project application dependencies have been selected or installed.

Primary references reviewed:

- Django migrations: https://docs.djangoproject.com/en/6.0/topics/migrations/
- FastAPI background-task guidance: https://fastapi.tiangolo.com/tutorial/background-tasks/
- SQLAlchemy reflection: https://docs.sqlalchemy.org/en/20/core/reflection.html
- `sqlacodegen`: https://github.com/agronholm/sqlacodegen
- Transactional outbox pattern: https://docs.aws.amazon.com/en_en/prescriptive-guidance/latest/cloud-design-patterns/transactional-outbox.html
- RabbitMQ reliability concepts: https://www.rabbitmq.com/docs/reliability
- Next.js route groups: https://nextjs.org/docs/app/api-reference/file-conventions/route-groups

## 2026-09-24 — Pipeline execution structure

Decision:

- The system has two connected flows: an asynchronous ingestion pipeline that publishes versioned knowledge snapshots, and an authenticated online retrieval pipeline that queries a selected published snapshot.
- PostgreSQL is the durable source of execution state. The message queue is a delivery mechanism and never the only record of work.
- Creating a run writes the run, initial stage graph, and an outbox event in one transaction. FastAPI returns `202 Accepted` with the run identity.
- An outbox publisher transfers committed work to the queue. Separate workers execute durable stages.
- Queue messages carry versioned identifiers, idempotency keys, and trace context, not document bodies.
- Stage outputs are immutable artifacts in the configured artifact store with PostgreSQL metadata and lineage edges.
- Every stage has explicit input/output contracts, configuration hashes, retry and timeout policies, validation results, telemetry, and terminal state.
- Stage attempts are separate from logical stages so retries do not erase failure history.
- Runs are resumable from the last valid immutable artifact.
- The approved ingestion stages are source resolution, acquisition, source validation, parallel filing and XBRL parsing, canonical normalization, cross-validation, optional enrichment, structure-aware chunking, embedding, indexing, quality gating, and atomic knowledge-snapshot publication.
- A required quality gate prevents incomplete or invalid indexes from becoming active.
- Retrieval records the authenticated subject, project, snapshot, query configuration, retrieved chunks and scores, evidence package, model and prompt versions, tool calls, citations, timing, and validation results.
- Deterministic ingestion creates trusted knowledge. Agentic behavior consumes it through bounded, typed, audited tools and cannot bypass required validation.
- Django models and migrations define relational tables. Pydantic/JSON Schema contracts independently define APIs, messages, and document artifacts. OpenTelemetry and OpenLineage conventions define telemetry and lineage contracts.

Initial relational domains identified:

- Pipeline definitions, runs, stages, attempts, outbox events, and processed-message identities.
- Source documents, source artifacts, derived artifacts, and artifact-lineage edges.
- Validation results, document sections, document tables, and financial facts.
- Knowledge snapshots, chunks, embeddings, and index records.
- Retrieval runs, retrieval hits, generation runs, agent tool calls, and citations.

## 2026-09-24 — Extensibility and component isolation

Decision:

- Flexibility comes from stable, versioned contracts and replaceable adapters, not from a generic workflow language or premature microservices.
- Source connectors discover and acquire immutable raw artifacts but do not parse or index them. API, scraper, upload, and object-storage connectors are separate implementations of the same contract.
- Parsers are selected by validated content capabilities and signatures rather than source names or filename extensions.
- Parsers produce a source-independent canonical block model with headings, paragraphs, lists, tables, figures, footnotes, facts, ordering, and precise source locators.
- Source-specific metadata uses validated, namespaced extensions instead of adding source-specific fields to every canonical document.
- Versioned pipeline definitions compose reusable connectors, parsers, validators, enrichers, chunkers, embedding providers, and index adapters for a particular source and document type.
- New sources begin as explicitly registered in-repository adapters. Standard Python package entry points may be introduced later only if independently distributed plugins become necessary.
- A central stage runner owns contract validation, attempt creation, trace propagation, idempotency, execution, artifact persistence, output validation, lineage, outbox emission, and terminal-state handling.
- Queue tasks are durable retry boundaries, not arbitrary function boundaries. Small internal operations remain ordinary functions with child trace spans.
- Every contract has a reusable compliance suite. Testing covers units, adapter contracts, golden fixtures, offline replay, infrastructure integration, injected failures, required telemetry, end-to-end snapshots, retrieval evaluation, and Django-to-SQLAlchemy schema drift.
- Standard stage telemetry includes lifecycle counts, duration, queue wait, artifact sizes, validation outcomes, worker concurrency, and oldest-message age. Unique run and artifact identities belong in correlated traces and logs rather than metric labels.
- All outputs record pipeline, stage, contract, configuration, source-artifact, and code versions so stored artifacts can be replayed through newer processing logic without redownloading the source.
- The orchestration core must not contain source-specific conditionals, queue document bodies, grant plugins arbitrary database access, or permit adapters to opt out of required telemetry and lineage.

Primary references reviewed:

- Python plugin discovery: https://packaging.python.org/en/latest/guides/creating-and-discovering-plugins/
- Python entry-point specification: https://packaging.python.org/en/latest/specifications/entry-points/
- OpenTelemetry instrumentation: https://opentelemetry.io/docs/concepts/instrumentation/
- OpenTelemetry instrumentation scopes: https://opentelemetry.io/docs/concepts/instrumentation-scope/

## 2026-09-24 — Tenancy direction

Decision:

- The system will use a multi-tenant design rather than individual-user-only ownership.
- A workspace or organization is the tenant boundary and owns projects, sources, pipeline runs, documents, artifacts, knowledge snapshots, retrieval activity, and usage records.
- Authentication remains centralized in Django, while authorization and tenant isolation must be enforced consistently in Django, the authenticated FastAPI API, background workers, artifact access, search and vector retrieval, agent tools, and operator actions.
- The exact PostgreSQL isolation mechanism, initial membership roles, token design, and quota model remain to be decided before the relational schema is specified.
- Initial application roles are `owner`, `admin`, `member`, `viewer`, and `service`. Authorization code will use named permissions mapped from these roles rather than hard-coded role-name checks throughout the application.
- Application roles are distinct from PostgreSQL connection roles. The database-role design remains part of the tenant-isolation decision.

## 2026-09-24 — Local artifact storage

Revised decision:

- The initial local system will not deploy an object-storage service.
- Pipeline code depends on an artifact-storage interface rather than filesystem calls spread across stages.
- The first adapter writes to a persistent local filesystem path mounted through a Docker Compose named volume.
- Database records store relative artifact keys rather than machine-specific absolute paths.
- The filesystem layout remains tenant-aware, for example `workspaces/{workspace_id}/projects/{project_id}/artifacts/{artifact_id}`.
- Artifact writes use a temporary file followed by atomic promotion after hashing and validation so readers never observe partial files.
- Raw and derived data remain outside Git.
- Backup, cleanup, quota, and deletion behavior must include the named volume.
- A future S3-compatible adapter may implement the same interface without changing pipeline-stage contracts.

Primary reference reviewed:

- Docker volumes: https://docs.docker.com/engine/storage/volumes/
