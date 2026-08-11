# Shizuku hardening — PR #1 receipt — 2026-08-11

Mode: `APPEND_ONLY_RECEIPT / CLAIM_ALLOWED=false`

Parent audit: `docs/rafaelia/SHIZUKU_HARDENING_AUDIT_2026-08-11.md`

PR: `#1` — `Hardening: reduce attach, build-config and Rish export failure modes`

Base: `master@b844bc491f1790c72328e1a8e5b2349f8978f0ea`

Head before this receipt: `audit/hardening-no-regression-20260811@7b11acc731efd7ad489e81eb24571d29bdfc3e20`

Observed PR state after creation:

- open: true
- draft: true
- merged: false
- mergeable: true
- commits before this receipt: 5
- changed files before this receipt: 5
- additions/deletions before this receipt: +170 / -13

Changed-file boundary observed before this receipt:

1. `docs/rafaelia/SHIZUKU_HARDENING_AUDIT_2026-08-11.md`
2. `manager/src/main/java/moe/shizuku/manager/shell/ShellTutorialActivity.kt`
3. `patches/shizuku-api/0001-restore-binder-calling-identity-finally.patch`
4. `server/src/main/java/rikka/shizuku/server/ShizukuService.java`
5. `settings.gradle`

Issue-tracker attempt:

- attempted to create a P0 issue for the external Binder identity-restoration risk;
- GitHub returned HTTP 410: Issues are disabled in this repository;
- tracking therefore remains in the audit ledger, proposed patch, and draft PR.

Remote status observation:

- combined commit statuses visible for head: none;
- therefore `REMOTE_BUILD = TOKEN_VAZIO_REMOTE_BUILD`;
- absence of a visible status is not treated as PASS or FAIL.

Promotion invariant:

`merge = false` until build evidence + review + runtime no-regression evidence exist.

F_ok: PR review surface exists and is mergeable without touching master.

F_gap: build/check evidence and physical Android runtime remain TOKEN_VAZIO; external Shizuku-API correction remains proposed-only.

F_next: collect CI/build evidence on the new receipt head, then execute device smoke matrix before changing draft/merge state.
