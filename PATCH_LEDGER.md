# Hybrid Mic patch ledger

Every change from upstream `v11.17.4` is recorded here before release. Security details that would create avoidable
exposure stay in a private advisory until coordinated disclosure; this ledger receives the public reference afterwards.

| Project release       | Upstream reference                                                                         | Impact and changed behavior                     | Regression proof                                                    | Rollback                                                                 |
| --------------------- | ------------------------------------------------------------------------------------------ | ----------------------------------------------- | ------------------------------------------------------------------- | ------------------------------------------------------------------------ |
| baseline (unreleased) | [`directus/directus@v11.17.4`](https://github.com/directus/directus/releases/tag/v11.17.4) | Exact upstream source; no Hybrid Mic core patch | Upstream tag resolves to `1f9a051041029e59b17a1a36249f232e59e60e9b` | Use upstream `directus/directus:11.17.4` with a compatible data snapshot |

## Entry checklist

- Advisory/issue/PR and upstream commit:
- Affected and fixed versions:
- Hybrid Mic exposure and severity:
- Minimal patch rationale:
- Regression test that fails on the previous image:
- Source, image, schema, extension, preview, and restore checks:
- SBOM/scan review and accepted findings:
- Release tag and GHCR digest:
- Database compatibility and rollback snapshot/image:
- Reviewer and release date:
