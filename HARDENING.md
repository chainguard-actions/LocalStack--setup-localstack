# Hardening Report: LocalStack--setup-localstack/v0.2.4

> This file was generated automatically by the hardening agent.

**Policy SHA:** `ff50f15e4b79bfbf764dafdfd2579175a6ea9771`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

Action **LocalStack--setup-localstack/v0.2.4** was hardened automatically. 3 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple action files reference external actions using mutable tags instead of pinned 40-character SHA digests, making them vulnerable to supply-chain attacks.

Failing references:
- action.yml: jenseng/dynamic-uses@v1 (×6)
- ephemeral/shutdown/action.yml: actions/download-artifact@v4, actions-cool/maintain-one-comment@v3.1.1
- ephemeral/startup/action.yml: jenseng/dynamic-uses@v1, actions/download-artifact@v4, actions/upload-artifact@v4
- finish/action.yml: actions/download-artifact@v4 (×2), dawidd6/action-download-artifact@v6 (×2), actions-cool/maintain-one-comment@v3.1.1
- local/action.yml: actions/download-artifact@v4, dawidd6/action-download-artifact@v6, actions/upload-artifact@v4
- prepare/action.yml: actions/upload-artifact@v4, actions-cool/maintain-one-comment@v3.1.1
- startup/action.yml: jenseng/dynamic-uses@v1

Locations:

- `action.yml:75`
- `action.yml:84`
- `action.yml:96`
- `action.yml:108`
- `action.yml:117`
- `action.yml:126`
- `ephemeral/shutdown/action.yml:13`
- `ephemeral/shutdown/action.yml:36`
- `ephemeral/startup/action.yml:31`
- `ephemeral/startup/action.yml:37`
- `ephemeral/startup/action.yml:80`
- `finish/action.yml:17`
- `finish/action.yml:23`
- `finish/action.yml:35`
- `finish/action.yml:41`
- `finish/action.yml:60`
- `local/action.yml:16`
- `local/action.yml:22`
- `local/action.yml:43`
- `prepare/action.yml:16`
- `prepare/action.yml:21`
- `startup/action.yml:35`
- `startup/action.yml:43`

### script-injection (severity: high)

Multiple run: blocks directly interpolate ${{ inputs.* }} or ${{ github.* }} expressions into shell commands without first routing them through environment variables. An attacker who controls these inputs can inject arbitrary shell commands.

- prepare/action.yml: `echo ${{ github.event.number }} > ./pr-id.txt` — github.event.number interpolated directly in shell.
- ephemeral/startup/action.yml: The entire `run:` value is `${{ inputs.preview-cmd }}`, allowing arbitrary command execution. Additionally, `${{ inputs.auto-load-pod }}`, `${{ inputs.extension-auto-install }}`, `${{ inputs.lifetime }}`, and `${{ inputs.localstack-api-key }}` are interpolated directly inside shell strings.
- ephemeral/shutdown/action.yml: `${{ inputs.localstack-api-key }}` interpolated directly inside a shell string in the curl command.
- startup/action.yml: `export CI_PROJECT=${{ inputs.ci-project }}` interpolates inputs directly in the run block.
- finish/action.yml: `${{ inputs.preview-url }}` interpolated directly inside a shell conditional.

Locations:

- `prepare/action.yml:10`
- `ephemeral/startup/action.yml:47`
- `ephemeral/startup/action.yml:51`
- `ephemeral/startup/action.yml:52`
- `ephemeral/startup/action.yml:53`
- `ephemeral/startup/action.yml:57`
- `ephemeral/startup/action.yml:62`
- `ephemeral/startup/action.yml:67`
- `ephemeral/startup/action.yml:75`
- `ephemeral/startup/action.yml:88`
- `ephemeral/shutdown/action.yml:26`
- `startup/action.yml:60`
- `finish/action.yml:52`
- `finish/action.yml:53`

### github-env-injection (severity: high)

finish/action.yml writes the attacker-controlled input `${{ inputs.preview-url }}` directly to $GITHUB_ENV without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`). A malicious value containing newlines could inject arbitrary environment variables into subsequent steps.

Specifically, in the 'Load the Ephemeral Instance URL' step:
  `echo "LS_PREVIEW_URL=${LS_PREVIEW_URL:-${{ inputs.preview-url }}}" >> $GITHUB_ENV`

The inputs.preview-url value is interpolated directly into the echo command that writes to $GITHUB_ENV, with no newline sanitization applied.

Locations:

- `finish/action.yml:52`
- `finish/action.yml:53`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, github-env-injection

**Notes:**

Fixed all three finding types across 7 action files:

1. unpinned-uses: Pinned all 23 unpinned action references to full 40-char SHAs:
   - jenseng/dynamic-uses@v1 → @8bc24f0360175e710da532c4d19eafdbed489a06 (action.yml ×6, ephemeral/startup/action.yml ×1, startup/action.yml ×1)
   - actions/download-artifact@v4 → @d3f86a106a0bac45b974a628896c90dbdf5c8093 (ephemeral/shutdown, ephemeral/startup, finish ×2, local)
   - actions-cool/maintain-one-comment@v3.1.1 → @b36a5b24a5293cc3bed7a8d4c0a74442cda865aa (ephemeral/shutdown, finish, prepare)
   - actions/upload-artifact@v4 → @ea165f8d65b6e75b540449e92b4886f43607fa02 (ephemeral/startup, local, prepare)
   - dawidd6/action-download-artifact@v6 → @bf251b5aa9c2f7eeb574a96ee720e24f801b7c11 (finish ×2, local)

2. script-injection: Moved all ${{ inputs.* }} and ${{ github.* }} expressions from run: blocks to env: blocks:
   - prepare/action.yml: github.event.number → PR_NUMBER env var
   - ephemeral/startup/action.yml: auto-load-pod/extension-auto-install/lifetime/localstack-api-key → env vars; preview-cmd → PREVIEW_CMD env var with eval
   - ephemeral/shutdown/action.yml: localstack-api-key → LOCALSTACK_API_KEY_INPUT env var
   - startup/action.yml: ci-project → INPUT_CI_PROJECT env var
   - finish/action.yml: preview-url → INPUT_PREVIEW_URL env var

3. github-env-injection: finish/action.yml now sanitizes preview URL with printf '%s' ... | tr -d '\n\r' before writing to $GITHUB_ENV, preventing newline injection.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed two script injection vulnerabilities:

1. ephemeral/startup/action.yml (line 116): Replaced `eval "$PREVIEW_CMD"` with writing the command to `/tmp/preview_cmd.sh` and executing it with `bash /tmp/preview_cmd.sh`. This eliminates the `eval` double-evaluation that allowed arbitrary shell code injection through the `preview-cmd` input.

2. startup/action.yml (line 68): Replaced `eval "${CONFIGURATION} localstack start -d"` with a safe alternative that parses the CONFIGURATION string into KEY=VALUE pairs using `grep -E '^[A-Za-z_][A-Za-z0-9_]*='` (validating the format), writes them to `/tmp/ls_config.env`, and passes them to the `env` command to run `localstack start -d`. This prevents arbitrary shell code injection through the `configuration` input while preserving the intended functionality of setting environment variables for the localstack process.

