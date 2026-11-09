<!-- markdownlint-disable -->

# Hardening Report: FirelyTeam--firely-terminal-pipeline/v0.8.22

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **FirelyTeam--firely-terminal-pipeline/v0.8.22** was hardened automatically. 24 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (b): 'Check if .NET is installed' step: `if $INPUT_DOTNET_VALIDATION_ENABLED || $INPUT_TERMINOLOGY_SERVICE_BFARM_ENABLED; then` expands inputs.* env vars without quotes as the command itself, so the shell runs whatever the input contains.

Locations:

- `action.yml:90`

### script-injection (severity: high)

Rule (b): 'Check .NET SDK Version' step: `if $INPUT_DOTNET_VALIDATION_ENABLED; then` runs an unquoted input value as a command.

Locations:

- `action.yml:105`

### script-injection (severity: high)

Rule (b): 'Install Firely.Terminal' step: `if $INPUT_DOTNET_VALIDATION_ENABLED || $INPUT_TERMINOLOGY_SERVICE_BFARM_ENABLED` runs input values as commands, and `dotnet tool install --global Firely.Terminal --version $FIRELY_TERMINAL_VERSION` expands an input without quotes.

Locations:

- `action.yml:116`
- `action.yml:119`

### script-injection (severity: high)

Rule (b): 'Check Firely Terminal Version' step: `if $INPUT_DOTNET_VALIDATION_ENABLED; then` runs an unquoted input as a command.

Locations:

- `action.yml:131`

### script-injection (severity: high)

Rule (b): 'Simplifier login' step: `if $INPUT_DOTNET_VALIDATION_ENABLED; then` and `fhir login email=$INPUT_SIMPLIFIER_USERNAME password=$INPUT_SIMPLIFIER_PASSWORD` expand input-derived env vars without quotes.

Locations:

- `action.yml:142`
- `action.yml:143`

### script-injection (severity: high)

Rule (b): 'Detect FHIR version and determine dependency source' step: `if $INPUT_DOTNET_VALIDATION_ENABLED || $INPUT_JAVA_VALIDATION_ENABLED`, `if $INPUT_SUSHI_USE_CONFIG_DEPENDENCIES; then` and `elif $INPUT_SUSHI_ENABLED && ...` run unquoted input values as commands.

Locations:

- `action.yml:168`
- `action.yml:183`
- `action.yml:202`

### script-injection (severity: high)

Rule (b): 'Validate FHIR version support' step: `if $INPUT_DOTNET_VALIDATION_ENABLED || $INPUT_JAVA_VALIDATION_ENABLED; then` runs unquoted input values as commands.

Locations:

- `action.yml:285`

### script-injection (severity: high)

Rule (b): 'Set FHIR specification and restore dependencies' step: `if $INPUT_DOTNET_VALIDATION_ENABLED; then` runs an unquoted input as a command.

Locations:

- `action.yml:363`

### script-injection (severity: high)

Rule (b): 'Check if npm is installed' step: `if $INPUT_SUSHI_ENABLED; then` runs an unquoted input as a command.

Locations:

- `action.yml:501`

### script-injection (severity: high)

Rule (b): 'Check npm Version' step: `if $INPUT_SUSHI_ENABLED; then` runs an unquoted input as a command.

Locations:

- `action.yml:514`

### script-injection (severity: high)

Rule (b): 'Install fsh-sushi' step: `if $INPUT_SUSHI_ENABLED; then`, `sudo chown -R $USER ...` and `sudo npm install -g fsh-sushi@$SUSHI_VERSION` expand workflow-controlled vars without quotes.

Locations:

- `action.yml:524`
- `action.yml:526`

### script-injection (severity: high)

Rule (b): 'Check SUSHI version' step: `if $INPUT_SUSHI_ENABLED; then` runs an unquoted input as a command.

Locations:

- `action.yml:535`

### script-injection (severity: high)

Rule (b): 'Generate conformance resources with SUSHI' step: `if $INPUT_SUSHI_ENABLED; then` and `sushi $INPUT_SUSHI_OPTIONS` expand inputs without quotes.

Locations:

- `action.yml:544`
- `action.yml:545`

### script-injection (severity: high)

Rule (b): 'Pre-processing FHIR profiles (Slicing open-closed)' step: `if $CLOSE_SLICING_FOR_VALIDATION; then` runs an unquoted input as a command.

Locations:

- `action.yml:558`

### script-injection (severity: high)

Rule (b): 'Configure Firely Terminal validator engine' step: `if $INPUT_DOTNET_VALIDATION_ENABLED; then` runs an unquoted input as a command.

Locations:

- `action.yml:613`

### script-injection (severity: high)

Rule (b): 'Run Quality Control checks' step: `if $INPUT_DOTNET_VALIDATION_ENABLED`, `echo $INPUT_EXPECTED_FAILS | grep ...`, `fhir check $INPUT_PATH_TO_QUALITY_CONTROL_RULES` and `fhir check --fail $INPUT_PATH_TO_QUALITY_CONTROL_RULES` expand inputs without quotes.

Locations:

- `action.yml:637`
- `action.yml:639`
- `action.yml:640`
- `action.yml:642`

### script-injection (severity: high)

Rule (b): 'Report Success - .NET Validator' step: `if $INPUT_DOTNET_VALIDATION_ENABLED; then` runs an unquoted input as a command.

Locations:

- `action.yml:653`

### script-injection (severity: high)

