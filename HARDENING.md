<!-- markdownlint-disable -->

# Hardening Report: giantswarm--install-binary-action/v4.0.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **giantswarm--install-binary-action/v4.0.0** was hardened automatically. 8 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Multiple run: blocks in zz_generated.create_release.yaml directly interpolate ${{ ... }} expressions inside shell commands, violating rule (a). Examples include: `${{ toJson(github) }}` echoed via heredoc (line 23), `git checkout -b ${{ env.branch }}` (unquoted, line ~107), `file="${{ needs.gather_facts.outputs.project_go_path }}"` and `version="${{ needs.gather_facts.outputs.version }}"` interpolated directly into shell (lines ~108-109, ~131, ~163-164), `git tag "v$version" ${{ github.sha }}` (line ~163), `gh pr create --title "${{ env.title }}" --body "" --base ${{ env.base }} --head ${{ env.branch }} --reviewer ${{ github.actor }}` (line ~122), and `gh pr merge --auto --squash "${{ env.branch }}"` (line ~130). Any of these allow an attacker-controlled value to be interpreted as shell code before the shell ever sees it.

Locations:

- `.github/workflows/zz_generated.create_release.yaml:23`
- `.github/workflows/zz_generated.create_release.yaml:107`
- `.github/workflows/zz_generated.create_release.yaml:108`
- `.github/workflows/zz_generated.create_release.yaml:109`
- `.github/workflows/zz_generated.create_release.yaml:122`
- `.github/workflows/zz_generated.create_release.yaml:130`
- `.github/workflows/zz_generated.create_release.yaml:163`
- `.github/workflows/zz_generated.create_release.yaml:164`

### script-injection (severity: high)

run: block in zz_generated.ensure_major_version_tags.yaml directly interpolates `${{ toJson(github) }}` inside a heredoc shell command (rule (a)). The entire github context — including attacker-controllable fields such as github.head_ref — is expanded by the YAML template engine before the shell sees it, enabling script injection.

Locations:

- `.github/workflows/zz_generated.ensure_major_version_tags.yaml:17`

### script-injection (severity: high)

run: block in zz_generated.add-team-labels.yaml directly interpolates `${{steps.download-users.outputs.download-path}}` inside a shell command (rule (a)): `TEAMS=$(cat ${{steps.download-users.outputs.download-path}}/users.yaml ...)`. A step output is expanded by the YAML template engine before the shell sees it, enabling script injection.

Locations:

- `.github/workflows/zz_generated.add-team-labels.yaml:36`

### script-injection (severity: high)

run: blocks in zz_generated.add-to-project-board.yaml directly interpolate step outputs inside shell commands (rule (a)): `${{steps.download-users.outputs.download-path}}` and `${{steps.download-labels.outputs.download-path}}` are expanded by the YAML template engine before the shell sees them, enabling script injection.

Locations:

- `.github/workflows/zz_generated.add-to-project-board.yaml:52`
- `.github/workflows/zz_generated.add-to-project-board.yaml:79`

### github-env-injection (severity: high)

In zz_generated.add-team-labels.yaml, the run: block writes the value of `${TEAMS}` — derived from the GitHub issue event's assignee login via `yq` — to $GITHUB_ENV using a multiline heredoc (`echo "LABEL<<EOF" >> $GITHUB_ENV; echo "team/${team}" >> $GITHUB_ENV; echo "EOF" >> $GITHUB_ENV`). The team names come from external YAML data keyed by the issue assignee (attacker-controlled). No `printf '%s' ... | tr -d '\n\r'` sanitization is applied before the write, so a newline in a team name could inject arbitrary environment variables.

Locations:

- `.github/workflows/zz_generated.add-team-labels.yaml:38`

### github-env-injection (severity: high)

In zz_generated.add-to-project-board.yaml, two run: blocks write `${BOARD}` — derived from the GitHub issue event's assignee login or label name via `yq` — to $GITHUB_ENV (`echo "BOARD=${BOARD}" >> $GITHUB_ENV`). The board URL comes from external YAML data keyed by attacker-controllable event fields. No `printf '%s' ... | tr -d '\n\r'` sanitization is applied before the write, so a newline in the board value could inject arbitrary environment variables.

Locations:

- `.github/workflows/zz_generated.add-to-project-board.yaml:57`
- `.github/workflows/zz_generated.add-to-project-board.yaml:84`

### unpinned-uses (severity: high)

