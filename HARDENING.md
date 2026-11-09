<!-- markdownlint-disable -->

# Hardening Report: FirelyTeam--firely-terminal-pipeline/v0.8.8

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **FirelyTeam--firely-terminal-pipeline/v0.8.8** was hardened automatically. 21 finding(s) were identified and resolved across 3 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (b): in the Download Java Validator step, input JAVA_VALIDATOR_DOWNLOAD_LOCATION is passed through eval: `JAVA_VALIDATOR_DOWNLOAD_LOCATION=$(eval echo "$JAVA_VALIDATOR_DOWNLOAD_LOCATION")`. Quoting does not help because eval parses the value again as shell code, so a value like `x; malicious_cmd` or `$(cmd)` runs. The step then expands it unquoted: `wget -q $JAVA_VALIDATOR_DOWNLOAD_LOCATION -O validator_cli.jar`. The same step also runs the boolean input as a command: `if $INPUT_JAVA_VALIDATION_ENABLED; then`.

Locations:

- `action.yml:656`

### script-injection (severity: high)

Rule (b): the Check if .NET is installed step runs workflow-controlled inputs unquoted as commands: `if $INPUT_DOTNET_VALIDATION_ENABLED || $INPUT_TERMINOLOGY_SERVICE_BFARM_ENABLED; then`. Whatever the input holds is executed as a shell command.

Locations:

- `action.yml:90`

### script-injection (severity: high)

Rule (b): the Check .NET SDK Version step runs an input unquoted as a command: `if $INPUT_DOTNET_VALIDATION_ENABLED; then`.

Locations:

- `action.yml:105`

### script-injection (severity: high)

Rule (b): the Install Firely.Terminal step expands inputs without quotes: `if $INPUT_DOTNET_VALIDATION_ENABLED || $INPUT_TERMINOLOGY_SERVICE_BFARM_ENABLED; then` and `dotnet tool install --global Firely.Terminal --version $FIRELY_TERMINAL_VERSION > /dev/null`.

Locations:

- `action.yml:116`
- `action.yml:119`

### script-injection (severity: high)

Rule (b): the Check Firely Terminal Version step runs an input unquoted as a command: `if $INPUT_DOTNET_VALIDATION_ENABLED; then`.

Locations:

- `action.yml:131`

### script-injection (severity: high)

Rule (b): the Simplifier login step expands the credentials without quotes: `fhir login email=$INPUT_SIMPLIFIER_USERNAME password=$INPUT_SIMPLIFIER_PASSWORD`. It also runs an input as a command: `if $INPUT_DOTNET_VALIDATION_ENABLED; then`.

Locations:

- `action.yml:143`

### script-injection (severity: high)

Rule (b): the Detect FHIR version step runs inputs unquoted as commands: `if $INPUT_DOTNET_VALIDATION_ENABLED || $INPUT_JAVA_VALIDATION_ENABLED; then`, `if $INPUT_SUSHI_USE_CONFIG_DEPENDENCIES; then` and `elif $INPUT_SUSHI_ENABLED && ...`.

Locations:

- `action.yml:168`

### script-injection (severity: high)

Rule (b): the Validate FHIR version support step runs inputs unquoted as commands: `if $INPUT_DOTNET_VALIDATION_ENABLED || $INPUT_JAVA_VALIDATION_ENABLED; then`.

Locations:

- `action.yml:292`

### script-injection (severity: high)

Rule (b): the Set FHIR specification and restore dependencies step runs an input unquoted as a command: `if $INPUT_DOTNET_VALIDATION_ENABLED; then`.

Locations:

- `action.yml:368`

### script-injection (severity: high)

Rule (b): the Check if npm is installed and Check npm Version steps run an input unquoted as a command: `if $INPUT_SUSHI_ENABLED; then`.

Locations:

- `action.yml:489`

### script-injection (severity: high)

Rule (b): the Install fsh-sushi step expands an input without quotes: `sudo npm install -g fsh-sushi@$SUSHI_VERSION`. The step also runs `if $INPUT_SUSHI_ENABLED; then`.

Locations:

- `action.yml:510`

### script-injection (severity: high)

Rule (b): the Check SUSHI version and Generate conformance resources with SUSHI steps expand inputs without quotes: `if $INPUT_SUSHI_ENABLED; then` and `sushi $INPUT_SUSHI_OPTIONS`.

Locations:

- `action.yml:522`

### script-injection (severity: high)

Rule (b): the Pre-processing FHIR profiles (Slicing open-closed) step runs an input unquoted as a command: `if $CLOSE_SLICING_FOR_VALIDATION; then`.

Locations:

- `action.yml:534`

### script-injection (severity: high)

Rule (b): the Configure Firely Terminal validator engine step runs an input unquoted as a command: `if $INPUT_DOTNET_VALIDATION_ENABLED; then`.

Locations:

- `action.yml:582`

### script-injection (severity: high)

