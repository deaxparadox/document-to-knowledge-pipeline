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
