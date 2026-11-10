<!-- markdownlint-disable -->

# Hardening Report: FirelyTeam--firely-terminal-pipeline/v0.8.9

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **FirelyTeam--firely-terminal-pipeline/v0.8.9** was hardened automatically. 23 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Step 'Download Java Validator', rule (b) plus eval: the input JAVA_VALIDATOR_DOWNLOAD_LOCATION goes through eval in `JAVA_VALIDATOR_DOWNLOAD_LOCATION=$(eval echo "$JAVA_VALIDATOR_DOWNLOAD_LOCATION")`, so the shell runs any $(...), backticks or ';' in the input. The result is also used unquoted in `wget -q $JAVA_VALIDATOR_DOWNLOAD_LOCATION -O validator_cli.jar`, and `if $INPUT_JAVA_VALIDATION_ENABLED; then` runs the input's value as a command. Fix: drop eval and replace only the literal `$JAVA_VALIDATOR_VERSION` placeholder with bash parameter substitution, quote all expansions, and compare booleans with `[[ "$VAR" == "true" ]]`.

Locations:

- `action.yml:679`
- `action.yml:683`
- `action.yml:686`

### script-injection (severity: high)

Step 'Check if .NET is installed', rule (b): `if $INPUT_DOTNET_VALIDATION_ENABLED || $INPUT_TERMINOLOGY_SERVICE_BFARM_ENABLED; then` expands inputs without quotes and runs their values as commands.

Locations:

- `action.yml:90`

### script-injection (severity: high)

Step 'Check .NET SDK Version', rule (b): `if $INPUT_DOTNET_VALIDATION_ENABLED; then` expands the input without quotes.

Locations:

- `action.yml:105`

### script-injection (severity: high)

Step 'Install Firely.Terminal', rule (b): `if $INPUT_DOTNET_VALIDATION_ENABLED || $INPUT_TERMINOLOGY_SERVICE_BFARM_ENABLED; then` and `dotnet tool install --global Firely.Terminal --version $FIRELY_TERMINAL_VERSION` expand inputs without quotes.

Locations:

- `action.yml:116`
- `action.yml:119`

### script-injection (severity: high)

Step 'Check Firely Terminal Version', rule (b): `if $INPUT_DOTNET_VALIDATION_ENABLED; then` expands the input without quotes.

Locations:

- `action.yml:131`

### script-injection (severity: high)

Step 'Simplifier login', rule (b): `if $INPUT_DOTNET_VALIDATION_ENABLED; then` and `fhir login email=$INPUT_SIMPLIFIER_USERNAME password=$INPUT_SIMPLIFIER_PASSWORD` expand inputs without quotes.

Locations:

- `action.yml:142`
- `action.yml:143`

### script-injection (severity: high)

Step 'Detect FHIR version and determine dependency source', rule (b): `if $INPUT_DOTNET_VALIDATION_ENABLED || $INPUT_JAVA_VALIDATION_ENABLED; then`, `if $INPUT_SUSHI_USE_CONFIG_DEPENDENCIES; then` and `elif $INPUT_SUSHI_ENABLED && ...` expand inputs without quotes.

Locations:

- `action.yml:168`
- `action.yml:183`
- `action.yml:202`

### script-injection (severity: high)

Step 'Validate FHIR version support', rule (b): `if $INPUT_DOTNET_VALIDATION_ENABLED || $INPUT_JAVA_VALIDATION_ENABLED; then` expands inputs without quotes.

Locations:

- `action.yml:285`

### script-injection (severity: high)

Step 'Set FHIR specification and restore dependencies', rule (b): `if $INPUT_DOTNET_VALIDATION_ENABLED; then` expands the input without quotes.

Locations:

- `action.yml:363`

### script-injection (severity: high)

Step 'Check if npm is installed', rule (b): `if $INPUT_SUSHI_ENABLED; then` expands the input without quotes.

Locations:

- `action.yml:487`

### script-injection (severity: high)

Step 'Check npm Version', rule (b): `if $INPUT_SUSHI_ENABLED; then` expands the input without quotes.

Locations:

- `action.yml:500`

### script-injection (severity: high)

Step 'Install fsh-sushi', rule (b): `if $INPUT_SUSHI_ENABLED; then`, `sudo chown -R $USER /usr/local/lib/node_modules` (USER is inherited env) and `sudo npm install -g fsh-sushi@$SUSHI_VERSION` expand values without quotes.

Locations:

- `action.yml:510`
- `action.yml:511`
- `action.yml:512`

### script-injection (severity: high)

Step 'Check SUSHI version', rule (b): `if $INPUT_SUSHI_ENABLED; then` expands the input without quotes.

Locations:

- `action.yml:521`

### script-injection (severity: high)

Step 'Generate conformance resources with SUSHI', rule (b): `if $INPUT_SUSHI_ENABLED; then` and `sushi $INPUT_SUSHI_OPTIONS` expand inputs without quotes.

Locations:

- `action.yml:530`
- `action.yml:531`

### script-injection (severity: high)

Step 'Pre-processing FHIR profiles (Slicing open-closed)', rule (b): `if $CLOSE_SLICING_FOR_VALIDATION; then` expands the input without quotes.

Locations:

- `action.yml:544`

### script-injection (severity: high)

Step 'Configure Firely Terminal validator engine', rule (b): `if $INPUT_DOTNET_VALIDATION_ENABLED; then` expands the input without quotes.

