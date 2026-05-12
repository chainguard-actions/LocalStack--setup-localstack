# Hardening Report: LocalStack--setup-localstack/v0.3.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `ff50f15e4b79bfbf764dafdfd2579175a6ea9771`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **LocalStack--setup-localstack/v0.3.1** was hardened automatically. 7 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Direct interpolation of ${{ github.event.number }} in a run: block. An attacker could craft a PR event with a malicious number value to inject shell commands. The value should be passed via an env: variable instead.

Locations:

- `prepare/action.yml:16`

### script-injection (severity: high)

Direct interpolation of ${{ inputs.preview-cmd }} as the entire body of a run: block. This allows any caller to inject arbitrary shell commands. The input should be passed via an env: variable and executed as a safe string, not interpolated directly.

Locations:

- `ephemeral/startup/action.yml:148`

### script-injection (severity: high)

Direct interpolation of ${{ inputs.ci-project }} in a run: block (export CI_PROJECT=${{ inputs.ci-project }}). An attacker-controlled input value is embedded directly into the shell command string. It should be passed via an env: variable.

Locations:

- `startup/action.yml:63`

### script-injection (severity: high)

Direct interpolation of ${{ inputs.localstack-api-key }} in run: blocks. The attacker-controlled input is embedded directly into the shell command string (AUTH_HEADER assignment). It should be passed via an env: variable.

Locations:

- `ephemeral/shutdown/action.yml:33`
- `ephemeral/startup/action.yml:57`
- `ephemeral/startup/action.yml:163`

### script-injection (severity: high)

Direct interpolation of ${{ inputs.auto-load-pod }}, ${{ inputs.extension-auto-install }}, and ${{ inputs.lifetime }} in a run: block in ephemeral/startup/action.yml. These attacker-controlled inputs are embedded directly into the shell command string and should be passed via env: variables.

Locations:

- `ephemeral/startup/action.yml:73`
- `ephemeral/startup/action.yml:74`
- `ephemeral/startup/action.yml:75`

### script-injection (severity: high)

Direct interpolation of ${{ inputs.preview-url }} in a run: block in finish/action.yml. The attacker-controlled input is embedded directly into the shell command string and should be passed via an env: variable.

Locations:

- `finish/action.yml:57`

### github-env-injection (severity: high)

The run: block in finish/action.yml writes ${{ inputs.preview-url }} directly to $GITHUB_ENV without sanitization (no printf '%s' ... | tr -d '\n\r' step). An attacker could inject newlines into the input to set arbitrary environment variables for subsequent steps.

Locations:

- `finish/action.yml:58`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection

**Notes:**

Fixed all 7 findings across 5 files:

1. prepare/action.yml (line 16): Moved `${{ github.event.number }}` to env var `PR_NUMBER`.

2. startup/action.yml (line 63): Moved `${{ inputs.ci-project }}` to env var `CI_PROJECT_INPUT`.

3. ephemeral/shutdown/action.yml (line 33): Moved `${{ inputs.localstack-api-key }}` to env var `LOCALSTACK_API_KEY_INPUT`; also moved `${{ github.action_path }}` to env var `ACTION_PATH`.

4. ephemeral/startup/action.yml (lines 57, 73-75): Moved `${{ inputs.localstack-api-key }}`, `${{ inputs.auto-load-pod }}`, `${{ inputs.extension-auto-install }}`, `${{ inputs.lifetime }}`, and `${{ github.action_path }}` to env vars in the 'Create preview environment' step.

5. ephemeral/startup/action.yml (line 148): Moved `${{ inputs.preview-cmd }}` to env var `PREVIEW_CMD` and used `eval "$PREVIEW_CMD"` in the run block.

6. ephemeral/startup/action.yml (line 163): Moved `${{ inputs.localstack-api-key }}` and `${{ github.action_path }}` to env vars in the 'Print logs' step.

7. finish/action.yml (lines 57-58): Moved `${{ inputs.preview-url }}` to env var `PREVIEW_URL_INPUT` and sanitized with `printf '%s' ... | tr -d '\n\r'` before writing to `$GITHUB_ENV` to prevent newline injection.

