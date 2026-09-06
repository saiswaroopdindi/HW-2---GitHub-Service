# Design Note

Author: OpenAI Codex

## Boundary and errors

FastAPI owns request validation and HTTP semantics. `GitHubClient` centralizes the required GitHub `Accept`, bearer, and API-version headers. GitHub 404s remain 404s, authentication and validation failures become a stable JSON error shape, rate exhaustion becomes 429 with `Retry-After` when GitHub supplies it, and upstream 5xx responses become 503.

## Pagination

The caller controls `state`, `labels`, `page`, and `per_page`. The gateway validates bounds and forwards GitHub’s `Link` header unchanged, preserving GitHub’s navigation semantics without guessing totals.

## Webhooks

The receiver reads the raw body, computes HMAC-SHA256, and compares signatures with `hmac.compare_digest`. It acknowledges supported events quickly after inserting a compact event summary into SQLite. The GitHub delivery ID is the primary dedupe key; a deterministic body hash is used only when a delivery header is absent.

## Security and operations

Credentials are environment-only and never logged. Logs contain request and delivery IDs plus event metadata, not raw payloads or signatures. SQLite is intentionally small and local for the assignment; production deployment should use a managed queue/store and a secret manager. Issue deletion is represented by `PATCH state=closed`, matching GitHub’s model.

