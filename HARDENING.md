# Hardening Report: LocalStack--setup-localstack/v0.3.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `ff50f15e4b79bfbf764dafdfd2579175a6ea9771`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

Action **LocalStack--setup-localstack/v0.3.0** was hardened automatically. 5 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

prepare/action.yml: The 'Save PR number' step directly interpolates ${{ github.event.number }} inside a run: shell command (`echo ${{ github.event.number }} > ./pr-id.txt`). An attacker can supply a crafted PR number to inject arbitrary shell commands. The value must be assigned to an env: variable and referenced as $VAR instead.

Locations:

- `prepare/action.yml:15`

### script-injection (severity: high)

ephemeral/shutdown/action.yml: The 'Shutdown ephemeral instance' step directly interpolates ${{ inputs.localstack-api-key }} inside a run: shell command to build AUTH_HEADER (`AUTH_HEADER="ls-api-key: ${LOCALSTACK_AUTH_TOKEN:-${LOCALSTACK_API_KEY:-${{ inputs.localstack-api-key }}}}"`). An attacker-controlled input value can break out of the string and inject arbitrary shell commands. The value must be passed via env: and referenced as $VAR.

Locations:

- `ephemeral/shutdown/action.yml:36`

### script-injection (severity: high)

ephemeral/startup/action.yml: Multiple run: blocks directly interpolate attacker-controlled inputs expressions: (1) 'Create preview environment' step uses ${{ inputs.localstack-api-key }} in AUTH_HEADER construction, ${{ inputs.auto-load-pod }}, ${{ inputs.extension-auto-install }}, and ${{ inputs.lifetime }} as shell variable assignments — all without env: indirection. (2) 'Run preview deployment' step executes ${{ inputs.preview-cmd }} directly as a shell command, allowing arbitrary command execution. (3) 'Print logs of ephemeral instance' step also uses ${{ inputs.localstack-api-key }} in AUTH_HEADER. All these inputs must be passed via env: variables.

Locations:

- `ephemeral/startup/action.yml:60`
- `ephemeral/startup/action.yml:88`
- `ephemeral/startup/action.yml:89`
- `ephemeral/startup/action.yml:90`
- `ephemeral/startup/action.yml:152`
- `ephemeral/startup/action.yml:163`

### script-injection (severity: high)

startup/action.yml: The 'Start LocalStack' run: block directly interpolates ${{ inputs.ci-project }} into a shell variable assignment (`export CI_PROJECT=${{ inputs.ci-project }}`). This value is attacker-controlled and can inject arbitrary shell commands. It must be passed via env: and referenced as $CI_PROJECT.

Locations:

- `startup/action.yml:75`

### github-env-injection (severity: high)

finish/action.yml: The 'Load the Ephemeral Instance URL' run: block writes ${{ inputs.preview-url }} directly to $GITHUB_ENV without sanitization (`echo "LS_PREVIEW_URL=${LS_PREVIEW_URL:-${{ inputs.preview-url }}}" >> $GITHUB_ENV`). An attacker can inject newlines into this input to set arbitrary environment variables for subsequent steps. The value must be sanitized with `printf '%s' "$VAR" | tr -d '\n\r'` before writing to $GITHUB_ENV.

Locations:

- `finish/action.yml:73`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection

**Notes:**

Fixed all 5 security findings across 5 files:

1. prepare/action.yml: Moved `${{ github.event.number }}` to env var `PR_NUMBER` and referenced as `"$PR_NUMBER"` in the run block.

2. ephemeral/shutdown/action.yml: Moved `${{ inputs.localstack-api-key }}` to `LOCALSTACK_API_KEY_INPUT` and `${{ github.action_path }}` to `ACTION_PATH` env vars. Updated AUTH_HEADER and source command to use env vars.

3. ephemeral/startup/action.yml: Three steps fixed:
   - 'Create preview environment': Moved localstack-api-key, auto-load-pod, extension-auto-install, lifetime, and action_path to env vars.
   - 'Run preview deployment': Moved preview-cmd to `PREVIEW_CMD` env var, used `eval "$PREVIEW_CMD"` instead of direct interpolation.
   - 'Print logs of ephemeral instance': Moved localstack-api-key and action_path to env vars.

4. startup/action.yml: Moved `${{ inputs.ci-project }}` to `CI_PROJECT_INPUT` env var and referenced as `"${CI_PROJECT_INPUT}"` in the shell script.

5. finish/action.yml: Moved `${{ inputs.preview-url }}` to `PREVIEW_URL_INPUT` env var, sanitized with `printf '%s' "..." | tr -d '\n\r'` before writing to `$GITHUB_ENV`.

