# Hardening Report: FirelyTeam--firely-terminal-pipeline/v0.8.9

> This file was generated automatically by the hardening agent.

**Policy SHA:** `ff50f15e4b79bfbf764dafdfd2579175a6ea9771`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **FirelyTeam--firely-terminal-pipeline/v0.8.9** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### suspicious-run-content (severity: high)

The 'Download Java Validator' step uses `eval echo "$JAVA_VALIDATOR_DOWNLOAD_LOCATION"` to perform shell variable expansion on an attacker-controlled input. The env var JAVA_VALIDATOR_DOWNLOAD_LOCATION is set directly from `${{ inputs.JAVA_VALIDATOR_DOWNLOAD_LOCATION }}`. Using `eval` with a `$`-prefixed variable (matching pattern `eval\s+[\x60$]`) allows an attacker who controls the input to inject arbitrary shell commands that will be executed during the eval. Sub-check: eval-dynamic.

Locations:

- `action.yml:630`

## Iteration Notes

### Iteration 1

**Fixes applied:** suspicious-run-content

**Notes:**

Replaced `eval echo "$JAVA_VALIDATOR_DOWNLOAD_LOCATION"` with safe bash parameter substitution `"${JAVA_VALIDATOR_DOWNLOAD_LOCATION/\$JAVA_VALIDATOR_VERSION/$JAVA_VALIDATOR_VERSION}"` in the 'Download Java Validator' step. This eliminates the eval-based command injection vulnerability (where an attacker controlling the JAVA_VALIDATOR_DOWNLOAD_LOCATION input could inject arbitrary shell commands) while preserving the intended behavior of expanding $JAVA_VALIDATOR_VERSION within the URL template.

