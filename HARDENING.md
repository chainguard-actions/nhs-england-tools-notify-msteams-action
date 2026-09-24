<!-- markdownlint-disable -->

# Hardening Report: nhs-england-tools--notify-msteams-action/v1.1.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **nhs-england-tools--notify-msteams-action/v1.1.0** was hardened automatically. 5 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a): Direct ${{ github.event.repository.default_branch }} expression interpolated inside a run: shell command. The github context value is substituted into the shell script before execution, enabling script injection if the value contains shell metacharacters.

Locations:

- `.github/actions/check-english-usage/action.yaml:9`
- `.github/actions/check-file-format/action.yaml:9`
- `.github/actions/check-markdown-format/action.yaml:9`

### script-injection (severity: high)

Rule (a): Direct ${{ inputs.* }} and ${{ github.event.pull_request.title }} expressions interpolated inside run: shell commands. In check-pr-title/action.yaml line 8, the env: block sets PR_TITLE from ${{ github.event.pull_request.title }} — this is attacker-controlled (PR title) and is a direct expression in the env: value within the step. More critically, in commit-release-files/action.yaml: line 15 uses ${{ inputs.github_token }} directly in run:, and line 22 uses ${{ inputs.commit_message }} directly in a shell variable assignment (commit_message="${{ inputs.commit_message }}"). In update-major-tag/action.yaml: line 16 uses ${{ inputs.github_token }} in run:; lines 22-23 use ${{ inputs.full_release_version }} and ${{ inputs.major_release_version }} directly in run:; line 26 uses ${{ github.repository }} directly in a gh api URL in run:. All of these allow shell metacharacter injection.

Locations:

- `.github/actions/check-pr-title/action.yaml:8`
- `.github/actions/commit-release-files/action.yaml:15`
- `.github/actions/commit-release-files/action.yaml:22`
- `.github/actions/update-major-tag/action.yaml:16`
- `.github/actions/update-major-tag/action.yaml:22`
- `.github/actions/update-major-tag/action.yaml:23`
- `.github/actions/update-major-tag/action.yaml:26`

### script-injection (severity: high)

Rule (a): Direct ${{ inputs.* }} expressions interpolated inside run: shell commands in perform-static-analysis/action.yaml. Line 17: echo "secret_exist=${{ inputs.sonar_token != '' }}" >> $GITHUB_OUTPUT — inputs.sonar_token is interpolated directly. Lines 22-25: export SONAR_ORGANISATION_KEY=${{ inputs.sonar_organisation_key }}, export SONAR_PROJECT_KEY=${{ inputs.sonar_project_key }}, export SONAR_TOKEN=${{ inputs.sonar_token }} — all inputs are interpolated directly into shell export statements without quoting or env: indirection.

Locations:

- `.github/actions/perform-static-analysis/action.yaml:17`
- `.github/actions/perform-static-analysis/action.yaml:22`
- `.github/actions/perform-static-analysis/action.yaml:23`
- `.github/actions/perform-static-analysis/action.yaml:24`

### script-injection (severity: high)

Rule (a): Direct ${{ inputs.* }} expressions interpolated inside run: shell commands in create-lines-of-code-report/action.yaml. Line 26: export BUILD_DATETIME=${{ inputs.build_datetime }} — input directly in shell export. Line 38: echo "secrets_exist=${{ inputs.idp_aws_report_upload_role_name != '' && inputs.idp_aws_report_upload_bucket_endpoint != '' }}" >> $GITHUB_OUTPUT — inputs interpolated directly. Lines 47-48: ${{ inputs.idp_aws_report_upload_bucket_endpoint }}/${{ inputs.build_timestamp }}-lines-of-code-report.json.zip — inputs interpolated directly into an aws s3 cp command.

Locations:

- `.github/actions/create-lines-of-code-report/action.yaml:26`
- `.github/actions/create-lines-of-code-report/action.yaml:38`
- `.github/actions/create-lines-of-code-report/action.yaml:47`

### script-injection (severity: high)

Rule (a): Direct ${{ inputs.* }} expressions interpolated inside run: shell commands in scan-dependencies/action.yaml. Line 27: export BUILD_DATETIME=${{ inputs.build_datetime }}. Line 41: export BUILD_DATETIME=${{ inputs.build_datetime }} (second occurrence in 'Scan vulnerabilities' step). Line 55: echo "secrets_exist=${{ inputs.idp_aws_report_upload_role_name != '' && inputs.idp_aws_report_upload_bucket_endpoint != '' }}" >> $GITHUB_OUTPUT. Lines 63-64 and 66-67: ${{ inputs.idp_aws_report_upload_bucket_endpoint }}/${{ inputs.build_timestamp }}-*.json.zip — inputs interpolated directly into aws s3 cp commands.

Locations:

- `.github/actions/scan-dependencies/action.yaml:27`
- `.github/actions/scan-dependencies/action.yaml:41`
- `.github/actions/scan-dependencies/action.yaml:55`
- `.github/actions/scan-dependencies/action.yaml:63`
- `.github/actions/scan-dependencies/action.yaml:66`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection

**Notes:**

Fixed script injection in 8 action files by moving all ${{ }} expressions from run: shell commands into env: blocks:

1. check-english-usage/action.yaml: Moved `${{ github.event.repository.default_branch }}` to env var `DEFAULT_BRANCH`.

2. check-file-format/action.yaml: Same fix as above.

3. check-markdown-format/action.yaml: Same fix as above.

4. check-pr-title/action.yaml: Already correctly used env block for PR_TITLE — no change needed.

5. commit-release-files/action.yaml: Moved `${{ inputs.github_token }}` in the mask step to env var `INPUT_GITHUB_TOKEN`; moved `${{ inputs.commit_message }}` to env var `INPUT_COMMIT_MESSAGE`.

6. update-major-tag/action.yaml: Moved `${{ inputs.github_token }}` in mask step to env var; moved `${{ inputs.full_release_version }}`, `${{ inputs.major_release_version }}`, and `${{ github.repository }}` to env vars `INPUT_FULL_RELEASE_VERSION`, `INPUT_MAJOR_RELEASE_VERSION`, `INPUT_GITHUB_REPOSITORY`.

7. perform-static-analysis/action.yaml: Replaced inline `${{ inputs.sonar_token != '' }}` expression with env var `INPUT_SONAR_TOKEN` and shell conditional; moved all three sonar inputs to env vars in the perform step.

8. create-lines-of-code-report/action.yaml: Moved `${{ inputs.build_datetime }}` to env var; replaced inline boolean expression with shell conditional using env vars; moved bucket endpoint and timestamp to env vars in the send step.

9. scan-dependencies/action.yaml: Same pattern applied to both Generate SBOM and Scan vulnerabilities steps (build_datetime); replaced inline boolean expression with shell conditional; moved bucket endpoint and timestamp to env vars in the send step.

