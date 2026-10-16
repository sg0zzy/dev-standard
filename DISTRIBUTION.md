# Distribution matrix and delivery policy

Build success, GitHub Release success and external destination success MUST be separately observable. External delivery MUST NOT be reported successful merely because a package exists or an upload request was sent.

## Independent matrix

Each destination entry MUST specify repository/project, target/artifact selection, platform, channel (`beta` and/or `stable`), mode, required checks, credential/environment references, success evidence and retry identity. Architecture selection MUST be explicit where relevant. A build target alone never authorizes delivery. A missing external destination/channel entry defaults to `disabled`.

| Mode | Contract |
| --- | --- |
| `disabled` | MUST NOT send artifacts to that destination. Local generation is allowed. |
| `manual` | Generate verified artifacts and human instructions; MUST NOT automatically upload/publish to the destination. |
| `approval` | Automation MAY send only after an explicit approval bound to that artifact, channel and destination; denial/absence MUST block sending. |
| `automatic` | MAY send after configured checks and standing authorization succeed; MUST still respect security and environment approvals. |

These modes govern the transfer/publication operation defined in the entry. A store upload to a draft track is still external delivery and requires an authorized mode; projects MAY model upload and public rollout as separate explicit entries. Whether completion means uploaded, accepted, available to users, or rollout complete MUST be declared. Pending processing/review MUST be reported as pending, not live. Mode selection does not guarantee access to a store or override its review rules.

Default GitHub stable behavior SHOULD be automatic after an explicitly authorized stable tag and successful release gates; default beta behavior SHOULD be an explicit manual `workflow_dispatch`. Here manual beta means human initiation of the GitHub release workflow, not the external `manual` mode's prohibition on automatic store upload. External stores SHOULD remain manual or disabled until explicitly configured. Local builds and ordinary CI MUST never deliver externally.

## Artifact consumption and platform requirements

Distribution MUST consume artifacts verified against the completed release manifest, rather than rebuilding ad hoc. It MUST NOT trigger additional compilation unless an explicit documented platform requirement demands it. Signing, notarization, store-generated packages or destination-specific transformations MAY be required; record the input hash, output hash/version/signer where available, tool/service identity and reason. A compilation exception MUST pin the original source/dependencies and repeat the relevant build/security gates. Do not claim byte identity after a transformation.

Example: build Android APK/AAB, Linux AppImage and Windows installer for stable; attach all declared mandatory packages to GitHub; submit only AAB to Google Play; upload only AppImage to the website; keep Windows store delivery disabled. No missing store entry creates an implicit build or destination. See [configuration](examples/project-config.example.yml).

Possible destinations include Google Play, AMO, Chrome Web Store, Microsoft Store, Apple channels, websites and SFTP hosts. Each configured destination MUST document package rules, identifiers, signing/credentials, applicable approvals and platform-version mapping. These standards do not require accounts or services. Projects MUST verify current official platform documentation before implementing an integration.

## Idempotency and failure recovery

A delivery MUST be keyed by destination, channel, product/version, platform package identifier/version and verified artifact hash. Persist external submission/deployment IDs and resulting status where available. A retry MUST query/reconcile existing state before uploading again; already successful identical delivery is a no-op. Conflicting content/version MUST fail visibly and MUST NOT overwrite silently. Website/hosting replacement MAY be explicitly configured as a versioned atomic deployment with a recorded previous deployment; immutable versioned release files remain protected.

Authentication, upload, review rejection and rollout failures MUST identify the failing destination and leave its state failed or pending as appropriate. They MUST NOT change an already completed GitHub Release into a failed build, nor mark the whole delivery set successful. Retry only the failed destination using verified inputs. A rejection requiring changed source or binary MUST follow the new-release/version rules. Rollback MUST use a separately authorized platform procedure; it cannot be assumed to permit lower Android codes or replacement of immutable release assets.
