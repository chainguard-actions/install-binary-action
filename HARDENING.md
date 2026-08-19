<!-- markdownlint-disable -->

# Hardening Report: giantswarm--install-binary-action/v4.1.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **giantswarm--install-binary-action/v4.1.0** was hardened automatically. 6 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple workflow files reference reusable workflows from giantswarm/github-workflows at the mutable @main branch ref instead of a pinned 40-character SHA commit hash. This exposes the action to supply-chain attacks. Affected: js-dependency-audit.yaml uses giantswarm/github-workflows/.github/workflows/js-dependency-audit.yaml@main; zz_generated.create_release.yaml uses create-release.yaml@main; zz_generated.create_release_pr.yaml uses create-release-pr.yaml@main; zz_generated.gitleaks.yaml uses gitleaks.yaml@main; zz_generated.run_ossf_scorecard.yaml uses ossf-scorecard.yaml@main; zz_generated.validate_changelog.yaml uses validate-changelog.yaml@main.

Locations:

- `.github/workflows/js-dependency-audit.yaml:12`
- `.github/workflows/zz_generated.create_release.yaml:22`
- `.github/workflows/zz_generated.create_release_pr.yaml:36`
- `.github/workflows/zz_generated.gitleaks.yaml:13`
- `.github/workflows/zz_generated.run_ossf_scorecard.yaml:27`
- `.github/workflows/zz_generated.validate_changelog.yaml:17`

### missing-permissions (severity: medium)

test.yaml has no top-level permissions: key and its only job (test) also has no job-level permissions: key. Without explicit permissions the workflow inherits default repository permissions which may be overly broad.

Locations:

- `.github/workflows/test.yaml:1`

### missing-permissions (severity: medium)

zz_generated.add-team-labels.yaml has no top-level permissions: key and the add_label job has no job-level permissions: key (only build_user_list has permissions: contents: read). The add_label job inherits default repository permissions.

Locations:

- `.github/workflows/zz_generated.add-team-labels.yaml:22`

### missing-permissions (severity: medium)

zz_generated.add-to-project-board.yaml has no top-level permissions: key and the add_to_personal_board and add_to_team_board jobs have no job-level permissions: keys. Only build_user_list has permissions: contents: read. The other two jobs inherit default repository permissions.

Locations:

- `.github/workflows/zz_generated.add-to-project-board.yaml:38`
- `.github/workflows/zz_generated.add-to-project-board.yaml:60`

### script-injection (severity: high)

Rule (a): GitHub Actions expressions from the steps context are interpolated directly inside run: shell command strings. In zz_generated.add-team-labels.yaml line 35: `TEAMS=$(cat ${{steps.download-users.outputs.download-path}}/users.yaml ...)`. In zz_generated.add-to-project-board.yaml line 52: `BOARD=($(cat ${{steps.download-users.outputs.download-path}}/users.yaml ...))` and line 72: `BOARD=($(cat ${{steps.download-labels.outputs.download-path}}/labels.yaml ...))`. The steps.*.outputs.* context flows through YAML template substitution before the shell parses it, allowing injection of arbitrary shell commands.

Locations:

- `.github/workflows/zz_generated.add-team-labels.yaml:35`
- `.github/workflows/zz_generated.add-to-project-board.yaml:52`
- `.github/workflows/zz_generated.add-to-project-board.yaml:72`

### github-env-injection (severity: high)

Untrusted data derived from step outputs (injected via ${{ steps.*.outputs.download-path }} expressions in the same run: block) is written to $GITHUB_ENV without sanitization (no `printf '%s' ... | tr -d '\n\r'`). In zz_generated.add-team-labels.yaml, TEAMS (built from the step output path) is written to $GITHUB_ENV via heredoc at line 40. In zz_generated.add-to-project-board.yaml, BOARD is written as `echo "BOARD=${BOARD}" >> $GITHUB_ENV` at lines 57 and 77. An attacker who can influence the step output value could inject newlines to set arbitrary environment variables.

Locations:

- `.github/workflows/zz_generated.add-team-labels.yaml:40`
- `.github/workflows/zz_generated.add-to-project-board.yaml:57`
- `.github/workflows/zz_generated.add-to-project-board.yaml:77`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, missing-permissions, script-injection, github-env-injection

**Notes:**

Fixed all 6 findings:

1. unpinned-uses: Pinned all 6 giantswarm/github-workflows@main references to SHA fc6951257f3ae7513411f23ddc8093156f67c757 in js-dependency-audit.yaml, zz_generated.create_release.yaml, zz_generated.create_release_pr.yaml, zz_generated.gitleaks.yaml, zz_generated.run_ossf_scorecard.yaml, and zz_generated.validate_changelog.yaml.

2. missing-permissions (test.yaml): Added top-level `permissions: {}` block.

3. missing-permissions (zz_generated.add-team-labels.yaml): Added `permissions: issues: write` to the add_label job.

4. missing-permissions (zz_generated.add-to-project-board.yaml): Added `permissions: contents: read` to both add_to_personal_board and add_to_team_board jobs.

5. script-injection: Moved all `${{ steps.*.outputs.download-path }}` expressions out of run: shell strings into the step's env: block as DOWNLOAD_PATH, then referenced via ${DOWNLOAD_PATH} in the shell.

6. github-env-injection: Sanitized all values written to $GITHUB_ENV using `printf '%s' "${value}" | tr -d '\n\r'` before writing, preventing newline injection attacks.

