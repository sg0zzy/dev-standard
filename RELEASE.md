# Beta and stable releases

Version forms and the repository-wide high-water rule are owned by [VERSIONING.md](VERSIONING.md). Declared development state, counter independence and the separate bump operation are defined in [VERSIONING.md](VERSIONING.md#independent-operations). Permissions are owned by [SECURITY.md](SECURITY.md). A release is complete only after its mandatory build, validation and asset gates pass; external delivery is separate.

## Common transaction and evidence

Projects MUST implement an observable state model equivalent to:

```text
requested -> validated -> tag_reserved -> building -> staged -> published
                    failures -> failed/abandoned, retaining reservation and evidence
published -> external delivery pending/succeeded/failed (separate destination state)
```

Each release record MUST identify requested channel/version, exact source SHA, immutable tag, standards/config revision, mandatory/optional target outcomes, workflow/run attempt, and failure/recovery status. Records MUST survive failed jobs. An official remote tag is a permanent version reservation even if no Release exists. A local unpushed preparation tag is provisional and may be rolled back under the preparation rules below. Once an identity is recorded in coordinated durable reservation state or an official remote tag, it remains consumed even if the local tag is removed or remote tag creation fails. Published assets and their identities MUST NOT be silently replaced.

All beta and stable allocation paths, including local stable preparation and retries, MUST coordinate against the same remote reservation state. Repository-wide serialization with revalidation is recommended; atomic create-only remote tag creation MUST enforce the final uniqueness check. Never use force-push or an update-tag operation to allocate. A local lock or a concurrency group used by only one of several release workflows is insufficient. Compare-and-create alone prevents identical tags, but does not ensure cross-version publication ordering.

Before making a release visible, a serialized publication gate MUST re-read published state and reject any first publication that is not greater than the last published version. New reservations MUST pass the stronger high-water rule. An older reserved job MAY finish building after a newer release, but MUST then remain failed/abandoned rather than publish out of order. A project MAY serialize the entire release lifecycle instead. Local stable preparation cannot lock remote state indefinitely; remote reservation MUST revalidate and fail safely if the state advanced. Completed-publication retries MAY verify the existing identical release without republishing it.

Each release MUST build from the exact commit peeled from its official tag, using clean committed sources and recorded dependencies. Beta/stable SHOULD reuse platform packaging implementations. The manifest MUST verify product/package versions, source, expected filenames, sizes, checksums and signing requirements. Every mandatory artifact MUST exist and pass verification before publication; unconfigured targets are not successes. Optional omissions MUST be explained.

GitHub Release assets MUST include the mandatory packages, an artifact manifest and SHA-256 checksums. Checksums MUST cover final signed/package bytes; verify transport hashes between jobs. A draft Release or private equivalent staging area SHOULD collect and validate all outputs before visibility changes. There MUST be a final aggregate gate that cannot be bypassed by optional jobs, skipped dependencies, or an unconditional `always()` publish step. Missing mandatory uploads MUST block completion.

## Manual beta workflow

Projects with distributable applications MUST provide a GitHub Actions `workflow_dispatch` workflow named **Beta Release** (or an equally clear name). It MUST accept a branch or commit selector defaulting to `main`, resolve it once to a full committed SHA, record that SHA, and build only that revision. The workflow definition used for publication MUST be trusted; arbitrary source selection does not authorize executing unreviewed workflow/build code with production credentials.

The workflow MUST validate source/version/config state, allocate an unused `MAJOR.MINOR.PATCH-dev.N` under the common transaction, and create its immutable tag before building. Beta tags SHOULD be annotated. Source eligibility, the beta core, initial N and unchanged declaration follow [VERSIONING.md](VERSIONING.md#generated-identifiers-and-counters); workflow inputs MUST NOT select a different core, bypass allocation or permit official `local` or `dev.0` version tags. Remote tag creation requires the developer authorization described in SECURITY.

After mandatory packaging, checksums and notes succeed, create/publish a GitHub Release with `prerelease: true` and explicit `make_latest: false` (or equivalent API behavior). A beta MUST NOT be designated stable or latest stable. The workflow MUST attach all mandatory and successful selected optional release artifacts. It MUST report incomplete attempts as failures and MUST NOT expose them as completed prereleases. Beta distribution to stores is disabled unless separately configured under [DISTRIBUTION.md](DISTRIBUTION.md).

GitHub's [release API](https://docs.github.com/en/rest/releases/releases) exposes draft, prerelease and latest controls; projects MUST check the result rather than assuming a UI label alone provides the intended state.

## Explicit stable preparation and tag workflow

The developer MUST select and initiate the stable MAJOR.MINOR.PATCH. A project SHOULD offer a local release command automating validation, version edits, a dedicated commit when needed, and an immutable annotated `vMAJOR.MINOR.PATCH` tag. A dry-run MUST show intended edits, source/commit plan, version checks, tag and remote actions without modifying source, Git refs/index or remote state. It MAY use temporary validation outputs if documented.

Stable finalization removes the development suffix for the explicitly selected stable version; it is separate from the [development bump](VERSIONING.md#explicit-version-bump) and does not start the next cycle. Preparation MUST check a clean intended source state, full official history, version precedence, prerequisites and required pre-release validation. It MUST update the authoritative version metadata and any generated mirrors coherently, creating a dedicated release commit when necessary. If the existing committed source already carries the selected version, a redundant commit is unnecessary. The tag MUST identify the reviewed intended commit and matching version metadata.

Preparation SHOULD use an isolated checkout or a documented transaction with rollback. Failure MUST preserve pre-existing user work, report created files/commits/tags and leave either the pre-operation state or an explicitly resumable consistent state. It MUST NOT perform a destructive reset of unrelated work. Local tags MUST be removed on rollback only if created by that operation, unpushed and still identifying its expected object.

Pushing the release commit/tag requires explicit remote authorization; local preparation alone does not grant it. Before pushing, revalidate remote reservations. Where supported, an atomic push of commit/ref changes and tag SHOULD avoid partial publication; otherwise detect/report partial success and recover by comparing remote objects. Never force a branch or tag to conceal a partial failure.

An authorized push of the annotated stable tag MUST initiate the official GitHub Actions release pipeline; `push` tag filtering SHOULD be used. The workflow MUST validate stable tag syntax, annotation, version-to-source match, reservation ordering, trusted source, and the release matrix before building. After all mandatory gates pass, it MUST publish a GitHub Release with `prerelease: false`. The repository SHOULD select it as latest stable according to its documented policy. A stable tag's existence does not prove release success.

If a workflow creates/pushes a tag using `GITHUB_TOKEN`, it MUST NOT assume that event will start another workflow. The project MUST use an explicit supported handoff or keep packaging within the original orchestrated workflow, tracking the required downstream result. See [GitHub workflow triggering](https://docs.github.com/en/actions/how-tos/write-workflows/choose-when-workflows-run/trigger-a-workflow) and the [workflow patterns](examples/github-actions-patterns.md).

## Failure, retries and cancellation

| Failure boundary | Required recovery |
| --- | --- |
| Before remote reservation | Repair/retry local preparation; undo only owned local changes or resume from a documented checkpoint. Revalidate all remote state. |
| Remote tag exists, packaging fails/cancels | Keep tag and version consumed. Keep draft/private staging and failure evidence. Retry the exact tag/SHA with the same inputs, or abandon and allocate a greater version for changed source. |
| Tag points to wrong source | Do not move it. Mark reservation abandoned and select a greater version with the correct source. |
| Draft has partial assets | Reconcile against manifest/checksums. Unpublished staging assets MAY be replaced with an audit record. A retry MUST NOT mix outputs from incompatible attempts. |
| Upload/publish API times out | Query actual remote state. Do not assume failure or create a duplicate. Verify tag, channel, asset identities and visibility before proceeding. |
| Already published | Verify identical assets and report idempotent success. Changes require a new greater version; missing/corrupt official assets require explicit incident handling, never silent replacement. |
| Newer version published before retry | Abandon first publication of the older reservation; make a greater version if still needed. |
| GitHub succeeded, store failed | Keep GitHub release successful; report failed destination separately and retry delivery of the verified artifact. |

An unpublished tag retry MAY rebuild with the same pinned source, toolchain, dependencies and package-code reservation; newly resolved inputs MUST be recorded and revalidated. Once an artifact has been submitted externally or published, retries MUST reuse those immutable bytes unless the platform explicitly requires a distinct documented transformation. Changing the source of an existing official reservation always requires a new official version; ordinary development commits do not themselves require a development bump. Lost or invalid credentials MUST fail the signing/publication gate; never fall back to debug/unsigned official packages.

## Release notes

Stable notes MUST cover all relevant changes since the previous successfully published stable tag, even after many betas. Determine that baseline at the publication gate and record it; if it changed since generation, regenerate/review notes. Use Git history in `previous-stable-tag..current-tag` as the authoritative commit range. For the first stable, document the initial reachable-history baseline. If the previous stable is not an ancestor, record that fact, review divergent history and account for carried changes; do not silently use a beta as the baseline.

Beta notes SHOULD cover the previous stable through the beta tag, with an optional preceding-beta comparison. Failed/abandoned reservations are not stable baselines. Links to comparisons SHOULD be included.

Projects SHOULD adopt Conventional Commits or another structured convention, but it is not mandatory without explicit adoption. Generated notes MUST be reviewed for omissions, duplicates, internal noise and secrets. Merge commits SHOULD describe the merged feature once, without duplicating constituent entries. Cherry-picked equivalent changes SHOULD be deduplicated by change identity while preserving relevant fixes and attribution. Generated files/automated commits MAY be grouped or omitted from user-facing prose if their meaningful changes remain covered; commit-range evidence MUST remain available.
