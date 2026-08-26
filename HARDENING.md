<!-- markdownlint-disable -->

# Hardening Report: nhs-england-tools--notify-msteams-action/v1.0.7

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **nhs-england-tools--notify-msteams-action/v1.0.7** was hardened automatically. 4 finding(s) were identified and resolved across 3 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

The workflow uses `nhs-england-tools/notify-msteams-action@main`, which is a mutable branch reference rather than a pinned 40-character commit SHA. This means the action can be silently updated to a different (potentially malicious) version at any time.

Locations:

- `.github/workflows/cicd-2-publish.yaml:200`

### missing-permissions (severity: medium)

Several workflow files have no top-level `permissions:` block and at least one job also lacks a job-level `permissions:` block, meaning those jobs run with the default (overly broad) token permissions.

- `cicd-3-deploy.yaml`: No top-level permissions and no job-level permissions on any job.
- `stage-2-test.yaml`: No top-level permissions; only the `perform-static-analysis` job has a `permissions:` block — jobs `test-unit`, `test-lint`, `test-coverage`, and `test-e2e` do not.
- `stage-3-build.yaml`: No top-level permissions and no job-level permissions on any job.
- `stage-4-acceptance.yaml`: No top-level permissions and no job-level permissions on any job.

Locations:

- `.github/workflows/cicd-3-deploy.yaml:1`
- `.github/workflows/stage-2-test.yaml:1`
- `.github/workflows/stage-3-build.yaml:1`
- `.github/workflows/stage-4-acceptance.yaml:1`

### script-injection (severity: high)

Multiple `run:` blocks interpolate `${{ ... }}` expressions directly into shell commands, violating sub-rule (a). An attacker who can control the referenced context value can inject arbitrary shell commands.

**cicd-1-pull-request.yaml**
- `origin/${{ github.base_ref }}` interpolated directly in a `git diff` command (guard-dist job). `github.base_ref` is attacker-controlled on pull_request events.
- `echo "version=${{steps.semantic.outputs.new_release_version}}" >> $GITHUB_OUTPUT` — step output interpolated directly in run block (metadata job).
- Multiple `export VAR="${{ steps.variables.outputs.* }}"` and `export VAR="${{ steps.pr_exists.outputs.* }}"` expressions interpolated directly in a run block (List variables step).

**cicd-2-publish.yaml**
- `echo "::add-mask::${{ steps.app-token.outputs.token }}"` — token interpolated directly in run block.
- `echo ${{ steps.semantic.outputs.new_release_version }}` (and major/minor/patch variants) — step outputs interpolated directly in run block (Output new release details step).
- `echo "secret_exist=${{ secrets.TEAMS_NOTIFICATION_WEBHOOK_URL != '' }}" >> $GITHUB_OUTPUT` — expression interpolated directly in run block.

**cicd-3-deploy.yaml**
- `echo "version=${{ github.event.ref }}" >> $GITHUB_OUTPUT` — `github.event.ref` is attacker-controllable and interpolated directly in a run block.
- Multiple `export VAR="${{ steps.variables.outputs.* }}"` and `export TAG="${{ steps.variables.outputs.tag }}"` expressions interpolated directly in a run block (List variables step).

**cicd-1-pull-request.yaml (pr-title-check context via stage)**
- `echo "Current=${{ steps.version_check.outputs.current_version }}, ..."` — step outputs interpolated directly in run block.

**.github/actions/check-english-usage/action.yaml**
- `export BRANCH_NAME=origin/${{ github.event.repository.default_branch }}` — github context interpolated directly in run block.

**.github/actions/check-file-format/action.yaml**
- `export BRANCH_NAME=origin/${{ github.event.repository.default_branch }}` — github context interpolated directly in run block.

**.github/actions/check-markdown-format/action.yaml**
- `export BRANCH_NAME=origin/${{ github.event.repository.default_branch }}` — github context interpolated directly in run block.

**.github/actions/check-pr-title/action.yaml**
- `PR_TITLE: '${{ github.event.pull_request.title }}'` is routed via env, but the env var `${PR_TITLE}` is used unquoted in the `if [[ -z "${PR_TITLE}" ]]` and `if ! [[ "${PR_TITLE}" =~ ${regex} ]]` checks — however the env routing is correct here. The primary issue is the `${{ github.event.pull_request.title }}` expression itself in the env block.

