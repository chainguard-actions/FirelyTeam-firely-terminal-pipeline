<!-- markdownlint-disable -->

# Hardening Report: FirelyTeam--firely-terminal-pipeline/v0.9.3

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **FirelyTeam--firely-terminal-pipeline/v0.9.3** was hardened automatically. 23 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (b) in step 'Check if .NET is installed': the input-derived variables are expanded unquoted and run as a command: `if $INPUT_DOTNET_VALIDATION_ENABLED || $INPUT_TERMINOLOGY_SERVICE_BFARM_ENABLED; then`. The value of the inputs DOTNET_VALIDATION_ENABLED and TERMINOLOGY_SERVICE_BFARM_ENABLED is executed as a shell command. Fix: compare against a literal, e.g. `[[ "$INPUT_DOTNET_VALIDATION_ENABLED" == "true" ]]`.

Locations:

- `action.yml:90`

### script-injection (severity: high)

Rule (b) in step 'Check .NET SDK Version': `if $INPUT_DOTNET_VALIDATION_ENABLED; then` runs the unquoted input value as a command.

Locations:

- `action.yml:105`

### script-injection (severity: high)

Rule (b) in step 'Install Firely.Terminal': `if $INPUT_DOTNET_VALIDATION_ENABLED || $INPUT_TERMINOLOGY_SERVICE_BFARM_ENABLED; then` runs input values as commands. `dotnet tool install --global Firely.Terminal --version $FIRELY_TERMINAL_VERSION` expands an input unquoted.

Locations:

- `action.yml:116`
- `action.yml:119`

### script-injection (severity: high)

Rule (b) in step 'Check Firely Terminal Version': `if $INPUT_DOTNET_VALIDATION_ENABLED; then` runs the unquoted input value as a command.

Locations:

- `action.yml:131`

### script-injection (severity: high)

Rule (b) in step 'Simplifier login': `if $INPUT_DOTNET_VALIDATION_ENABLED; then` runs an input value as a command. `fhir login email=$INPUT_SIMPLIFIER_USERNAME password=$INPUT_SIMPLIFIER_PASSWORD` expands the credential inputs unquoted.

Locations:

- `action.yml:142`
- `action.yml:143`

### script-injection (severity: high)

Rule (b) in step 'Detect FHIR version and determine dependency source': `if $INPUT_DOTNET_VALIDATION_ENABLED || $INPUT_JAVA_VALIDATION_ENABLED; then`, `if $INPUT_SUSHI_USE_CONFIG_DEPENDENCIES; then` and `elif $INPUT_SUSHI_ENABLED && ...` all run unquoted input values as commands.

Locations:

- `action.yml:168`
- `action.yml:183`

### script-injection (severity: high)

Rule (b) in step 'Validate FHIR version support': `if $INPUT_DOTNET_VALIDATION_ENABLED || $INPUT_JAVA_VALIDATION_ENABLED; then` runs unquoted input values as commands.

Locations:

- `action.yml:285`

### script-injection (severity: high)

Rule (b) in step 'Set FHIR specification and restore dependencies': `if $INPUT_DOTNET_VALIDATION_ENABLED; then` runs the unquoted input value as a command.

Locations:

- `action.yml:362`

### script-injection (severity: high)

Rule (b) in step 'Check if npm is installed': `if $INPUT_SUSHI_ENABLED; then` runs the unquoted input value as a command.

Locations:

- `action.yml:492`

### script-injection (severity: high)

Rule (b) in step 'Check npm Version': `if $INPUT_SUSHI_ENABLED; then` runs the unquoted input value as a command.

Locations:

- `action.yml:505`

### script-injection (severity: high)

Rule (b) in step 'Install fsh-sushi': `if $INPUT_SUSHI_ENABLED; then` runs an input value as a command. `sudo npm install -g fsh-sushi@$SUSHI_VERSION` expands the SUSHI_VERSION input unquoted.

Locations:

- `action.yml:515`
- `action.yml:517`

### script-injection (severity: high)

Rule (b) in step 'Check SUSHI version': `if $INPUT_SUSHI_ENABLED; then` runs the unquoted input value as a command.

Locations:

- `action.yml:526`

### script-injection (severity: high)

Rule (b) in step 'Generate conformance resources with SUSHI': `if $INPUT_SUSHI_ENABLED; then` runs an input value as a command. `sushi $INPUT_SUSHI_OPTIONS` expands the input unquoted (no array assignment with a shellcheck SC2086 disable).

Locations:

- `action.yml:535`
- `action.yml:536`

### script-injection (severity: high)

Rule (b) in step 'Pre-processing FHIR profiles (Slicing open-closed)': `if $CLOSE_SLICING_FOR_VALIDATION; then` runs the unquoted CLOSE_SLICING_FOR_VALIDATION input as a command.

Locations:

- `action.yml:548`

### script-injection (severity: high)

Rule (b) in step 'Configure Firely Terminal validator engine': `if $INPUT_DOTNET_VALIDATION_ENABLED; then` runs the unquoted input value as a command.

Locations:

- `action.yml:603`

### script-injection (severity: high)

Rule (b) in step 'Run Quality Control checks': `if $INPUT_DOTNET_VALIDATION_ENABLED; then` runs an input value as a command. `echo $INPUT_EXPECTED_FAILS | grep ...` and `fhir check [--fail] $INPUT_PATH_TO_QUALITY_CONTROL_RULES` expand inputs unquoted.

Locations:

- `action.yml:626`
- `action.yml:628`
- `action.yml:629`
- `action.yml:631`

### script-injection (severity: high)

Rule (b) in step 'Report Success - .NET Validator': `if $INPUT_DOTNET_VALIDATION_ENABLED; then` runs the unquoted input value as a command.

