<!-- markdownlint-disable -->

# Hardening Report: FirelyTeam--firely-terminal-pipeline/v0.8.9

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **FirelyTeam--firely-terminal-pipeline/v0.8.9** was hardened automatically. 3 finding(s) were identified and resolved across 4 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

All three workflow files reference `actions/checkout@v4` using a mutable version tag instead of a pinned 40-character commit SHA. This allows supply-chain attacks if the tag is moved to a malicious commit.

Locations:

- `.github/workflows/update_firelyterminal.yml:17`
- `.github/workflows/update_javavalidator.yml:17`
- `.github/workflows/update_sushi.yml:17`

### permissions (severity: medium)

None of the three workflow files define a top-level `permissions:` block, and none of the jobs define job-level `permissions:` blocks. This means the workflows run with the default (potentially broad) GITHUB_TOKEN permissions. Each workflow should declare minimal required permissions (e.g., `contents: write`, `pull-requests: write`).

Locations:

- `.github/workflows/update_firelyterminal.yml:1`
- `.github/workflows/update_javavalidator.yml:1`
- `.github/workflows/update_sushi.yml:1`

### script-injection (severity: high)

Sub-rule (b): Multiple `run:` blocks in action.yml expand user-controlled input variables without double-quoting, allowing shell metacharacter injection. Affected lines:
- `dotnet tool install --global Firely.Terminal --version $FIRELY_TERMINAL_VERSION` (FIRELY_TERMINAL_VERSION from inputs, unquoted)
- `fhir login email=$INPUT_SIMPLIFIER_USERNAME password=$INPUT_SIMPLIFIER_PASSWORD` (both from inputs, unquoted)
- `sudo npm install -g fsh-sushi@$SUSHI_VERSION` (SUSHI_VERSION from inputs, unquoted)
- `sushi $INPUT_SUSHI_OPTIONS` (SUSHI_OPTIONS from inputs, unquoted — can inject arbitrary sushi CLI flags or shell metacharacters)
- `fhir check $INPUT_PATH_TO_QUALITY_CONTROL_RULES` (PATH_TO_QUALITY_CONTROL_RULES from inputs, unquoted)
- `java -jar validator_cli.jar $GITHUB_WORKSPACE/$p*.xml ... $INPUT_JAVA_VALIDATION_OPTIONS $UNESCPAED_IG_DEPENDENCIES $LOCAL_IG_PARAMETERS` (multiple unquoted vars from inputs)
- `wget -q $JAVA_VALIDATOR_DOWNLOAD_LOCATION` (JAVA_VALIDATOR_DOWNLOAD_LOCATION from inputs, unquoted)

Sub-rule (a): `JAVA_VALIDATOR_DOWNLOAD_LOCATION=$(eval echo "$JAVA_VALIDATOR_DOWNLOAD_LOCATION")` — the env var is sourced from `inputs.JAVA_VALIDATOR_DOWNLOAD_LOCATION` and passed directly to `eval`, enabling arbitrary command execution by a caller who supplies a malicious value such as `$(malicious_command)`.

All these variables should be double-quoted in the shell commands.

Locations:

- `action.yml:130`
- `action.yml:148`
- `action.yml:490`
- `action.yml:500`
- `action.yml:580`
- `action.yml:640`
- `action.yml:643`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, permissions, script-injection

**Notes:**

Fixed all three findings:

1. **unpinned-uses**: Pinned `actions/checkout@v4` to full SHA `11d5960a326750d5838078e36cf38b85af677262` in all three workflow files (.github/workflows/update_firelyterminal.yml, update_javavalidator.yml, update_sushi.yml).

2. **permissions**: Added `permissions: { contents: write, pull-requests: write }` top-level blocks to all three workflow files (these workflows create branches and open PRs).

