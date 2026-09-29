<!-- markdownlint-disable -->

# Hardening Report: rjtngit--nunit-html-action/v2.0.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **rjtngit--nunit-html-action/v2.0.0** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

The composite action step 'Set up Python' uses 'actions/setup-python@v6', which is pinned to a mutable tag ('v6') rather than an immutable 40-character commit SHA. A tag can be silently moved to point to a different (potentially malicious) commit, enabling a supply-chain attack. It should be pinned to a full SHA, e.g. 'actions/setup-python@<40-char-sha> # v6'.

Locations:

- `action.yml:15`

### script-injection (severity: high)

The 'Run python script' step interpolates GitHub Actions expressions directly inside a 'run:' shell command string (sub-rule a). Three expressions are embedded: '${{ github.action_path }}' (line 21), '${{ inputs.inputXmlPath }}' (line 22), and '${{ inputs.outputHtmlPath }}' (line 22). Any ${{ ... }} expression in a run: block undergoes YAML template substitution before the shell ever sees the string, allowing shell metacharacters to be injected. The inputs.* values are fully attacker-controlled (e.g. via workflow_dispatch or a calling workflow), making this a direct command-injection vector. All three should be moved to env: variables and referenced as double-quoted shell variables (e.g. "$INPUT_XML_PATH"), with the optional outputHtmlPath using the guarded form ${OUTPUT_HTML_PATH:+"$OUTPUT_HTML_PATH"}.

Locations:

- `action.yml:21`
- `action.yml:22`

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

1. Pinned actions/setup-python@v6 to full SHA ece7cb06caefa5fff74198d8649806c4678c61a1 (kept # v6 comment for readability). 2. Moved all three ${{ }} expressions out of the run: block into an env: map: github.action_path → ACTION_PATH, inputs.inputXmlPath → INPUT_XML_PATH, inputs.outputHtmlPath → OUTPUT_HTML_PATH. Shell references use double-quoted variables; the optional outputHtmlPath uses the guarded form ${OUTPUT_HTML_PATH:+"$OUTPUT_HTML_PATH"} to avoid passing an empty argument when the input is not set.