**.github/actions/create-lines-of-code-report/action.yaml**
- `export BUILD_DATETIME=${{ inputs.build_datetime }}` — inputs interpolated directly in run block.
- `echo "secrets_exist=${{ inputs.idp_aws_report_upload_role_name != '' && inputs.idp_aws_report_upload_bucket_endpoint != '' }}" >> $GITHUB_OUTPUT` — inputs interpolated directly in run block.
- `${{ inputs.idp_aws_report_upload_bucket_endpoint }}/${{ inputs.build_timestamp }}-lines-of-code-report.json.zip` — inputs interpolated directly in aws s3 cp command.

**.github/actions/perform-static-analysis/action.yaml**
- `echo "secret_exist=${{ inputs.sonar_token != '' }}" >> $GITHUB_OUTPUT` — input interpolated directly in run block.
- `export SONAR_ORGANISATION_KEY=${{ inputs.sonar_organisation_key }}`, `export SONAR_PROJECT_KEY=${{ inputs.sonar_project_key }}`, `export SONAR_TOKEN=${{ inputs.sonar_token }}` — inputs interpolated directly in run block.

**.github/actions/scan-dependencies/action.yaml**
- `export BUILD_DATETIME=${{ inputs.build_datetime }}` (twice) — inputs interpolated directly in run block.
- `echo "secrets_exist=${{ inputs.idp_aws_report_upload_role_name != '' && inputs.idp_aws_report_upload_bucket_endpoint != '' }}" >> $GITHUB_OUTPUT` — inputs interpolated directly in run block.
- `${{ inputs.idp_aws_report_upload_bucket_endpoint }}/${{ inputs.build_timestamp }}-*.zip` — inputs interpolated directly in aws s3 cp commands.

**.github/actions/update-major-tag/action.yaml**
- `full_tag="v${{ inputs.full_release_version }}"`, `major_tag="v${{ inputs.major_release_version }}"` — inputs interpolated directly in run block.
- `gh api repos/${{ github.repository }}/git/ref/tags/${full_tag}` — github context interpolated directly in run block.

**.github/actions/commit-release-files/action.yaml**
- `commit_message="${{ inputs.commit_message }}"` — input interpolated directly in run block.
- `echo "::add-mask::${{ inputs.github_token }}"` — input interpolated directly in run block.

Locations:

- `.github/workflows/cicd-1-pull-request.yaml:18`
- `.github/workflows/cicd-1-pull-request.yaml:97`
- `.github/workflows/cicd-1-pull-request.yaml:107`
- `.github/workflows/cicd-2-publish.yaml:57`
- `.github/workflows/cicd-2-publish.yaml:163`
- `.github/workflows/cicd-2-publish.yaml:196`
- `.github/workflows/cicd-3-deploy.yaml:47`
- `.github/workflows/cicd-3-deploy.yaml:52`
- `.github/actions/check-english-usage/action.yaml:9`
- `.github/actions/check-file-format/action.yaml:9`
- `.github/actions/check-markdown-format/action.yaml:9`
- `.github/actions/create-lines-of-code-report/action.yaml:24`
- `.github/actions/create-lines-of-code-report/action.yaml:33`
- `.github/actions/create-lines-of-code-report/action.yaml:48`
- `.github/actions/perform-static-analysis/action.yaml:17`
- `.github/actions/perform-static-analysis/action.yaml:23`
- `.github/actions/scan-dependencies/action.yaml:24`
- `.github/actions/scan-dependencies/action.yaml:38`
- `.github/actions/scan-dependencies/action.yaml:51`
- `.github/actions/update-major-tag/action.yaml:21`
- `.github/actions/update-major-tag/action.yaml:26`
- `.github/actions/commit-release-files/action.yaml:14`
- `.github/actions/commit-release-files/action.yaml:20`

### github-env-injection (severity: high)

