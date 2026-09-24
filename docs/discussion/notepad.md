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
