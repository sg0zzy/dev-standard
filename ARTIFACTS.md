# Artifact identity, manifests and retention

## Local layout and state

All final local build artifacts MUST be collected under `local_artifacts/` at the project root. Projects MUST exclude it from source snapshots and apply the [Git exclusions](#git-exclusions) below. Each invocation MUST reserve a unique direct child directory using exclusive creation. A directory MAY hold multiple platforms. Staging outside this root MAY be used for tool intermediates; final local output belongs here.

```text
local_artifacts/
  0.9.5-local.21/
    app-0.9.5-local.21-android-arm64.apk
    app-0.9.5-local.21-linux-x86_64.AppImage
    build-info.json
    .keep                         # optional protection
  0.9.5-local.22/
    build-info.json
```

A managed build MUST have explicit `in_progress`, `succeeded` or `failed` state, unique build ID and a schema/manager marker. Completion metadata MUST be committed atomically only after artifacts and their verification are complete. A crash or lost process MUST NOT cause an in-progress directory to become collectible automatically; recovery is a separate explicit inspection. Completed artifacts MUST NOT be overwritten by subsequent builds. Retention below is an explicit, bounded deletion policy, not an overwrite exception.

## Git exclusions

Each project's root `.gitignore` MUST include the following directory patterns. `/local_artifacts/` applies at the project root; the other patterns exclude dependency, cache and build-output directories at any depth. Effective ignore rules MUST NOT re-include their contents, and files under these directories MUST NOT remain tracked.

```gitignore
/local_artifacts/
node_modules/
dist/
__pycache__/
target/
```

Adding an ignore pattern does not untrack existing files. Adoption MUST check the Git index and remove any already-tracked files in these directories from the index while preserving local files. Git ignore status alone does not define source snapshot scope: relevant ignored inputs remain subject to the [metadata contract](#metadata-contract).

## Metadata contract

Every successful local directory MUST contain a manifest (recommended filename `build-info.json`). Failed/in-progress builds MUST retain enough state for cleanup to exclude them. The manifest MUST record:

- Manifest schema and manager identity, unique build ID, product version, development core, channel, state, UTC start/completion timestamps and a durable completion sequence used for retention.
- Full Git commit SHA, dirty indicator, submodule source identities where used, and a source snapshot digest plus algorithm and scope description. A SHA alone identifies committed history, not dirty source.
- The scope MUST cover tracked modifications/deletions, relevant untracked files and every other source/configuration input used, including relevant ignored inputs. Use a canonical path/content inventory or equivalent snapshot hash; document exclusions such as secrets, caches and outputs. Secret contents MUST NOT be copied into the manifest or snapshot archive; record nonsecret dependency/key identifiers where needed. Different effective dirty sources MUST have different recorded identities.
- Build environment: OS/architecture, compiler/runtime/SDK/build tool versions, dependency lock identity and relevant options. Container image digest SHOULD be included when used. Avoid usernames, absolute private paths and secret environment values.
- Per-target requiredness and outcomes, artifact relative paths, byte sizes, package versions and signing identity fingerprints where relevant. Failure/skipped reasons MUST be visible.

Manifests SHOULD record SHA-256 for each artifact; official release checksums are mandatory under [RELEASE.md](RELEASE.md). Artifact names SHOULD contain project, version, platform and architecture where relevant. Filenames and manifest paths MUST NOT escape the build directory.

CI/release manifests MUST use the applicable fields above, including exact source, environment and target results; clean official builds record `dirty: false` and tagged SHA. CI identities need not use a local completion sequence. Published manifests MUST omit secrets and private source archives.

## Automatic local retention

The effective limit MUST default to **10 most recent successful, unprotected builds**. Projects MAY explicitly configure another nonnegative integer; zero means keep no unprotected successful builds, including the just-completed one. The chosen value and consequences MUST be documented. A `.keep` entry protects a build regardless of its contents; protected builds do not count against the limit.

Garbage collection MUST run only after a successful build, and MUST retain the newest effective-limit successful unprotected managed directories. It MUST NOT delete anything because a build failed. It MUST NOT delete active, failed, unknown, malformed or protected directories automatically. A successful build MAY be pruned later by this policy; users needing permanent preservation SHOULD add `.keep` before the next collection.

Order MUST use reliable completion metadata, preferably a sequence allocated under the storage lock at successful completion, with UTC timestamps for explanation. Directory names, filesystem modification times, counters allocated at build start and clock time alone are insufficient to order concurrent completions. Missing/invalid sequences or duplicate ambiguous identities MUST cause those directories to be excluded and reported, never guessed at.

Creation, completion, protection changes through the project interface, and cleanup MUST coordinate using a lock or equivalent transaction across every process sharing the root. Cleanup MUST revalidate protection, state, ownership and path immediately before deletion. Active builds MUST remain excluded even when long-running. Out-of-band `.keep` creation MUST be respected if observed before deletion; projects MUST document the locking interface for race-free protection and SHOULD provide a protect command. A raw filesystem edit during an active deletion cannot be promised atomic protection.

Deletion MUST be restricted to positively validated managed direct children of the canonical `local_artifacts/` root. Reject symlinked roots/children, traversal, external paths, symlink-following recursive deletion, malformed manifests and substituted directories. Safely unlinking an internal symlink MAY be allowed only if its target is never traversed. Use race-resistant path/identity checks while holding the lock. If validation is uncertain, skip and report. Cleanup MUST NOT follow a manifest-supplied deletion path.

## Manual cleanup and recovery

Projects MUST document a manual cleanup procedure with a nonmutating dry-run reporting candidate paths, reasons, identities, protection, and retained/skipped builds. Actual deletion MUST replan/revalidate under the lock; a dry-run is not a permanent authorization for stale paths. Manual deletion MUST retain the same containment/ownership protections. Removing `.keep`, abandoned/in-progress or failed builds requires explicit selection and confirmation in the project's interface; automatic retention MUST NOT do it.

Illustrative procedure: stop or inspect abandoned jobs; run `<project-tool> artifacts clean --dry-run`; review candidates; run the documented apply operation; inspect a deletion audit/summary. Projects SHOULD report cleanup failure separately from compilation success, return a clear maintenance failure, and preserve unvalidated directories. A cleanup failure MUST NOT retroactively claim compilation failed or erase successful output metadata.
