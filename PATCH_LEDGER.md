# Hybrid Mic patch ledger

Every change from upstream `v11.17.4` is recorded here before release. Security details that would create avoidable
exposure stay in a private advisory until coordinated disclosure; this ledger receives the public reference afterwards.

| Project release       | Upstream reference                                                                         | Impact and changed behavior                     | Regression proof                                                    | Rollback                                                                 |
| --------------------- | ------------------------------------------------------------------------------------------ | ----------------------------------------------- | ------------------------------------------------------------------- | ------------------------------------------------------------------------ |
| baseline (unreleased) | [`directus/directus@v11.17.4`](https://github.com/directus/directus/releases/tag/v11.17.4) | Exact upstream source; no Hybrid Mic core patch | Upstream tag resolves to `1f9a051041029e59b17a1a36249f232e59e60e9b` | Use upstream `directus/directus:11.17.4` with a compatible data snapshot |

## Baseline security gate

The initial import reported 193 open dependency alerts: 6 critical, 72 high, 96 moderate, and 19 low. No project image
may be released until runtime reachability and available patches are triaged and the release image passes the blocking
Trivy scan. The imported upstream image remains the rollback reference; no Hybrid Mic fork image exists yet.

The first review found that `aquasecurity/trivy-action@0.33.1` in the new release workflow was itself affected by
[GHSA-69fq-xp46-6x23](https://github.com/advisories/GHSA-69fq-xp46-6x23) and
[GHSA-9p44-j4g5-cfx5](https://github.com/advisories/GHSA-9p44-j4g5-cfx5). It was replaced before use with the patched,
immutable `0.35.0` commit `57a97c7e7821a5776cebc9bb87c984fa69cba8f1`.

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
