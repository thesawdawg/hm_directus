# Hybrid Mic Directus 11 maintenance policy

## Purpose and support boundary

This repository is the project-owned maintenance fork for Hybrid Mic. It stays on Directus 11 because the current typed
content model exceeds the unlicensed collection limit introduced by Directus 12. The fork is deliberately narrow:

1. Security fixes have first priority.
2. Defects that block the supported Hybrid Mic deployment have second priority.
3. Hybrid Mic features belong in Directus extensions or the application, not in core.
4. Unrelated Directus feature development is out of scope.

Upstream history, `BUSL-1.1` license files, authorship, and notices must remain intact. A backport retains its upstream
attribution and uses `cherry-pick -x`.

## Branches, remotes, and releases

- `origin`: `https://github.com/thesawdawg/hm_directus.git`
- `upstream`: `https://github.com/directus/directus.git`
- default branch: `11.x-hybrid-mic`
- baseline: upstream `v11.17.4`, commit `1f9a051041029e59b17a1a36249f232e59e60e9b`
- release tags: `11.<minor>.<patch>-hm.<revision>`
- container: `ghcr.io/thesawdawg/hm_directus:<release tag>` and its digest

Set up a maintainer checkout with:

```bash
git remote add upstream https://github.com/directus/directus.git
git fetch --tags upstream
git switch 11.x-hybrid-mic
```

Tags and GHCR digests are immutable release identities. Do not use `latest`, a major/minor floating tag, or the
maintenance branch in Compose. A release is not complete until its digest, SBOM, scan, provenance, source commit, build
metadata, test result, and rollback image are recorded.

## Backport procedure

1. Record the advisory, upstream issue/PR/commit, affected versions, severity, and Hybrid Mic exposure in
   `PATCH_LEDGER.md`.
2. Reproduce the defect on the prior project image and add a regression test that fails there.
3. Prefer the smallest upstream patch. Use `git cherry-pick -x <commit>` when possible; explain any adaptation in the
   commit and ledger.
4. Run source checks, MySQL bootstrap/blackbox tests, the Hybrid Mic schema provisioner twice, extension/preview
   contracts, application tests, and the portable restore drill.
5. Review the complete diff against the prior release; verify licenses and generated lockfile changes.
6. Merge only after the regression test and existing checks pass.
7. Create an annotated release tag only from `11.x-hybrid-mic`, then monitor the GHCR workflow and record its digest in
   the ledger.
8. Deploy by digest. Retain the prior digest and a compatible portable export until the new image completes its smoke
   window.

Database migrations are forward changes. Rollback means restoring the pre-change portable export and starting the prior
immutable image; never assume that starting an older image reverses a migrated database.

## Required release proof

Every candidate must prove:

- a clean, reproducible multi-architecture image build;
- an empty MySQL 8.4 bootstrap and supported migration path;
- Directus schema provisioning twice without drift;
- all 31 Hybrid Mic collection and permission contracts;
- extension loading with `@directus/extensions-sdk` 17.1.4;
- Visual Editing compatibility with `@directus/visual-editing` 2.0.1;
- authenticated preview/versioning, assets, and YouTube import;
- restore of a representative portable ZIP followed by public/preview smoke;
- application tests and browser smoke against the candidate image;
- SBOM, high/critical vulnerability scan, and build provenance;
- rollback with the prior image and a compatible restored snapshot.

The release workflow publishes only the exact release tag and a commit tag to GHCR. It does not publish npm packages,
Docker Hub images, `latest`, or moving major/minor aliases.

## Maintenance cadence

- Weekly: review Directus advisories, upstream 11 commits, relevant fixes from newer majors, Node 22 security releases,
  CodeQL, and transitive advisories.
- Monthly: rebuild the candidate on the current approved Node 22 Alpine base, even when no fork source changed, and
  compare the resulting scan/SBOM.
- Quarterly: run the full portable restore and rollback drill and reassess the cost/risk of maintaining 11 versus a
  supported licensed major upgrade.
- Emergency: privately reproduce and patch an undisclosed vulnerability; do not publish exploit detail before a
  coordinated fix is available.

Security and deployment-blocking work may ship outside the cadence. A clean scan is not evidence by itself: record
accepted findings and compensating controls explicitly in the patch ledger.

## Compatibility metadata

The GHCR workflow records the source commit, build time, Node and pnpm versions, lockfile hash, Directus version, image
digest, extension SDK version, and Visual Editing version in `build-metadata.json`. The file, SBOM, and scan are
attached to the GitHub release. The Hybrid Mic application export records the deployed image digest so a data archive
can be matched to its server build.
