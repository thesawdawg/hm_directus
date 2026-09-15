# Hybrid Mic patch ledger

Every change from upstream `v11.17.4` is recorded here before release. Security details that would create avoidable
exposure stay in a private advisory until coordinated disclosure; this ledger receives the public reference afterwards.

| Project release       | Upstream reference                                                                         | Impact and changed behavior                     | Regression proof                                                    | Rollback                                                                 |
| --------------------- | ------------------------------------------------------------------------------------------ | ----------------------------------------------- | ------------------------------------------------------------------- | ------------------------------------------------------------------------ |
| baseline (unreleased) | [`directus/directus@v11.17.4`](https://github.com/directus/directus/releases/tag/v11.17.4) | Exact upstream source; no Hybrid Mic core patch | Upstream tag resolves to `1f9a051041029e59b17a1a36249f232e59e60e9b` | Use upstream `directus/directus:11.17.4` with a compatible data snapshot |

## Baseline security gate

A fresh API snapshot taken 2026-09-07 reported 200 open dependency alerts: 5 critical, 74 high, 102 medium, and 19 low.
The earlier counts in spec 019 and prior ledger drafts (193/6/72/96/19) were point-in-time snapshots and are stale; the
fresh count is the current reference. No project image may be released until runtime reachability and available patches
are triaged and the release image passes the blocking Trivy scan. The imported upstream image remains the rollback
reference; no Hybrid Mic fork image exists yet.

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

## Security batch 1 — seven approved runtime dependency upgrades

_Committed on branch `fix/security-alerts-batch-1`, based on `910fff0d8b`; no tag, push, or release made.
Changeset: `.changeset/security-batch-1.md` (`@directus/api: patch`)._

Catalog and override bumps applied via `pnpm-workspace.yaml` catalog and `package.json` pnpm overrides; lockfile
regenerated with `pnpm install --lockfile-only` (no hand-edit). All seven targets are published >7 days and resolve
exactly as approved.

| Package  | Old (installed) | New (approved) | Advisory scope                                                                             | Hybrid Mic exposure                                                                                         |
| -------- | --------------- | -------------- | ------------------------------------------------------------------------------------------ | ----------------------------------------------------------------------------------------------------------- |
| tar      | 7.5.13          | 7.5.21         | Combined floor 7.5.21 for marketplace tar extraction criticals                             | Runtime: marketplace extension `InstallationManager.install` calls `tar.extract` on downloaded tarballs     |
| liquidjs | 10.25.7         | 10.27.1        | Advisory metadata says no patch, but fix commit `3129d46` is ancestor of v10.26.0/v10.27.1 | Runtime: mail template rendering via Liquid; default sandbox marketplace available, no custom template path |
| axios    | 1.15.0          | 1.16.0         | High-severity request/files advisories                                                     | Runtime: `getAxios` request helper, file import from URL, deployment drivers                                |
| mysql2   | 3.15.3          | 3.22.0         | High-severity MySQL client advisories                                                      | Runtime: supported MySQL 8.4 deployment via Knex                                                            |
| ws       | 8.18.3          | 8.21.0         | High-severity WebSocket advisories                                                         | Runtime: realtime WebSocket API and collab                                                                  |
| samlify  | 2.12.0          | 2.13.0         | SAML auth driver advisories                                                                | Not deployed (no SAML in Hybrid Mic), but driver ships in the image                                         |
| undici   | 7.24.5          | 7.29.0         | High-severity HTTP client advisories                                                       | Runtime: Cloudinary and Supabase storage drivers                                                            |

### Resolution proof

- `pnpm install --lockfile-only` succeeded with no further diff; `pnpm install --frozen-lockfile` succeeded.
- All seven targets linked to the approved versions in `api/node_modules/` and
  `packages/storage-driver-*/node_modules/`.
- Old versions remain in the pnpm store but are no longer linked to any importer.

### Production bundle proof

`pnpm --filter directus deploy --legacy --prod dist` built successfully. `dist/node_modules` contains exactly:

- tar 7.5.21, axios 1.16.0, liquidjs 10.27.1, mysql2 3.22.0, ws 8.21.0, samlify 2.13.0, undici 7.29.0.
- Old vulnerable versions (tar 7.5.13, axios 1.15.0, liquidjs 10.25.7, mysql2 3.15.3, ws 8.18.3, samlify 2.12.0, undici
  7.24.5) are absent from the prod bundle.
- Residual: `ws@7.5.10` ships via `pm2@6.0.14` / `@pm2/js-api` / `@pm2/agent`; it is outside the seven-target
  scope but inside this production bundle. `ws@8.18.3` remains a dev-only transitive (happy-dom, jsdom) and does not
  ship in the prod bundle.

### Tests run

Unit tests (vitest, Node 22, pnpm 10.27.0):

- tar/marketplace: `api/src/extensions/lib/installation/manager.test.ts` — 4 passed.
- liquid/mail: `api/src/services/mail/index.test.ts`, `api/src/mailer.test.ts`, `api/src/operations/mail/index.test.ts`,
  `api/src/operations/mail/rate-limiter.test.ts`, `api/src/services/mail/rate-limiter.test.ts` — 28 passed.
- axios/request: `api/src/request/index.test.ts`, `api/src/operations/request/index.test.ts` — 7 passed.
- mysql: `api/src/database/helpers/schema/dialects/mysql.test.ts` — 10 passed.
- ws/websocket: `api/src/websocket/` authenticate, errors, handlers, collab suites — 212 passed.
- undici/cloudinary: `packages/storage-driver-cloudinary` — 84 passed.
- undici/supabase: `packages/storage-driver-supabase` — 41 passed.
- samlify: no unit test for `api/src/auth/drivers/saml.ts`; import sanity check confirmed samlify 2.13.0 loads and
  exports `IdentityProvider`, `ServiceProvider`, etc.

Blackbox (test-only MySQL 8.4 containers, never the Hybrid Mic live stack):

- The initial run used a stale four-day-old test database: `--project db` had one `case-when.test.ts` failure
  ("Could not create user"). `--project common` also failed because the OpenLDAP service was not running. Neither result
  is accepted as regression proof.
- Sawyer approved a destructive reset of blackbox containers/volumes. The reset was performed; MySQL bootstrapped and
  connectivity passed. The full DB matrix then exceeded a hard 15-minute bound before reporting a test summary.
- Sawyer approved two bounded single-file attempts for the prior `case-when.test.ts` failure. Both timed out before
  reporting: the first because the shell ran Node 24 while the deployed bundle's native modules were compiled for
  Node 22 (`isolated-vm` NODE_MODULE_VERSION mismatch), and single-file runs also hang in the vitest environment's
  `tests_flow_completed` wait loop because the sequencer's prerequisite list is never satisfied.
- On 2026-09-08 the prior failure was reproduced clean: under Node 22.23.2 with the sequencer temporarily pinned to
  the required chain (seed-database → common → case-when), `TEST_DB=mysql` on the fresh database passed 3 files /
  24 tests in ~39s, including `case-when.test.ts`. The sequencer pin was reverted immediately after; no test-infra
  change is part of this diff.
- OpenLDAP itself exits before tests because its image cannot write `/var/lib/openldap/openldap-data` on this host.
  Sawyer deferred that fixture repair to mission `AD-directus-openldap-fixture`.
- The specific prior failure is proven clean on a fresh database. The full blackbox matrix (all db tests + common
  project, which needs OpenLDAP) has not run to completion and remains a release-candidate gate.

Repository checks:

- `pnpm install --frozen-lockfile`: passed on Node 22.23.2 / pnpm 10.27.0.
- Repo-wide `pnpm lint`, `pnpm lint:style`, and `pnpm format`: passed.
- Full `pnpm build`: passed.
- Full `pnpm test` completed but its SQLite E2E portion had six service-fixture failures (MinIO file/assets, mail response,
  and storage-health 503). The exact base commit `910fff0d8b` was installed/built in a separate worktree and reproduced
  the identical four failing files / six failing tests. The failures are therefore pre-existing host fixture failures,
  not introduced by this dependency diff; repair belongs to the test-fixture mission.
- API and app builds also passed in the initial targeted run. `tsc --noEmit` on api reported pre-existing errors only;
  no new typecheck regression was identified.

### Image and scanner proof

- Local candidate image `hm-directus:security-batch-1-candidate` built successfully (not a release tag, not pushed).
  Image contains all seven patched versions verified via `docker run`.
- No local Trivy/Syft/Grype scanner installed. The blocking Trivy scan (`severity: HIGH,CRITICAL`,
  `ignore-unfixed: true`, `exit-code: 1`) runs only in `hybrid-mic-release.yml` on the immutable release tag in CI. **CI
  evidence still needed: multi-architecture build, SBOM, Trivy HIGH/CRITICAL gate, and provenance attestation from the
  release workflow on the approved commit.**

### Rollback

No Hybrid Mic fork image has been released yet. The upstream `directus/directus:11.17.4` image remains the rollback
reference. To revert this batch, restore `package.json`, `pnpm-workspace.yaml`, and `pnpm-lock.yaml` to commit
`910fff0d8b` and reinstall. Database migrations are unaffected (no schema changes).

### Residual findings requiring remediation/decision

- `ws@7.5.10` ships in this candidate image via `pm2@6.0.14` / `@pm2/js-api` / `@pm2/agent` (runtime process manager).
  It is outside the seven-target scope of batch 1 but inside the production bundle. Sawyer explicitly chose isolated
  PM2/ws7 remediation before release; this is not a risk acceptance — it requires a separate fix before the release
  gate.
- vitest + shell-quote remain dev/build cleanup; they do not ship in the prod bundle. Non-shipped proof is pending the
  CI scanner gate; this is not accepted as production risk.
- sharp 0.35.x, TinyMCE 7, Nodemailer 9 — approved as separate isolated upgrade batches after batch 1; not remediated
  here.
