<!-- markdownlint-disable -->

# Hardening Report: nhs-england-tools--notify-msteams-action/v1.0.5

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **nhs-england-tools--notify-msteams-action/v1.0.5** was hardened automatically. 10 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): Direct ${{ }} expression interpolation inside run: shell blocks. In check-english-usage, check-file-format, and check-markdown-format, `${{ github.event.repository.default_branch }}` is interpolated directly into a shell export statement (e.g. `export BRANCH_NAME=origin/${{ github.event.repository.default_branch }}`). An attacker who controls the repository's default branch name could inject shell metacharacters.

Locations:

- `.github/actions/check-english-usage/action.yaml:9`
- `.github/actions/check-file-format/action.yaml:9`
- `.github/actions/check-markdown-format/action.yaml:9`

### script-injection (severity: high)

Sub-rule (a) and (b): In lint-terraform/action.yaml, `stacks=${{ inputs.root-modules }}` directly interpolates the user-supplied input into the shell without quoting. The variable is then used unquoted in `echo ${stacks//,/$'\n'}` inside a command substitution, allowing shell metacharacter injection via the `root-modules` input.

Locations:

- `.github/actions/lint-terraform/action.yaml:17`

### script-injection (severity: high)

Sub-rule (a): In perform-static-analysis/action.yaml, multiple ${{ inputs.* }} expressions are interpolated directly into run: shell commands: `echo "secret_exist=${{ inputs.sonar_token != '' }}" >> $GITHUB_OUTPUT` (line 18), `export SONAR_ORGANISATION_KEY=${{ inputs.sonar_organisation_key }}` (line 23), `export SONAR_PROJECT_KEY=${{ inputs.sonar_project_key }}` (line 24), and `export SONAR_TOKEN=${{ inputs.sonar_token }}` (line 25). Attacker-controlled input values are expanded by the YAML template engine before the shell sees them.

Locations:

- `.github/actions/perform-static-analysis/action.yaml:18`
- `.github/actions/perform-static-analysis/action.yaml:23`
- `.github/actions/perform-static-analysis/action.yaml:24`
- `.github/actions/perform-static-analysis/action.yaml:25`

### script-injection (severity: high)

Sub-rule (a): In create-lines-of-code-report/action.yaml, ${{ inputs.* }} expressions are interpolated directly into run: shell commands: `export BUILD_DATETIME=${{ inputs.build_datetime }}` (line 24), `echo "secrets_exist=${{ inputs.idp_aws_report_upload_role_name != '' && inputs.idp_aws_report_upload_bucket_endpoint != '' }}" >> $GITHUB_OUTPUT` (line 37), and `${{ inputs.idp_aws_report_upload_bucket_endpoint }}/${{ inputs.build_timestamp }}-lines-of-code-report.json.zip` in an aws s3 cp command (line 47). Attacker-controlled input values are expanded before the shell sees them.

Locations:

- `.github/actions/create-lines-of-code-report/action.yaml:24`
- `.github/actions/create-lines-of-code-report/action.yaml:37`
- `.github/actions/create-lines-of-code-report/action.yaml:47`

### script-injection (severity: high)

Sub-rule (a): In scan-dependencies/action.yaml, ${{ inputs.* }} expressions are interpolated directly into run: shell commands: `export BUILD_DATETIME=${{ inputs.build_datetime }}` (lines 25 and 34), `echo "secrets_exist=${{ inputs.idp_aws_report_upload_role_name != '' && inputs.idp_aws_report_upload_bucket_endpoint != '' }}" >> $GITHUB_OUTPUT` (line 44), and `${{ inputs.idp_aws_report_upload_bucket_endpoint }}/${{ inputs.build_timestamp }}-*.zip` in aws s3 cp commands (lines 50-55). Attacker-controlled input values are expanded before the shell sees them.

Locations:

- `.github/actions/scan-dependencies/action.yaml:25`
- `.github/actions/scan-dependencies/action.yaml:34`
- `.github/actions/scan-dependencies/action.yaml:44`
- `.github/actions/scan-dependencies/action.yaml:50`

### script-injection (severity: high)

Sub-rule (a): In commit-release-files/action.yaml, `commit_message="${{ inputs.commit_message }}"` (line 22) directly interpolates the user-supplied commit message into the shell. A malicious commit message containing shell metacharacters or newlines could alter command execution.

Locations:

- `.github/actions/commit-release-files/action.yaml:22`

