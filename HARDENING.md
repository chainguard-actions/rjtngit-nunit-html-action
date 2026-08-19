<!-- markdownlint-disable -->

# Hardening Report: rjtngit--nunit-html-action/v1.0.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **rjtngit--nunit-html-action/v1.0.1** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): The first `run:` step interpolates `${{ github.action_path }}` directly into the shell command string: `sudo python -m pip install -r ${{ github.action_path }}/requirements.txt`. Any `${{ ... }}` expression inside a `run:` block is expanded by the Actions template engine before the shell ever sees it, allowing an attacker who can influence the context value to inject arbitrary shell commands. Even though `github.action_path` is typically GitHub-controlled, the pattern is still a script-injection violation per the check rules — no `${{ ... }}` expression should appear directly inside a `run:` script.

Locations:

- `action.yml:11`

### script-injection (severity: high)

Sub-rule (a): The second `run:` step interpolates three `${{ ... }}` expressions directly into the shell command string: `sudo python ${{ github.action_path }}/main.py ${{ inputs.inputXmlPath }} ${{ inputs.outputHtmlPath }}`. Both `${{ inputs.inputXmlPath }}` and `${{ inputs.outputHtmlPath }}` are attacker-controlled inputs that are passed unquoted and unsanitised directly into the shell, enabling arbitrary command injection (e.g. a value like `foo; malicious-command`). These must be moved to an `env:` block and referenced as double-quoted shell variables (e.g. `"$INPUT_XML_PATH"`) instead.

Locations:

- `action.yml:13`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.inputXmlPath }}" appears directly in run: block of step ""; move to env: map

Locations:

- `action.yml:17`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.outputHtmlPath }}" appears directly in run: block of step ""; move to env: map

Locations:

- `action.yml:17`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, static-inline-injection

**Notes:**

Rewrote action.yml to move all ${{ ... }} expressions from run: blocks into env: blocks:
1. Step 1 (pip install): ACTION_PATH env var holds ${{ github.action_path }}; run: uses "$ACTION_PATH/requirements.txt".
2. Step 2 (main.py): ACTION_PATH, INPUT_XML_PATH, and OUTPUT_HTML_PATH env vars hold the respective ${{ ... }} expressions; run: uses "$ACTION_PATH/main.py" "$INPUT_XML_PATH" ${OUTPUT_HTML_PATH:+"$OUTPUT_HTML_PATH"}. The optional outputHtmlPath uses the ${VAR:+"$VAR"} form so no empty argument is passed when the value is absent.

