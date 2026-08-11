# SHIZUKU POST-MERGE RECONCILIATION — 2026-08-11

State: `TOKEN_VAZIO_RECONCILIATION_IN_PROGRESS`

PR #1 was merged externally at 2026-08-11T18:45:11Z with merge commit `a12627b447aeee16939779b3cb0cfc7a3648c21c` while the audit branch continued to receive additional commits afterward.

Observed relation immediately after detection:

- `master`: `a12627b447aeee16939779b3cb0cfc7a3648c21c`
- merge base with audit branch: `155baf5c80d67e2a8954e8c2eefc627dac0c0f7d`
- audit branch: 5 commits ahead / 1 commit behind master
- post-merge file delta: `.github/workflows/app.yml`, `ShellTutorialActivity.kt`, `settings.gradle`, `signing.gradle`

Invariant: do not rewrite merged history. Rebase/replay only the post-merge delta onto a new branch from current master, then open a new draft PR with `claim_allowed=false` until remote build/device gates are observed.