Locations:

- `action.yml:599`

### script-injection (severity: high)

Step 'Run Quality Control checks', rule (b): `if $INPUT_DOTNET_VALIDATION_ENABLED; then`, `if echo $INPUT_EXPECTED_FAILS | grep ...`, `fhir check $INPUT_PATH_TO_QUALITY_CONTROL_RULES` and `fhir check --fail $INPUT_PATH_TO_QUALITY_CONTROL_RULES` expand inputs without quotes.

Locations:

- `action.yml:623`
- `action.yml:625`
- `action.yml:626`
- `action.yml:628`

### script-injection (severity: high)

Step 'Report Success - .NET Validator', rule (b): `if $INPUT_DOTNET_VALIDATION_ENABLED; then` expands the input without quotes.

Locations:

- `action.yml:639`

### script-injection (severity: high)

Step 'Remove package cache to enable the Java validator to create snapshots', rule (b): `if $INPUT_JAVA_SNAPSHOT_ENABLED; then` expands the input without quotes.

Locations:

- `action.yml:656`

### script-injection (severity: high)

Step 'Check if Java is installed', rule (b): `if $INPUT_JAVA_VALIDATION_ENABLED; then` expands the input without quotes.

Locations:

- `action.yml:666`

### script-injection (severity: high)

Step 'Validate all conformance resources in scope of the repository', rule (b): `if $INPUT_JAVA_VALIDATION_ENABLED; then`, `for p in $INPUT_PATH_TO_CONFORMANCE_RESOURCES;`, `if echo $INPUT_EXPECTED_FAILS | grep ...` and `java -jar validator_cli.jar ... -version $FHIR_VERSION $INPUT_JAVA_VALIDATION_OPTIONS ...` expand workflow-controlled values without quotes.

Locations:

- `action.yml:704`
- `action.yml:727`
- `action.yml:732`
- `action.yml:742`
- `action.yml:743`
- `action.yml:745`

### script-injection (severity: high)

Step 'Validate all example resources in scope of the repository', rule (b): `if $INPUT_JAVA_VALIDATION_ENABLED; then`, `for p in $INPUT_PATH_TO_CONFORMANCE_RESOURCES;`, `for p in $PATH_TO_EXAMPLES;`, `if echo $INPUT_EXPECTED_FAILS | grep ...` and `java -jar validator_cli.jar ... -version $FHIR_VERSION $INPUT_JAVA_VALIDATION_OPTIONS ...` expand workflow-controlled values without quotes.

Locations:

- `action.yml:778`
- `action.yml:795`
- `action.yml:800`
- `action.yml:810`
- `action.yml:811`
- `action.yml:813`

### script-injection (severity: high)

Step 'Re-run SUSHI to restore conformance resources after slicing modification', rule (b): `if $CLOSE_SLICING_FOR_VALIDATION && $INPUT_SUSHI_ENABLED; then` and `sushi $INPUT_SUSHI_OPTIONS` expand inputs without quotes.

Locations:

- `action.yml:841`
- `action.yml:843`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection

**Notes:**

All changes are in action.yml. Nothing has been run or tested.

- **True/false inputs:** every `if $INPUT_X; then` check (DOTNET/JAVA/SUSHI/BFARM/JAVA_SNAPSHOT/CLOSE_SLICING/SUSHI_USE_CONFIG_DEPENDENCIES) is now `[[ "$X" == "true" ]]`, including the combined `||`/`&&` checks. Upstream ran the input's value as a command; now only the exact lowercase string "true" turns a step on. Something like "True" counts as off.
- **Download Java Validator:** removed the `eval`. Only the literal `$JAVA_VALIDATOR_VERSION` / `${JAVA_VALIDATOR_VERSION}` placeholder is swapped out, using bash substitution with quoted pattern and replacement. The URL passed to wget is quoted.
- **Single values are quoted:** the Firely.Terminal `--version`, `fsh-sushi@$SUSHI_VERSION`, `$USER` in `chown`, the `fhir login` email/password arguments, `-version "$FHIR_VERSION"` and `rm -rf "$fhirCacheLocation"`. The optional PATH_TO_QUALITY_CONTROL_RULES uses `${VAR:+"$VAR"}`, so it drops out entirely when empty.
- **EXPECTED_FAILS:** checked with `printf '%s\n' "$INPUT_EXPECTED_FAILS" | grep -w -q ...`.
- **List inputs:** SUSHI_OPTIONS, JAVA_VALIDATION_OPTIONS, PATH_TO_CONFORMANCE_RESOURCES and PATH_TO_EXAMPLES go into unquoted arrays (`arr=($VAR)`), which split on whitespace and expand globs exactly like upstream's unquoted use. The arrays are then expanded as `"${arr[@]}"`.
- **Values built inside the script:** the IG dependency list (read from package.json) also goes into an array. The `-ig <path>` arguments are now built as an array, so a path containing spaces stays one argument instead of being split as it was upstream.
- **File globs:** the `*.xml` / `*.json` patterns are kept as `"$GITHUB_WORKSPACE/$p"*.xml`, so the globs still expand.

### Iteration 2

**Fixes applied:** github-env-injection

**Notes:**

In action.yml, the 'Detect FHIR version' step now strips \n and \r from fhirVersion (printf '%s' | tr -d '\n\r') before it writes FHIR_VERSION to $GITHUB_ENV. Earlier steps and log output still use the original value.

