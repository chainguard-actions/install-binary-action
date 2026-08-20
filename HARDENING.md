<!-- markdownlint-disable -->

# Hardening Report: giantswarm--install-binary-action/v4.0.2

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **giantswarm--install-binary-action/v4.0.2** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple workflow files reference external reusable workflows using the mutable `@main` branch ref instead of a pinned 40-character SHA commit hash. This allows supply-chain attacks if the referenced repository is compromised. Affected references: `giantswarm/github-workflows/.github/workflows/create-release.yaml@main`, `giantswarm/github-workflows/.github/workflows/create-release-pr.yaml@main`, `giantswarm/github-workflows/.github/workflows/gitleaks.yaml@main`, `giantswarm/github-workflows/.github/workflows/ossf-scorecard.yaml@main`, `giantswarm/github-workflows/.github/workflows/validate-changelog.yaml@main`.

Locations:

- `.github/workflows/zz_generated.create_release.yaml:22`
- `.github/workflows/zz_generated.create_release_pr.yaml:30`
- `.github/workflows/zz_generated.gitleaks.yaml:14`
- `.github/workflows/zz_generated.run_ossf_scorecard.yaml:30`
- `.github/workflows/zz_generated.validate_changelog.yaml:15`

### missing-permissions (severity: medium)

Several workflow files have no top-level `permissions:` key and at least one job also lacks a job-level `permissions:` block, meaning those jobs inherit the default broad token permissions. (1) `test.yaml` has no top-level permissions and the `test` job has no job-level permissions. (2) `zz_generated.add-team-labels.yaml` has no top-level permissions and the `add_label` job has no job-level permissions. (3) `zz_generated.add-to-project-board.yaml` has no top-level permissions and both the `add_to_personal_board` and `add_to_team_board` jobs have no job-level permissions.

Locations:

- `.github/workflows/test.yaml:1`
- `.github/workflows/zz_generated.add-team-labels.yaml:1`
- `.github/workflows/zz_generated.add-to-project-board.yaml:1`

### script-injection (severity: high)

GitHub Actions expressions (`${{ steps.*.outputs.* }}`) are interpolated directly inside `run:` shell command strings, violating sub-rule (a). Before the shell ever sees the string, the YAML template substitution inserts the raw value, allowing an attacker who controls the output value to inject arbitrary shell commands. (1) `zz_generated.add-team-labels.yaml`: `TEAMS=$(cat ${{steps.download-users.outputs.download-path}}/users.yaml ...)` — the step output is spliced directly into a shell command. (2) `zz_generated.add-to-project-board.yaml`: same pattern with `${{steps.download-users.outputs.download-path}}` and `${{steps.download-labels.outputs.download-path}}`.

Locations:

- `.github/workflows/zz_generated.add-team-labels.yaml:37`
- `.github/workflows/zz_generated.add-to-project-board.yaml:55`
- `.github/workflows/zz_generated.add-to-project-board.yaml:88`

### github-env-injection (severity: high)

Untrusted data derived from the GitHub event payload is written to `$GITHUB_ENV` without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`). An attacker who controls the issue assignee login or label name can inject newlines to set arbitrary environment variables for subsequent steps. (1) `zz_generated.add-team-labels.yaml`: `event_assignee` (from `$GITHUB_EVENT_PATH`) is used to build `TEAMS`, which is then written line-by-line to `$GITHUB_ENV` via a heredoc (`echo "LABEL<<EOF" >> $GITHUB_ENV` / `echo "team/${team}" >> $GITHUB_ENV`). (2) `zz_generated.add-to-project-board.yaml`: `BOARD` (derived from event assignee/label data) is written unsanitized via `echo "BOARD=${BOARD}" >> $GITHUB_ENV` in both the `add_to_personal_board` and `add_to_team_board` jobs.

Locations:

- `.github/workflows/zz_generated.add-team-labels.yaml:40`
- `.github/workflows/zz_generated.add-team-labels.yaml:43`
- `.github/workflows/zz_generated.add-to-project-board.yaml:61`
- `.github/workflows/zz_generated.add-to-project-board.yaml:94`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, missing-permissions, script-injection, github-env-injection

**Notes:**

1. unpinned-uses: Pinned all 5 @main references in generated workflow files to SHA 4014c2d7776bc9f81df0d7a84e8f597cc3699b9d with # main comment.
2. missing-permissions: Added permissions: {} top-level to test.yaml, zz_generated.add-team-labels.yaml, and zz_generated.add-to-project-board.yaml. Added job-level permissions to add_label (issues: write), add_to_personal_board (contents: read), and add_to_team_board (contents: read).
3. script-injection: Moved ${{steps.download-users.outputs.download-path}} and ${{steps.download-labels.outputs.download-path}} expressions out of run: blocks into env: blocks as DOWNLOAD_PATH, referenced as ${DOWNLOAD_PATH} in shell.
4. github-env-injection: Sanitized team label values and BOARD values with printf '%s' ... | tr -d '\n\r' before writing to $GITHUB_ENV in both affected workflow files.

