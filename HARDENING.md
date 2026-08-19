<!-- markdownlint-disable -->

# Hardening Report: giantswarm--install-binary-action/v4.0.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **giantswarm--install-binary-action/v4.0.1** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Five workflow files reference the reusable workflow `giantswarm/github-workflows` using the mutable branch ref `@main` instead of a pinned 40-character commit SHA. This allows the referenced workflow to change at any time without notice, enabling supply-chain attacks. Failing references:
- `giantswarm/github-workflows/.github/workflows/create-release.yaml@main`
- `giantswarm/github-workflows/.github/workflows/create-release-pr.yaml@main`
- `giantswarm/github-workflows/.github/workflows/gitleaks.yaml@main`
- `giantswarm/github-workflows/.github/workflows/ossf-scorecard.yaml@main`
- `giantswarm/github-workflows/.github/workflows/validate-changelog.yaml@main`

Locations:

- `.github/workflows/zz_generated.create_release.yaml:22`
- `.github/workflows/zz_generated.create_release_pr.yaml:33`
- `.github/workflows/zz_generated.gitleaks.yaml:14`
- `.github/workflows/zz_generated.run_ossf_scorecard.yaml:30`
- `.github/workflows/zz_generated.validate_changelog.yaml:17`

### missing-permissions (severity: medium)

Three workflow files have no top-level `permissions:` key and contain at least one job that also lacks a `permissions:` key, meaning those jobs run with the default (potentially broad) token permissions.

- `test.yaml`: No top-level permissions; the `test` job has no `permissions:` block.
- `zz_generated.add-team-labels.yaml`: No top-level permissions; the `add_label` job has no `permissions:` block (only `build_user_list` does).
- `zz_generated.add-to-project-board.yaml`: No top-level permissions; the `add_to_personal_board` and `add_to_team_board` jobs have no `permissions:` blocks (only `build_user_list` does).

Locations:

- `.github/workflows/test.yaml:1`
- `.github/workflows/zz_generated.add-team-labels.yaml:1`
- `.github/workflows/zz_generated.add-to-project-board.yaml:1`

### script-injection (severity: high)

GitHub Actions expressions (`${{ ... }}`) are interpolated directly inside `run:` shell command strings (sub-rule a). The expression `${{steps.download-users.outputs.download-path}}` and `${{steps.download-labels.outputs.download-path}}` are embedded in shell commands, meaning the YAML template substitution injects the value before the shell ever sees it. A malicious value in a step output could inject arbitrary shell commands.

- `zz_generated.add-team-labels.yaml`: `TEAMS=$(cat ${{steps.download-users.outputs.download-path}}/users.yaml | ...)` inside a `run:` block.
- `zz_generated.add-to-project-board.yaml`: `BOARD=($(cat ${{steps.download-users.outputs.download-path}}/users.yaml | ...))` and `BOARD=($(cat ${{steps.download-labels.outputs.download-path}}/labels.yaml | ...))` inside `run:` blocks.

Locations:

- `.github/workflows/zz_generated.add-team-labels.yaml:33`
- `.github/workflows/zz_generated.add-to-project-board.yaml:52`
- `.github/workflows/zz_generated.add-to-project-board.yaml:80`

### github-env-injection (severity: high)

Values derived from step outputs (an untrusted source via `steps.*.outputs.*`) are written to `$GITHUB_ENV` without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`). This allows newline injection that can override subsequent environment variables in the same workflow run.

- `zz_generated.add-team-labels.yaml`: The `TEAMS` variable (populated from `${{steps.download-users.outputs.download-path}}`) is iterated and written to `$GITHUB_ENV` via `echo "team/${team}" >> $GITHUB_ENV` with no sanitization.
- `zz_generated.add-to-project-board.yaml`: The `BOARD` variable (populated from `${{steps.download-users.outputs.download-path}}` and `${{steps.download-labels.outputs.download-path}}`) is written to `$GITHUB_ENV` via `echo "BOARD=${BOARD}" >> $GITHUB_ENV` with no sanitization in two separate jobs.

Locations:

- `.github/workflows/zz_generated.add-team-labels.yaml:37`
- `.github/workflows/zz_generated.add-to-project-board.yaml:56`
- `.github/workflows/zz_generated.add-to-project-board.yaml:84`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, missing-permissions, script-injection, github-env-injection

**Notes:**

Fixed all four findings:

1. unpinned-uses: Pinned all 5 `giantswarm/github-workflows` references from `@main` to `@4014c2d7776bc9f81df0d7a84e8f597cc3699b9d # main` in: zz_generated.create_release.yaml, zz_generated.create_release_pr.yaml, zz_generated.gitleaks.yaml, zz_generated.run_ossf_scorecard.yaml, zz_generated.validate_changelog.yaml.

2. missing-permissions: Added `permissions: {}` at top-level and appropriate job-level permissions to test.yaml (contents:read for test job), zz_generated.add-team-labels.yaml (issues:write for add_label job), and zz_generated.add-to-project-board.yaml (contents:read for add_to_personal_board and add_to_team_board jobs).

3. script-injection: Moved `${{steps.download-users.outputs.download-path}}` and `${{steps.download-labels.outputs.download-path}}` expressions from inline `run:` shell strings into `env:` blocks as `DOWNLOAD_PATH`, then referenced as `${DOWNLOAD_PATH}` in the shell scripts.

4. github-env-injection: Added sanitization using `printf '%s' "${value}" | tr -d '\n\r'` before writing team names and BOARD values to `$GITHUB_ENV` in both affected workflow files.

