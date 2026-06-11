<!-- markdownlint-disable -->

# Hardening Report: rjtngit--nunit-html-action/v1.0.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **rjtngit--nunit-html-action/v1.0.1** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): The first `run:` step directly interpolates `${{ github.action_path }}` into a shell command string. Although `github.action_path` is not attacker-controlled in the same way as `inputs.*`, any `${{ ... }}` expression inside a `run:` block is a script-injection finding per the check rules, as the value flows through YAML template substitution before the shell ever sees it.

Offending line: `sudo python -m pip install -r ${{ github.action_path }}/requirements.txt`

Locations:

- `action.yml:14`

### script-injection (severity: high)

Sub-rule (a): The second `run:` step directly interpolates `${{ github.action_path }}`, `${{ inputs.inputXmlPath }}`, and `${{ inputs.outputHtmlPath }}` into a shell command string. The `inputs.inputXmlPath` and `inputs.outputHtmlPath` values are fully attacker-controlled and are passed unquoted directly to the shell, enabling arbitrary command injection (e.g. a value like `foo; rm -rf /` would be executed). These must be moved to `env:` variables and double-quoted in the script.

Offending line: `sudo python ${{ github.action_path }}/main.py ${{ inputs.inputXmlPath }} ${{ inputs.outputHtmlPath }}`

Locations:

- `action.yml:16`

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

Fixed all four script-injection findings in action.yml:

1. Step 1 (pip install): Moved `${{ github.action_path }}` into an `env:` block as `ACTION_PATH`, then referenced it as `"$ACTION_PATH/requirements.txt"` in the shell command.

2. Step 2 (python main.py): Moved `${{ github.action_path }}`, `${{ inputs.inputXmlPath }}`, and `${{ inputs.outputHtmlPath }}` into an `env:` block as `ACTION_PATH`, `INPUT_XML_PATH`, and `OUTPUT_HTML_PATH` respectively. Referenced them as double-quoted variables in the shell command. Since `outputHtmlPath` is optional (default: ''), used `${OUTPUT_HTML_PATH:+"$OUTPUT_HTML_PATH"}` so the argument is omitted entirely when empty, preserving the original command semantics.

