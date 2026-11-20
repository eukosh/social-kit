# A platform-neutral core with provider packages

Meta is the only provider today, yet `core/` is written as if it weren't, because TikTok is the next platform and must arrive as a new package without reshaping anything. `core/` holds the operation registry, errors and the reason catalogue, the login store, defaults and `--as`, parsing, pagination, output, the write ledger, media handling and redaction, and imports nothing from `meta/` or a platform package (a test enforces it). `meta/` holds what Meta's three platforms share: the Graph transport (one instance per platform, version pinned per host), the error table, `appsecret_proof`, usage headers, containers, OAuth and the setup steps. `instagram/`, `facebook/` and `threads/` hold models and operations per noun. App credentials are keyed by provider (`app.json` as `{"meta": {...}}`, `SKIT_META_APP_ID` and its siblings), and a provider's raw error codes sit under `details.<provider>`.

## Consequences

- A TikTok package adds its own transport, error table, OAuth and setup steps; nothing in `core/` changes.
- A few reasons in the shared catalogue are provider-specific (`blocked_by_meta`); the catalogue says which provider each belongs to.
