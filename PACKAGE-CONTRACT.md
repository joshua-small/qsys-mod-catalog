# Package and repository contract

## Repository-owned presentation

Place these files in each mod repository before creating its release tag:

- `catalog/mod.json`: schema 1, `id`, canonical `name`, `description` (1–4000 characters), GitHub username `author`, repository-relative `thumbnail` and `helpUrl` within that repository.
- `catalog/<image>.png`, `.jpg`, `.jpeg` or `.ico`: at most 1 MiB. The publisher pins its content commit and hash.
- `catalog/releases/<tag>.md`: 1–12000 characters of UTF-8 release notes. Notes are displayed verbatim as plain text.

Use release tags such as `v1.2.3`; the module and manifest use the corresponding four-part assembly version, for example `1.2.3.0`. Publish one package asset per stable GitHub release. Drafts and prereleases are not collected. Presentation and notes can change in later releases; a published record cannot be replaced.

## Package contents

A `.qdsmod` is a ZIP containing exactly `manifest.json` and the files listed as `payload/<flat-filename>`. No scripts, paths, extraction instructions, Core payloads or vendor binaries are accepted.

A modern manifest has:

```json
{
  "schema": 2,
  "id": "example",
  "name": "Example",
  "version": "1.2.3.0",
  "description": "Description owned by the mod author.",
  "hostSha256": "<approved Q-Sys Designer.dll SHA-256>",
  "commonSha256": "<approved ModuleCommon.dll SHA-256>",
  "files": [
    {
      "name": "Designer.Modules.Example.dll",
      "sha256": "<module SHA-256>",
      "length": 12345,
      "shared": false
    }
  ]
}
```

The ID, canonical name, source repository and exact file set must match a catalog approval. The module's assembly name and version must match its filename and manifest. The currently supported target remains the exact approved Q-SYS Designer 10.5 build; declaring a different hash does not add compatibility.

Modern packages have **no enable marker**. QMM enables a mod by placing its managed module in the selected installation; disabling or suspending removes its private files while retaining the package for restoration. A mod must retain its own exact-build runtime checks.

Harmony is optional. A mod that needs it declares the approved shared DLL and license file; the two existing mods use their current pinned Harmony bytes and license. A mod that does not need shared files can contain only its own module. All packages sharing a filename must use the same approved hash. QMM preserves shared files when removing a mod and refuses to overwrite a different shared version.

Additional private files must use the `Qmm.<mod-id>.` namespace; additional shared files use `Qmm.Shared.` and an approved hash. Known vendor module names are reserved. Each approved layout is an exact file set, not a wildcard permission. Paths, duplicate names, unsupported members and undeclared files are rejected. Existing approved layouts remain available for old version pins.

Limits: 24 MiB archive, 16 MiB combined expanded payload, 8 MiB per payload file, 64 KiB manifest, at most 32 payload files. The catalog currently bounds 256 approved mods and 100 releases per mod; reaching these explicit bounds requires raising and validating them, not hand-maintaining mod code.

## Build and verify

The manager repository supplies the packer and publisher, pinned to a reviewed commit. Keep vendor references local; do not commit or redistribute Q-SYS binaries.

```text
dotnet run --project Pack/QdsModManager.Pack.csproj -- <id> <payload-directory> <new-output.qdsmod> <approvals.json> <catalog/mod.json>
```

The packer selects the last approved schema-2 layout, hashes each file, checks the module metadata and validates the finished archive. It never includes an activation marker. If a mod needs a different layout, obtain approval before publishing it.

Package import remains available for separately supplied archives that match an approved identity and layout. Import verifies structure, all payload hashes, assembly identity, shared hashes and exact compatibility; it does not establish that the archive was published by its claimed author. Downloads from the catalog additionally verify the approved repository address, archive size and archive hash.

The publisher compares the release record with the package and with the mod repository's immutable source files. Collection repeats these checks before appending the version to the signed feed. Historical package pins and receipts cannot be changed or removed by routine automation.

