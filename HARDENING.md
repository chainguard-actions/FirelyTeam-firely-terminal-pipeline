<!-- markdownlint-disable -->

# Hardening Report: FirelyTeam--firely-terminal-pipeline/v0.8.10

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **FirelyTeam--firely-terminal-pipeline/v0.8.10** was hardened automatically. 16 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (b), plus evaluation of an input, in step 'Download Java Validator'. The line `JAVA_VALIDATOR_DOWNLOAD_LOCATION=$(eval echo "$JAVA_VALIDATOR_DOWNLOAD_LOCATION")` evals inputs.JAVA_VALIDATOR_DOWNLOAD_LOCATION, so any $(...), backticks or ';' in the value run as commands. After that, `wget -q $JAVA_VALIDATOR_DOWNLOAD_LOCATION -O validator_cli.jar` expands it without quotes. The step also uses unquoted `if $INPUT_JAVA_VALIDATION_ENABLED; then`, which runs the input value as a command. Fix: drop eval and substitute only the literal $JAVA_VALIDATOR_VERSION placeholder, e.g. with "${JAVA_VALIDATOR_DOWNLOAD_LOCATION//\$JAVA_VALIDATOR_VERSION/$JAVA_VALIDATOR_VERSION}". Then quote every expansion.

Locations:

- `action.yml:712`
- `action.yml:716`

### script-injection (severity: high)

Rule (b) in step 'Simplifier login': `fhir login email=$INPUT_SIMPLIFIER_USERNAME password=$INPUT_SIMPLIFIER_PASSWORD`. Both input-derived values are expanded without quotes, and `if $INPUT_DOTNET_VALIDATION_ENABLED; then` runs the input as a command.

Locations:

- `action.yml:143`

### script-injection (severity: high)

Rule (b) in step 'Install Firely.Terminal': `dotnet tool install --global Firely.Terminal --version $FIRELY_TERMINAL_VERSION` (unquoted input) and `if $INPUT_DOTNET_VALIDATION_ENABLED || $INPUT_TERMINOLOGY_SERVICE_BFARM_ENABLED; then`, which runs input values as commands.

Locations:

- `action.yml:116`
- `action.yml:119`

### script-injection (severity: high)

Rule (b) in step 'Install fsh-sushi': `sudo npm install -g fsh-sushi@$SUSHI_VERSION` and `sudo chown -R $USER ...` are expanded without quotes, and `if $INPUT_SUSHI_ENABLED; then` runs the input as a command.

Locations:

- `action.yml:544`

### script-injection (severity: high)

Rule (b) in step 'Generate conformance resources with SUSHI': `sushi $INPUT_SUSHI_OPTIONS` (unquoted input, not the exempt array-assignment form) and `if $INPUT_SUSHI_ENABLED; then`.

Locations:

- `action.yml:555`

### script-injection (severity: high)

Rule (b) in step 'Re-run SUSHI to restore conformance resources after slicing modification': `if $CLOSE_SLICING_FOR_VALIDATION && $INPUT_SUSHI_ENABLED; then` runs input values as commands, and `sushi $INPUT_SUSHI_OPTIONS` is unquoted.

Locations:

- `action.yml:902`

### script-injection (severity: high)

Rule (b) in step 'Run Quality Control checks': `if echo $INPUT_EXPECTED_FAILS | grep ...`, `fhir check $INPUT_PATH_TO_QUALITY_CONTROL_RULES` and `fhir check --fail $INPUT_PATH_TO_QUALITY_CONTROL_RULES` are unquoted, and `if $INPUT_DOTNET_VALIDATION_ENABLED; then` runs the input as a command.

Locations:

- `action.yml:654`

### script-injection (severity: high)

Rule (b) in step 'Validate all conformance resources in scope of the repository'. Unquoted expansions: `for p in $INPUT_PATH_TO_CONFORMANCE_RESOURCES;`, `echo $INPUT_EXPECTED_FAILS | grep ...`, and `java -jar validator_cli.jar $GITHUB_WORKSPACE/$p*.xml ... -version $FHIR_VERSION $INPUT_JAVA_VALIDATION_OPTIONS ...`. The step also runs the input as a command with `if $INPUT_JAVA_VALIDATION_ENABLED; then`.

Locations:

- `action.yml:724`
- `action.yml:777`

### script-injection (severity: high)

Rule (b) in step 'Validate all example resources in scope of the repository'. Unquoted expansions: `for p in $INPUT_PATH_TO_CONFORMANCE_RESOURCES;`, `for p in $PATH_TO_EXAMPLES;`, `echo $INPUT_EXPECTED_FAILS | grep ...`, and `java -jar validator_cli.jar ... -version $FHIR_VERSION $INPUT_JAVA_VALIDATION_OPTIONS ...`. The step also has `if $INPUT_JAVA_VALIDATION_ENABLED; then`.

Locations:

- `action.yml:813`
- `action.yml:859`

### script-injection (severity: high)

