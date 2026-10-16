# Compliance acceptance cases

These cases test contracts, not a particular script language or CI implementation. Apply [normative levels](README.md#normative-language-and-defaults) and each owning policy's scope. A project MUST record applicable case results and evidence before claiming full compliance. A recommended feature not adopted with a justified departure is recorded as such; an unmet MUST is a gap. Mark Android/other nonexistent targets not applicable with a reason. Retention cases use default 10 unless an explicit override exists; record and substitute its effective value.

Fault injection, isolated temporary repositories/storage and mocked destination APIs SHOULD be used for destructive/failure/concurrency cases. They MUST NOT modify real official tags, send unapproved deployments or expose production credentials. Device upgrade and real destination claims require corresponding evidence; a simulation alone does not prove installation or store acceptance. Useful evidence includes manifests/hashes, job conclusions, tag objects, permission configuration, draft visibility, device certificates/codes and destination receipts.

Each table row is a test with preconditions, action, expected result and an explicit failure condition. A project MAY combine executions if each result remains independently evidenced. Policy owners: [architecture](ARCHITECTURE.md), [build](BUILD.md), [artifacts](ARTIFACTS.md), [versioning](VERSIONING.md), [release](RELEASE.md), [Android](ANDROID.md), [distribution](DISTRIBUTION.md), [security](SECURITY.md), [adoption](ADOPTION.md).

## Architecture review

These cases distinguish mandatory safeguards from recommended design/stack choices. For a SHOULD, record whether the recommendation applies, was followed, or has a justified departure; an alternative stack alone is not a failure. Reviews MAY use design/source evidence; data recovery claims require exercised recovery paths in isolated fixtures. No case requires rewriting an existing application or installing a preferred framework.

| ID | Preconditions | Action | Expected result | Failure condition |
| --- | --- | --- | --- | --- |
| R01 | New project or substantive technology selection; requirements known | Review applicable preferences and selected stack | Relevant defaults considered; alternatives permitted with brief rationale under the recommendation convention | Preferred stack treated as mandatory or alternatives rejected solely for standardization |
| R02 | Working application uses another stack | Review proposed standards-adoption change | Existing stack retained unless change has a concrete independent benefit; incremental compliance possible | Rewrite/framework added solely to match preferences |
| R03 | Project has local functions and optional network/account features | Disconnect network/sign out; review documented feasibility | Promised offline functions remain usable; pending work survives as promised; justified network-only scope documented | Offline promise broken, local work silently lost, or unsupported SHOULD treated as mandatory stack change |
| R04 | Application stores valuable user data and attachments/relationships | Export, back up, and restore representative data in an isolated environment | Documented procedures recover required data accurately and securely; sync alone is not claimed as backup without historical recovery | Missing recovery procedure, failed restore or incomplete export represented as sufficient |
| R05 | Storage/protocol and dependencies selected | Review format portability, licensing, maintenance and operational burden | Appropriate open/portable choices and simplicity considered; justified alternatives accepted | Unassessed SHOULD departure or unnecessary framework demanded as proof of compliance |
| R06 | Native capabilities or extension permissions used | Inspect permissions/data flows and deny optional permissions in a test | Only necessary permissions requested; sensitive data protected; denial behavior predictable per adopted recommendations | Unnecessary privilege/collection, data exposure or required function misrepresented after denial |
| R07 | Synchronization implemented; isolated replicas can diverge | Make concurrent offline edits/deletions, interrupt and retry transfer | Documented conflict policy preserves integrity, reports incomplete work and recovers or obtains informed action before irreversible loss | Silent lost edits/corruption, false success or no recoverable source after failure |
| R08 | Schema/format migration implemented; backup fixtures available | Interrupt migration and supply incompatible/corrupt input, then recover | Failure visible; recoverable source retained; restoration demonstrated; no silent destructive conversion | Unrecoverable silent loss or untested claimed recovery path |
| R09 | Project does not need replication or complex reactive persistence | Review proposed RxDB/WebRTC adoption and compliance decision | No database/sync framework required merely for standardization; appropriate simple storage accepted | Unneeded RxDB, WebRTC or service treated as compulsory |
| R10 | Peer-to-peer feature proposed or implemented | Review identity, signaling/relay, offline/conflict recovery and service assumptions | Conditional transport preference evaluated; infrastructure is explicit; application-level sync safety independently verified when implemented | Claims of guaranteed serverless/direct connectivity or transport alone treated as data-integrity proof |
| R11 | Multiple platforms or frontend/native boundaries exist | Review integration/accessibility and exercise relevant business rules | Platform conventions and useful logic separation considered; focused testing possible or departures justified | Mandatory identical UI/layering imposed, or adopted integration/testability promises broken |
| R12 | New architecture guidance is being adopted | Inspect project mappings, dependencies and delivery matrices | Repository owns implementation; decisions and scope recorded; framework choice adds no shared runtime or implicit destination | Architecture choice silently enables deployment or introduces a mandatory shared framework |

## Local builds and retention

| ID | Preconditions | Action | Expected result | Failure condition |
| --- | --- | --- | --- | --- |
| L01 | Local host supports a declared target; one successful build exists | Build twice without a commit | Distinct build IDs/directories; prior bytes preserved | Reuse, overwrite, commit prerequisite or remote action |
| L02 | Same HEAD, two different dirty source states including an input untracked file | Build each state | Same SHA permitted; dirty flag true; different verified source digests | SHA alone or identical identity for different used inputs |
| L03 | Stable version known; mutable branch available | Run local build | Stable authority unchanged; no push/tag/publish; coherent source snapshot recorded | Version mutation, remote operation or incoherent source |
| L04 | 12 unprotected successful managed builds; limit 10 | Complete another successful build and collect | Exactly newest 10 unprotected successful builds retained by completion order | Wrong count, wrong ordering or protected counts included |
| L05 | Old successful build contains `.keep`; 12 unprotected builds | Complete build and collect | Protected bytes preserved plus newest 10 unprotected | `.keep` build deleted or consumes a retention slot |
| L06 | Existing success/protected/unknown directories | Inject mandatory build failure | Failure visible; no automatic collection or deletion | Any deletion caused by failure or success status |
| L07 | Active slow build; completed successes; two simultaneous collectors | Complete another build/collect concurrently | Locks coordinate; active output preserved; final valid retention state | Partial deletion, active removal or race corrupts state |
| L08 | Unknown/malformed manifests, symlinked child/root and traversal candidates in isolated fixtures | Run automatic and manual dry-run cleanup | Unsafe entries skipped/reported; no followed paths; dry-run changes nothing | Unknown deletion, escape or dry-run mutation |
| L09 | Dry-run selected an old managed build | Protect via locking interface, then apply cleanup | Revalidation preserves it; deletion summary accurate | Stale dry-run list causes deletion |
| L10 | Concurrent builds complete opposite their start order; clock/mtime manipulated | Complete and collect | Reliable completion sequence determines newest builds | Name, start order, mtime or wall clock chooses retention |
| L11 | Configured limit 2, then invalid negative limit in fixtures | Build/collect with each config | Valid override keeps 2; invalid config fails before deletion | Hardcoded 10 despite override or unsafe invalid cleanup |
| L12 | Build is interrupted; partial state remains | Run future automatic collection | Interrupted directory remains excluded; explicit recovery available | Automatic deletion or false success |
| L13 | Inputs can change during compilation | Modify used input during a local build | Captured snapshot remains consistent or build detects/fails change | Manifest describes inputs other than those compiled |
| L14 | Multi-platform local selection has one failed mandatory target | Finish build | Failed status and per-target results; partial artifacts identified; no retention | Partial output marked complete or failed target hidden |
| L15 | Root `.gitignore` and Git index available; isolated fixture includes root `local_artifacts/` and root/nested `node_modules/`, `dist/`, `__pycache__/`, `target/` files, including an already-tracked file | Inspect required patterns under [ARTIFACTS.md](ARTIFACTS.md#git-exclusions), run `git check-ignore --no-index` on fixture paths, and inspect `git ls-files`; apply adoption remediation to the tracked fixture file | All required patterns present; every fixture path effectively ignored; excluded files absent from index after remediation and local bytes preserved | Missing pattern, negation re-includes contents, excluded file remains tracked, or remediation deletes local bytes |

## Versions and ordinary CI

| ID | Preconditions | Action | Expected result | Failure condition |
| --- | --- | --- | --- | --- |
| V01 | Implemented version comparator | Evaluate all precedence examples in VERSIONING.md | Exact stated inequalities/equality, including numeric 2/10 and metadata equivalence | Lexical whole-string sort or metadata affects precedence |
| V02 | Last stable 0.9.4; clean committed beta sources | Exercise the full documented transition through 0.9.7 and local 0.9.8 | Each official candidate increases; skipped 0.9.5 never backfilled; local counters independent | Rejected valid transition or accepted lower official version |
| V03 | Reserved 0.9.6-dev.1, even if failed; misleading shallow local checkout | Request 0.9.5 stable, then 0.9.7 | First rejected; second eligible after complete remote inventory | Failed reservation ignored or shallow state trusted |
| C01 | Push CI configured; known push SHA | Push to declared primary branch in an authorized test repository | Checks/build run on exact SHA; temporary packages uploaded with recorded identity/retention | Moving-source build, missing mandatory output or official release |
| C02 | Untrusted PR targets primary branch; production secret exists only in protected context | Run/inspect PR CI and privilege boundaries | Declared PR revision recorded; no production secrets/write token exposed | Secret access, privileged execution of PR output, or inaccurate source |
| C03 | Ordinary push/PR jobs have artifacts | Inspect events and remote mutation audit | No release/tag/deployment/version increment | Any ordinary-CI publication side effect |
| C04 | A declared required test, toolchain or target unavailable | Run CI | Required gate fails visibly; skipped work is not success | Green result with missing required checks |
| C05 | Optional target fails; mandatory targets succeed | Finish CI/release staging | Optional failure explicitly reported per policy; no fabricated artifact | Hidden failure or unconfigured target counted as built |

## Beta releases

| ID | Preconditions | Action | Expected result | Failure condition |
| --- | --- | --- | --- | --- |
| B01 | Trusted committed source; authorized manual dispatch; all required targets pass | Dispatch Beta Release with default source | Resolved SHA tagged; all mandatory assets/checksums; visible prerelease, not latest stable | Moving source, wrong channel/latest, incomplete assets |
| B02 | Same beta core and two simultaneous requests | Allocate/build through both paths | Distinct immutable tags or explicit safely failed/cancelled request; each reservation above high-water | Duplicate allocation, tag force-update or silent lost request |
| B03 | Beta tag already reserved | Retry same SHA, then request changed source under same tag | Same reservation safely retried; changed source rejected/new greater version required | Tag moved or identity reused for other source |
| B04 | Beta tag created; Windows mandatory packaging fails | Finish pipeline | Tag stays reserved; draft/private failure evidence; no completed prerelease | Visible completed beta or freed/reused reservation |
| B05 | Failed tagged beta; source correction committed | Recover | Exact-tag retry only for unchanged source; correction gets greater version | Corrected source built under old tag |
| B06 | Older beta slow, newer version ready/published | Attempt older first publication afterward | Serialized gate blocks older publication or whole lifecycle was serialized | Published official sequence decreases |
| B07 | Malformed/unknown official history entry | Allocate beta | Block and reconcile rather than guess | Allocation ignores uncertain official high-water state |

## Stable releases and recovery

| ID | Preconditions | Action | Expected result | Failure condition |
| --- | --- | --- | --- | --- |
| S01 | Existing official stream and explicit developer selection | Try equal/lower, malformed and valid greater stable versions | Only valid greater candidate passes; metadata/tag match | Equal/lower accepted, odd/even restriction or automatic stable selection |
| S02 | User work exists; preparation supports dry-run | Dry-run, then inject pre-push preparation failure | Dry-run no source/ref/remote changes; failure preserves user work and coherent checkpoint/rollback | Dirty reset, hidden leftover inconsistent state or remote dry-run effect |
| S03 | Version metadata selected; intended commit reviewed | Prepare authorized release locally; inspect annotated tag | Commit contains intended version/source; tag annotation/peeled SHA correct | Wrong commit, lightweight stable tag or metadata mismatch |
| S04 | Authorized stable tag, declared multi-platform release | Execute tag workflow | Every mandatory target built from tag; verified staging before public release | Builds branch head, omitted target or premature publication |
| S05 | Prior stable v0.9.4; several betas; stable v0.9.7 | Generate/review notes | Range v0.9.4..v0.9.7 covers relevant changes, with merge/cherry-pick handling | Range only covers latest beta or meaningful changes omitted |
| S06 | Stable Android/Linux succeed, mandatory Windows fails | Finish release | Failed workflow; retained immutable tag; no completed stable Release | Completed stable published with missing Windows |
| S07 | Pushed tag with failed packaging or wrong source | Retry or repair | Same correct tag retry; wrong source abandoned and greater version selected | Forced/moved tag, released identity reclaimed |
| S08 | Publication request timed out after possible success | Retry with remote reconciliation | Existing identical result verified or incomplete stage resumed once | Duplicate publication or blind overwrite |
| S09 | Official assets already published | Retry with identical then different bytes | Identical no-op; differing content blocked and incident/new-version path | Silent replacement of official assets |
| S10 | Tag created by automation using GITHUB_TOKEN; separate tag workflow intended | Exercise documented dispatch/orchestration in fixture | Required build actually runs through explicit supported handoff | Assumes token-created tag automatically triggered another workflow |

## Android

| ID | Preconditions | Action | Expected result | Failure condition |
| --- | --- | --- | --- | --- |
| A01 | Beta→beta promised; device has beta code 4101 | Install next beta code 4102 over it | Compatible ID/certificate, update succeeds without uninstall; data checks pass | Signature/code mismatch, uninstall needed or data promise broken |
| A02 | Beta→stable promised; installed beta code 4103 | Install stable code 4104 | Direct upgrade succeeds with declared compatible signer | Equal/lower code, incompatible signer or reinstall used as proof |
| A03 | APK/AAB signing profiles and installed cert inventory exist | Verify final packages and any promised GitHub↔Play edge | Correct APK signer, AAB upload signer; actual delivered Play signer checked separately | Upload key assumed equal to app signing key or wrong-signed release accepted |
| A04 | Skipped cores and future major/minor examples | Allocate codes for all worked examples | Strict code progression independent of SemVer encoding; failed codes not reused | Reset on new core/major or abandoned code reused |
| A05 | Test ledger near bounds | Try -1, 0, fractional, 2100000000 and 2100000001; request next after max | Invalid values rejected; max allowed only if greater than high-water; exhaustion blocks without wrap | Overflow, reset, noninteger or above-Play-limit accepted |
| A06 | Concurrent beta/stable allocations in same ID domain | Reserve codes from both | Unique coordinated greater reservations, valid promised delivery ordering | Same code for distinct releases or shared-domain ledger bypass |
| A07 | Existing Play/GitHub submissions exceed proposed migration seed | Migrate allocator | Seed exceeds every allocated/submitted code; existing IDs/signers retained | Only visible releases counted or downgrade path introduced |
| A08 | Untrusted CI; missing production signer in release fixture | Build debug; attempt official APK signing | Debug works without production secret; official target fails closed | Debug key fallback or unsigned installable release |

## Distribution, security and adoption

| ID | Preconditions | Action | Expected result | Failure condition |
| --- | --- | --- | --- | --- |
| D01 | Targets built; destinations span all four modes | Run authorized release/delivery | Disabled sends nothing; manual yields instructions; approval waits; automatic meets checks | Build implies upload, approval bypass or mode ignored |
| D02 | Different beta/stable/platform destination entries | Publish beta then stable in fixture | Only selected channel/platform/artifacts delivered; no additional compilation absent documented requirement | Extra store/platform upload or undeclared rebuild |
| D03 | GitHub release complete; store returns rejection/processing | Record delivery state | GitHub success preserved; destination failed/pending, receipt recorded; no claim of live success | Entire delivery marked successful from artifact/upload alone |
| D04 | Prior successful store submission or website deployment | Retry identical then conflicting delivery | Existing success reconciled/no-op; conflict blocked; retry identity durable | Duplicate/conflicting publish or silent overwrite |
| D05 | Local and ordinary CI have credentials available incidentally | Run both | No external upload/deployment regardless of credential presence | Credentials or config trigger external publication |
| Q01 | Publication permissions, key material and logs in isolated audit | Inspect job scopes, package contents, secret handling and integrity gates | Least privilege; no leaked keys/secrets; hashes/signatures checked | Production credentials in untrusted code or artifacts/logs |
| Q02 | Agent asked only to implement local infrastructure | Prepare/validate changes; inspect attempted remote operations | Reviewable local work; no unauthorized remote action | Agent treats config mode/token as approval |
| P01 | Project pinned to older standards with verified snapshot; newer central edition exists | Resolve policy online, then offline | Exact pinned policy used in both; missing unverified source explicitly reported | Silent latest adoption or guessed policy |
| P02 | Explicit migration request with old/new pins | Review delta, implement, validate and update adoption | Snapshot/mapping/pin/evidence updated coherently; identities preserved; extra distribution separately authorized | Pin-only upgrade, erased reservations or accidental new deployment |
| P03 | Native metadata used instead of example YAML | Audit mapping and inject unknown target/invalid mode | Equivalent concepts documented; invalid operation blocked; no duplicated editable version | Configuration silently permissive or mandatory concept missing |

## Recording results

Record case ID, applicability/rationale, source/config/standards revisions, preconditions, test action, observed result, pass/fail, evidence location and any gap owner. Logs MUST be redacted. A failed release test passes its acceptance case when it fails safely as specified; a green workflow is not always the expected result.
