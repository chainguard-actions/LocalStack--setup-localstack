# Hardening Report: LocalStack--setup-localstack/v0.3.2

> This file was generated automatically by the hardening agent.

**Policy SHA:** `ff50f15e4b79bfbf764dafdfd2579175a6ea9771`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **LocalStack--setup-localstack/v0.3.2** was hardened automatically. 6 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

In prepare/action.yml, the 'Save PR number' step directly interpolates `${{ github.event.number }}` inside a `run:` shell command (`echo ${{ github.event.number }} > ./pr-id.txt`). The github context value is not first assigned to an environment variable, allowing an attacker to inject shell metacharacters via a crafted event payload.

Locations:

- `prepare/action.yml:16`

### script-injection (severity: high)

In startup/action.yml, the 'Start LocalStack' step directly interpolates `${{ inputs.ci-project }}` inside a `run:` shell command (`export CI_PROJECT=${{ inputs.ci-project }}`). The inputs value is not first assigned to an environment variable, allowing an attacker to inject arbitrary shell commands via the ci-project input.

Locations:

- `startup/action.yml:80`

### script-injection (severity: high)

In ephemeral/shutdown/action.yml, the 'Shutdown ephemeral instance' step directly interpolates `${{ inputs.localstack-api-key }}` and `${{ github.action_path }}` inside a `run:` shell command block. These expressions are embedded directly in shell strings without being routed through env vars, enabling shell injection.

Locations:

- `ephemeral/shutdown/action.yml:38`

### script-injection (severity: high)

In ephemeral/startup/action.yml, multiple run: blocks directly interpolate attacker-controlled expressions: (1) the 'Create preview environment' step embeds `${{ inputs.localstack-api-key }}`, `${{ github.action_path }}`, `${{ inputs.auto-load-pod }}`, `${{ inputs.extension-auto-install }}`, and `${{ inputs.lifetime }}` directly in shell commands; (2) the 'Run preview deployment' step executes `${{ inputs.preview-cmd }}` directly as a shell command, allowing arbitrary command execution by an attacker who controls the preview-cmd input.

Locations:

- `ephemeral/startup/action.yml:57`
- `ephemeral/startup/action.yml:155`

### script-injection (severity: high)

In finish/action.yml, the 'Load the Ephemeral Instance URL' step directly interpolates `${{ inputs.preview-url }}` inside a `run:` shell command block (used in a bash conditional and written to $GITHUB_ENV). The inputs value is not first assigned to an environment variable, enabling shell injection.

Locations:

- `finish/action.yml:65`

### github-env-injection (severity: high)

In finish/action.yml, the 'Load the Ephemeral Instance URL' step writes `${{ inputs.preview-url }}` directly to `$GITHUB_ENV` without sanitization (`echo "LS_PREVIEW_URL=${LS_PREVIEW_URL:-${{ inputs.preview-url }}}" >> $GITHUB_ENV`). An attacker can inject newlines into the preview-url input to set arbitrary environment variables for subsequent steps, enabling environment variable injection attacks.

Locations:

- `finish/action.yml:65`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection

**Notes:**

Fixed all 6 findings across 5 files:

1. prepare/action.yml (line 16): Moved `${{ github.event.number }}` to env var `PR_NUMBER` and referenced it as `"$PR_NUMBER"` in the run block.

2. startup/action.yml (line 80): Moved `${{ inputs.ci-project }}` to env var `CI_PROJECT_INPUT` and referenced it as `"${CI_PROJECT_INPUT}"` in the run block.

3. ephemeral/shutdown/action.yml (line 38): Moved `${{ inputs.localstack-api-key }}` to `LOCALSTACK_API_KEY_INPUT` and `${{ github.action_path }}` to `ACTION_PATH` env vars; updated shell references accordingly.

4. ephemeral/startup/action.yml (lines 57 and 155):
   - 'Create preview environment' step: Moved `inputs.localstack-api-key`, `github.action_path`, `inputs.auto-load-pod`, `inputs.extension-auto-install`, and `inputs.lifetime` to env vars (`LOCALSTACK_API_KEY_INPUT`, `ACTION_PATH`, `AUTO_LOAD_POD_INPUT`, `EXTENSION_AUTO_INSTALL_INPUT`, `LIFETIME_INPUT`).
   - 'Run preview deployment' step: Moved `inputs.preview-cmd` to env var `PREVIEW_CMD` and used `eval "$PREVIEW_CMD"` instead of direct interpolation.
   - 'Print logs of ephemeral instance' step: Also fixed `inputs.localstack-api-key` and `github.action_path` injections.

5. finish/action.yml (line 65): Fixed both script-injection and github-env-injection by moving `inputs.preview-url` to env var `PREVIEW_URL_INPUT`, resolving the URL into a local variable, and sanitizing with `printf '%s' ... | tr -d '\n\r'` before writing to `$GITHUB_ENV`.

