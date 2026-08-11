# Shizuku hardening audit — 2026-08-11

State: `AUDIT_BRANCH / NON_DESTRUCTIVE / CLAIM_ALLOWED=false`

Branch: `audit/hardening-no-regression-20260811`

Base/master SHA: `b844bc491f1790c72328e1a8e5b2349f8978f0ea`

Upstream `RikkaApps/Shizuku` HEAD observed at audit start: same SHA as base. No sync commit was necessary.

## Invariant

Reduce failure modes without silently changing privilege semantics, vendoring external dependencies, or mutating `master` before evidence exists.

Operational chain:

`source -> finding -> severity -> patch -> verification -> falsifier -> decision`

Unknown or unverified states remain `TOKEN_VAZIO`.

## Findings ledger

| ID | Severity | Finding | State | Evidence / action |
|---|---:|---|---|---|
| SHZ-001 | P1 | `attachApplication()` could leave `clientRecord == null` when the same UID/PID was already attached, then dereference it for non-manager clients. The pre-lock check also allowed a duplicate-add race. | `FIXED_BRANCH_UNMERGED` | Existing record is now captured; missing record is double-checked inside `synchronized (this)` before add. |
| SHZ-002 | P1 | `settings.gradle` called `.equals("true")` on a possibly absent `api.useLocal` property and accepted an enabled local API override without a validated `api.dir`. | `FIXED_BRANCH_UNMERGED` | Null-safe property comparison plus fail-fast validation for `api.dir`. |
| SHZ-003 | P2 | Rish export opened document/asset streams without scoped close and did not cover document creation with the same failure boundary. | `FIXED_BRANCH_UNMERGED` | Both streams now use Kotlin `use`; the operation is inside `runCatching` and failures are logged. |
| SHZ-004 | P0 | Pinned external `Shizuku-API` `Service.transactRemote()` clears Binder calling identity and restores it only after successful `targetBinder.transact()`. Exception path can skip restoration. | `BLOCKED_EXTERNAL_SUBMODULE` | Parent repo is not silently vendoring/changing the external submodule. Required patch is `restoreCallingIdentity` in `finally`, followed by exception-path identity test and build. |
| SHZ-005 | P2 | Legacy local `BuildUtils` contains an explicit TODO to replace it with `rikka.core.util.BuildUtils`. | `TOKEN_VAZIO_MIGRATION` | Migration value/compatibility not proven; no churn introduced. |
| SHZ-006 | P2 | `ShizukuService` still contains TODOs around user-service termination/runtime permission listener plus small dead/diagnostic code. | `TOKEN_VAZIO_BEHAVIOR` | Kept untouched because behavior impact needs tests; cosmetic cleanup is lower priority than regression control. |
| SHZ-007 | P1 | Repository had only `master`, no PR/issue ledger, and code search was not indexed at audit start. | `PARTIAL_REMEDIATION` | Dedicated audit branch created. Issue creation for SHZ-004 was attempted but GitHub returned HTTP 410 because Issues are disabled. PR is the remaining governance surface. |

## Commits already materialized

1. `5b90cdb7f812db30c972d426f11ac2fe26d51164` — `build: harden local API override configuration`
2. `83e8d9dbb56a79cc3993653e34b818d6be51a231` — `manager: close rish export streams and log failures`
3. `8b027ef07e9bfff7265e563ef2675ad343037610` — `server: make application attach idempotent and race-safe`

Before this audit document, the branch was 3 commits ahead and 0 behind `master`; code delta was limited to three files.

## Verification gates

### G0 — provenance

`PASS_OBSERVED`

The parent base SHA matched the observed upstream HEAD at audit start. This audit therefore does not pretend upstream synchronization is work performed by the branch.

### G1 — bounded parent diff

`PASS_OBSERVED`

Before this document, changes were confined to:

- `settings.gradle`
- `manager/src/main/java/moe/shizuku/manager/shell/ShellTutorialActivity.kt`
- `server/src/main/java/rikka/shizuku/server/ShizukuService.java`

### G2 — remote build

`TOKEN_VAZIO_REMOTE_BUILD`

No combined status/check was exposed for branch HEAD at the time of recording. No build PASS is claimed.

Falsifier: a Gradle/Android build failure on this branch invalidates promotion until fixed.

### G3 — runtime Android behavior

`TOKEN_VAZIO_DEVICE`

No physical-device ADB/root Shizuku startup, permission grant/revoke, reattach, Rish export, work-profile path, or binder-death test was executed by this audit environment.

Falsifier: any reproducible regression against unchanged `master` blocks merge.

### G4 — external Binder identity restoration

`BLOCKED_EXTERNAL_SUBMODULE`

Required minimal shape in the pinned API implementation:

```java
long id = Binder.clearCallingIdentity();
try {
    targetBinder.transact(targetCode, newData, reply, targetFlags);
} finally {
    Binder.restoreCallingIdentity(id);
}
```

This is a proposed correction, **not an applied parent-repository fix**.

## No-regression decision

`master` remains unchanged.

Promotion rule:

`MERGE_ALLOWED = remote_build_PASS && review_PASS && no_new_regression && external_P0_not_misrepresented`

Until those gates exist:

`claim_allowed=false`

## F_ok / F_gap / F_next

**F_ok** — three bounded hardening fixes are committed in an isolated branch; original upstream-aligned base remains intact.

**F_gap** — remote build and device runtime remain `TOKEN_VAZIO`; external Binder identity issue remains blocked in the pinned Shizuku-API dependency; repository Issues are disabled.

**F_next** — open a draft PR as the auditable review surface, collect CI/build evidence, then run Android/Termux device smoke tests before any merge.
