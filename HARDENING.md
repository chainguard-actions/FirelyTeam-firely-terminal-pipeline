# Hardening Report: FirelyTeam--firely-terminal-pipeline/v0.8.10

> This file was generated automatically by the hardening agent.

**Policy SHA:** `ff50f15e4b79bfbf764dafdfd2579175a6ea9771`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **FirelyTeam--firely-terminal-pipeline/v0.8.10** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

The 'Download Java Validator' step passes the attacker-controlled input `inputs.JAVA_VALIDATOR_DOWNLOAD_LOCATION` into an `eval echo` call: `JAVA_VALIDATOR_DOWNLOAD_LOCATION=$(eval echo "$JAVA_VALIDATOR_DOWNLOAD_LOCATION")`. Although the value is routed through an `env:` variable (`JAVA_VALIDATOR_DOWNLOAD_LOCATION: ${{ inputs.JAVA_VALIDATOR_DOWNLOAD_LOCATION }}`), the subsequent `eval` expands the variable's contents as a shell expression, allowing an attacker to inject arbitrary shell commands (e.g., by supplying a value containing `$(malicious_command)`). The safe pattern is to use parameter expansion or `envsubst` without `eval`.

Locations:

- `action.yml:706`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection

**Notes:**

Replaced `eval echo "$JAVA_VALIDATOR_DOWNLOAD_LOCATION"` with `printf '%s' "$JAVA_VALIDATOR_DOWNLOAD_LOCATION" | JAVA_VALIDATOR_VERSION="$JAVA_VALIDATOR_VERSION" envsubst '$JAVA_VALIDATOR_VERSION'` in the 'Download Java Validator' step of action.yml. The `eval` was used to expand `$JAVA_VALIDATOR_VERSION` within the download URL, but it also allowed arbitrary command execution if an attacker supplied a value containing `$(malicious_command)`. The fix uses `envsubst` with an explicit, restricted variable list so only `$JAVA_VALIDATOR_VERSION` is substituted — no shell commands can be injected.

