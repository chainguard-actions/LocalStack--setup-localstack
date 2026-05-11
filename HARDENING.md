# Hardening Report: LocalStack--setup-localstack/v0.3.2

> This file was generated automatically by the hardening agent.

**Policy SHA:** `ff50f15e4b79bfbf764dafdfd2579175a6ea9771`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **LocalStack--setup-localstack/v0.3.2** was hardened automatically. 8 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

prepare/action.yml: The 'Save PR number' run: step directly interpolates ${{ github.event.number }} into the shell command without first assigning it to an env: variable. An attacker controlling the event number could inject shell metacharacters.

Locations:

- `prepare/action.yml:13`

### script-injection (severity: high)

ephemeral/startup/action.yml: The 'Create preview environment' run: step directly interpolates ${{ inputs.localstack-api-key }}, ${{ inputs.auto-load-pod }}, ${{ inputs.extension-auto-install }}, and ${{ inputs.lifetime }} into shell commands without routing through env: variables. These inputs are attacker-controlled and can inject arbitrary shell code.

Locations:

- `ephemeral/startup/action.yml:56`
- `ephemeral/startup/action.yml:72`
- `ephemeral/startup/action.yml:73`
- `ephemeral/startup/action.yml:74`

### script-injection (severity: high)

ephemeral/startup/action.yml: The 'Run preview deployment' run: step directly executes ${{ inputs.preview-cmd }} as a shell command. This is a critical script injection: an attacker can supply arbitrary shell commands via the preview-cmd input, which are executed verbatim on the runner.

Locations:

- `ephemeral/startup/action.yml:143`

### script-injection (severity: high)

ephemeral/startup/action.yml: The 'Print logs of ephemeral instance' run: step directly interpolates ${{ inputs.localstack-api-key }} into the AUTH_HEADER shell variable without routing through env:. An attacker-controlled input value is embedded directly in the shell command string.

Locations:

- `ephemeral/startup/action.yml:152`

### script-injection (severity: high)

ephemeral/shutdown/action.yml: The 'Shutdown ephemeral instance' run: step directly interpolates ${{ inputs.localstack-api-key }} into the AUTH_HEADER shell variable without routing through env:. An attacker-controlled input value is embedded directly in the shell command string.

Locations:

- `ephemeral/shutdown/action.yml:30`

### script-injection (severity: high)

startup/action.yml: The 'Start LocalStack' run: step directly interpolates ${{ inputs.ci-project }} into the shell command (export CI_PROJECT=${{ inputs.ci-project }}) without routing through env:. An attacker-controlled input is embedded directly in the shell command string.

Locations:

- `startup/action.yml:57`

### script-injection (severity: high)

finish/action.yml: The 'Load the Ephemeral Instance URL' run: step directly interpolates ${{ inputs.preview-url }} into shell command strings without routing through env:. An attacker-controlled input is embedded directly in the shell command string.

Locations:

- `finish/action.yml:55`
- `finish/action.yml:56`

### github-env-injection (severity: high)

finish/action.yml: The 'Load the Ephemeral Instance URL' run: step writes the attacker-controlled value ${{ inputs.preview-url }} directly to $GITHUB_ENV (echo "LS_PREVIEW_URL=${LS_PREVIEW_URL:-${{ inputs.preview-url }}}" >> $GITHUB_ENV) without applying the required sanitization pipeline (printf '%s' ... | tr -d '\n\r'). A malicious newline in the input can inject arbitrary environment variables into subsequent steps.

Locations:

- `finish/action.yml:56`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection

**Notes:**

Fixed all 8 findings across 5 files:

1. **prepare/action.yml** (line 13): Moved `${{ github.event.number }}` to `env: PR_NUMBER` and referenced as `$PR_NUMBER` in the run block.

2. **ephemeral/startup/action.yml** (lines 56, 72-74): In 'Create preview environment', moved `${{ inputs.localstack-api-key }}`, `${{ inputs.auto-load-pod }}`, `${{ inputs.extension-auto-install }}`, `${{ inputs.lifetime }}`, and `${{ github.action_path }}` to env: block as `LOCALSTACK_API_KEY_INPUT`, `AUTO_LOAD_POD_INPUT`, `EXTENSION_AUTO_INSTALL_INPUT`, `LIFETIME_INPUT`, `ACTION_PATH`.

3. **ephemeral/startup/action.yml** (line 143): In 'Run preview deployment', moved `${{ inputs.preview-cmd }}` to `env: PREVIEW_CMD` and executed via `eval "$PREVIEW_CMD"` instead of direct interpolation.

4. **ephemeral/startup/action.yml** (line 152): In 'Print logs of ephemeral instance', moved `${{ inputs.localstack-api-key }}` and `${{ github.action_path }}` to env: block.

5. **ephemeral/shutdown/action.yml** (line 30): In 'Shutdown ephemeral instance', moved `${{ inputs.localstack-api-key }}` and `${{ github.action_path }}` to env: block.

6. **startup/action.yml** (line 57): In 'Start LocalStack', moved `${{ inputs.ci-project }}` to `env: CI_PROJECT_INPUT` and referenced as `$CI_PROJECT_INPUT`.

7. **finish/action.yml** (lines 55-56): In 'Load the Ephemeral Instance URL', moved `${{ inputs.preview-url }}` to `env: PREVIEW_URL_INPUT` and applied `printf '%s' | tr -d '\n\r'` sanitization before writing to `$GITHUB_ENV`, fixing both the script-injection and github-env-injection findings.