Rule (b) in step 'Remove package cache to enable the Java validator to create snapshots': `if $INPUT_JAVA_SNAPSHOT_ENABLED; then` runs the input as a command, and `rm -rf $fhirCacheLocation` is unquoted.

Locations:

- `action.yml:685`
- `action.yml:690`

### script-injection (severity: high)

Rule (b) in step 'Pre-processing FHIR profiles (Slicing open-closed)': `if $CLOSE_SLICING_FOR_VALIDATION; then` expands inputs.CLOSE_SLICING_FOR_VALIDATION without quotes and runs it as a command.

Locations:

- `action.yml:567`

### script-injection (severity: high)

Rule (b) in step 'Detect FHIR version and determine dependency source'. Three boolean inputs are expanded without quotes and run as commands: `if $INPUT_DOTNET_VALIDATION_ENABLED || $INPUT_JAVA_VALIDATION_ENABLED; then`, `if $INPUT_SUSHI_USE_CONFIG_DEPENDENCIES; then` and `elif $INPUT_SUSHI_ENABLED && ...`. Compare them as strings instead, e.g. [[ "$VAR" == "true" ]].

Locations:

- `action.yml:168`

### script-injection (severity: high)

Rule (b) in step 'Check if .NET is installed': `if $INPUT_DOTNET_VALIDATION_ENABLED || $INPUT_TERMINOLOGY_SERVICE_BFARM_ENABLED; then` runs the input values as commands without quotes.

Locations:

- `action.yml:90`

### script-injection (severity: high)

Rule (b): these steps run the input as a command with an unquoted `if $INPUT_DOTNET_VALIDATION_ENABLED; then`: 'Check .NET SDK Version', 'Check Firely Terminal Version', 'Set FHIR specification and restore dependencies', 'Configure Firely Terminal validator engine' and 'Report Success - .NET Validator'. 'Set FHIR specification...' also has an unquoted `echo $FHIR_RESTORE`.

Locations:

- `action.yml:105`
- `action.yml:131`
- `action.yml:386`
- `action.yml:625`
- `action.yml:668`

### script-injection (severity: high)

Rule (b) in step 'Validate FHIR version support': `if $INPUT_DOTNET_VALIDATION_ENABLED || $INPUT_JAVA_VALIDATION_ENABLED; then` runs input values as commands without quotes.

Locations:

- `action.yml:307`

### script-injection (severity: high)

Rule (b): steps 'Check if npm is installed', 'Check npm Version' and 'Check SUSHI version' run the input as a command with an unquoted `if $INPUT_SUSHI_ENABLED; then`. Step 'Check if Java is installed' does the same with `if $INPUT_JAVA_VALIDATION_ENABLED; then`.

Locations:

- `action.yml:526`
- `action.yml:535`
- `action.yml:550`
- `action.yml:696`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection

**Notes:**

All the script-injection findings are fixed in action.yml. Nothing has been run yet: the changes are untested.

- **Boolean inputs:** every `if $INPUT_X; then` (and the `||`/`&&`/`elif` forms) is now `[[ "$X" == "true" ]]`. Upstream ran the value as a command, so only `true` and `false` ever worked; `true` still runs the step and anything else still skips it.
- **Download Java Validator:** the `eval` is gone. The step now replaces only the literal `$JAVA_VALIDATOR_VERSION` or `${JAVA_VALIDATOR_VERSION}` in the URL with the version, and passes the URL to `wget` in quotes. Side effect: any other `$VAR` in a custom URL is no longer expanded, because that expansion was the eval.
- **Quoted single values:** the Firely Terminal version, the `fhir login` email and password arguments, `$USER`, `fsh-sushi@$SUSHI_VERSION`, `echo "$FHIR_RESTORE"`, `rm -rf "$fhirCacheLocation"` and the FHIR version passed to `-version`.
- **Quality-control rules path:** this optional value is passed as `${VAR:+"$VAR"}`, so it drops out entirely when empty, as before.
- **EXPECTED_FAILS:** checked with `printf '%s\n' "$INPUT_EXPECTED_FAILS" | grep -w -q ...`, which gives the same match result as before.
- **List inputs (SUSHI_OPTIONS, JAVA_VALIDATION_OPTIONS, PATH_TO_CONFORMANCE_RESOURCES, PATH_TO_EXAMPLES):** each is assigned to a bash array unquoted (`arr=($VAR)`), which splits on whitespace and expands wildcards exactly like upstream's unquoted use, then passed as `"${arr[@]}"`.
- **IG dependency list:** built from package.json and split the same way.
- **Local `-ig <path>` arguments:** now built as arrays, so each path stays one argument instead of one long string that gets re-split.
- **Java file globs:** written as `"$GITHUB_WORKSPACE/$p"*.xml` / `*.json`, so the path part is quoted and the wildcard still works.

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

In action.yml, the 'Detect FHIR version' step now strips \n and \r from fhirVersion using printf '%s' | tr -d '\n\r' before it writes FHIR_VERSION to $GITHUB_ENV.

