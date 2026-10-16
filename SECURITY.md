# Security, permissions and agent authority

## Workflow boundaries

GitHub Actions MUST use least-privilege explicit permissions. Ordinary build jobs SHOULD use `contents: read`; publication permission such as `contents: write` MUST be limited to trusted jobs that need it. Build and publication permissions MUST be separated. Signing SHOULD occur in narrowly scoped trusted jobs/environments. Do not give untrusted compilation a write token just because a later job publishes.

Untrusted PR code MUST NOT receive production signing material, deployment credentials or write-scoped publication tokens, including through caches, artifacts, environment files or a later privileged workflow. Do not execute PR code with secrets under `pull_request_target`, or download untrusted executable output into a privileged publisher without a verified trust boundary. Trusted-source validation MUST include workflow/build/dependency code, not only the selector's branch name. Repository/fork trust and environment restrictions MUST be documented.

Required checks MUST fail closed on unavailable secrets or invalid signatures. Secrets MUST NOT be committed, embedded in packages/manifests, printed in logs or included in uploaded build directories. Masking alone is insufficient protection; avoid command echoing, secret-bearing arguments and diagnostic dumps. Projects SHOULD use protected environments for sensitive signing/distribution and restrict eligible refs and reviewers as appropriate.

Third-party Actions SHOULD be pinned to full reviewed commit SHAs and updated deliberately. Toolchain/dependency integrity SHOULD be pinned/verified where feasible. Artifact SHA-256, signing identities and source provenance MUST be checked at official publication boundaries. Checksums prove byte integrity, not authorship by themselves. Native signatures/attestations MAY strengthen verification. Release tags MUST be immutable; permissions/rulesets SHOULD enforce this, including protections from administrative bypass where available.

See GitHub's [secure use reference](https://docs.github.com/en/actions/reference/security/secure-use) and [deployment controls](https://docs.github.com/en/actions/how-tos/deploy/configure-and-manage-deployments/control-deployments). Environment availability depends on repository/plan configuration; projects MUST verify their actual controls.

## Agent operations

When requested to implement or maintain infrastructure, coding agents MAY inspect sources and history, edit local policies/workflows/configuration, perform local builds/tests/dry-runs, and prepare reviewable changes. They MUST protect existing user work. A version bump and its metadata-only local commit require an explicit developer request under [VERSIONING.md](VERSIONING.md#explicit-version-bump); completing a feature, bug fix or local build MUST NOT imply authorization to bump or commit. Bump-and-commit authorization does not authorize a tag, push, release or deployment. Local release commits/tags MAY be prepared only when release preparation is in the authorized task scope; a generic build request does not authorize them.

Agents MUST NOT push commits, create remote tags, upload secrets, publish Releases or deploy without explicit authorization. Authorization MAY be a specific user instruction or an explicitly adopted standing policy scoped to the operation, channel, destination and conditions. Agents MUST record/cite the applicable scope; merely finding `mode: automatic`, an example config, or a writable token is not authorization. Existing valid session authorization MUST NOT be requested again. If scope is absent, finish local reviewable work and obtain authorization before the remote step.

A developer's explicit beta dispatch or stable tag push MAY constitute authorization for that workflow's declared GitHub publication. It does not implicitly authorize stores. A standing automatic distribution policy MUST be explicitly adopted by the developer and cannot be introduced by an agent without authorization. An `approval` destination always requires its declared approval even with standing automation authority.

Dry-runs MUST NOT modify remotes, upload credentials, publish or deploy. Local builds MUST remain local regardless of available credentials. Publication jobs MUST NOT silently replace published assets or move existing tags. Security incidents involving exposed keys or corrupted official assets require explicit remediation and an audit record, not an automated key replacement or hidden rebuild.
