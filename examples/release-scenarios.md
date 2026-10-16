# End-to-end scenarios

These examples illustrate the contracts; commands are placeholders for repository-owned tooling. Product names and build IDs are illustrative. Acceptance evidence is recorded against [COMPLIANCE.md](../COMPLIANCE.md).

## 1. Local build without committing

Stable authority remains 0.9.4; next development core is 0.9.5. The developer modifies tracked code and adds an input file without committing. `<project-tool> build --local` snapshots those inputs and creates `local_artifacts/0.9.5-local.1/`. Its manifest records full HEAD, `dirty: true`, snapshot SHA-256/scope, unique build ID, environment, target outcomes and completion order. A second modification/build produces another directory and digest even with the same Git SHA. No tag/push/publication occurs. Successful completion collects only validated eligible old builds; a failed build collects nothing. Cases L01–L14.

## 2. Push to main

A push supplies exact commit P. CI builds the declared Android debug/Linux/Windows CI matrix, runs required tests, and uploads temporary Actions artifacts named with P, target, run and attempt, with 14-day configured retention. Required failures yield a failing result. These files are not GitHub Release assets and do not consume beta N. Cases C01, C03–C05.

## 3. Pull request CI

A fork PR targets main. The project tests GitHub's proposed merge revision M, recording head H and base B. It has read-only build authority and no production secrets. A Windows test-only installer and Android debug APK are permitted. Required check failure is visible; no protected publisher executes PR-produced code. A head-only project would instead document and test H. Case C02.

## 4. Manually publish several betas

The developer dispatches Beta Release against main. It resolves committed source A, reserves `v0.9.5-dev.1`, packages all mandatory targets, verifies signatures/checksums and publishes a prerelease with latest disabled. A second dispatch against committed B reserves `v0.9.5-dev.2`. Concurrent requests serialize/revalidate; they cannot both allocate dev.2. GitHub output is official Release assets; no external store entry is enabled merely by beta publication. Cases B01–B03.

## 5. Skip a development version

After dev.2, the developer changes the development base to 0.9.6 and records that stable 0.9.5 was skipped. The next reservation is `v0.9.6-dev.1`, greater than every 0.9.5 prerelease. N restarts for the new core but local numbering is independent. A later proposal to publish 0.9.5 stable fails the global high-water rule. No synthetic stable 0.9.5 release is created. Cases V02–V03.

## 6. Stable after multiple betas

The developer explicitly selects 0.9.7. A local dry-run reports version edits, validation, commit/tag plan and requested remote actions. Authorized preparation records authoritative 0.9.7 in the intended commit and makes annotated `v0.9.7`. A separately authorized push triggers stable packaging from that commit. Notes cover **v0.9.4..v0.9.7**, including changes carried through all betas; no assumption is made that the latest beta contains the entire stable changelog range. After mandatory packages, manifest, checksums and uploads pass, the draft becomes stable. Next local development can use 0.9.8-local.1. Cases S01–S05.

## 7. Failed Windows build blocks release

Android and Linux succeed for `v0.9.7`, but Windows is mandatory and fails. The pipeline reports failure; the staged Release remains draft/private. The tag remains consumed and unchanged. If a transient environment fault caused the failure, retry the exact tag with pinned inputs. If a source fix is necessary, abandon this reservation and choose a greater version, for example 0.9.8. Making Windows optional retroactively is not a retry fix. Cases B04–B06, S06–S09.

If instead Windows was explicitly optional before this request, the mandatory gate may pass with its omission clearly disclosed. Optionality is project policy, not an inference from which runner happened to work.

## 8. Android beta-to-stable update

An installed 0.9.6-dev.1 APK uses production ID `org.example.sample`, code 4103 and certificate K. Stable 0.9.7 gets code 4104 and the same compatible signer K. Verification and a device installation without uninstalling confirm the promised update/data path. An AAB signed with Play upload key U does not prove the installed Play APK uses K; verify Play's actual app-signing certificate before promising that cross-destination path. Cases A01–A08; [numeric examples](android-versioning.md).

## 9. GitHub succeeds, store fails

Stable 0.9.7 is complete on GitHub. The separately authorized Google Play delivery is rejected. GitHub success remains recorded; Google Play state is failed with its submission ID and reason. A website may independently be awaiting approval. Retry the verified AAB after reconciling the store's state; do not rebuild from main, duplicate successful submissions or claim an upload request means public availability. If repair requires changed source/package bytes, create a new official version/code. Cases D01–D05.

## 10. Migrate standards

A project pins edition 1.0.0 at commit S1 with a verified snapshot. Central edition 1.1.0 at S2 does not change its behavior. On an explicit migration task, the agent compares S1/S2, updates affected local implementation, tests the changed contracts, records justified overrides/gaps and prepares one reviewable change updating snapshot, pin, mapping and evidence. Existing tags and Android code high-water remain intact. Newly available automatic delivery stays inactive until authorized. Case P02.

## 11. Concurrent channels and failed-tag recovery

An older beta is slow while a higher stable reservation becomes ready. With whole-lifecycle serialization, stable waits. With split allocation/publication locks, stable may publish first; then the older beta's final publication gate rejects it as superseded. A retry cannot make the public stream decrease. An abandoned tag remains part of allocation history. Never delete the tag to reclaim its version. Cases B06 and V03.