### script-injection (severity: high)

Sub-rule (a): In update-major-tag/action.yaml, multiple ${{ }} expressions are interpolated directly into run: shell commands: `full_tag="v${{ inputs.full_release_version }}"` (line 22), `major_tag="v${{ inputs.major_release_version }}"` (line 23), and `gh api repos/${{ github.repository }}/git/ref/tags/${full_tag}` (line 26) plus two more gh api calls using `${{ github.repository }}` (lines 29, 32). Attacker-controlled version strings or repository names could inject shell metacharacters.

Locations:

- `.github/actions/update-major-tag/action.yaml:22`
- `.github/actions/update-major-tag/action.yaml:23`
- `.github/actions/update-major-tag/action.yaml:26`

### github-env-injection (severity: high)

In perform-static-analysis/action.yaml, the expression `${{ inputs.sonar_token != '' }}` is written directly to $GITHUB_OUTPUT without sanitization: `echo "secret_exist=${{ inputs.sonar_token != '' }}" >> $GITHUB_OUTPUT`. Although this is a boolean expression, the pattern of writing ${{ }} expressions directly to special environment files without `printf '%s' ... | tr -d '\n\r'` sanitization is unsafe.

Locations:

- `.github/actions/perform-static-analysis/action.yaml:18`

### github-env-injection (severity: high)

In create-lines-of-code-report/action.yaml, the expression `${{ inputs.idp_aws_report_upload_role_name != '' && inputs.idp_aws_report_upload_bucket_endpoint != '' }}` is written directly to $GITHUB_OUTPUT without sanitization: `echo "secrets_exist=${{ ... }}" >> $GITHUB_OUTPUT`. No `printf '%s' ... | tr -d '\n\r'` sanitization is applied before the write.

Locations:

- `.github/actions/create-lines-of-code-report/action.yaml:37`

### github-env-injection (severity: high)

In scan-dependencies/action.yaml, the expression `${{ inputs.idp_aws_report_upload_role_name != '' && inputs.idp_aws_report_upload_bucket_endpoint != '' }}` is written directly to $GITHUB_OUTPUT without sanitization: `echo "secrets_exist=${{ ... }}" >> $GITHUB_OUTPUT`. No `printf '%s' ... | tr -d '\n\r'` sanitization is applied before the write.

Locations:

- `.github/actions/scan-dependencies/action.yaml:44`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection

**Notes:**

Fixed all script-injection and github-env-injection findings across 7 action files:

1. check-english-usage/action.yaml: Moved `${{ github.event.repository.default_branch }}` to env block as DEFAULT_BRANCH, referenced as ${DEFAULT_BRANCH} in shell.

2. check-file-format/action.yaml: Same fix as above.

3. check-markdown-format/action.yaml: Same fix as above.

4. lint-terraform/action.yaml: Moved `${{ inputs.root-modules }}` to env block as INPUT_ROOT_MODULES, referenced as "${INPUT_ROOT_MODULES}" in shell.

5. perform-static-analysis/action.yaml: Moved all ${{ inputs.* }} expressions to env blocks (SONAR_ORGANISATION_KEY, SONAR_PROJECT_KEY, SONAR_TOKEN). Fixed github-env-injection by moving `${{ inputs.sonar_token != '' }}` to env block as SECRET_EXIST and sanitizing with printf '%s' | tr -d '\n\r' before writing to $GITHUB_OUTPUT.

6. create-lines-of-code-report/action.yaml: Moved BUILD_DATETIME, BUCKET_ENDPOINT, BUILD_TIMESTAMP to env blocks. Fixed github-env-injection by moving the boolean expression to env block as SECRETS_EXIST and sanitizing before writing to $GITHUB_OUTPUT.

7. scan-dependencies/action.yaml: Same pattern as create-lines-of-code-report — moved all ${{ inputs.* }} expressions to env blocks and sanitized the GITHUB_OUTPUT write.

8. commit-release-files/action.yaml: Moved `${{ inputs.commit_message }}` to env block as INPUT_COMMIT_MESSAGE. Also moved `${{ inputs.github_token }}` in the mask step to an env block.

9. update-major-tag/action.yaml: Moved `${{ inputs.full_release_version }}`, `${{ inputs.major_release_version }}`, and `${{ github.repository }}` to env blocks. Also moved `${{ inputs.github_token }}` in the mask step to an env block. All gh api calls now use quoted "${VAR}" references.

