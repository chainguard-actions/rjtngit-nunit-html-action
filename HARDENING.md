<!-- markdownlint-disable -->

# Hardening Report: rjtngit--nunit-html-action/v1.0.2

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **rjtngit--nunit-html-action/v1.0.2** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

The composite action uses 'actions/setup-python@v4', which is pinned to a mutable version tag ('v4') rather than an immutable 40-character commit SHA. If the tag is moved (e.g. by a supply-chain compromise), the action will silently execute different code. Fix: pin to a full SHA, e.g. 'actions/setup-python@a26af69be951a213d495a4c3e4e4022e16d87065 # v4'.

Locations:

- `action.yml:14`

### script-injection (severity: high)

Sub-rule (a): The 'run:' block in action.yml directly interpolates GitHub Actions expressions inside shell command strings. Three expressions are used:
  1. '${{ github.action_path }}' — interpolated directly into 'python -m pip install -r ${{ github.action_path }}/requirements.txt'
  2. '${{ inputs.inputXmlPath }}' — caller-supplied input interpolated directly as a shell argument: 'python ${{ github.action_path }}/main.py ${{ inputs.inputXmlPath }} ...'
  3. '${{ inputs.outputHtmlPath }}' — caller-supplied optional input interpolated directly: '... ${{ inputs.outputHtmlPath }}'
Any ${{ }} expression inside a run: block is substituted by the Actions runner before the shell ever sees the string, allowing an attacker-controlled value (especially inputs.*) to inject arbitrary shell commands. Fix: move all values into env: variables and reference them as quoted shell variables, e.g.:
  env:
    INPUT_XML_PATH: ${{ inputs.inputXmlPath }}
    OUTPUT_HTML_PATH: ${{ inputs.outputHtmlPath }}
  run: |
    python -m pip install -r "$GITHUB_ACTION_PATH/requirements.txt"
    python "$GITHUB_ACTION_PATH/main.py" "$INPUT_XML_PATH" ${OUTPUT_HTML_PATH:+"$OUTPUT_HTML_PATH"}

Locations:

- `action.yml:19`
- `action.yml:20`

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

Fixed action.yml with two changes: (1) Pinned actions/setup-python@v4 to its full commit SHA 7f4fc3e22c37d6ff65e88745f38bd3157c663f7c, keeping the tag as a comment. (2) Moved all ${{ }} expressions out of the run: block — github.action_path replaced with the built-in $GITHUB_ACTION_PATH env var, inputs.inputXmlPath and inputs.outputHtmlPath moved to an env: block as INPUT_XML_PATH and OUTPUT_HTML_PATH, then referenced as quoted shell variables. The optional outputHtmlPath uses ${OUTPUT_HTML_PATH:+"$OUTPUT_HTML_PATH"} to avoid passing an empty positional argument when the input is not set.

