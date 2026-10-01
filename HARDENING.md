<!-- markdownlint-disable -->

# Hardening Report: nhs-england-tools--notify-msteams-action/v1.0.7

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **nhs-england-tools--notify-msteams-action/v1.0.7** was hardened automatically. 6 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): Direct ${{ }} expression interpolation inside run: shell commands. In check-english-usage/action.yaml, check-file-format/action.yaml, and check-markdown-format/action.yaml, the github context value is interpolated directly into a shell assignment: `export BRANCH_NAME=origin/${{ github.event.repository.default_branch }}`. An attacker who can control the repository's default_branch name could inject shell metacharacters.

Locations:

- `.github/actions/check-english-usage/action.yaml:9`
- `.github/actions/check-file-format/action.yaml:9`
- `.github/actions/check-markdown-format/action.yaml:9`

### script-injection (severity: high)

Sub-rule (a): Direct ${{ }} expression interpolation inside run: shell commands. In commit-release-files/action.yaml, the input value is interpolated directly into a shell variable assignment: `commit_message="${{ inputs.commit_message }}"`. A caller supplying a crafted commit_message containing shell metacharacters (e.g. `$(...)`, backticks) could achieve command injection.

Locations:

- `.github/actions/commit-release-files/action.yaml:22`

### script-injection (severity: high)

Sub-rule (a): Direct ${{ }} expression interpolation inside run: shell commands. In create-lines-of-code-report/action.yaml, inputs are interpolated directly into shell: `export BUILD_DATETIME=${{ inputs.build_datetime }}` and `${{ inputs.idp_aws_report_upload_bucket_endpoint }}/${{ inputs.build_timestamp }}-lines-of-code-report.json.zip` (passed as an unquoted argument to aws s3 cp). Any of these inputs could contain shell metacharacters.

Locations:

- `.github/actions/create-lines-of-code-report/action.yaml:27`
- `.github/actions/create-lines-of-code-report/action.yaml:47`

### script-injection (severity: high)

Sub-rule (a): Direct ${{ }} expression interpolation inside run: shell commands. In perform-static-analysis/action.yaml, multiple inputs are interpolated directly into shell variable assignments: `export SONAR_ORGANISATION_KEY=${{ inputs.sonar_organisation_key }}`, `export SONAR_PROJECT_KEY=${{ inputs.sonar_project_key }}`, and `export SONAR_TOKEN=${{ inputs.sonar_token }}`. A caller supplying crafted values could inject shell commands.

Locations:

- `.github/actions/perform-static-analysis/action.yaml:22`
- `.github/actions/perform-static-analysis/action.yaml:23`
- `.github/actions/perform-static-analysis/action.yaml:24`

### script-injection (severity: high)

Sub-rule (a): Direct ${{ }} expression interpolation inside run: shell commands. In scan-dependencies/action.yaml, inputs are interpolated directly into shell: `export BUILD_DATETIME=${{ inputs.build_datetime }}` (appears in two steps), and `${{ inputs.idp_aws_report_upload_bucket_endpoint }}/${{ inputs.build_timestamp }}-sbom-repository-report.json.zip` and the equivalent for the vulnerabilities report (unquoted arguments to aws s3 cp). Any of these inputs could contain shell metacharacters.

Locations:

- `.github/actions/scan-dependencies/action.yaml:27`
- `.github/actions/scan-dependencies/action.yaml:40`
- `.github/actions/scan-dependencies/action.yaml:57`
- `.github/actions/scan-dependencies/action.yaml:60`

### script-injection (severity: high)

Sub-rule (a): Direct ${{ }} expression interpolation inside run: shell commands. In update-major-tag/action.yaml, inputs and github context are interpolated directly into shell: `full_tag="v${{ inputs.full_release_version }}"`, `major_tag="v${{ inputs.major_release_version }}"`, and `gh api repos/${{ github.repository }}/git/ref/tags/${full_tag}` (and the DELETE/POST variants). A caller supplying crafted version strings or a repository name with special characters could inject shell commands.

Locations:

- `.github/actions/update-major-tag/action.yaml:21`
- `.github/actions/update-major-tag/action.yaml:22`
- `.github/actions/update-major-tag/action.yaml:25`
- `.github/actions/update-major-tag/action.yaml:28`
- `.github/actions/update-major-tag/action.yaml:31`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection

**Notes:**

Fixed all 6 script-injection findings across 8 action files by moving ${{ }} expressions from run: shell commands into step env: blocks and referencing them as plain environment variables:

1. check-english-usage/action.yaml: github.event.repository.default_branch → env: DEFAULT_BRANCH
2. check-file-format/action.yaml: github.event.repository.default_branch → env: DEFAULT_BRANCH
3. check-markdown-format/action.yaml: github.event.repository.default_branch → env: DEFAULT_BRANCH
4. commit-release-files/action.yaml: inputs.commit_message → env: INPUT_COMMIT_MESSAGE
5. create-lines-of-code-report/action.yaml: inputs.build_datetime → env: INPUT_BUILD_DATETIME; inputs.idp_aws_report_upload_bucket_endpoint + inputs.build_timestamp → env: INPUT_BUCKET_ENDPOINT + INPUT_BUILD_TIMESTAMP
6. perform-static-analysis/action.yaml: inputs.sonar_organisation_key/sonar_project_key/sonar_token → env: INPUT_SONAR_ORGANISATION_KEY/INPUT_SONAR_PROJECT_KEY/INPUT_SONAR_TOKEN
7. scan-dependencies/action.yaml: Two occurrences of inputs.build_datetime → env: INPUT_BUILD_DATETIME; inputs.idp_aws_report_upload_bucket_endpoint + inputs.build_timestamp → env: INPUT_BUCKET_ENDPOINT + INPUT_BUILD_TIMESTAMP
8. update-major-tag/action.yaml: inputs.full_release_version + inputs.major_release_version + github.repository → env: INPUT_FULL_RELEASE_VERSION + INPUT_MAJOR_RELEASE_VERSION + GITHUB_REPOSITORY_NAME

All shell scripts now reference environment variables using ${VAR_NAME} syntax instead of inline ${{ }} expressions. Remaining ${{ }} in run: blocks are boolean comparisons (always true/false) or ::add-mask:: commands, which were not flagged in the findings.

### Iteration 2

**Fixes applied:** script-injection, github-env-injection

**Notes:**

Fixed 5 locations across 5 files:
1. commit-release-files/action.yaml (line 16): Moved `${{ inputs.github_token }}` to an `env:` block as `INPUT_GITHUB_TOKEN` and referenced it as `$INPUT_GITHUB_TOKEN` in the shell command.
2. update-major-tag/action.yaml (line 16): Same fix as above.
3. perform-static-analysis/action.yaml (line 18): Replaced template expression `${{ inputs.sonar_token != '' }}` with shell logic evaluating `$INPUT_SONAR_TOKEN` (from env block), with `tr -d '\n\r'` sanitization before writing to `$GITHUB_OUTPUT`.
4. create-lines-of-code-report/action.yaml (line 46): Replaced template boolean expression with shell logic using `INPUT_ROLE_NAME` and `INPUT_BUCKET_ENDPOINT` env vars, with sanitization before writing to `$GITHUB_OUTPUT`.
5. scan-dependencies/action.yaml (line 57): Same fix as #4.

