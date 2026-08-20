<!-- markdownlint-disable -->

# Hardening Report: giantswarm--install-binary-action/v3.1.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **giantswarm--install-binary-action/v3.1.1** was hardened automatically. 4 finding(s) were identified and resolved across 3 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple workflow files reference actions or reusable workflows by mutable tag or branch refs instead of full 40-character commit SHAs, making them vulnerable to supply-chain attacks:
- test.yaml: `actions/setup-node@v4` and `actions/checkout@v4` (tag refs)
- zz_generated.create_release_pr.yaml: `giantswarm/github-workflows/.github/workflows/create-release-pr.yaml@main` (branch ref)
- zz_generated.gitleaks.yaml: `giantswarm/github-workflows/.github/workflows/gitleaks.yaml@main` (branch ref)
- zz_generated.run_ossf_scorecard.yaml: `giantswarm/github-workflows/.github/workflows/ossf-scorecard.yaml@main` (branch ref)
- zz_generated.validate_changelog.yaml: `giantswarm/github-workflows/.github/workflows/validate-changelog.yaml@main` (branch ref)

Locations:

- `.github/workflows/test.yaml:13`
- `.github/workflows/test.yaml:16`
- `.github/workflows/zz_generated.create_release_pr.yaml:36`
- `.github/workflows/zz_generated.gitleaks.yaml:14`
- `.github/workflows/zz_generated.run_ossf_scorecard.yaml:27`
- `.github/workflows/zz_generated.validate_changelog.yaml:19`

### script-injection (severity: high)

Multiple `run:` blocks directly interpolate `${{ ... }}` expressions into shell commands, allowing template substitution before the shell parses the string — a classic script injection vector.

(a) zz_generated.create_release.yaml — `debug_info` job: `${{ toJson(github) }}` is interpolated directly into a heredoc run block. The entire github context (including attacker-controlled fields like head_commit.message) is injected into the shell.

(a) zz_generated.create_release.yaml — `update_project_go` job: `git checkout -b ${{ env.branch }}`, `file="${{ needs.gather_facts.outputs.project_go_path }}"`, `version="${{ needs.gather_facts.outputs.version }}"`, `git tag "v$version" ${{ github.sha }}`, `gh pr create ... --base ${{ env.base }} --head ${{ env.branch }} --reviewer ${{ github.actor }}`, `gh pr merge --auto --squash "${{ env.branch }}"` — all directly interpolated into run blocks.

(a) zz_generated.create_release.yaml — `create_release` job: `file="${{ needs.gather_facts.outputs.project_go_path }}"`, `version="${{ needs.gather_facts.outputs.version }}"`, `git tag "v$version" ${{ github.sha }}` — directly interpolated.

(a) zz_generated.create_release.yaml — `create-release-branch` job: `current_version="${{ needs.gather_facts.outputs.version }}"` — directly interpolated.

(a) zz_generated.ensure_major_version_tags.yaml — `debug_info` job: `${{ toJson(github) }}` directly in heredoc run block.

(a) zz_generated.add-team-labels.yaml — `add_label` job: `cat ${{steps.download-users.outputs.download-path}}/users.yaml` — steps output directly interpolated into a shell path in a run block.

(a) zz_generated.add-to-project-board.yaml — `add_to_personal_board` job: `cat ${{steps.download-users.outputs.download-path}}/users.yaml` — steps output directly interpolated.

(a) zz_generated.add-to-project-board.yaml — `add_to_team_board` job: `cat ${{steps.download-labels.outputs.download-path}}/labels.yaml` — steps output directly interpolated.

Locations:

- `.github/workflows/zz_generated.create_release.yaml:22`
- `.github/workflows/zz_generated.create_release.yaml:88`
- `.github/workflows/zz_generated.create_release.yaml:89`
- `.github/workflows/zz_generated.create_release.yaml:90`
- `.github/workflows/zz_generated.create_release.yaml:100`
- `.github/workflows/zz_generated.create_release.yaml:108`
- `.github/workflows/zz_generated.create_release.yaml:117`
- `.github/workflows/zz_generated.create_release.yaml:136`
- `.github/workflows/zz_generated.create_release.yaml:148`
- `.github/workflows/zz_generated.create_release.yaml:196`
- `.github/workflows/zz_generated.ensure_major_version_tags.yaml:17`
- `.github/workflows/zz_generated.add-team-labels.yaml:33`
- `.github/workflows/zz_generated.add-to-project-board.yaml:44`
- `.github/workflows/zz_generated.add-to-project-board.yaml:68`

### github-env-injection (severity: high)

Unsanitized external data is written to $GITHUB_ENV without the required `printf '%s' ... | tr -d '\n\r'` sanitization step, enabling environment variable injection attacks.

