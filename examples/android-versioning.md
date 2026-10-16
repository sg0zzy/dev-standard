# Worked Android versionCode progressions

Informative examples of [ANDROID.md](../ANDROID.md)'s recommended durable sequence. Numbers are illustrative, not a universal starting offset. Every real migration first inventories existing allocated/submitted codes. `versionName` carries the product version or documented package mapping; versionCode is independent.

## 1. Consecutive betas, skipped core, stable

| Product version | Reserved code | Observation |
| --- | --- | --- |
| 0.9.4 | 4100 | Existing stable |
| 0.9.5-dev.1 | 4101 | Beta→beta with same ID/compatible signer |
| 0.9.5-dev.2 | 4102 | Another committed beta |
| 0.9.6-dev.1 | 4103 | Skip stable 0.9.5 without resetting code |
| 0.9.7 | 4104 | Stable can update beta 4103 if all other compatibility gates pass |

Install 4101→4102→4103→4104 on a supported device without uninstalling. Verify certificate, code and preserved data at each step. Different certificates invalidate the promise even though the numbers increase.

## 2. Minor/major changes and failed packaging

| Product version | Reserved code | Outcome |
| --- | --- | --- |
| 0.9.7 | 4104 | Published stable |
| 0.10.0-dev.1 | 4105 | Packaging failed; reservation retained |
| 0.10.0-dev.2 | 4106 | Corrected source, new tag/code |
| 0.10.0 | 4107 | Stable |
| 1.0.0-dev.1 | 4108 | Major increment does not reset code |
| 1.0.0 | 4109 | Stable |

Retrying the unchanged 0.10.0-dev.1 before a newer publication could use 4105; after later publication it cannot first publish out of order. Corrected source always takes a new official identity. Gaps are harmless and MUST NOT be reclaimed.

## 3. Shared GitHub/Play domain

| Product version | Code | Artifacts/delivery |
| --- | --- | --- |
| 1.0.0 | 7200 | GitHub APK and Play AAB for same release; platform-valid shared code |
| 1.0.1-dev.1 | 7201 | GitHub APK; no automatic Play beta |
| 1.0.1 | 7202 | GitHub APK and Play AAB; one code reservation |
| 1.1.0 | 7203 | Next stable; no SemVer multiplication |

A shared code ledger supports numerical order but does not establish signer compatibility. Verify the actual Play-delivered APK certificate against the GitHub APK signer before promising cross-destination upgrades. If Play holds a different private app-signing key, an upload-signed GitHub APK alone is not proof of compatibility. Promoting an existing Play artifact may avoid reupload; retries must reconcile actual platform state.

## 4. Bounds and migration

If historical/submitted high-water is 900000, first migrated reservation is at least 900001. If it is 2099999999, one remaining Play-compatible code is 2100000000. Any next reservation MUST fail without overflow or reset. Test zero, negative, fractional, equal-to-high-water and over-limit inputs; test concurrent allocations and preservation of abandoned codes. A new application ID is a different application/update domain and is not an automatic in-place migration solution.

Acceptance cases A01–A08 in [COMPLIANCE.md](../COMPLIANCE.md) cover these transitions. Platform constraints: [Android versioning](https://developer.android.com/studio/publish/versioning) and [signing](https://developer.android.com/studio/publish/app-signing).