3. **script-injection**: Fixed all instances in action.yml:
   - Quoted `$FIRELY_TERMINAL_VERSION` in `dotnet tool install` command
   - Quoted `$INPUT_SIMPLIFIER_USERNAME` and `$INPUT_SIMPLIFIER_PASSWORD` in `fhir login` command
   - Quoted `fsh-sushi@$SUSHI_VERSION` in `npm install` command
   - Replaced `sushi $INPUT_SUSHI_OPTIONS` with xargs-based array tokenization in both SUSHI invocation steps
   - Replaced `fhir check $INPUT_PATH_TO_QUALITY_CONTROL_RULES` with `${INPUT_PATH_TO_QUALITY_CONTROL_RULES:+"$INPUT_PATH_TO_QUALITY_CONTROL_RULES"}` (optional single path value)
   - Replaced `eval echo "$JAVA_VALIDATOR_DOWNLOAD_LOCATION"` with safe bash string substitution `"${JAVA_VALIDATOR_DOWNLOAD_LOCATION//'$JAVA_VALIDATOR_VERSION'/$JAVA_VALIDATOR_VERSION}"` to expand the version placeholder without arbitrary code execution
   - Quoted `$JAVA_VALIDATOR_DOWNLOAD_LOCATION` in `wget` command
   - Converted `$INPUT_JAVA_VALIDATION_OPTIONS` and `$UNESCPAED_IG_DEPENDENCIES` to arrays using xargs tokenization in both Java validator steps
   - Converted `LOCAL_IG_PARAMETERS` and `COMBINED_IG_PARAMETERS` from space-concatenated strings to proper bash arrays with separate `-ig` and path elements

### Iteration 2

**Fixes applied:** script-injection, github-env-injection

**Notes:**

Fixed 2 security findings in hardened/action/action.yml:

1. github-env-injection: Added sanitization of fhirVersion before writing to GITHUB_ENV. Added `safe_fhirVersion=$(printf '%s' "$fhirVersion" | tr -d '\n\r')` and changed the GITHUB_ENV write to use the sanitized variable.

2. script-injection (6 locations):
   - 'Run Quality Control checks' step: Double-quoted $INPUT_EXPECTED_FAILS in the echo|grep pipeline.
   - 'Validate all conformance resources' step: Converted both `for p in $INPUT_PATH_TO_CONFORMANCE_RESOURCES` loops to use xargs-based array tokenization (conformance_paths array), and double-quoted $INPUT_EXPECTED_FAILS in the echo|grep pipeline.
   - 'Validate all example resources' step: Converted `for p in $INPUT_PATH_TO_CONFORMANCE_RESOURCES` loop to use xargs-based array tokenization (conformance_paths array), converted `for p in $PATH_TO_EXAMPLES` loop to use xargs-based array tokenization (example_paths array), and double-quoted $INPUT_EXPECTED_FAILS in the echo|grep pipeline.

PATH_TO_CONFORMANCE_RESOURCES and PATH_TO_EXAMPLES are space-separated lists of paths, so they correctly use xargs-based tokenization (which handles quoted paths) rather than simple double-quoting.

### Iteration 1

**Fixes applied:** script-injection

**Notes:**

Fixed all 20+ script injection vulnerabilities in hardened/action/action.yml. Every instance where a boolean input variable was used directly as a shell command (e.g., `if $INPUT_DOTNET_VALIDATION_ENABLED; then`) was replaced with a safe string comparison (e.g., `if [[ "$INPUT_DOTNET_VALIDATION_ENABLED" == "true" ]]; then`). This affects the following variables: INPUT_DOTNET_VALIDATION_ENABLED, INPUT_TERMINOLOGY_SERVICE_BFARM_ENABLED, INPUT_JAVA_VALIDATION_ENABLED, INPUT_SUSHI_ENABLED, INPUT_SUSHI_USE_CONFIG_DEPENDENCIES, INPUT_JAVA_SNAPSHOT_ENABLED, and CLOSE_SLICING_FOR_VALIDATION. Compound conditions with || and && were also fixed to use the safe string comparison form for each operand.

### Iteration 2

**Fixes applied:** github-env-injection

**Notes:**

Fixed all 6 github-env-injection instances across 3 workflow files by adding `safe=$(printf '%s' "$VAR" | tr -d '\n\r')` sanitization before each write to $GITHUB_ENV. Affected variables: FIRELY_TERMINAL_VERSION and LATEST_RELEASE in update_firelyterminal.yml; JAVA_VALIDATOR_VERSION and LATEST_RELEASE in update_javavalidator.yml; SUSHI_VERSION and LATEST_RELEASE in update_sushi.yml. Also quoted $GITHUB_ENV references consistently.

