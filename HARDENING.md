<!-- markdownlint-disable -->

# Hardening Report: nhs-england-tools--notify-msteams-action/v1.0.6

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **nhs-england-tools--notify-msteams-action/v1.0.6** was hardened automatically. 4 finding(s) were identified and resolved across 4 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Multiple run: blocks directly interpolate ${{ }} expressions inside shell commands, violating rule (a). This allows template substitution to inject arbitrary shell metacharacters before the shell ever sees the value.

1. cicd-1-pull-request.yaml — 'Fail if dist/ contains changes' step: `git diff --name-only origin/${{ github.base_ref }}...HEAD` — attacker-controlled branch name injected directly into shell.
2. cicd-1-pull-request.yaml — 'Set CI/CD variables' step: `echo "version=${{steps.semantic.outputs.new_release_version}}"` — step output interpolated directly.
3. cicd-1-pull-request.yaml — 'List variables' step: multiple `export VAR="${{ steps.variables.outputs.* }}"` lines — step outputs interpolated directly.
4. cicd-2-publish.yaml — 'List variables' step: multiple `export VAR="${{ steps.variables.outputs.* }}"` lines.
5. cicd-2-publish.yaml — 'Output new release details' step: `echo ${{ steps.semantic.outputs.new_release_version }}` (and similar) — unquoted step outputs interpolated directly.
6. cicd-2-publish.yaml — 'Check prerequisites for notification' step: `echo "secret_exist=${{ secrets.TEAMS_NOTIFICATION_WEBHOOK_URL != '' }}"` — secret expression interpolated directly.
7. cicd-3-deploy.yaml — 'Set CI/CD variables' step: `echo "version=${{ github.event.ref }}"` — github context interpolated directly.
8. cicd-3-deploy.yaml — 'List variables' step: multiple `export VAR="${{ steps.variables.outputs.* }}"` lines.

Locations:

- `.github/workflows/cicd-1-pull-request.yaml:19`
- `.github/workflows/cicd-1-pull-request.yaml:75`
- `.github/workflows/cicd-1-pull-request.yaml:100`
- `.github/workflows/cicd-2-publish.yaml:35`
- `.github/workflows/cicd-2-publish.yaml:152`
- `.github/workflows/cicd-2-publish.yaml:163`
- `.github/workflows/cicd-3-deploy.yaml:44`
- `.github/workflows/cicd-3-deploy.yaml:50`

### github-env-injection (severity: high)

Two run: blocks write values derived from untrusted/workflow-controlled expressions directly to $GITHUB_OUTPUT without the required sanitization step (printf '%s' ... | tr -d '\n\r').

1. cicd-1-pull-request.yaml — 'Set CI/CD variables' step writes `echo "version=${{steps.semantic.outputs.new_release_version}}" >> $GITHUB_OUTPUT` — a step output (workflow-controllable) is written directly to GITHUB_OUTPUT with no newline sanitization.
2. cicd-3-deploy.yaml — 'Set CI/CD variables' step writes `echo "version=${{ github.event.ref }}" >> $GITHUB_OUTPUT` — the github.event.ref context value (attacker-controllable via workflow_dispatch tag input) is written directly to GITHUB_OUTPUT with no newline sanitization.

Locations:

- `.github/workflows/cicd-1-pull-request.yaml:75`
- `.github/workflows/cicd-3-deploy.yaml:44`

### unpinned-uses (severity: high)

The following uses: reference is pinned to a mutable branch name rather than a full 40-character commit SHA, making it vulnerable to supply-chain attacks if the referenced branch is compromised:

- `uses: nhs-england-tools/notify-msteams-action@main` — the `@main` ref is a branch, not a SHA digest. Any commit pushed to that branch will be silently picked up by this workflow.

Locations:

- `.github/workflows/cicd-2-publish.yaml:167`

### missing-permissions (severity: medium)

Several workflow files have no top-level `permissions:` key and also lack job-level `permissions:` blocks on every job, meaning they run with the default (potentially broad) GITHUB_TOKEN permissions:

- cicd-3-deploy.yaml: No top-level permissions; neither the `metadata` nor `deploy` job has a permissions block.
- stage-1-commit.yaml: No top-level permissions; only `count-lines-of-code` and `scan-dependencies` jobs have job-level permissions — the remaining jobs (scan-secrets, check-file-format, check-markdown-format, check-english-usage, lint-terraform) do not.
- stage-2-test.yaml: No top-level permissions; only `perform-static-analysis` has job-level permissions — the remaining jobs (test-unit, test-lint, test-coverage, test-e2e) do not.
- stage-3-build.yaml: No top-level permissions and no job-level permissions on the `artefact-1` job.
- stage-4-acceptance.yaml: No top-level permissions and no job-level permissions on any of its many jobs.

Locations:

