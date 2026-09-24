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