Rule (b): 'Remove package cache...' step: `if $INPUT_JAVA_SNAPSHOT_ENABLED; then` runs an unquoted input as a command, and `rm -rf $fhirCacheLocation` is also unquoted.

Locations:

- `action.yml:670`
- `action.yml:672`

### script-injection (severity: high)

Rule (b): 'Check if Java is installed' step: `if $INPUT_JAVA_VALIDATION_ENABLED; then` runs an unquoted input as a command.

Locations:

- `action.yml:680`

### script-injection (severity: high)

Rule (b): 'Download Java Validator' step: `if $INPUT_JAVA_VALIDATION_ENABLED; then` runs an unquoted input as a command. `JAVA_VALIDATOR_DOWNLOAD_LOCATION=$(eval echo "$JAVA_VALIDATOR_DOWNLOAD_LOCATION")` passes inputs.JAVA_VALIDATOR_DOWNLOAD_LOCATION to eval, so any $(...), backticks or ';' in the input get executed. `wget -q $JAVA_VALIDATOR_DOWNLOAD_LOCATION -O validator_cli.jar` is unquoted.

Locations:

- `action.yml:693`
- `action.yml:697`
- `action.yml:700`

### script-injection (severity: high)

Rule (b): 'Validate all conformance resources in scope of the repository' step: `if $INPUT_JAVA_VALIDATION_ENABLED`, `for p in $INPUT_PATH_TO_CONFORMANCE_RESOURCES`, `echo $INPUT_EXPECTED_FAILS | grep ...` and `java -jar validator_cli.jar $GITHUB_WORKSPACE/$p*.xml ... -version $FHIR_VERSION $INPUT_JAVA_VALIDATION_OPTIONS ...` expand input/env-derived vars without quotes.

Locations:

- `action.yml:718`
- `action.yml:741`
- `action.yml:746`
- `action.yml:756`
- `action.yml:757`

### script-injection (severity: high)

Rule (b): 'Validate all example resources in scope of the repository' step: `if $INPUT_JAVA_VALIDATION_ENABLED`, `for p in $INPUT_PATH_TO_CONFORMANCE_RESOURCES`, `for p in $PATH_TO_EXAMPLES`, `echo $INPUT_EXPECTED_FAILS | grep ...` and `java -jar validator_cli.jar ... -version $FHIR_VERSION $INPUT_JAVA_VALIDATION_OPTIONS ...` expand input/env-derived vars without quotes.

Locations:

- `action.yml:792`
- `action.yml:809`
- `action.yml:814`
- `action.yml:824`
- `action.yml:825`

### script-injection (severity: high)

Rule (b): 'Re-run SUSHI to restore conformance resources...' step: `if $CLOSE_SLICING_FOR_VALIDATION && $INPUT_SUSHI_ENABLED; then` runs inputs as commands, and `sushi $INPUT_SUSHI_OPTIONS` is unquoted.

Locations:

- `action.yml:855`
- `action.yml:857`

### github-env-injection (severity: high)

'Detect FHIR version and determine dependency source' step: `echo "FHIR_VERSION=$fhirVersion" >> "$GITHUB_ENV"` writes a value read from repository-controlled package.json / sushi-config.yaml (via jq/yq, including a base64-decoded fallback). That value is not a literal computed in the script, and it is written without the `printf '%s' ... | tr -d '\n\r'` sanitization step. A multi-line value could inject extra environment variables.

Locations:

- `action.yml:271`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection

**Notes:**

Edited action.yml only. Not tested by running the action.

- **Boolean inputs:** every `if $INPUT_X` / `elif` / `&&` / `||` that ran an input as a command (DOTNET/JAVA validation, BFARM, SUSHI, SUSHI_USE_CONFIG_DEPENDENCIES, CLOSE_SLICING, JAVA_SNAPSHOT) is now `[ "$INPUT_X" = "true" ]`. Only the exact string `true` turns a step on. Before, an empty value also counted as on, because an empty command succeeds.
- **Quoting:** these values are now double-quoted: FIRELY_TERMINAL_VERSION, the Simplifier login (`"email=..." "password=..."`), `$USER`, `fsh-sushi@$SUSHI_VERSION`, `rm -rf "$fhirCacheLocation"`, every `echo "$INPUT_EXPECTED_FAILS"`, `echo "$IG_DEPENDENCIES"` and `echo "$FHIR_RESTORE"`. The optional quality-control rules path is passed as `${VAR:+"$VAR"}`, so it drops out when empty.
- **No more `eval` on the download URL:** JAVA_VALIDATOR_DOWNLOAD_LOCATION only gets the documented `$JAVA_VALIDATOR_VERSION` / `${JAVA_VALIDATOR_VERSION}` placeholder replaced by plain text substitution. The URL is quoted for wget. Any other `$VAR`, `$(...)` or backticks in the input now stay as literal text.
- **List inputs** (SUSHI_OPTIONS, JAVA_VALIDATION_OPTIONS, PATH_TO_CONFORMANCE_RESOURCES, PATH_TO_EXAMPLES, and the IG dependency string): each is split into a bash array with an unquoted assignment. That gives the same word splitting and globbing upstream got, and the array is expanded quoted. The `-ig` parameters are built as arrays too. The `*.xml` / `*.json` file globs still work, with the directory part quoted: `"$GITHUB_WORKSPACE/$p"*.xml`. One small difference: a conformance path containing spaces now stays a single `-ig` argument, where upstream split it.
- **github-env-injection:** fhirVersion is passed through `printf '%s' | tr -d '\n\r'` before it is written to GITHUB_ENV.

