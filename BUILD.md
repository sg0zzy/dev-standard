# Development and build behavior

## Build matrix

Each project MUST explicitly declare target IDs, platforms, architectures, package kinds, and required/optional status separately for local, ordinary CI, beta and stable contexts. An empty matrix MUST be explicit. An absent target is unconfigured and MUST NOT be reported as built. Different contexts MAY have different matrices. Projects MUST NOT be required to build every possible platform.

Each selected target MUST produce a recorded outcome: successful, failed, or explicitly skipped with a reason. A mandatory target's failure or skipped validation MUST fail that context's completion gate. Optional failures MAY allow completion but MUST be visible in results and release notes; the project MUST state how absent optional assets are handled. Supported release targets are those declared for the channel, not all local or CI targets. Distribution selection is independent: see [DISTRIBUTION.md](DISTRIBUTION.md).

Projects MUST document each target's packaging and verification requirements. Examples include Android APK/AAB, Linux AppImage, Windows MSIX/EXE, macOS packages, Firefox XPI, Chrome extension packages, and web bundles. Package architecture MUST be explicit where applicable. Installation/signature validation SHOULD be included where feasible. Platform package-version restrictions MUST be mapped under [VERSIONING.md](VERSIONING.md).

## Per-platform signing profiles

Every declared platform MUST have its own signing profile, including an explicit unsigned/checksum-only decision where the platform and adopted distribution policy permit it. Profiles MUST identify required credentials by reference, expected public identity, final-package verification, and whether a signing/notarization service interaction is required. Required service interactions MUST have explicit authorization under SECURITY; a test-only package must not be mistaken for an installable official package.

| Platform/package | Project profile must address |
| --- | --- |
| Android APK/AAB | Debug, APK release signer, Play upload/app-signing distinction and update edges under [ANDROID.md](ANDROID.md). |
| Linux AppImage | Native/detached signing choice and verification, or explicit checksum-only policy; any separate repository/website signature requirements. |
| Windows MSIX / EXE | Package type, publisher identity, certificate trust/timestamp policy and signature verification; a signed MSIX profile does not automatically define the EXE policy. [MSIX requires valid package signing for deployment](https://learn.microsoft.com/en-us/windows/msix/package/signing-package-overview). |
| macOS | Intended distribution path, appropriate signing identity, entitlements, and notarization/stapling requirements where applicable; inspect submission results and final package verification. Follow [Apple's notarization guidance](https://developer.apple.com/documentation/security/notarizing-macos-software-before-distribution) for the selected path. |
| Firefox XPI | Declared extension ID, listed/unlisted signing path, signed-output verification and update compatibility. [Release/beta Firefox normally requires Mozilla-signed extensions](https://www.extensionworkshop.com/documentation/publish/signing-and-distribution-overview/); a locally zipped unsigned XPI is not equivalent. |
| Chrome extension | Stable extension identity, store-upload versus self-hosted package requirements and key custody where relevant; distinguish an upload ZIP from an installable CRX. Record the native numeric version and display-name mapping; see [Chrome version fields](https://developer.chrome.com/docs/extensions/reference/manifest/version). |
| Web bundle | Checksum/integrity or signing policy for the bundle; hosting/TLS/deployment authority is separately configured and does not follow from producing the bundle. |

Signing/notarization services used only to obtain a final package are distinct from listing it publicly in a store. Projects MUST document such service use even if public external distribution is disabled. For example, Mozilla signing of an unlisted XPI still sends code externally and needs an authorized signing operation; it does not authorize an AMO public listing. If the required signer is unavailable, fail that mandatory target rather than weaken its profile.

## Local procedure

Each project MUST document prerequisites, available host capabilities, target selection, commands, output paths, and failure reporting. Unsupported host/target combinations MUST fail clearly or be explicitly excluded before building. Local builds MUST work with uncommitted changes and MUST NOT require creating a commit. They MUST NOT automatically push, tag, publish or distribute. Version derivation and metadata preservation follow [VERSIONING.md](VERSIONING.md#generated-identifiers-and-counters). Native tool-generated transient files MAY change if documented.

Each completed build MUST preserve its final artifacts in its own `local_artifacts/` directory under [ARTIFACTS.md](ARTIFACTS.md), subject only to documented retention/manual deletion. No implicit overwrite is permitted. All mandatory selected targets and manifest verification MUST succeed before a build is marked successful. Failed multi-target builds MUST record partial outcomes and MUST NOT trigger retention.

The source used across targets MUST be a coherent snapshot matching the recorded source identity. Projects SHOULD capture a private source snapshot before compiling; otherwise they MUST detect input changes during building and fail or capture the actual per-target snapshots. Recording a digest before building while continuing to compile changing files is insufficient. Build commands SHOULD reuse the same underlying compilation and packaging logic as CI/releases where feasible. Debug/release flags and signing MAY differ.

Illustrative interfaces only (not scripts supplied by these standards):

```text
<project-tool> build --local --target android-debug
<project-tool> build --local --target linux-x86_64 --target windows-x64
<project-tool> artifacts clean --dry-run
```

Projects MAY use native IDE tasks, Make, Gradle, PowerShell or other interfaces. A failure MUST return a failing process/workflow status where applicable, identify the failing stage/target, and report retained logs/output without exposing secrets.

## Ordinary continuous integration

Every repository SHOULD implement automatic CI on push to its primary development branch (normally `main`) and pull requests targeting it. A project choosing different triggers MUST document why.

Ordinary CI MUST:

1. Check out the exact event revision: the push SHA, or the PR head or proposed merge revision according to an explicit project policy. Record which policy and the relevant head/base/merge SHAs; do not resolve a moving branch again mid-run.
2. Install the declared toolchain and dependencies; SHOULD use reproducible locks/pins where available.
3. Run the declared required checks/tests and build the CI matrix. Projects MUST inventory available validation and document any exclusions; missing required tooling MUST fail instead of silently skipping tests.
4. Verify generated packages, record source/run identity, and upload successful packages as temporary GitHub Actions artifacts. On failure, diagnostic/partial artifacts MAY be uploaded but MUST be labeled incomplete.
5. Report job and aggregate failure accurately, including mandatory matrix failures even if other jobs succeed.

CI MUST NOT bump declared project versions (see [VERSIONING.md](VERSIONING.md#independent-operations)), create official tags or GitHub Releases, or deploy externally. It MUST NOT require production signing secrets for untrusted PRs. Debug or otherwise test-only packages MUST be labeled accordingly. CI identities MAY be native run IDs or local-style identifiers, but MUST NOT consume beta N or Android production codes merely for compilation.

Temporary artifact names SHOULD include project, target, source SHA, run ID and attempt. Their retention SHOULD be explicit, with 14 days the recommended default subject to hosting limits. Upload failure for mandatory expected CI output MUST fail the CI output gate. Actions artifacts are temporary transport/storage; their presence MUST NOT imply official publication. See [GitHub workflow artifacts](https://docs.github.com/en/actions/tutorials/store-and-share-data).

Security requirements for checkout, credentials, build/publish separation, and workflow triggers are owned by [SECURITY.md](SECURITY.md). Release builds additionally satisfy [RELEASE.md](RELEASE.md).
