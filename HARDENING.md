<!-- markdownlint-disable -->

# Hardening Report: rjtngit--nunit-html-action/v1.0.2

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **rjtngit--nunit-html-action/v1.0.2** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

The 'Run python script' step directly interpolates GitHub Actions expressions into the `run:` shell command string (sub-rule a). Line 21 uses `${{ github.action_path }}` and line 22 uses `${{ github.action_path }}`, `${{ inputs.inputXmlPath }}`, and `${{ inputs.outputHtmlPath }}`. The `inputs.*` values are attacker-controlled and are passed directly as positional arguments to `python`, enabling command injection. All `${{ ... }}` expressions must be moved to `env:` variables and then referenced as double-quoted shell variables (e.g., `"$INPUT_XML_PATH"`).

Locations:

- `action.yml:21`
- `action.yml:22`

### unpinned-uses (severity: high)

The step 'Set up Python' references `actions/setup-python@v4`, which uses a mutable version tag (`v4`) instead of a full 40-character commit SHA. A tag can be moved to point to a different (potentially malicious) commit, enabling supply-chain attacks. Pin to a specific commit SHA, e.g., `actions/setup-python@a26af69be951a213d495a4c3e4e4022e16d87065 # v4`.

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

**Fixes applied:** unpinned-uses, script-injection, static-inline-injection

**Notes:**

Fixed action.yml: (1) Pinned actions/setup-python@v4 to full SHA 7f4fc3e22c37d6ff65e88745f38bd3157c663f7c with # v4 comment. (2) Moved all ${{ github.action_path }}, ${{ inputs.inputXmlPath }}, and ${{ inputs.outputHtmlPath }} expressions out of the run: block into an env: map as ACTION_PATH, INPUT_XML_PATH, and OUTPUT_HTML_PATH respectively. Referenced them as double-quoted shell variables in the run: block. Used ${OUTPUT_HTML_PATH:+"$OUTPUT_HTML_PATH"} for the optional outputHtmlPath argument to avoid passing an empty positional argument when the value is unset.