Rule (b): the Run Quality Control checks step expands inputs without quotes: `if $INPUT_DOTNET_VALIDATION_ENABLED; then`, `if echo $INPUT_EXPECTED_FAILS | grep ...`, `fhir check $INPUT_PATH_TO_QUALITY_CONTROL_RULES` and `fhir check --fail $INPUT_PATH_TO_QUALITY_CONTROL_RULES`.

Locations:

- `action.yml:605`

### script-injection (severity: high)

Rule (b): the Report Success - .NET Validator step runs an input unquoted as a command: `if $INPUT_DOTNET_VALIDATION_ENABLED; then`.

Locations:

- `action.yml:620`

### script-injection (severity: high)

Rule (b): the Remove package cache step runs an input unquoted as a command: `if $INPUT_JAVA_SNAPSHOT_ENABLED; then`.

Locations:

- `action.yml:634`

### script-injection (severity: high)

Rule (b): the Check if Java is installed step runs an input unquoted as a command: `if $INPUT_JAVA_VALIDATION_ENABLED; then`.

Locations:

- `action.yml:642`

### script-injection (severity: high)

Rule (b): the Validate all conformance resources step expands inputs and env values without quotes: `if $INPUT_JAVA_VALIDATION_ENABLED; then`, `for p in $INPUT_PATH_TO_CONFORMANCE_RESOURCES;`, `if echo $INPUT_EXPECTED_FAILS | grep ...` and `java -jar validator_cli.jar $GITHUB_WORKSPACE/$p*.xml ... -version $FHIR_VERSION $INPUT_JAVA_VALIDATION_OPTIONS ...`.

Locations:

- `action.yml:668`

### script-injection (severity: high)

Rule (b): the Validate all example resources step expands inputs and env values without quotes: `if $INPUT_JAVA_VALIDATION_ENABLED; then`, `for p in $INPUT_PATH_TO_CONFORMANCE_RESOURCES;`, `for p in $PATH_TO_EXAMPLES;`, `if echo $INPUT_EXPECTED_FAILS | grep ...` and `java -jar validator_cli.jar ... -version $FHIR_VERSION $INPUT_JAVA_VALIDATION_OPTIONS ...`.

Locations:

- `action.yml:747`

### script-injection (severity: high)

Rule (b): the Re-run SUSHI step expands inputs without quotes: `if $CLOSE_SLICING_FOR_VALIDATION && $INPUT_SUSHI_ENABLED; then` and `sushi $INPUT_SUSHI_OPTIONS`.

Locations:

- `action.yml:822`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection

**Notes:**

Fixed all 21 script-injection findings in action.yml. Nothing was run or tested; the only check was a final search.

- **Boolean inputs:** every step used to run the input as a command (`if $INPUT_X; then`). They now use string comparisons (`if [ "$INPUT_X" = "true" ]`). One difference from upstream: an empty value used to count as true and now counts as false. Every boolean input has a default, so this only matters if a caller passes an empty string.
- **Download Java Validator:** removed the `eval`. The step now replaces only the documented `$JAVA_VALIDATOR_VERSION` placeholder (also `${JAVA_VALIDATOR_VERSION}`) in the URL, then runs `wget -q "$URL"`. Other `$VAR` text in a custom URL is no longer expanded, but the default URL works as before.
- **Single values are now quoted:** the Firely Terminal version, the `fsh-sushi@$SUSHI_VERSION` package, the Simplifier login (`"email=..." "password=..."`), `$USER`, the cache directory passed to `rm -rf`, and `-version "$FHIR_VERSION"`. The quality-control rules path is passed as `${VAR:+"$VAR"}`, so it drops out when empty. EXPECTED_FAILS is checked with `printf '%s\n' "$VAR" | grep -w`.
- **List inputs** (SUSHI_OPTIONS, JAVA_VALIDATION_OPTIONS, PATH_TO_CONFORMANCE_RESOURCES, PATH_TO_EXAMPLES, and the IG dependency string read from package.json): upstream expanded these unquoted, so they are now split into arrays the same way (`arr=($VAR)`). Splitting on whitespace and glob expansion behave as before, and nothing in the values is evaluated.
- **Validator arguments:** the `-ig` arguments are now built as arrays instead of space-joined strings. The validator file globs keep the path quoted and leave only `*.xml` / `*.json` unquoted, so matching still works.

### Iteration 2

**Fixes applied:** github-env-injection

**Notes:**

In action.yml, the 'Detect FHIR version' step now strips newlines from fhirVersion with printf '%s' | tr -d '\n\r' before it writes FHIR_VERSION to $GITHUB_ENV.

### Iteration 3

**Fixes applied:** script-injection

**Notes:**

Replaced all 9 '# shellcheck disable=SC2206' comments with '# shellcheck disable=SC2086' above the unquoted array assignments (sushi_args, conformance_paths, example_paths, java_options, ig_dependency_args). Variables left unquoted to keep upstream's whitespace split and glob behaviour.

