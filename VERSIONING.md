# Application version policy

This file owns version identity and ordering. [RELEASE.md](RELEASE.md) owns transactional allocation and publication; [ANDROID.md](ANDROID.md) owns Android numeric update ordering.

## Forms

| Context | Package/product identity | Official tag |
| --- | --- | --- |
| Stable | `MAJOR.MINOR.PATCH` | `vMAJOR.MINOR.PATCH` |
| Published beta | `MAJOR.MINOR.PATCH-dev.N` | `vMAJOR.MINOR.PATCH-dev.N` |
| Local | Prefer `MAJOR.MINOR.PATCH-local.N` | Forbidden |

Core fields MUST be nonnegative integers without leading zeroes; N MUST be a positive integer without leading zeroes. Stable and beta product versions MUST use the forms above, without build metadata. A platform that cannot encode these strings MUST document a lossless association from its package version fields to the product version and record both in the manifest. Tag prefixes are not part of SemVer.

The developer MUST explicitly select stable versions. There is no odd/even rule and no requirement to release intermediate development cores. Projects SHOULD describe their compatibility surface (APIs, file formats, user data and integrations) and choose major/minor/patch changes accordingly. Version zero denotes initial development; projects MUST document their compatibility promises there. See [SemVer 2.0.0](https://semver.org/spec/v2.0.0.html).

Projects MUST identify one authoritative application-version source. A development base is a proposed future core, not an automatically incremented stable release. It MAY be derived or configured separately from the last stable version, with its role documented. Local and beta build overrides MUST NOT rewrite the authoritative stable version as a side effect.

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

Local identifiers are not release progression. At the same core, `local` sorts after `dev` regardless of build time; local counters MUST NOT be used to infer official ordering. A project MAY choose another clearly local prerelease label, but MUST document its ordering and MUST NOT publish it as an official tag.

Within a repository there is one official forward-moving version stream across stable and beta. Every newly reserved official version MUST exceed all previously published official versions AND all earlier official tag reservations, including failed/abandoned releases. A retry of the same reservation is exempt from being greater than itself. Local builds and temporary CI artifacts are excluded. This deliberate rule rules out publishing an older maintenance release after a newer core beta; projects needing multiple independent product streams need a separately adopted policy and cannot claim this rule is satisfied by silently splitting the stream.

The allocator MUST inspect complete remote official tags, releases and durable reservation/tombstone records, rather than a shallow checkout or local counter. Migrating repositories MUST inventory historical aliases and nonstandard tags so a naming change cannot erase the high-water mark. Unexpected or unparseable official records MUST block allocation pending reconciliation. Deleting a release MUST NOT free its identity.

For a beta core, select N greater than every reserved N for that core; N MAY start at 1 for a new greater core. The complete version, not N alone, is globally unique within the repository. A collision MUST trigger revalidation and a fresh allocation, never tag replacement. Only committed source is eligible for official releases.

## Skipping and transitions

Skipping a core or N creates no implicit release. Record the decision in the release record/notes when it affects the development plan. Once a higher core is reserved, a lower skipped core MUST NOT be filled in later. Stable selection may skip development cores, but MUST pass the same high-water check. Choosing `0.9.7` after `0.9.6-dev.1` is valid; choosing `0.9.5` then is invalid. Choosing `0.9.6` then is also valid if no higher reservation exists.

```text
0.9.4                  explicit stable
0.9.5-local.1          dirty or clean local build; excluded from high-water mark
0.9.5-dev.1            committed beta reservation and publication
0.9.5-dev.2            next beta
0.9.6-dev.1            development base advanced; no stable 0.9.5
0.9.7                  explicitly selected stable; notes start at v0.9.4
0.9.8-local.1          next development base; no official reservation
```

Local numbering MUST be independent of beta numbering. Local allocation MUST be collision-safe across concurrent builds and workspaces sharing artifact storage; a durable counter or collision-resistant build identifier MAY be used. A reset MUST NOT reuse an existing build identity. The distinct build ID, source digest and metadata required by [ARTIFACTS.md](ARTIFACTS.md) remain mandatory even if the display counter is machine-scoped.
