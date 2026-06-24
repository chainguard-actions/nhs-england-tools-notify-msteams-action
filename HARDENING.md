<!-- markdownlint-disable -->

# Hardening Report: nhs-england-tools--notify-msteams-action/v1.0.5

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **nhs-england-tools--notify-msteams-action/v1.0.5** was hardened automatically. 10 finding(s) were identified and resolved across 3 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a): `${{ github.event.repository.default_branch }}` is interpolated directly inside a `run:` shell command string. An attacker who can control the repository's default branch name could inject arbitrary shell commands.

Offending line: `export BRANCH_NAME=origin/${{ github.event.repository.default_branch }}`

Locations:

- `.github/actions/check-english-usage/action.yaml:9`
- `.github/actions/check-file-format/action.yaml:9`
- `.github/actions/check-markdown-format/action.yaml:9`

### script-injection (severity: high)

Rule (a): `${{ inputs.commit_message }}` and `${{ inputs.github_token }}` are interpolated directly inside `run:` shell command strings. An attacker supplying a crafted input value could inject arbitrary shell commands.

Offending lines:
- `echo "::add-mask::${{ inputs.github_token }}"` (line 15)
- `commit_message="${{ inputs.commit_message }}"` (line 23)

Locations:

- `.github/actions/commit-release-files/action.yaml:15`
- `.github/actions/commit-release-files/action.yaml:23`

### script-injection (severity: high)

Rule (a): Multiple `${{ inputs.* }}` expressions are interpolated directly inside `run:` shell command strings. An attacker supplying crafted input values could inject arbitrary shell commands.

Offending lines:
- `echo "secret_exist=${{ inputs.sonar_token != '' }}" >> $GITHUB_OUTPUT` (line 19)
- `export SONAR_ORGANISATION_KEY=${{ inputs.sonar_organisation_key }}` (line 24)
- `export SONAR_PROJECT_KEY=${{ inputs.sonar_project_key }}` (line 25)
- `export SONAR_TOKEN=${{ inputs.sonar_token }}` (line 26)

Locations:

- `.github/actions/perform-static-analysis/action.yaml:19`
- `.github/actions/perform-static-analysis/action.yaml:24`
- `.github/actions/perform-static-analysis/action.yaml:25`
- `.github/actions/perform-static-analysis/action.yaml:26`

### script-injection (severity: high)

Rule (a): `${{ inputs.root-modules }}` is interpolated directly inside a `run:` shell command string. An attacker supplying a crafted input value could inject arbitrary shell commands.

Offending line: `stacks=${{ inputs.root-modules }}`

Locations:

- `.github/actions/lint-terraform/action.yaml:17`

### script-injection (severity: high)

Rule (a): Multiple `${{ inputs.* }}` expressions are interpolated directly inside `run:` shell command strings. An attacker supplying crafted input values could inject arbitrary shell commands.

Offending lines:
- `export BUILD_DATETIME=${{ inputs.build_datetime }}` (line 27)
- `echo "secrets_exist=${{ inputs.idp_aws_report_upload_role_name != '' && inputs.idp_aws_report_upload_bucket_endpoint != '' }}" >> $GITHUB_OUTPUT` (line 40)
- `${{ inputs.idp_aws_report_upload_bucket_endpoint }}/${{ inputs.build_timestamp }}-lines-of-code-report.json.zip` (line 52)

Locations:

- `.github/actions/create-lines-of-code-report/action.yaml:27`
- `.github/actions/create-lines-of-code-report/action.yaml:40`
- `.github/actions/create-lines-of-code-report/action.yaml:52`

### script-injection (severity: high)

Rule (a): Multiple `${{ inputs.* }}` expressions are interpolated directly inside `run:` shell command strings. An attacker supplying crafted input values could inject arbitrary shell commands.

Offending lines:
- `export BUILD_DATETIME=${{ inputs.build_datetime }}` (lines 27 and 43)
- `echo "secrets_exist=${{ ... }}" >> $GITHUB_OUTPUT` (line 57)
- `${{ inputs.idp_aws_report_upload_bucket_endpoint }}/${{ inputs.build_timestamp }}-sbom-repository-report.json.zip` (line 68)
- `${{ inputs.idp_aws_report_upload_bucket_endpoint }}/${{ inputs.build_timestamp }}-vulnerabilities-repository-report.json.zip` (line 71)

Locations:

- `.github/actions/scan-dependencies/action.yaml:27`
- `.github/actions/scan-dependencies/action.yaml:43`
- `.github/actions/scan-dependencies/action.yaml:57`
- `.github/actions/scan-dependencies/action.yaml:68`
- `.github/actions/scan-dependencies/action.yaml:71`

### script-injection (severity: high)

Rule (a): Multiple `${{ inputs.* }}` and `${{ github.repository }}` expressions are interpolated directly inside `run:` shell command strings. An attacker supplying crafted input values could inject arbitrary shell commands.