Locations:

- `action.yml:642`

### script-injection (severity: high)

Rule (b) in step 'Remove package cache to enable the Java validator to create snapshots': `if $INPUT_JAVA_SNAPSHOT_ENABLED; then` runs the unquoted input value as a command.

Locations:

- `action.yml:659`

### script-injection (severity: high)

Rule (b) in step 'Check if Java is installed': `if $INPUT_JAVA_VALIDATION_ENABLED; then` runs the unquoted input value as a command.

Locations:

- `action.yml:669`

### script-injection (severity: high)

Rule (b) in step 'Download Java Validator': `if $INPUT_JAVA_VALIDATION_ENABLED; then` runs an input value as a command. `JAVA_VALIDATOR_DOWNLOAD_LOCATION=$(eval echo "$JAVA_VALIDATOR_DOWNLOAD_LOCATION")` passes the JAVA_VALIDATOR_DOWNLOAD_LOCATION input to eval, so any `$(...)`, `;` or backticks in it run as commands. `wget -q $JAVA_VALIDATOR_DOWNLOAD_LOCATION -O validator_cli.jar` expands it unquoted. Fix: expand only the version placeholder with a fixed string replacement (e.g. `${JAVA_VALIDATOR_DOWNLOAD_LOCATION//\$JAVA_VALIDATOR_VERSION/$JAVA_VALIDATOR_VERSION}`) and quote the URL.

Locations:

- `action.yml:682`
- `action.yml:686`
- `action.yml:689`

### script-injection (severity: high)

Rule (b) in step 'Validate all conformance resources in scope of the repository': `if $INPUT_JAVA_VALIDATION_ENABLED; then` runs an input value as a command. Inputs are also expanded unquoted in `for p in $INPUT_PATH_TO_CONFORMANCE_RESOURCES;`, `echo $INPUT_EXPECTED_FAILS | grep ...` and `java -jar validator_cli.jar ... -version $FHIR_VERSION $INPUT_JAVA_VALIDATION_OPTIONS ...`.

Locations:

- `action.yml:707`
- `action.yml:730`
- `action.yml:745`
- `action.yml:746`

### script-injection (severity: high)

Rule (b) in step 'Validate all example resources in scope of the repository': `if $INPUT_JAVA_VALIDATION_ENABLED; then` runs an input value as a command. Inputs are also expanded unquoted in `for p in $INPUT_PATH_TO_CONFORMANCE_RESOURCES;`, `for p in $PATH_TO_EXAMPLES;`, `echo $INPUT_EXPECTED_FAILS | grep ...` and `java -jar validator_cli.jar ... $INPUT_JAVA_VALIDATION_OPTIONS ...`.

Locations:

- `action.yml:779`
- `action.yml:796`
- `action.yml:801`
- `action.yml:811`
- `action.yml:812`

### script-injection (severity: high)

Rule (b) in step 'Re-run SUSHI to restore conformance resources after slicing modification': `if $CLOSE_SLICING_FOR_VALIDATION && $INPUT_SUSHI_ENABLED; then` runs input values as commands. `sushi $INPUT_SUSHI_OPTIONS` expands an input unquoted.

Locations:

- `action.yml:841`
- `action.yml:843`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection

**Notes:**

Fixed all 23 script-injection findings in action.yml. Nothing was run, so the changes are untested.

- **Boolean inputs:** every `if $INPUT_X; then` (and the `||`/`&&` forms, `elif`, `CLOSE_SLICING_FOR_VALIDATION`, `JAVA_SNAPSHOT_ENABLED`) is now `[[ "$X" == "true" ]]`. Upstream only ever acted on the exact value "true" ("false" ran `false`, anything else failed as an unknown command). Now any value other than "true" quietly skips the step instead of failing it.
- **Java validator download:** the `eval echo` on JAVA_VALIDATOR_DOWNLOAD_LOCATION is gone. Instead, the literal text `$JAVA_VALIDATOR_VERSION` (or `${JAVA_VALIDATOR_VERSION}`) in the URL is replaced with the version, and the URL passed to `wget` is quoted. A custom URL that uses any other `$` variable would not be expanded any more.
- **Single values now quoted:** `--version "$FIRELY_TERMINAL_VERSION"`, `"fsh-sushi@$SUSHI_VERSION"`, `"$USER"`, `fhir login "email=..." "password=..."`, `"$fhirCacheLocation"` and `-version "$FHIR_VERSION"`. The optional quality-control rules path uses `${VAR:+"$VAR"}`, so an empty value passes no argument. EXPECTED_FAILS goes to grep via `printf '%s\n' "$INPUT_EXPECTED_FAILS"`.
- **List inputs:** SUSHI_OPTIONS, JAVA_VALIDATION_OPTIONS, PATH_TO_CONFORMANCE_RESOURCES and PATH_TO_EXAMPLES are split into arrays with an unquoted assignment (`arr=($VAR)`, shellcheck SC2086 disabled). This splits on whitespace and expands globs exactly as upstream's unquoted use did, and each array is expanded quoted. The `-ig` workspace path arguments are now built as an array too. The `*.xml`/`*.json` wildcards still work because only the directory part before them is quoted.
- **Left as upstream had it:** the dependency list built from package.json (`UNESCPAED_IG_DEPENDENCIES`) is still unquoted, because it comes from the repository's file rather than an input and has to split into separate `-ig` arguments.

### Iteration 2

**Fixes applied:** github-env-injection

**Notes:**

In the 'Detect FHIR version' step of action.yml, fhirVersion now has CR/LF stripped (printf '%s' | tr -d '\n\r') before it is written to $GITHUB_ENV, so a crafted value can't add extra env entries. I didn't add a version-pattern check because the next step already checks the version against the list of supported versions.

