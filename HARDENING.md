<!-- markdownlint-disable -->

# Hardening Report: rjtngit--nunit-html-action/v2.0.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **rjtngit--nunit-html-action/v2.0.0** was hardened automatically. 5 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

The `run:` block in action.yml directly interpolates GitHub Actions expressions inside shell commands, violating rule (a). The expressions `${{ github.action_path }}`, `${{ inputs.inputXmlPath }}`, and `${{ inputs.outputHtmlPath }}` are substituted into the shell command string before the shell ever sees them. An attacker controlling `inputs.inputXmlPath` or `inputs.outputHtmlPath` can inject arbitrary shell commands (e.g. via semicolons, backticks, or `$(...)` in the input value). Even `${{ github.action_path }}` should be passed via an env var and referenced as `$GITHUB_ACTION_PATH`. All three expressions must be moved to `env:` variables and double-quoted in the shell script.

Locations:

- `action.yml:21`

### unpinned-uses (severity: high)

Multiple `uses:` references are pinned to mutable tags rather than immutable 40-character commit SHAs, making the action vulnerable to supply-chain attacks if the tag is moved:
- `action.yml`: `actions/setup-python@v6`
- `.github/workflows/test.yml`: `actions/checkout@v6`
- `.github/workflows/test.yml`: `actions/upload-artifact@v7`
All should be pinned to full SHA digests (e.g. `actions/checkout@<40-hex-sha> # v6`).

Locations:

- `action.yml:14`
- `.github/workflows/test.yml:7`
- `.github/workflows/test.yml:13`

### missing-permissions (severity: medium)

The workflow file `.github/workflows/test.yml` has no top-level `permissions:` key and the single job `generate-html` also has no job-level `permissions:` key. Without explicit permissions, the workflow inherits the repository default (often `write-all` for private repos or broad read access for public repos). A minimal `permissions:` block (e.g. `contents: read`) should be added at the top level or on the job.

Locations:

- `.github/workflows/test.yml:1`

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

**Fixes applied:** script-injection, static-inline-injection, unpinned-uses, missing-permissions

**Notes:**

Fixed action.yml: moved ${{ github.action_path }}, ${{ inputs.inputXmlPath }}, and ${{ inputs.outputHtmlPath }} from the run: block into an env: map (ACTION_PATH, INPUT_XML_PATH, OUTPUT_HTML_PATH); used ${OUTPUT_HTML_PATH:+"$OUTPUT_HTML_PATH"} for the optional argument to avoid passing an empty string. Pinned actions/setup-python@v6 to SHA ece7cb06caefa5fff74198d8649806c4678c61a1. Fixed .github/workflows/test.yml: added top-level permissions: { contents: read }; pinned actions/checkout@v6 to SHA d23441a48e516b6c34aea4fa41551a30e30af803 and actions/upload-artifact@v7 to SHA 043fb46d1a93c77aae656e7c1c64a875d1fc6a0a.

