# Hardening Report: LocalStack--setup-localstack/v0.2.5

> This file was generated automatically by the hardening agent.

**Policy SHA:** `ff50f15e4b79bfbf764dafdfd2579175a6ea9771`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **LocalStack--setup-localstack/v0.2.5** was hardened automatically. 7 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

prepare/action.yml directly interpolates `${{ github.event.number }}` inside a `run:` shell command (`echo ${{ github.event.number }} > ./pr-id.txt`). The github context value is expanded by the Actions template engine before the shell sees it, allowing an attacker to inject arbitrary shell commands via a crafted event payload. The value must be assigned to an env var and referenced as `$VAR` instead.

Locations:

- `prepare/action.yml:15`

### script-injection (severity: high)

ephemeral/startup/action.yml directly interpolates multiple `inputs.*` expressions inside `run:` shell commands without routing through env vars: (1) `${{ inputs.preview-cmd }}` is executed verbatim as a shell command — an attacker can supply arbitrary shell code; (2) `inputs.localstack-api-key`, `inputs.auto-load-pod`, `inputs.extension-auto-install`, and `inputs.lifetime` are all interpolated directly into shell variable assignments inside the run block. All of these must be passed via `env:` and referenced as shell variables.

Locations:

- `ephemeral/startup/action.yml:130`
- `ephemeral/startup/action.yml:75`
- `ephemeral/startup/action.yml:76`
- `ephemeral/startup/action.yml:77`
- `ephemeral/startup/action.yml:78`

### script-injection (severity: high)

ephemeral/shutdown/action.yml directly interpolates `${{ inputs.localstack-api-key }}` inside a `run:` shell command, embedding it into the AUTH_HEADER variable assignment. An attacker-controlled input value is expanded by the Actions template engine before the shell sees it, enabling shell injection. The value must be passed via `env:` and referenced as a shell variable.

Locations:

- `ephemeral/shutdown/action.yml:36`

### script-injection (severity: high)

startup/action.yml directly interpolates `${{ inputs.ci-project }}` inside a `run:` shell command (`export CI_PROJECT=${{ inputs.ci-project }}`). The Actions template engine expands this before the shell sees it, allowing an attacker to inject arbitrary shell code. Additionally, `eval "${CONFIGURATION} localstack start -d"` is called where CONFIGURATION is derived from `inputs.configuration` via env var — while the env var indirection is used, the eval of an attacker-influenced variable is still dangerous. The ci-project value must be passed via `env:` and referenced as a shell variable.

Locations:

- `startup/action.yml:62`

### script-injection (severity: high)

finish/action.yml directly interpolates `${{ inputs.preview-url }}` inside a `run:` shell command (used in a conditional expression and in an echo statement writing to $GITHUB_ENV). The Actions template engine expands this before the shell sees it, allowing an attacker to inject arbitrary shell code. The value must be passed via `env:` and referenced as a shell variable.

Locations:

- `finish/action.yml:57`
- `finish/action.yml:58`

### github-env-injection (severity: high)

finish/action.yml writes `${{ inputs.preview-url }}` directly to `$GITHUB_ENV` without sanitization. The run block does `echo "LS_PREVIEW_URL=${LS_PREVIEW_URL:-${{ inputs.preview-url }}}" >> $GITHUB_ENV`. An attacker can supply a value containing newlines to inject arbitrary environment variables (e.g. `ACTIONS_RUNTIME_TOKEN=...`). The required sanitization step `printf '%s' "$VAR" | tr -d '\n\r'` is absent before the write.

Locations:

- `finish/action.yml:58`

### suspicious-run-content (severity: high)

eval-dynamic: startup/action.yml uses `eval "${CONFIGURATION} localstack start -d"` where the CONFIGURATION shell variable is set from `inputs.configuration` via the step's `env:` block. While routing through an env var avoids direct template injection, `eval` with an attacker-controlled variable allows arbitrary command execution — an attacker can supply a value like `; malicious-command #` as the configuration input to execute arbitrary code on the runner.

Locations:

- `startup/action.yml:63`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection, suspicious-run-content

**Notes:**

Fixed all 7 findings across 5 files:

1. prepare/action.yml: Moved `${{ github.event.number }}` to env var `PR_NUMBER` in the env: block.

2. ephemeral/startup/action.yml: (a) Moved `inputs.localstack-api-key`, `inputs.auto-load-pod`, `inputs.extension-auto-install`, `inputs.lifetime`, and `github.action_path` to env vars in the Create preview environment step. (b) Moved `inputs.preview-cmd` to env var `PREVIEW_CMD` and used `eval "$PREVIEW_CMD"` in the Run preview deployment step. (c) Fixed the Print logs step similarly.

3. ephemeral/shutdown/action.yml: Moved `inputs.localstack-api-key` and `github.action_path` to env vars `INPUT_LOCALSTACK_API_KEY` and `ACTION_PATH`.

4. startup/action.yml: (a) Moved `inputs.ci-project` to env var `INPUT_CI_PROJECT`. (b) Replaced `eval "${CONFIGURATION} localstack start -d"` with `env $(echo "$CONFIGURATION" | xargs) localstack start -d` to eliminate eval of attacker-controlled input.

5. finish/action.yml: Moved `inputs.preview-url` to env var `INPUT_PREVIEW_URL`, used shell variable references, and added `printf '%s' "$var" | tr -d '\n\r'` sanitization before writing to $GITHUB_ENV to prevent newline injection.