(1) zz_generated.add-team-labels.yaml — `add_label` job: The `TEAMS` variable is populated from an external YAML file via `yq`, then iterated to write `team/${team}` values directly to `$GITHUB_ENV` using a heredoc (`echo "LABEL<<EOF" >> $GITHUB_ENV` ... `echo "team/${team}" >> $GITHUB_ENV`). An attacker who can influence the users.yaml content could inject arbitrary environment variables.

(2) zz_generated.add-to-project-board.yaml — `add_to_personal_board` job: `BOARD` is populated from an external YAML file via `yq` and written directly as `echo "BOARD=${BOARD}" >> $GITHUB_ENV` without sanitization.

(3) zz_generated.add-to-project-board.yaml — `add_to_team_board` job: Same pattern — `BOARD` from external YAML written to `$GITHUB_ENV` unsanitized.

Locations:

- `.github/workflows/zz_generated.add-team-labels.yaml:36`
- `.github/workflows/zz_generated.add-to-project-board.yaml:49`
- `.github/workflows/zz_generated.add-to-project-board.yaml:73`

### missing-permissions (severity: medium)

The following workflow files have no top-level `permissions:` key and no job-level `permissions:` keys on any job, meaning they run with the default (potentially broad) token permissions:
- test.yaml
- zz_generated.add-team-labels.yaml
- zz_generated.add-to-project-board.yaml
- zz_generated.create_release.yaml
- zz_generated.create_release_pr.yaml
- zz_generated.ensure_major_version_tags.yaml
- zz_generated.gitleaks.yaml
- zz_generated.run_ossf_scorecard.yaml

(zz_generated.validate_changelog.yaml is the only file with a `permissions:` block.)

Locations:

- `.github/workflows/test.yaml:1`
- `.github/workflows/zz_generated.add-team-labels.yaml:1`
- `.github/workflows/zz_generated.add-to-project-board.yaml:1`
- `.github/workflows/zz_generated.create_release.yaml:1`
- `.github/workflows/zz_generated.create_release_pr.yaml:1`
- `.github/workflows/zz_generated.ensure_major_version_tags.yaml:1`
- `.github/workflows/zz_generated.gitleaks.yaml:1`
- `.github/workflows/zz_generated.run_ossf_scorecard.yaml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, github-env-injection, missing-permissions

**Notes:**

Fixed all 4 findings across 9 workflow files:

1. unpinned-uses: Pinned actions/setup-node@v4 to SHA 49933ea5..., actions/checkout@v4 to SHA 11d5960a..., and all 4 giantswarm/github-workflows@main reusable workflow refs to SHA ab31694b... (the current HEAD of main).

2. script-injection: Moved all ${{ }} expressions out of run: blocks into env: blocks in zz_generated.create_release.yaml (debug_info heredoc, update_project_go step, create_release step, create-release-branch step), zz_generated.ensure_major_version_tags.yaml (debug_info heredoc), zz_generated.add-team-labels.yaml (download-path), and zz_generated.add-to-project-board.yaml (both download-path references).

3. github-env-injection: Added printf '%s' ... | tr -d '\n\r' sanitization before writing external data to $GITHUB_ENV in all 3 locations (team labels in add-team-labels.yaml, BOARD in both jobs of add-to-project-board.yaml).

4. missing-permissions: Added permissions: {} top-level block to all 8 workflow files that lacked it (test.yaml, zz_generated.add-team-labels.yaml, zz_generated.add-to-project-board.yaml, zz_generated.create_release.yaml, zz_generated.create_release_pr.yaml, zz_generated.ensure_major_version_tags.yaml, zz_generated.gitleaks.yaml, zz_generated.run_ossf_scorecard.yaml).

### Iteration 2

**Fixes applied:** script-injection, github-env-injection

**Notes:**

Fixed 3 script-injection instances by moving `${{ secrets.ISSUE_AUTOMATION }}` expressions into `env:` blocks and referencing them as shell variables in wget --header arguments. Fixed 2 github-env-injection instances in zz_generated.create_release.yaml by adding explicit sanitization (printf + tr -d '\n\r') before writing `version` and `new_version` to $GITHUB_OUTPUT.

### Iteration 3

**Fixes applied:** script-injection

**Notes:**

Fixed four unquoted shell variable expansions in hardened/action/.github/workflows/zz_generated.create_release.yaml:
1. Line ~107: `semver bump patch $version` → `semver bump patch "$version"` in the 'Update project.go' step
2. Line ~127: `git add $file` → `git add "$file"` in the 'Commit changes' step
3. Line ~175: `grep -qE "..." $file` → `grep -qE "..." "$file"` in the 'Ensure correct version in project.go' step
4. Lines ~247-250: Four `semver get major/minor $current_version/$parent_version` calls → all variables now properly double-quoted in the 'Create long-lived release branch' step

