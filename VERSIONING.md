# Application version policy

This file owns version identity, declared development state, explicit bump behavior and ordering. [RELEASE.md](RELEASE.md) owns transactional allocation and publication; [ANDROID.md](ANDROID.md) owns Android numeric update ordering.

## Forms

| Context | Declared/product identity | Official version tag |
| --- | --- | --- |
| Repository development state | `MAJOR.MINOR.PATCH-dev.0` | Forbidden |
| Stable | `MAJOR.MINOR.PATCH` | `vMAJOR.MINOR.PATCH` |
| Published beta | `MAJOR.MINOR.PATCH-dev.N` | `vMAJOR.MINOR.PATCH-dev.N` |
| Local artifact | `MAJOR.MINOR.PATCH-local.N` | Forbidden |

Core fields MUST be nonnegative integers without leading zeroes; N in local artifacts and published betas MUST be a positive integer without leading zeroes. Reserved `dev.0` denotes the repository development cycle, not a published beta. An official tag named `vMAJOR.MINOR.PATCH-dev.0` is forbidden; a beta tag `vMAJOR.MINOR.PATCH-dev.N` points to committed source whose declaration remains `-dev.0`. Stable and beta product versions MUST use the forms above, without build metadata. A platform that cannot encode these strings MUST document a lossless association from its package version fields to the product version and record both in the manifest. Tag prefixes are not part of SemVer.

