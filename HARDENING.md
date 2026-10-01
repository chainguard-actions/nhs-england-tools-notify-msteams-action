<!-- markdownlint-disable -->

# Hardening Report: nhs-england-tools--notify-msteams-action/v1.1.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **nhs-england-tools--notify-msteams-action/v1.1.0** was hardened automatically. 2 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a): ${{ }} expressions are interpolated directly inside run: shell command strings, allowing an attacker who controls the input values to inject arbitrary shell commands. Affected steps and offending lines:

• check-english-usage/action.yaml line 9: `export BRANCH_NAME=origin/${{ github.event.repository.default_branch }}`
• check-file-format/action.yaml line 9: `export BRANCH_NAME=origin/${{ github.event.repository.default_branch }}`
• check-markdown-format/action.yaml line 9: `export BRANCH_NAME=origin/${{ github.event.repository.default_branch }}`
• commit-release-files/action.yaml line 15: `echo "::add-mask::${{ inputs.github_token }}"`
• commit-release-files/action.yaml line 20: `commit_message="${{ inputs.commit_message }}"`
• commit-release-files/action.yaml lines 57-58: `gh api repos/${{ github.repository }}/...`
• create-lines-of-code-report/action.yaml line 25: `export BUILD_DATETIME=${{ inputs.build_datetime }}`
• create-lines-of-code-report/action.yaml line 38: `echo "secrets_exist=${{ inputs.idp_aws_report_upload_role_name != '' && ... }}" >> $GITHUB_OUTPUT`
• create-lines-of-code-report/action.yaml lines 52-53: `${{ inputs.idp_aws_report_upload_bucket_endpoint }}/${{ inputs.build_timestamp }}`
• perform-static-analysis/action.yaml line 17: `echo "secret_exist=${{ inputs.sonar_token != '' }}" >> $GITHUB_OUTPUT`
• perform-static-analysis/action.yaml lines 22-24: `export SONAR_ORGANISATION_KEY=${{ inputs.sonar_organisation_key }}` etc.
• scan-dependencies/action.yaml line 27: `export BUILD_DATETIME=${{ inputs.build_datetime }}`
• scan-dependencies/action.yaml line 41: `export BUILD_DATETIME=${{ inputs.build_datetime }}`
• scan-dependencies/action.yaml line 55: `echo "secrets_exist=${{ ... }}" >> $GITHUB_OUTPUT`
• scan-dependencies/action.yaml lines 67-70: `${{ inputs.idp_aws_report_upload_bucket_endpoint }}/${{ inputs.build_timestamp }}`
• update-major-tag/action.yaml line 15: `echo "::add-mask::${{ inputs.github_token }}"`
• update-major-tag/action.yaml lines 20-21: `${{ inputs.full_release_version }}`, `${{ inputs.major_release_version }}`
• update-major-tag/action.yaml lines 24-26: `gh api repos/${{ github.repository }}/...`

Fix: move all ${{ }} values into env: variables and reference them as quoted shell variables (e.g. "$VAR").

Locations:

- `.github/actions/check-english-usage/action.yaml:9`
- `.github/actions/check-file-format/action.yaml:9`
- `.github/actions/check-markdown-format/action.yaml:9`
- `.github/actions/commit-release-files/action.yaml:15`
- `.github/actions/commit-release-files/action.yaml:20`
- `.github/actions/commit-release-files/action.yaml:57`
- `.github/actions/create-lines-of-code-report/action.yaml:25`
- `.github/actions/create-lines-of-code-report/action.yaml:38`
- `.github/actions/create-lines-of-code-report/action.yaml:52`
- `.github/actions/perform-static-analysis/action.yaml:17`
- `.github/actions/perform-static-analysis/action.yaml:22`
- `.github/actions/scan-dependencies/action.yaml:27`
- `.github/actions/scan-dependencies/action.yaml:41`
- `.github/actions/scan-dependencies/action.yaml:55`
- `.github/actions/scan-dependencies/action.yaml:67`
- `.github/actions/update-major-tag/action.yaml:15`
- `.github/actions/update-major-tag/action.yaml:20`
- `.github/actions/update-major-tag/action.yaml:24`

