<!-- markdownlint-disable -->

# Hardening Report: rjtngit--nunit-html-action/v2.0.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **rjtngit--nunit-html-action/v2.0.0** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): the 'Run python script' step puts expressions straight into the run: script. Offending lines: `python -m pip install -r ${{ github.action_path }}/requirements.txt` and `python ${{ github.action_path }}/main.py ${{ inputs.inputXmlPath }} ${{ inputs.outputHtmlPath }}`. Because inputs.inputXmlPath and inputs.outputHtmlPath come from the calling workflow, a crafted value can inject shell commands. Fix: move the values into env: variables (ACTION_PATH from github.action_path, INPUT_XML_PATH, OUTPUT_HTML_PATH). Then double-quote them in the script, using ${OUTPUT_HTML_PATH:+"$OUTPUT_HTML_PATH"} for the optional output path.

Locations:

- `action.yml:21`
- `action.yml:22`

### unpinned-uses (severity: high)

A composite step uses a tag ref instead of a full 40-character commit SHA: `uses: actions/setup-python@v6`.

Locations:

- `action.yml:15`

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

**Fixes applied:** script-injection, unpinned-uses, static-inline-injection

**Notes:**

Pinned actions/setup-python to commit ece7cb06caefa5fff74198d8649806c4678c61a1 (tag v6). Moved github.action_path, inputs.inputXmlPath and inputs.outputHtmlPath into the step's env: block as ACTION_PATH, INPUT_XML_PATH and OUTPUT_HTML_PATH. The script now reads them as double-quoted variables. The optional output path uses ${OUTPUT_HTML_PATH:+"$OUTPUT_HTML_PATH"}, so when it's empty no argument is passed and the call to main.py gets the same number of arguments as before.