- `.github/workflows/cicd-3-deploy.yaml:1`
- `.github/workflows/stage-1-commit.yaml:1`
- `.github/workflows/stage-2-test.yaml:1`
- `.github/workflows/stage-3-build.yaml:1`
- `.github/workflows/stage-4-acceptance.yaml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection, unpinned-uses, missing-permissions

**Notes:**

Fixed all 4 finding types across 7 workflow files:

1. script-injection: Moved all ${{ }} expressions out of run: shell strings and into step env: blocks across cicd-1-pull-request.yaml, cicd-2-publish.yaml, and cicd-3-deploy.yaml. Shell scripts now reference plain environment variables.

2. github-env-injection: Added printf '%s' ... | tr -d '\n\r' sanitization before writing to GITHUB_OUTPUT in cicd-1-pull-request.yaml (new_release_version) and cicd-3-deploy.yaml (github.event.ref).

3. unpinned-uses: Pinned nhs-england-tools/notify-msteams-action@main to full commit SHA c0cbc559871271fa387121eb519057d58d1c0d82 in cicd-2-publish.yaml.

4. missing-permissions: Added top-level 'permissions: {}' and job-level 'permissions: contents: read' to all jobs lacking permissions in cicd-3-deploy.yaml, stage-1-commit.yaml, stage-2-test.yaml, stage-3-build.yaml, and stage-4-acceptance.yaml.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed script injection vulnerabilities in 9 files by moving all ${{ }} expressions out of run: shell commands and into env: blocks:

1. `.github/actions/update-major-tag/action.yaml`: Moved `inputs.github_token`, `inputs.full_release_version`, `inputs.major_release_version`, and `github.repository` to env blocks. Shell script now references `$GITHUB_TOKEN_INPUT`, `$FULL_RELEASE_VERSION`, `$MAJOR_RELEASE_VERSION`, `$GITHUB_REPOSITORY_INPUT`.

2. `.github/actions/create-lines-of-code-report/action.yaml`: Moved `inputs.build_datetime` to env block for CLOC report step. Replaced the `secrets_exist` expression with a shell conditional using env vars `$ROLE_NAME` and `$BUCKET_ENDPOINT`. Moved `inputs.idp_aws_report_upload_bucket_endpoint` and `inputs.build_timestamp` to env block for the S3 upload step.

3. `.github/actions/scan-dependencies/action.yaml`: Same pattern as create-lines-of-code-report — moved `inputs.build_datetime` (used twice) to env blocks, replaced `secrets_exist` expression with shell conditional, moved bucket endpoint and timestamp to env block for S3 uploads.

4. `.github/actions/perform-static-analysis/action.yaml`: Replaced `inputs.sonar_token != ''` expression with a shell conditional using `$SONAR_TOKEN_INPUT` env var. Moved `inputs.sonar_organisation_key`, `inputs.sonar_project_key`, and `inputs.sonar_token` to env block.

5. `.github/actions/check-english-usage/action.yaml`: Moved `github.event.repository.default_branch` to env block as `DEFAULT_BRANCH`, referenced as `${DEFAULT_BRANCH}` in shell.

6. `.github/actions/check-file-format/action.yaml`: Same fix as check-english-usage.

7. `.github/actions/check-markdown-format/action.yaml`: Same fix as check-english-usage.

8. `.github/actions/lint-terraform/action.yaml`: Moved `inputs.root-modules` to env block as `INPUT_ROOT_MODULES`.

9. `.github/actions/commit-release-files/action.yaml`: Moved `inputs.github_token` to env block for mask step, moved `inputs.commit_message` to env block as `COMMIT_MESSAGE_INPUT`.

10. `.github/workflows/cicd-1-pull-request.yaml`: Moved `steps.app-token.outputs.token` to env block as `APP_TOKEN` for the mask step.

11. `.github/workflows/cicd-2-publish.yaml`: Same fix as cicd-1-pull-request.yaml.

### Iteration 3

**Fixes applied:** script-injection

**Notes:**

Fixed the script injection vulnerability in `.github/actions/lint-terraform/action.yaml` at line 20. The `${stacks//,/$'\n'}` expansion inside the `for` loop's command substitution was unquoted, allowing attacker-controlled values in `inputs.root-modules` to inject shell metacharacters. Fixed by double-quoting the expansion: `"${stacks//,/$'\n'}"`. The `inputs.root-modules` value was already correctly placed in the `env:` block as `INPUT_ROOT_MODULES`, so no other changes were needed.

### Iteration 4

**Fixes applied:** script-injection

**Notes:**

Fixed all four script-injection findings:
1. `.github/workflows/cicd-1-pull-request.yaml` line 108: Quoted `$branch_name` in `gh pr list --head "$branch_name"` and also quoted `$GITHUB_REF` in the sed command.
2. `.github/actions/check-english-usage/action.yaml` line 11: Quoted `${DEFAULT_BRANCH}` → `export BRANCH_NAME="origin/${DEFAULT_BRANCH}"`.
3. `.github/actions/check-file-format/action.yaml` line 11: Same fix as above.
4. `.github/actions/check-markdown-format/action.yaml` line 11: Same fix as above.
5. `.github/actions/lint-terraform/action.yaml` lines 18-19: Quoted `${INPUT_ROOT_MODULES}` assignment → `stacks="${INPUT_ROOT_MODULES}"`.
6. `.github/actions/update-major-tag/action.yaml` lines 32/35/38: Quoted all `gh api` URL path arguments containing `${GITHUB_REPOSITORY_INPUT}`, `${full_tag}`, and `${major_tag}` to prevent shell metacharacter injection.

