<!-- markdownlint-disable -->

# Hardening Report: nhs-england-tools--notify-msteams-action/v1.0.6

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **nhs-england-tools--notify-msteams-action/v1.0.6** was hardened automatically. 8 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a): Direct ${{ }} expression interpolation inside run: shell commands. In check-english-usage/action.yaml, check-file-format/action.yaml, and check-markdown-format/action.yaml, the github.event.repository.default_branch context is interpolated directly into a shell assignment: `export BRANCH_NAME=origin/${{ github.event.repository.default_branch }}`. This allows an attacker who controls the repository's default branch name to inject shell metacharacters.

Locations:

- `.github/actions/check-english-usage/action.yaml:9`
- `.github/actions/check-file-format/action.yaml:9`
- `.github/actions/check-markdown-format/action.yaml:9`

### script-injection (severity: high)

Rule (a): Direct ${{ }} expression interpolation inside run: shell commands in commit-release-files/action.yaml. The inputs.github_token is interpolated directly in `echo "::add-mask::${{ inputs.github_token }}"` and inputs.commit_message is interpolated in `commit_message="${{ inputs.commit_message }}"`. A caller-controlled commit_message value could inject shell metacharacters.

Locations:

- `.github/actions/commit-release-files/action.yaml:14`
- `.github/actions/commit-release-files/action.yaml:22`

### script-injection (severity: high)

Rule (a): Direct ${{ }} expression interpolation inside run: shell commands in create-lines-of-code-report/action.yaml. The inputs.build_datetime is interpolated directly in `export BUILD_DATETIME=${{ inputs.build_datetime }}`. Additionally, inputs.idp_aws_report_upload_bucket_endpoint and inputs.build_timestamp are interpolated directly in the aws s3 cp command. Caller-controlled input values could inject shell metacharacters.

Locations:

- `.github/actions/create-lines-of-code-report/action.yaml:24`
- `.github/actions/create-lines-of-code-report/action.yaml:49`

### script-injection (severity: high)

Rule (a): Direct ${{ }} expression interpolation inside run: shell commands in lint-terraform/action.yaml. The inputs.root-modules is interpolated directly in `stacks=${{ inputs.root-modules }}`, allowing a caller to inject shell metacharacters into the variable assignment and subsequent loop.

Locations:

- `.github/actions/lint-terraform/action.yaml:14`

### script-injection (severity: high)

Rule (a): Direct ${{ }} expression interpolation inside run: shell commands in perform-static-analysis/action.yaml. The inputs.sonar_organisation_key, inputs.sonar_project_key, and inputs.sonar_token are all interpolated directly into shell export statements (e.g. `export SONAR_TOKEN=${{ inputs.sonar_token }}`). Caller-controlled values could inject shell metacharacters.

Locations:

- `.github/actions/perform-static-analysis/action.yaml:20`
- `.github/actions/perform-static-analysis/action.yaml:21`
- `.github/actions/perform-static-analysis/action.yaml:22`

### script-injection (severity: high)

Rule (a): Direct ${{ }} expression interpolation inside run: shell commands in scan-dependencies/action.yaml. The inputs.build_datetime is interpolated directly in two separate `export BUILD_DATETIME=${{ inputs.build_datetime }}` statements, and inputs.idp_aws_report_upload_bucket_endpoint and inputs.build_timestamp are interpolated directly in aws s3 cp commands.

Locations:

- `.github/actions/scan-dependencies/action.yaml:24`
- `.github/actions/scan-dependencies/action.yaml:35`
- `.github/actions/scan-dependencies/action.yaml:49`

### script-injection (severity: high)

Rule (a): Direct ${{ }} expression interpolation inside run: shell commands in update-major-tag/action.yaml. The inputs.github_token is interpolated in `echo "::add-mask::${{ inputs.github_token }}"`, and inputs.full_release_version, inputs.major_release_version, and github.repository are interpolated directly into shell variable assignments and gh api commands (e.g. `full_tag="v${{ inputs.full_release_version }}"`). Caller-controlled values could inject shell metacharacters.

Locations:

- `.github/actions/update-major-tag/action.yaml:14`
- `.github/actions/update-major-tag/action.yaml:20`
- `.github/actions/update-major-tag/action.yaml:21`
- `.github/actions/update-major-tag/action.yaml:24`

