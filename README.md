# Approved Q-SYS Mod Catalog

This repository owns the approval list and aggregated release catalog for [Q-SYS Mod Manager](https://github.com/joshua-small/qsys-mod-manager). The manager software and publishing tools live in that separate repository. Each mod repository owns its description, author identity, artwork and release notes.

## Add a mod

1. Prepare a repository with `catalog/mod.json`, an image and `catalog/releases/<tag>.md`, plus a schema-2 `.qdsmod` package. Follow [the package contract](PACKAGE-CONTRACT.md).
2. Fork this repository and submit a pull request adding your mod to [approvals.json](approvals.json). Declare the canonical ID/name/repository, module filename and exact permitted file layouts. Shared files require pinned SHA-256 hashes. Include license information, an initial package and evidence for the supported Designer build using the PR template.
3. A catalog maintainer reviews the source identity, license, file ownership, compatibility evidence and initial package. Automated checks reject unsafe paths, conflicting files, unsupported layouts and changes that would break historical restoration. A new supported mod does **not** require changing or rebuilding QMM.
4. After approval is merged, the scheduled collector publishes the new signed approval data. Add the [pinned reusable publisher workflow](https://github.com/joshua-small/qsys-mod-manager/blob/codex/repository-owned-catalog/.github/workflows/publish-mod-catalog.yml) to your repository; use the exact tooling commit recorded in this repository's workflow.
5. Publish stable GitHub releases containing exactly one `.qdsmod` asset. The publisher validates your source-owned content and attaches an immutable `qmm-catalog.json` record. The hourly collector verifies that record, its source commit, artwork and package, then publishes it automatically. Maintainers do not hand-edit each version or approve every routine release.

Release automation never executes code from mod repositories and never changes the approval list. A new file layout, repository identity or supported Designer build is a separate review. Package publication is verification against declared constraints, not a claim that every behavior has been tested by the catalog maintainer.

## Repository structure and trust

- **main / approvals.json:** editable approval policy, changed through reviewed pull requests. CODEOWNERS and branch protection require maintainer review; repository administrators retain their explicit override.
- **published / catalog/:** generated index, presentation, immutable source receipts, approved definitions and signed `feed.json`. The collector writes only this branch with a normal fast-forward push.
- **Manager repository / catalog/:** frozen offline/bootstrap snapshot and legacy endpoint for older managers. It is not a second live package registry.
- **Mod repositories / catalog/:** author-maintained copy, images and version-specific notes. No summaries or README scraping are performed.

The feed is signed with a dedicated ECDSA P-256 key. QMM verifies the signature before accepting catalog or cached approval data, and its elevated worker verifies the same signed snapshot carried with the preview. The public key is checked into both repositories; the private key exists as the catalog repository's `CATALOG_SIGNING_KEY` Actions secret. PR jobs cannot access it. Only the trusted main-branch collector signs. A key replacement requires updating QMM's embedded public key and its bundled signed fixture before publishing feeds with that key.

New managers fetch [the signed feed](https://raw.githubusercontent.com/joshua-small/qsys-mod-catalog/published/catalog/feed.json). Offline use relies on the last verified signed snapshot or the manager's bundled bootstrap approvals. A new mod must have been learned from a signed catalog before its separately supplied package can be imported offline.

## Historical releases and migration

Approved IDs, repositories and existing file layouts are retained. New layouts may be added; incompatible permission changes cannot silently invalidate old packages. Published package bytes, version pins and source receipts are immutable. Keep all historical release assets so exported profiles remain resolvable.

The initial catalog carries forward the two existing approved packages and their verified hashes. Their original releases predate repository-owned metadata, so their migrated text remains a bootstrap snapshot until the authors publish descriptors or new releases. New descriptors must match repository-owned content at an immutable commit. The explicit publisher `content_ref` input supports older tags without moving those tags or replacing packages.

Schema-1 historical packages still include their old activation markers because those binaries require them. Schema-2 packages do not. A QMM update removes an old marker only when it belongs to the managed previous package; undo can restore that complete historical version. User and unrelated files remain untouched.

## Operations

The collector runs hourly and can be started with **Actions → Approved catalog → Run workflow** on main. Validation failure leaves the published catalog unchanged and appears as a failed Actions run. An unchanged collection produces no commit or new signature. Concurrent publication fails without overwriting another writer.

Approval is not a malware guarantee. Reports about an approved mod should be filed in this repository and the mod's source repository. Revocation policy and emergency blocking require an explicit design because deleting a mod's approval would also prevent restoring existing profiles; the current contract preserves approvals and pins.