Multiple `run:` blocks write values derived from untrusted `${{ ... }}` expressions directly to `$GITHUB_OUTPUT` without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`). A newline character in the value can be used to inject arbitrary key=value pairs into the output file, potentially overwriting subsequent step outputs.

**cicd-1-pull-request.yaml** — `echo "version=${{steps.semantic.outputs.new_release_version}}" >> $GITHUB_OUTPUT`: step output written directly to GITHUB_OUTPUT without sanitization.

**cicd-2-publish.yaml** — `echo "secret_exist=${{ secrets.TEAMS_NOTIFICATION_WEBHOOK_URL != '' }}" >> $GITHUB_OUTPUT`: expression result written directly to GITHUB_OUTPUT without sanitization.

**cicd-3-deploy.yaml** — `echo "version=${{ github.event.ref }}" >> $GITHUB_OUTPUT`: `github.event.ref` (attacker-controllable via workflow_dispatch `tag` input) written directly to GITHUB_OUTPUT without sanitization.

**.github/actions/create-lines-of-code-report/action.yaml** — `echo "secrets_exist=${{ inputs.idp_aws_report_upload_role_name != '' && inputs.idp_aws_report_upload_bucket_endpoint != '' }}" >> $GITHUB_OUTPUT`: input-derived expression written directly to GITHUB_OUTPUT without sanitization.

**.github/actions/perform-static-analysis/action.yaml** — `echo "secret_exist=${{ inputs.sonar_token != '' }}" >> $GITHUB_OUTPUT`: input-derived expression written directly to GITHUB_OUTPUT without sanitization.

**.github/actions/scan-dependencies/action.yaml** — `echo "secrets_exist=${{ inputs.idp_aws_report_upload_role_name != '' && inputs.idp_aws_report_upload_bucket_endpoint != '' }}" >> $GITHUB_OUTPUT`: input-derived expression written directly to GITHUB_OUTPUT without sanitization.

Locations:

- `.github/workflows/cicd-1-pull-request.yaml:97`
- `.github/workflows/cicd-2-publish.yaml:196`
- `.github/workflows/cicd-3-deploy.yaml:47`
- `.github/actions/create-lines-of-code-report/action.yaml:33`
- `.github/actions/perform-static-analysis/action.yaml:17`
- `.github/actions/scan-dependencies/action.yaml:51`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, missing-permissions, script-injection, github-env-injection

**Notes:**

Fixed all 4 findings across 13 files:

1. unpinned-uses: Pinned nhs-england-tools/notify-msteams-action@main to SHA 3d354c122e09513108d34d93c729874da45d26a2 in cicd-2-publish.yaml.

2. missing-permissions: Added top-level permissions: {contents: read} and job-level permissions blocks to cicd-3-deploy.yaml (2 jobs), stage-2-test.yaml (4 jobs), stage-3-build.yaml (1 job), and stage-4-acceptance.yaml (9 jobs).

3. script-injection: Moved all ${{ }} expressions from run: blocks to env: blocks in cicd-1-pull-request.yaml, cicd-2-publish.yaml, cicd-3-deploy.yaml, check-english-usage/action.yaml, check-file-format/action.yaml, check-markdown-format/action.yaml, create-lines-of-code-report/action.yaml, perform-static-analysis/action.yaml, scan-dependencies/action.yaml, update-major-tag/action.yaml, and commit-release-files/action.yaml.

4. github-env-injection: Added printf '%s' "$VAR" | tr -d '\n\r' sanitization before all GITHUB_OUTPUT writes that were derived from ${{ }} expressions in cicd-1-pull-request.yaml, cicd-2-publish.yaml, cicd-3-deploy.yaml, create-lines-of-code-report/action.yaml, perform-static-analysis/action.yaml, and scan-dependencies/action.yaml.

### Iteration 2

**Fixes applied:** script-injection, missing-permissions

**Notes:**

1. Fixed script-injection in .github/workflows/pr-title-check.yaml: Moved ${{ steps.version_check.outputs.current_version }}, ${{ steps.version_check.outputs.next_increment }}, and ${{ steps.version_check.outputs.next_version }} from the 'Show predicted release summary' run: block into an env: block (as CURRENT_VERSION, NEXT_INCREMENT, NEXT_VERSION), then referenced them as plain shell variables. 2. Fixed missing-permissions in .github/workflows/stage-1-commit.yaml: Added 'permissions: contents: read' to the four jobs that lacked explicit permissions (scan-secrets, check-file-format, check-markdown-format, check-english-usage). These jobs only need read access to checkout and inspect code.

### Iteration 3

**Fixes applied:** script-injection

**Notes:**

Fixed the script injection vulnerability in `.github/workflows/cicd-1-pull-request.yaml` at the 'Check if pull request exists for this branch' step. Changed `gh pr list --head $branch_name` to `gh pr list --head "$branch_name"` so that the branch name (derived from the attacker-controlled GITHUB_HEAD_REF environment variable) is properly double-quoted, preventing shell metacharacters in branch names from being interpreted as shell commands.