### github-env-injection (severity: high)

Untrusted ${{ inputs.* }} expressions are written directly to $GITHUB_OUTPUT without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`). An attacker-controlled newline in the input value can inject additional key=value pairs into the GitHub output context.

• perform-static-analysis/action.yaml line 17: `echo "secret_exist=${{ inputs.sonar_token != '' }}" >> $GITHUB_OUTPUT` — the expression result is written unsanitized.
• create-lines-of-code-report/action.yaml line 38: `echo "secrets_exist=${{ inputs.idp_aws_report_upload_role_name != '' && inputs.idp_aws_report_upload_bucket_endpoint != '' }}" >> $GITHUB_OUTPUT` — same pattern.
• scan-dependencies/action.yaml line 55: `echo "secrets_exist=${{ inputs.idp_aws_report_upload_role_name != '' && inputs.idp_aws_report_upload_bucket_endpoint != '' }}" >> $GITHUB_OUTPUT` — same pattern.

Fix: sanitize values before writing: `safe=$(printf '%s' "$VALUE" | tr -d '\n\r'); echo "key=$safe" >> "$GITHUB_OUTPUT"`.

Locations:

- `.github/actions/perform-static-analysis/action.yaml:17`
- `.github/actions/create-lines-of-code-report/action.yaml:38`
- `.github/actions/scan-dependencies/action.yaml:55`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection

**Notes:**

Fixed all script-injection and github-env-injection findings across 7 action files:

1. check-english-usage/action.yaml: Moved `github.event.repository.default_branch` to env var DEFAULT_BRANCH.
2. check-file-format/action.yaml: Same fix as above.
3. check-markdown-format/action.yaml: Same fix as above.
4. commit-release-files/action.yaml: Moved `inputs.github_token` to env var GITHUB_TOKEN_INPUT in mask step; moved `inputs.commit_message` to env var INPUT_COMMIT_MESSAGE in commit step.
5. create-lines-of-code-report/action.yaml: Moved `inputs.build_datetime` to INPUT_BUILD_DATETIME; moved secrets_exist boolean expression to SECRETS_EXIST env var with sanitization (printf | tr -d '\n\r') before writing to GITHUB_OUTPUT; moved bucket endpoint and timestamp to env vars.
6. perform-static-analysis/action.yaml: Moved secret_exist boolean expression to SECRET_EXIST env var with sanitization before writing to GITHUB_OUTPUT; moved sonar_organisation_key, sonar_project_key, sonar_token to env vars.
7. scan-dependencies/action.yaml: Moved build_datetime to INPUT_BUILD_DATETIME in both SBOM and vulnerability steps; moved secrets_exist boolean to SECRETS_EXIST env var with sanitization; moved bucket endpoint and timestamp to env vars.
8. update-major-tag/action.yaml: Moved github_token to GITHUB_TOKEN_INPUT in mask step; moved full_release_version, major_release_version, and github.repository to env vars in update step.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed 4 script-injection vulnerabilities across 4 files:

1. `.github/actions/check-english-usage/action.yaml` (line 10): Changed `export BRANCH_NAME=origin/$DEFAULT_BRANCH` to `export BRANCH_NAME="origin/$DEFAULT_BRANCH"` — quoting prevents word-splitting and glob expansion on the attacker-controlled `DEFAULT_BRANCH` env var.

2. `.github/actions/check-file-format/action.yaml` (line 10): Same fix as above.

3. `.github/actions/check-markdown-format/action.yaml` (line 10): Same fix as above.

4. `.github/actions/perform-static-analysis/action.yaml` (line 31): Changed `echo $GITHUB_REF` to `echo "$GITHUB_REF"` inside the command substitution — quoting prevents word-splitting and glob expansion on the attacker-controlled `GITHUB_REF` environment variable, which could otherwise allow shell metacharacter injection via a crafted ref value.