The developer MUST explicitly select stable versions. There is no odd/even rule and no requirement to release intermediate development cores. Projects SHOULD describe their compatibility surface (APIs, file formats, user data and integrations) and choose major/minor/patch changes accordingly. Version zero denotes initial development; projects MUST document their compatibility promises there. See [SemVer 2.0.0](https://semver.org/spec/v2.0.0.html).

Projects MUST identify one authoritative application-version source and document all required platform metadata and generated mirrors. After an explicit bump, this source MUST declare the intended development version as `MAJOR.MINOR.PATCH-dev.0`. The development core is read from that declaration, not maintained as a second independently editable version. Stable preparation finalizes the explicitly selected core without a development suffix under [RELEASE.md](RELEASE.md). Publication MUST NOT automatically start a new development cycle or rewrite the declaration or development branch. After stable preparation, the declared stable version remains until a later explicitly requested version operation (a development bump or another stable preparation); publication of an older tagged commit MUST NOT overwrite newer development state.

## Independent operations

| Operation | Repository version state | Result |
| --- | --- | --- |
| Explicit version bump | Advances the core and commits `-dev.0` metadata | Local metadata-only commit; no tag/publication |
| Local build | Reads declared development version; leaves it unchanged | Unique `-local.N` artifacts; no commit prerequisite, tag or publication |
| Manual GitHub beta release | Reads committed development state; leaves `-dev.0` unchanged | Allocated immutable `-dev.N` tag and GitHub prerelease |
| Explicit stable release | Finalizes the selected stable version | Official release commit when needed and annotated tag; authorized tag push initiates the GitHub pipeline |

These operations MUST remain independent. A bump MUST NOT create tags, push, publish releases, trigger deployments or invoke a release operation. Ordinary builds and tests MUST NOT bump versions or create version commits. Agent authorization is governed by [SECURITY.md](SECURITY.md#agent-operations).

## Explicit version bump

Every adopting repository MUST provide and document a repository-owned operation supporting MAJOR, MINOR and PATCH increments. The script name and implementation language are project-specific; these standards supply no executable implementation. Recommended interface (including an illustrative dry-run flag):

```bash
./scripts/bump.sh patch "Start next development cycle"
./scripts/bump.sh minor "Prepare next minor release"
./scripts/bump.sh major "Prepare next major release"
./scripts/bump.sh --dry-run patch
```

The operation MUST follow the adopted compatibility policy. PATCH increments the current core's patch; MINOR increments minor and resets patch to zero; MAJOR increments major and resets minor/patch to zero. Existing prerelease fields are discarded, not incremented, and the resulting declaration MUST end in `-dev.0`. Explicit bumps MAY skip development cores without publishing a stable release; no odd/even convention applies.

Required transition examples (each starts independently from its left-hand input, including a legacy development declaration):

```text
0.5.3          -> 0.5.4-dev.0   (patch bump)
0.5.4-dev.12   -> 0.5.5-dev.0   (explicit patch bump)
0.5.4-dev.12   -> 0.6.0-dev.0   (minor bump)
0.5.4-dev.12   -> 1.0.0-dev.0   (major bump)
```

The operation MUST:

1. Refuse a dirty working tree by default, including staged changes, unstaged tracked changes and nonignored untracked files. Any project-specific explicit override MUST be documented and preserve unrelated work and index entries; it MUST NOT weaken the selective-commit requirement.
2. Update the authority and all required platform metadata/mirrors coherently. Validate the resulting version, compatibility policy and metadata mappings before creating a commit. A bump is not an official reservation and MUST NOT consume beta counters or production package-code reservations.
3. Accept an optional custom commit description and generate a sensible default when omitted, for example `chore: bump version to 0.5.4-dev.0`. Treat the description as data.
4. Stage and create a local Git commit containing only necessary version metadata changes made by this operation. Never stage or commit unrelated changes, including files created by validators/hooks or concurrent edits. Revalidate HEAD, the index and intended diff before committing; concurrent changes MUST cause safe refusal unless isolation preserves this guarantee.
5. Provide a dry-run that reports the before/after version, affected metadata, validation and commit plan without modifying files, the index, refs, counters or remote state.
6. Fail visibly on invalid input, inconsistent metadata, validation, staging or commit failure. Document recovery: preserve pre-existing/concurrent work, undo only owned changes or leave an explicitly reported consistent resumable checkpoint. Report whether a commit was created; retries MUST reconcile that state rather than blindly increment again. No automatic destructive reset or cleanup of unrelated files is permitted.

## Generated identifiers and counters

Local builds MUST derive their core from the repository's declared version and use `MAJOR.MINOR.PATCH-local.N`. If the declaration is still stable, local builds MAY use that same core without starting a new development cycle; selecting the next development core requires an explicit bump. Local builds never require a new commit. Local builds MUST NOT modify declared version metadata or the published beta counter. Source identity, input digest and dirty-state traceability remain governed by [BUILD.md](BUILD.md#local-procedure) and [ARTIFACTS.md](ARTIFACTS.md).

Local N MUST be a locally managed unique build counter, independent of beta numbering. Projects MUST document its allocation scope; counters MAY be scoped per core and machine, but allocation MUST be collision-safe across concurrent builds and workspaces sharing artifact storage. Within that scope and core, local version identifiers MUST NOT be reused, including after artifact deletion. Counter state MUST survive artifact retention/cleanup; a reset or lost counter MUST recover a safe high-water mark or fail visibly rather than reuse identifiers. The distinct build ID, source digest and metadata required by [ARTIFACTS.md](ARTIFACTS.md) remain mandatory even if the display counter is machine-scoped.

New beta reservations MUST use committed source declaring `MAJOR.MINOR.PATCH-dev.0`; retries of historical reservations keep their original source under [RELEASE.md](RELEASE.md#failure-retries-and-cancellation). Published beta N MUST be allocated by the GitHub release workflow independently of local counters and MUST start at 1 for an eligible core without prior reservations. Subsequent allocations obey the high-water and recovery rules below; failed reservations remain consumed. A beta MUST NOT rewrite the committed `-dev.0` declaration: its package version is a build-time override recorded alongside the tagged source identity. Beta publication and stable tags MUST NOT reset local counters or automatically bump the repository version.

## Precedence and forward progress

Comparisons MUST use SemVer precedence, never lexical sorting of whole version strings or publication timestamps. Core fields compare numerically; prereleases precede their matching stable core. Prerelease fields compare left to right: numeric fields numerically, nonnumeric fields by ASCII lexical order, numeric before nonnumeric, and a longer equal prefix after its shorter prefix. Build metadata has no precedence effect. The [SemVer specification](https://semver.org/spec/v2.0.0.html) defines the complete algorithm.

Examples required in project validation:

```text
0.9.7 < 0.10.0 < 1.0.0
0.9.5-dev.2 < 0.9.5-dev.10 < 0.9.5
0.9.5-dev.99 < 0.9.6-dev.1
0.9.5-dev.2 < 0.9.5-local.1 < 0.9.5
0.9.5-dev.2+one = 0.9.5-dev.2+two   (precedence only)
```

Local identifiers are not release progression. At the same core, `local` sorts after `dev` regardless of build time; local counters MUST NOT be used to infer official ordering. Local identifiers MUST NOT be published as official tags.

Within a repository there is one official forward-moving version stream across stable and beta. Every newly reserved official version MUST exceed all previously published official versions AND all earlier durable official reservations, including failed/abandoned releases (see [RELEASE.md](RELEASE.md#common-transaction-and-evidence) for the reservation boundary). A retry of the same reservation is exempt from being greater than itself. Local builds and temporary CI artifacts are excluded. This deliberate rule rules out publishing an older maintenance release after a newer core beta; projects needing multiple independent product streams need a separately adopted policy and cannot claim this rule is satisfied by silently splitting the stream.

The allocator MUST inspect complete remote official tags, releases and durable reservation/tombstone records, rather than a shallow checkout or local counter. Migrating repositories MUST inventory historical aliases and nonstandard tags so a naming change cannot erase the high-water mark. Unexpected or unparseable official records MUST block allocation pending reconciliation. Deleting a release MUST NOT free its identity.

For a beta core, select N greater than every reserved N for that core, using the initial allocation defined above. The complete version, not N alone, is globally unique within the repository. A collision MUST trigger revalidation and a fresh allocation, never tag replacement. Only committed source is eligible for official releases.

## Skipping and transitions

Skipping a core or N creates no implicit release; skipping N applies after the initial beta reservation at 1. Record the decision in the release record/notes when it affects the development plan. Once a higher core is reserved, a lower skipped core MUST NOT be filled in later. Stable selection may skip development cores, but MUST pass the same high-water check. Choosing `0.9.7` after `0.9.6-dev.1` is valid; choosing `0.9.5` then is invalid. Choosing `0.9.6` then is also valid if no higher reservation exists.

The following sequence illustrates repository state separately from generated identities. Each bump is explicitly requested; the local counter shown is scoped to the core and continues independently of beta publication.

| Action | Declared repository version afterward | Generated artifact / official identity |
| --- | --- | --- |
| Explicitly release stable | `0.9.4` | `v0.9.4` |
| Explicit patch bump and local metadata commit | `0.9.5-dev.0` | None |
| Two local builds, clean or dirty | `0.9.5-dev.0` | `0.9.5-local.1`, `0.9.5-local.2` |
| Two manual beta dispatches from committed source | `0.9.5-dev.0` | `v0.9.5-dev.1`, `v0.9.5-dev.2` |
| Another local build | `0.9.5-dev.0` | `0.9.5-local.3` |
| Explicit patch bump; skip stable 0.9.5 | `0.9.6-dev.0` | None |
| Manual beta dispatch | `0.9.6-dev.0` | `v0.9.6-dev.1` |
| Explicitly select and prepare stable 0.9.7 | `0.9.7` | Release commit and `v0.9.7`; notes start at `v0.9.4` |
| Publish stable from exact tagged commit | `0.9.7` | GitHub stable release; no automatic bump |
| Explicit patch bump, then local build | `0.9.8-dev.0` | Metadata-only bump commit, then `0.9.8-local.1` |