### github-env-injection (severity: high)

Unsanitized ${{ inputs.* }} expressions are written directly to $GITHUB_OUTPUT without the required `printf '%s' ... | tr -d '\n\r'` sanitization. In perform-static-analysis/action.yaml: `echo "secret_exist=${{ inputs.sonar_token != '' }}" >> $GITHUB_OUTPUT`. In create-lines-of-code-report/action.yaml and scan-dependencies/action.yaml: `echo "secrets_exist=${{ inputs.idp_aws_report_upload_role_name != '' && inputs.idp_aws_report_upload_bucket_endpoint != '' }}" >> $GITHUB_OUTPUT`. Although these are boolean expressions, the ${{ }} interpolation happens before the shell runs, so a crafted input containing newlines could inject additional key=value pairs into GITHUB_OUTPUT.

Locations:

- `.github/actions/perform-static-analysis/action.yaml:14`
- `.github/actions/create-lines-of-code-report/action.yaml:37`
- `.github/actions/scan-dependencies/action.yaml:48`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection

**Notes:**

Fixed all script-injection and github-env-injection findings across 8 action files:

1. check-english-usage/action.yaml: Moved `github.event.repository.default_branch` to env var `DEFAULT_BRANCH`, referenced as `${DEFAULT_BRANCH}` in shell.

2. check-file-format/action.yaml: Same fix as above.

3. check-markdown-format/action.yaml: Same fix as above.

4. commit-release-files/action.yaml: Moved `inputs.github_token` to env var `GITHUB_TOKEN_INPUT` for the mask step; moved `inputs.commit_message` to env var `INPUT_COMMIT_MESSAGE` for the commit step.

5. create-lines-of-code-report/action.yaml: Moved `inputs.build_datetime` to env var `BUILD_DATETIME`; sanitized the boolean expression written to GITHUB_OUTPUT using `printf '%s' ... | tr -d '\n\r'`; moved `inputs.idp_aws_report_upload_bucket_endpoint` and `inputs.build_timestamp` to env vars `BUCKET_ENDPOINT` and `BUILD_TIMESTAMP`.

6. lint-terraform/action.yaml: Moved `inputs.root-modules` to env var `INPUT_ROOT_MODULES`, referenced as `${INPUT_ROOT_MODULES}` in shell.

7. perform-static-analysis/action.yaml: Sanitized the boolean expression written to GITHUB_OUTPUT; moved `inputs.sonar_organisation_key`, `inputs.sonar_project_key`, and `inputs.sonar_token` to env vars in the analysis step.

8. scan-dependencies/action.yaml: Moved `inputs.build_datetime` to env var `BUILD_DATETIME` in both SBOM and vulnerability scan steps; sanitized the boolean expression written to GITHUB_OUTPUT; moved `inputs.idp_aws_report_upload_bucket_endpoint` and `inputs.build_timestamp` to env vars.

9. update-major-tag/action.yaml: Moved `inputs.github_token` to env var `GITHUB_TOKEN_INPUT` for the mask step; moved `inputs.full_release_version`, `inputs.major_release_version`, and `github.repository` to env vars `INPUT_FULL_RELEASE_VERSION`, `INPUT_MAJOR_RELEASE_VERSION`, and `GITHUB_REPOSITORY_NAME` respectively, with all gh api URL arguments properly quoted.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed two script-injection findings:
1. hardened/action/.github/actions/lint-terraform/action.yaml (line 20): Replaced unquoted `echo ${stacks//,/$'\n'}` with `printf '%s' "${stacks}" | tr ',' '\n'`. The stacks variable is now double-quoted, preventing shell metacharacters in the `root-modules` input from being interpreted by the shell.
2. hardened/action/.github/actions/perform-static-analysis/action.yaml (line 32): Added double-quotes around the BRANCH_NAME assignment and around `$GITHUB_REF` inside the echo command: `export BRANCH_NAME="${GITHUB_HEAD_REF:-$(echo "$GITHUB_REF" | sed 's#refs/heads/##')}"`. This prevents shell metacharacters in the workflow-controlled GITHUB_HEAD_REF and GITHUB_REF environment variables from being interpreted.