Offending lines:
- `echo "::add-mask::${{ inputs.github_token }}"` (line 17)
- `full_tag="v${{ inputs.full_release_version }}"` (line 23)
- `major_tag="v${{ inputs.major_release_version }}"` (line 24)
- `tag_sha=$(gh api repos/${{ github.repository }}/git/ref/tags/${full_tag} ...)` (line 27)
- `gh api --method DELETE repos/${{ github.repository }}/git/refs/tags/${major_tag} ...` (line 30)
- `gh api --method POST repos/${{ github.repository }}/git/refs ...` (line 33)

Locations:

- `.github/actions/update-major-tag/action.yaml:17`
- `.github/actions/update-major-tag/action.yaml:23`
- `.github/actions/update-major-tag/action.yaml:24`
- `.github/actions/update-major-tag/action.yaml:27`
- `.github/actions/update-major-tag/action.yaml:30`
- `.github/actions/update-major-tag/action.yaml:33`

### github-env-injection (severity: high)

A `${{ inputs.* }}` expression is written directly to `$GITHUB_OUTPUT` without sanitization (no `printf '%s' ... | tr -d '\n\r'` step). A newline embedded in the input value could inject additional key=value pairs into the output file.

Offending line: `echo "secret_exist=${{ inputs.sonar_token != '' }}" >> $GITHUB_OUTPUT`

Locations:

- `.github/actions/perform-static-analysis/action.yaml:19`

### github-env-injection (severity: high)

A `${{ inputs.* }}` expression is written directly to `$GITHUB_OUTPUT` without sanitization (no `printf '%s' ... | tr -d '\n\r'` step). A newline embedded in the input value could inject additional key=value pairs into the output file.

Offending line: `echo "secrets_exist=${{ inputs.idp_aws_report_upload_role_name != '' && inputs.idp_aws_report_upload_bucket_endpoint != '' }}" >> $GITHUB_OUTPUT`

Locations:

- `.github/actions/create-lines-of-code-report/action.yaml:40`

### github-env-injection (severity: high)

A `${{ inputs.* }}` expression is written directly to `$GITHUB_OUTPUT` without sanitization (no `printf '%s' ... | tr -d '\n\r'` step). A newline embedded in the input value could inject additional key=value pairs into the output file.

Offending line: `echo "secrets_exist=${{ inputs.idp_aws_report_upload_role_name != '' && inputs.idp_aws_report_upload_bucket_endpoint != '' }}" >> $GITHUB_OUTPUT`

Locations:

- `.github/actions/scan-dependencies/action.yaml:57`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection

**Notes:**

Fixed all script-injection and github-env-injection findings across 9 action files:

1. check-english-usage/action.yaml: Moved `github.event.repository.default_branch` to env block as DEFAULT_BRANCH.
2. check-file-format/action.yaml: Same fix as above.
3. check-markdown-format/action.yaml: Same fix as above.
4. commit-release-files/action.yaml: Moved `inputs.github_token` to env block (INPUT_GITHUB_TOKEN) for the mask step, and `inputs.commit_message` to env block (INPUT_COMMIT_MESSAGE) for the commit step.
5. perform-static-analysis/action.yaml: Replaced the boolean expression interpolation with a shell conditional using env var SONAR_TOKEN_VALUE; sanitized GITHUB_OUTPUT write with printf/tr; moved sonar inputs to env block.
6. lint-terraform/action.yaml: Moved `inputs.root-modules` to env block as ROOT_MODULES.
7. create-lines-of-code-report/action.yaml: Moved `inputs.build_datetime` to env block; replaced boolean expression interpolation with shell conditional for GITHUB_OUTPUT write (sanitized with printf/tr); moved bucket endpoint and timestamp to env block for aws s3 cp.
8. scan-dependencies/action.yaml: Same pattern as create-lines-of-code-report for both build_datetime steps, the GITHUB_OUTPUT write, and the aws s3 cp commands.
9. update-major-tag/action.yaml: Moved `inputs.github_token` to env block for mask step; moved `inputs.full_release_version`, `inputs.major_release_version`, and `github.repository` to env block for the update step.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed two script injection vulnerabilities: (1) In lint-terraform/action.yaml, double-quoted the `${stacks//,/$'\n'}` expansion inside the command substitution (`echo "${stacks//,/$'\n'}"`), preventing attacker-controlled `inputs.root-modules` values from injecting shell commands. (2) In perform-static-analysis/action.yaml, double-quoted the outer `${GITHUB_HEAD_REF:-...}` expansion and `$GITHUB_REF` inside the subshell (`export BRANCH_NAME="${GITHUB_HEAD_REF:-$(echo "$GITHUB_REF" | sed 's#refs/heads/##')}"`), preventing an attacker-controlled branch name from injecting shell commands.

### Iteration 3

**Fixes applied:** script-injection

**Notes:**

Fixed the script injection vulnerability in `.github/actions/lint-terraform/action.yaml` in the 'Validate Terraform' step. Replaced the unquoted `for dir in $(find ...; echo "${stacks//,/$'\n'}")` construct with a `while IFS= read -r dir; do ... done < <(find ...; printf '%s\n' "${stacks//,/$'\n'}")` pattern. This eliminates word-splitting and glob expansion on the attacker-controllable `ROOT_MODULES` input, as each line is now read atomically by `read -r` rather than being subject to shell field-splitting. Empty lines are skipped with a guard, and the directory variable is properly quoted in the make invocation.

