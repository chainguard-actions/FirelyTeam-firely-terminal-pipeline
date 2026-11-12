<!-- markdownlint-disable -->

# Hardening Report: FirelyTeam--firely-terminal-pipeline/v0.10.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **FirelyTeam--firely-terminal-pipeline/v0.10.0** was hardened automatically. 23 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

(b) 'Check if .NET is installed': `if $INPUT_DOTNET_VALIDATION_ENABLED || $INPUT_TERMINOLOGY_SERVICE_BFARM_ENABLED; then`. Inputs are expanded without quotes and run as commands, so an input value such as 'true; malicious' executes arbitrary code. Fix: compare against a literal, e.g. `[[ "$VAR" == "true" ]]`.

Locations:

- `action.yml:90`

### script-injection (severity: high)

(b) 'Check .NET SDK Version': `if $INPUT_DOTNET_VALIDATION_ENABLED; then`. The input is expanded without quotes and run as a command.

Locations:

- `action.yml:105`

### script-injection (severity: high)

(b) 'Install Firely.Terminal': `if $INPUT_DOTNET_VALIDATION_ENABLED || $INPUT_TERMINOLOGY_SERVICE_BFARM_ENABLED; then` and `dotnet tool install --global Firely.Terminal --version $FIRELY_TERMINAL_VERSION`. The input-derived vars are not quoted.

Locations:

- `action.yml:116`
- `action.yml:119`

### script-injection (severity: high)

(b) 'Check Firely Terminal Version': `if $INPUT_DOTNET_VALIDATION_ENABLED; then`. The input is not quoted.

Locations:

- `action.yml:131`

### script-injection (severity: high)

(b) 'Simplifier login': `if $INPUT_DOTNET_VALIDATION_ENABLED; then` and `fhir login email=$INPUT_SIMPLIFIER_USERNAME password=$INPUT_SIMPLIFIER_PASSWORD`. The input-derived vars are not quoted.

Locations:

- `action.yml:142`
- `action.yml:143`

### script-injection (severity: high)

(b) 'Detect FHIR version and determine dependency source': `if $INPUT_DOTNET_VALIDATION_ENABLED || $INPUT_JAVA_VALIDATION_ENABLED; then`, `if $INPUT_SUSHI_USE_CONFIG_DEPENDENCIES; then` and `elif $INPUT_SUSHI_ENABLED && ...`. The inputs are not quoted and run as commands.

Locations:

- `action.yml:168`
- `action.yml:183`
- `action.yml:202`

### script-injection (severity: high)

(b) 'Validate FHIR version support': `if $INPUT_DOTNET_VALIDATION_ENABLED || $INPUT_JAVA_VALIDATION_ENABLED; then`. The inputs are not quoted.

Locations:

- `action.yml:285`

### script-injection (severity: high)

(b) 'Set FHIR specification and restore dependencies': `if $INPUT_DOTNET_VALIDATION_ENABLED; then`. The input is not quoted.

Locations:

- `action.yml:363`

### script-injection (severity: high)

(b) 'Check if npm is installed': `if $INPUT_SUSHI_ENABLED; then`. The input is not quoted.

Locations:

- `action.yml:503`

### script-injection (severity: high)

(b) 'Check npm Version': `if $INPUT_SUSHI_ENABLED; then`. The input is not quoted.

Locations:

- `action.yml:516`

### script-injection (severity: high)

(b) 'Install fsh-sushi': `if $INPUT_SUSHI_ENABLED; then` and `sudo npm install -g fsh-sushi@$SUSHI_VERSION`. The input-derived vars are not quoted.

Locations:

- `action.yml:526`
- `action.yml:528`

### script-injection (severity: high)

(b) 'Check SUSHI version': `if $INPUT_SUSHI_ENABLED; then`. The input is not quoted.

Locations:

- `action.yml:537`

### script-injection (severity: high)

(b) 'Generate conformance resources with SUSHI': `if $INPUT_SUSHI_ENABLED; then` and `sushi $INPUT_SUSHI_OPTIONS`. The input-derived vars are not quoted.

Locations:

- `action.yml:546`
- `action.yml:547`

### script-injection (severity: high)

(b) 'Pre-processing FHIR profiles (Slicing open-closed)': `if $CLOSE_SLICING_FOR_VALIDATION; then`. The input is not quoted and runs as a command.

Locations:

- `action.yml:560`

### script-injection (severity: high)

(b) 'Configure Firely Terminal validator engine': `if $INPUT_DOTNET_VALIDATION_ENABLED; then`. The input is not quoted.

Locations:

- `action.yml:615`

### script-injection (severity: high)

(b) 'Run Quality Control checks': `if $INPUT_DOTNET_VALIDATION_ENABLED; then`, `if echo $INPUT_EXPECTED_FAILS | grep ...` and `fhir check $INPUT_PATH_TO_QUALITY_CONTROL_RULES` / `fhir check --fail $INPUT_PATH_TO_QUALITY_CONTROL_RULES`. The input-derived vars are not quoted.

Locations:

- `action.yml:639`
- `action.yml:641`
- `action.yml:642`
- `action.yml:644`

### script-injection (severity: high)

(b) 'Report Success - .NET Validator': `if $INPUT_DOTNET_VALIDATION_ENABLED; then`. The input is not quoted.

Locations:

- `action.yml:655`

### script-injection (severity: high)

