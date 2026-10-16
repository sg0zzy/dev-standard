# GitHub Actions patterns

Informative fragments and implementation guidance, **not complete runnable workflows**. Each application repository supplies its own commands, toolchains and reviewed Action SHAs. These examples are not a central reusable workflow dependency. Normative gates are in [BUILD.md](../BUILD.md), [RELEASE.md](../RELEASE.md) and [SECURITY.md](../SECURITY.md).

## Ordinary CI trigger and outputs

```yaml
name: CI
on:
  push:
    branches: [main]
  pull_request:
    branches: [main]
permissions:
  contents: read
```

The repository adds checkout, toolchain setup, checks, matrix packaging and temporary upload jobs. A checkout of the event's `github.sha` can implement push-SHA and PR proposed-merge policies; record PR head/base separately. `github.sha` on a PR merge test is not the head SHA. A head-only policy needs an explicit checkout of the recorded PR head instead. Use `persist-credentials: false` where Git credentials are not needed. Fetch complete official history only in the trusted release validation path; shallow CI may be sufficient for ordinary builds.

An upload step fragment (replace the Action pin with a real reviewed full SHA):

```yaml
- name: Preserve verified CI packages
  uses: actions/upload-artifact@REVIEWED_FULL_COMMIT_SHA
  with:
    name: sample-${{ matrix.target }}-${{ github.sha }}-${{ github.run_id }}-${{ github.run_attempt }}
    path: ci-output/${{ matrix.target }}/
    if-no-files-found: error
    retention-days: 14
```

Use a different diagnostic upload step for partial/failed output, labeled incomplete. Keep production signing and publication out of these jobs. A Windows CI installer MAY be explicitly test-only unsigned even if its official release counterpart requires signing. A successful temporary upload does not create a Release asset. GitHub documents [workflow artifact storage](https://docs.github.com/en/actions/tutorials/store-and-share-data).

## Beta dispatch and exact source

```yaml
name: Beta Release
on:
  workflow_dispatch:
    inputs:
      source:
        description: Committed source branch or full SHA
        required: true
        default: main
        type: string
permissions:
  contents: read
```

GitHub requires the dispatchable workflow to exist on the default branch for `workflow_dispatch` triggering. The workflow ref selected in the UI and the application's `source` input are different concepts. Use a trusted workflow definition; resolve the input once, validate that commit's trust, then pass the immutable resolved SHA between jobs. Do not interpolate untrusted inputs directly into shell code; pass them through safe environment/argument handling and validate ref syntax. See [workflow events](https://docs.github.com/en/actions/reference/workflows-and-actions/events-that-trigger-workflows).

The project adds a narrowly privileged reservation job, read-only compilation jobs, controlled signing, an aggregate mandatory-output gate, draft asset upload, notes and checksums, and a trusted final publication job. Only the jobs creating tags/releases need the corresponding write permissions. The final beta API operation explicitly sets `prerelease: true`, `make_latest: false` and transitions the fully verified draft to visible. Store delivery is a separate job/workflow governed by the distribution matrix.

## Coordination fragment

```yaml
concurrency:
  group: official-release-lifecycle
  cancel-in-progress: false
```

If lifecycle serialization is chosen, every beta/stable workflow must use the same repository-scoped group, not a group containing version/channel/workflow name. This fragment alone is insufficient: external local stable pushes do not acquire an Actions lock. Use atomic create-only remote tag reservations, revalidate the complete high-water mark, and serialize the publication gate as specified in RELEASE.md.

Do not assume GitHub concurrency scheduling is a durable FIFO queue or that every pending request will eventually run. Inspect the available [concurrency controls](https://docs.github.com/en/actions/concepts/workflows-and-actions/concurrency) and [workflow syntax](https://docs.github.com/en/actions/reference/workflows-and-actions/workflow-syntax) for the actual host; handle cancelled/replaced pending requests explicitly. `cancel-in-progress: false` prevents deliberate cancellation of the running lifecycle but does not replace durable reservation records or protect against manual cancellation.

## Stable tag trigger and handoff

```yaml
name: Stable Release
on:
  push:
    tags: ['v*']
permissions:
  contents: read
```

This broad glob is only an event filter. Add a validation gate that accepts **only** annotated `vMAJOR.MINOR.PATCH` stable tags, rejects local/dev tags for this workflow, peels the exact commit, validates authoritative metadata and checks ordering before packaging. Add platform jobs matching the stable build matrix and a final gate depending on all mandatory results. Do not use `continue-on-error` to bypass mandatory jobs or publish merely because a matrix ran.

A tag pushed by a developer can trigger the tag workflow. Events generated using a workflow's `GITHUB_TOKEN` generally do not start another workflow, with documented dispatch exceptions. Thus a beta reservation job creating a tag with that token should continue its packaging in the same orchestrated workflow or explicitly dispatch a trusted workflow and track its completion. Do not assume the tag starts a second pipeline. See [triggering workflows](https://docs.github.com/en/actions/how-tos/write-workflows/choose-when-workflows-run/trigger-a-workflow).

## Separate external delivery

After a completed GitHub Release, select destination entries by channel/target. A website `approval` entry waits for its declared protected-environment approval bound to artifact hash/version. An `automatic` entry still needs adopted authorization and all security gates. Give only that delivery job its destination credential, reconcile existing delivery state before retrying, and record pending/failed/success separately. Never rebuild from the current branch to upload to a store. See [deployment controls](https://docs.github.com/en/actions/how-tos/deploy/configure-and-manage-deployments/control-deployments).
