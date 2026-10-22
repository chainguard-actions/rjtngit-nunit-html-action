<!-- markdownlint-disable -->

# Hardening Report: rjtngit--nunit-html-action/v1.0.2

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **rjtngit--nunit-html-action/v1.0.2** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

A composite action step uses a version tag instead of a full 40-character commit SHA: `uses: actions/setup-python@v4`.

Locations:

- `action.yml:15`

### script-injection (severity: high)

Sub-rule (a): expressions are placed directly inside the run: script. The offending lines are `python -m pip install -r ${{ github.action_path }}/requirements.txt` and `python ${{ github.action_path }}/main.py ${{ inputs.inputXmlPath }} ${{ inputs.outputHtmlPath }}`. Both inputs are user-controlled and unquoted, so they can inject shell commands. Fix: move them to env: variables (INPUT_XML_PATH, OUTPUT_HTML_PATH) and use "$GITHUB_ACTION_PATH". Quote the required input and pass the optional one as ${OUTPUT_HTML_PATH:+"$OUTPUT_HTML_PATH"}.

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

Pinned actions/setup-python to 7f4fc3e22c37d6ff65e88745f38bd3157c663f7c (v4). Moved inputs into env (INPUT_XML_PATH, OUTPUT_HTML_PATH), replaced github.action_path with "$GITHUB_ACTION_PATH", quoted the required input and passed the optional output path as ${OUTPUT_HTML_PATH:+"$OUTPUT_HTML_PATH"} so main.py's len(sys.argv) default logic still works.