(b) 'Remove package cache to enable the Java validator to create snapshots': `if $INPUT_JAVA_SNAPSHOT_ENABLED; then`. The input is not quoted.

Locations:

- `action.yml:672`

### script-injection (severity: high)

(b) 'Check if Java is installed': `if $INPUT_JAVA_VALIDATION_ENABLED; then`. The input is not quoted.

Locations:

- `action.yml:682`

### script-injection (severity: high)

(b) 'Download Java Validator': `if $INPUT_JAVA_VALIDATION_ENABLED; then`, `JAVA_VALIDATOR_DOWNLOAD_LOCATION=$(eval echo "$JAVA_VALIDATOR_DOWNLOAD_LOCATION")` and `wget -q $JAVA_VALIDATOR_DOWNLOAD_LOCATION -O validator_cli.jar`. The eval runs the inputs.JAVA_VALIDATOR_DOWNLOAD_LOCATION value through the shell, so any $(...), backticks or ';' in it execute. The wget argument is also unquoted. Fix: replace the eval with a literal substitution, e.g. `${JAVA_VALIDATOR_DOWNLOAD_LOCATION//\$JAVA_VALIDATOR_VERSION/$JAVA_VALIDATOR_VERSION}`, and quote the variable.

Locations:

- `action.yml:695`
- `action.yml:699`
- `action.yml:702`

### script-injection (severity: high)

(b) 'Validate all conformance resources in scope of the repository': `if $INPUT_JAVA_VALIDATION_ENABLED; then`, `for p in $INPUT_PATH_TO_CONFORMANCE_RESOURCES;`, `if echo $INPUT_EXPECTED_FAILS | grep ...` and `java -jar validator_cli.jar ... -version $FHIR_VERSION $INPUT_JAVA_VALIDATION_OPTIONS ...`. The input-derived vars are not quoted.

Locations:

- `action.yml:720`
- `action.yml:736`
- `action.yml:744`
- `action.yml:759`
- `action.yml:760`

### script-injection (severity: high)

(b) 'Validate all example resources in scope of the repository': `if $INPUT_JAVA_VALIDATION_ENABLED; then`, `for p in $PATH_TO_EXAMPLES;`, `if echo $INPUT_EXPECTED_FAILS | grep ...` and `java -jar validator_cli.jar ... $FHIR_VERSION $INPUT_JAVA_VALIDATION_OPTIONS ...`. The input-derived vars are not quoted.

Locations:

- `action.yml:795`
- `action.yml:818`
- `action.yml:826`
- `action.yml:827`

### script-injection (severity: high)

(b) 'Re-run SUSHI to restore conformance resources after slicing modification': `if $CLOSE_SLICING_FOR_VALIDATION && $INPUT_SUSHI_ENABLED; then` and `sushi $INPUT_SUSHI_OPTIONS`. The input-derived vars are not quoted.

Locations:

- `action.yml:858`
- `action.yml:860`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection

**Notes:**

All changes are in action.yml. Nothing has been run or tested yet.

- **Boolean inputs:** steps no longer run these inputs as commands. Each `if $X;` (alone, or joined with `||` / `&&`) is now `[[ "$X" == "true" ]]`. This covers DOTNET/JAVA/SUSHI_ENABLED, TERMINOLOGY_SERVICE_BFARM_ENABLED, SUSHI_USE_CONFIG_DEPENDENCIES, JAVA_SNAPSHOT_ENABLED and CLOSE_SLICING_FOR_VALIDATION. One behaviour change: a step now runs only for the exact string 'true'. Upstream also ran it for an explicitly empty value, which counted as true when run as a command.
- **Download URL:** `eval echo` on JAVA_VALIDATOR_DOWNLOAD_LOCATION is gone. A plain text substitution now swaps `$JAVA_VALIDATOR_VERSION` (and `${JAVA_VALIDATOR_VERSION}`) for the version, so the default URL still resolves. The wget URL is quoted.
- **Single values are quoted:** Firely Terminal version, `fsh-sushi@version`, the Simplifier email and password, `$USER`, the cache path given to `rm -rf`, and `-version "$FHIR_VERSION"`. The optional quality-control rules path uses `${VAR:+"$VAR"}`, so it drops out when empty.
- **EXPECTED_FAILS check:** `echo $INPUT_EXPECTED_FAILS | grep -w` is now `printf '%s\n' "$INPUT_EXPECTED_FAILS" | grep -w`.
- **List inputs keep upstream's splitting:** SUSHI_OPTIONS, JAVA_VALIDATION_OPTIONS, PATH_TO_CONFORMANCE_RESOURCES, PATH_TO_EXAMPLES and the IG dependency list are now unquoted array assignments (`arr=($VAR)`). These split on whitespace and expand globs exactly as upstream's unquoted use did, then expand as `"${arr[@]}"`.
- **IG parameter lists:** the `-ig <path>` lists are built as arrays instead of strings.
- **Validator file globs:** these are written as `"$GITHUB_WORKSPACE/$p"*.xml` and `*.json`, so `*` still expands but the path part does not split.

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

In action.yml, the detected FHIR version is now cleaned with printf '%s' | tr -d '\n\r' before it is written to $GITHUB_ENV as FHIR_VERSION. Its value comes from package.json or sushi-config.yaml, which a pull request can change. The step's other $GITHUB_ENV writes only hold true/false values the action sets itself, so they were left alone.