The following workflow files use `uses:` references pinned to a mutable branch name (`@main`) rather than an immutable 40-character commit SHA. This means the referenced workflow can be silently changed by the upstream repository at any time, enabling supply-chain attacks:
- zz_generated.create_release_pr.yaml: `giantswarm/github-workflows/.github/workflows/create-release-pr.yaml@main`
- zz_generated.gitleaks.yaml: `giantswarm/github-workflows/.github/workflows/gitleaks.yaml@main`
- zz_generated.run_ossf_scorecard.yaml: `giantswarm/github-workflows/.github/workflows/ossf-scorecard.yaml@main`
- zz_generated.validate_changelog.yaml: `giantswarm/github-workflows/.github/workflows/validate-changelog.yaml@main`

Locations:

- `.github/workflows/zz_generated.create_release_pr.yaml:35`
- `.github/workflows/zz_generated.gitleaks.yaml:14`
- `.github/workflows/zz_generated.run_ossf_scorecard.yaml:27`
- `.github/workflows/zz_generated.validate_changelog.yaml:17`

### missing-permissions (severity: medium)

The following workflow files have no top-level `permissions:` key and contain at least one job that also lacks a `permissions:` key, meaning those jobs run with the default (potentially broad) token permissions:
- test.yaml: no permissions at any level
- zz_generated.create_release.yaml: no top-level permissions; jobs debug_info, gather_facts, update_project_go, create_release, ensure_floating_major_version_tags, create-release-branch all lack permissions
- zz_generated.create_release_pr.yaml: no permissions at any level
- zz_generated.ensure_major_version_tags.yaml: no permissions at any level
- zz_generated.gitleaks.yaml: no permissions at any level
- zz_generated.run_ossf_scorecard.yaml: no permissions at any level
- zz_generated.add-team-labels.yaml: only build_user_list job has permissions; add_label job does not
- zz_generated.add-to-project-board.yaml: only build_user_list job has permissions; add_to_personal_board and add_to_team_board jobs do not

Locations:

- `.github/workflows/test.yaml:1`
- `.github/workflows/zz_generated.create_release.yaml:1`
- `.github/workflows/zz_generated.create_release_pr.yaml:1`
- `.github/workflows/zz_generated.ensure_major_version_tags.yaml:1`
- `.github/workflows/zz_generated.gitleaks.yaml:1`
- `.github/workflows/zz_generated.run_ossf_scorecard.yaml:1`
- `.github/workflows/zz_generated.add-team-labels.yaml:1`
- `.github/workflows/zz_generated.add-to-project-board.yaml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection, unpinned-uses, missing-permissions

**Notes:**

Fixed all findings across 8 workflow files:

1. script-injection (zz_generated.create_release.yaml): Moved all ${{ }} expressions from run: blocks into env: blocks — toJson(github) heredoc, git checkout branch, project_go_path/version interpolations, gh pr create args (title/base/branch/reviewer), gh pr merge branch, git tag sha.

2. script-injection (zz_generated.ensure_major_version_tags.yaml): Moved ${{ toJson(github) }} from heredoc into env: GITHUB_CONTEXT variable.

3. script-injection (zz_generated.add-team-labels.yaml): Moved steps.download-users.outputs.download-path into env: DOWNLOAD_PATH.

4. script-injection (zz_generated.add-to-project-board.yaml): Moved both download-path step outputs into env: DOWNLOAD_PATH in their respective jobs.

5. github-env-injection (zz_generated.add-team-labels.yaml): Added printf '%s' | tr -d '\n\r' sanitization before writing team names to GITHUB_ENV.

6. github-env-injection (zz_generated.add-to-project-board.yaml): Added printf '%s' | tr -d '\n\r' sanitization before writing board URLs to GITHUB_ENV in both personal and team board jobs.

7. unpinned-uses: Replaced all four @main references with pinned SHA 4014c2d7776bc9f81df0d7a84e8f597cc3699b9d in create_release_pr.yaml, gitleaks.yaml, run_ossf_scorecard.yaml, and validate_changelog.yaml.

8. missing-permissions: Added top-level permissions blocks to all 8 affected workflow files, plus per-job permissions blocks to jobs that lacked them.

### Iteration 2

**Fixes applied:** github-env-injection

**Notes:**

Fixed two github-env-injection vulnerabilities in `.github/workflows/zz_generated.create_release.yaml`:
1. In the `Get version` step: added `safe_version=$(printf '%s' "${version}" | tr -d '\n\r')` and changed the GITHUB_OUTPUT write to use `safe_version` instead of `version`.
2. In the `Check if reference version` step: added `safe_refversion=$(printf '%s' "${refversion}" | tr -d '\n\r')` and changed the GITHUB_OUTPUT write to use `safe_refversion` instead of `refversion`.
Both values are derived from `COMMIT_MESSAGE` (set to `${{ github.event.head_commit.message }}`), which is attacker-controlled. The sanitization strips newline characters that could be used to inject additional key=value pairs into `$GITHUB_OUTPUT`.

