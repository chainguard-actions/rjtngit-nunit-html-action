<!-- markdownlint-disable -->

# Hardening Report: rjtngit--nunit-html-action/v1.0.2

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **rjtngit--nunit-html-action/v1.0.2** was hardened automatically. 5 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

The `run:` block in action.yml directly interpolates GitHub Actions expressions inside shell commands without routing them through env vars. Specifically: `${{ github.action_path }}` (rule a — any expression in a run block is a script-injection risk), `${{ inputs.inputXmlPath }}` (rule a — attacker-controlled input interpolated directly into shell), and `${{ inputs.outputHtmlPath }}` (rule a — attacker-controlled input interpolated directly into shell). An attacker calling this composite action can supply a crafted `inputXmlPath` or `outputHtmlPath` value containing shell metacharacters (`;`, `|`, `$(...)`, etc.) that will be executed by the runner shell. The fix is to move all expressions into `env:` variables and double-quote their expansions in the script.

Locations:

- `action.yml:20`

### unpinned-uses (severity: high)

Multiple `uses:` references are pinned to mutable version tags instead of immutable 40-character commit SHAs, making the action vulnerable to supply-chain attacks if the upstream tag is moved or compromised. Failing references:
- `action.yml`: `uses: actions/setup-python@v4` (tag `v4`)
- `.github/workflows/test.yml`: `uses: actions/checkout@v3` (tag `v3`)
- `.github/workflows/test.yml`: `uses: actions/upload-artifact@v3` (tag `v3`)
All should be pinned to a full SHA, e.g. `actions/setup-python@<40-hex-char-sha> # v4`.

Locations:

- `action.yml:17`
- `.github/workflows/test.yml:8`
- `.github/workflows/test.yml:14`

### missing-permissions (severity: medium)

The workflow file `.github/workflows/test.yml` has no top-level `permissions:` key and the only job (`generate-html`) also has no job-level `permissions:` key. Without an explicit permissions block the workflow inherits the repository's default token permissions, which may be overly broad (e.g. `write` on `contents`). A minimal explicit `permissions:` block (e.g. `permissions: {}` or only the scopes actually needed) should be added at the top level or on the job.

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

Fixed action.yml: (1) pinned actions/setup-python@v4 to SHA 7f4fc3e22c37d6ff65e88745f38bd3157c663f7c, (2) moved all ${{ }} expressions (github.action_path, inputs.inputXmlPath, inputs.outputHtmlPath) into an env: block and referenced them as double-quoted shell variables in the run: script, using ${OUTPUT_HTML_PATH:+"$OUTPUT_HTML_PATH"} for the optional argument. Fixed .github/workflows/test.yml: (1) added top-level `permissions: {}`, (2) pinned actions/checkout@v3 to SHA a37ce9120846195fa4ece8f58b268e6043cb2f26, (3) pinned actions/upload-artifact@v3 to SHA ff15f0306b3f739f7b6fd43fb5d26cd321bd4de5.

