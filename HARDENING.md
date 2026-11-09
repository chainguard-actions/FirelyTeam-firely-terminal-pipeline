<!-- markdownlint-disable -->

# Hardening Report: FirelyTeam--firely-terminal-pipeline/v0.8.7

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **FirelyTeam--firely-terminal-pipeline/v0.8.7** was hardened automatically. 23 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (b): in 'Check if .NET is installed', `if $INPUT_DOTNET_VALIDATION_ENABLED || $INPUT_TERMINOLOGY_SERVICE_BFARM_ENABLED; then` expands inputs without quotes and runs them as commands. Any input value is run as a shell command.

Locations:

- `action.yml:90`

### script-injection (severity: high)

Rule (b): in 'Check .NET SDK Version', `if $INPUT_DOTNET_VALIDATION_ENABLED; then` runs an input without quotes as a command.

Locations:

- `action.yml:105`

### script-injection (severity: high)

Rule (b): in 'Install Firely.Terminal', `if $INPUT_DOTNET_VALIDATION_ENABLED || $INPUT_TERMINOLOGY_SERVICE_BFARM_ENABLED` and `dotnet tool install --global Firely.Terminal --version $FIRELY_TERMINAL_VERSION` expand inputs without quotes.

Locations:

- `action.yml:116`

### script-injection (severity: high)

Rule (b): in 'Check Firely Terminal Version', `if $INPUT_DOTNET_VALIDATION_ENABLED; then` runs an input without quotes as a command.

Locations:

- `action.yml:131`

### script-injection (severity: high)

Rule (b): in 'Simplifier login', `if $INPUT_DOTNET_VALIDATION_ENABLED` and `fhir login email=$INPUT_SIMPLIFIER_USERNAME password=$INPUT_SIMPLIFIER_PASSWORD` expand inputs without quotes.

Locations:

- `action.yml:143`

### script-injection (severity: high)

Rule (b): in 'Detect FHIR version and determine dependency source', `if $INPUT_DOTNET_VALIDATION_ENABLED || $INPUT_JAVA_VALIDATION_ENABLED; then`, `if $INPUT_SUSHI_USE_CONFIG_DEPENDENCIES; then` and `elif $INPUT_SUSHI_ENABLED && ...` run inputs without quotes as commands.

Locations:

- `action.yml:168`

### script-injection (severity: high)

Rule (b): in 'Validate FHIR version support', `if $INPUT_DOTNET_VALIDATION_ENABLED || $INPUT_JAVA_VALIDATION_ENABLED; then` runs inputs without quotes as commands.

Locations:

- `action.yml:280`

### script-injection (severity: high)

Rule (b): in 'Set FHIR specification and restore dependencies', `if $INPUT_DOTNET_VALIDATION_ENABLED; then` runs an input without quotes as a command.

Locations:

- `action.yml:360`

### script-injection (severity: high)

Rule (b): in 'Check if npm is installed', `if $INPUT_SUSHI_ENABLED; then` runs an input without quotes as a command.

Locations:

- `action.yml:530`

### script-injection (severity: high)

Rule (b): in 'Check npm Version', `if $INPUT_SUSHI_ENABLED; then` runs an input without quotes as a command.

Locations:

- `action.yml:545`

### script-injection (severity: high)

Rule (b): in 'Install fsh-sushi', `if $INPUT_SUSHI_ENABLED` and `sudo npm install -g fsh-sushi@$SUSHI_VERSION` expand inputs without quotes, and the npm install runs with sudo.

Locations:

- `action.yml:567`

### script-injection (severity: high)

Rule (b): in 'Check SUSHI version', `if $INPUT_SUSHI_ENABLED; then` runs an input without quotes as a command.

Locations:

- `action.yml:575`

### script-injection (severity: high)

Rule (b): in 'Generate conformance resources with SUSHI', `if $INPUT_SUSHI_ENABLED` and `sushi $INPUT_SUSHI_OPTIONS` expand inputs without quotes.

Locations:

- `action.yml:580`

### script-injection (severity: high)

Rule (b): in 'Pre-processing FHIR profiles (Slicing open-closed)', `if $CLOSE_SLICING_FOR_VALIDATION; then` runs an input without quotes as a command.

Locations:

- `action.yml:592`

### script-injection (severity: high)

Rule (b): in 'Configure Firely Terminal validator engine', `if $INPUT_DOTNET_VALIDATION_ENABLED; then` runs an input without quotes as a command.

Locations:

- `action.yml:650`

### script-injection (severity: high)

Rule (b): in 'Run Quality Control checks', `if $INPUT_DOTNET_VALIDATION_ENABLED`, `echo $INPUT_EXPECTED_FAILS | grep ...` and `fhir check --fail $INPUT_PATH_TO_QUALITY_CONTROL_RULES` expand inputs without quotes.

Locations:

- `action.yml:680`

### script-injection (severity: high)

Rule (b): in 'Report Success - .NET Validator', `if $INPUT_DOTNET_VALIDATION_ENABLED; then` runs an input without quotes as a command.

Locations:

- `action.yml:700`

### script-injection (severity: high)

Rule (b): in 'Remove package cache to enable the Java validator to create snapshots', `if $INPUT_JAVA_SNAPSHOT_ENABLED; then` runs an input without quotes as a command, and `rm -rf $fhirCacheLocation` is also unquoted.

Locations:

- `action.yml:720`

### script-injection (severity: high)

Rule (b): in 'Check if Java is installed', `if $INPUT_JAVA_VALIDATION_ENABLED; then` runs an input without quotes as a command.

Locations:

- `action.yml:730`

### script-injection (severity: high)

Rule (b): in 'Download Java Validator', `JAVA_VALIDATOR_DOWNLOAD_LOCATION=$(eval echo "$JAVA_VALIDATOR_DOWNLOAD_LOCATION")` passes the input inputs.JAVA_VALIDATOR_DOWNLOAD_LOCATION to eval. Any $(...), backticks or ';' in the value runs. `if $INPUT_JAVA_VALIDATION_ENABLED` and `wget -q $JAVA_VALIDATOR_DOWNLOAD_LOCATION` are also unquoted.

Locations:

- `action.yml:745`

### script-injection (severity: high)

Rule (b): in 'Validate all conformance resources in scope of the repository', `if $INPUT_JAVA_VALIDATION_ENABLED`, `for p in $INPUT_PATH_TO_CONFORMANCE_RESOURCES;`, `echo $INPUT_EXPECTED_FAILS | grep ...` and `java -jar validator_cli.jar ... -version $FHIR_VERSION $INPUT_JAVA_VALIDATION_OPTIONS ...` expand inputs and env values without quotes.

Locations:

- `action.yml:800`

### script-injection (severity: high)

Rule (b): in 'Validate all example resources in scope of the repository', `if $INPUT_JAVA_VALIDATION_ENABLED`, `for p in $INPUT_PATH_TO_CONFORMANCE_RESOURCES;`, `for p in $PATH_TO_EXAMPLES;`, `echo $INPUT_EXPECTED_FAILS | grep ...` and `java -jar validator_cli.jar ... -version $FHIR_VERSION $INPUT_JAVA_VALIDATION_OPTIONS ...` expand inputs without quotes.

Locations:

- `action.yml:880`

### script-injection (severity: high)

Rule (b): in 'Re-run SUSHI to restore conformance resources after slicing modification', `if $CLOSE_SLICING_FOR_VALIDATION && $INPUT_SUSHI_ENABLED; then` and `sushi $INPUT_SUSHI_OPTIONS` expand inputs without quotes.

Locations:

- `action.yml:945`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection

**Notes:**

All changes are in action.yml. Nothing has been run or tested yet.

- **Boolean flags:** every `if $INPUT_X; then` (and the same pattern in `||`, `&&` and `elif`) now reads `[ "$INPUT_X" = "true" ]`. This covers DOTNET_VALIDATION_ENABLED, TERMINOLOGY_SERVICE_BFARM_ENABLED, JAVA_VALIDATION_ENABLED, SUSHI_ENABLED, SUSHI_USE_CONFIG_DEPENDENCIES, CLOSE_SLICING_FOR_VALIDATION and JAVA_SNAPSHOT_ENABLED. Behaviour changes for any value other than `true` or `false`: upstream ran the value as a command, and now only the exact word `true` turns a flag on. So `True`, `TRUE` or `yes` now silently turn the flag off.
- **Single values are now quoted:** FIRELY_TERMINAL_VERSION, SUSHI_VERSION, `$USER`, the Simplifier login (now `"email=..."` and `"password=..."`), FHIR_VERSION, the cache path in `rm -rf`, and the download URL in `wget`.
- **Quality-control rules path:** this optional input is now passed as `${VAR:+"$VAR"}`, so it adds no argument when empty.
- **EXPECTED_FAILS:** now checked with `printf '%s\n' "$VAR" | grep` instead of an unquoted `echo`.
- **Java download URL:** the `eval echo` is gone. The `$JAVA_VALIDATOR_VERSION` or `${JAVA_VALIDATOR_VERSION}` placeholder is now swapped for the version by plain text replacement. Unlike `eval`, this expands no other `$...` text in a custom URL.
- **List inputs** (SUSHI_OPTIONS, JAVA_VALIDATION_OPTIONS, PATH_TO_CONFORMANCE_RESOURCES, PATH_TO_EXAMPLES): each is turned into a bash array with an unquoted assignment (`arr=($VAR)`), which splits and globs exactly as upstream's unquoted use did, then passed as `"${arr[@]}"`.
- **Internal argument lists:** the `-ig` lists the script builds itself (local IG parameters and package dependencies) are now arrays too.
- **Validator file globs:** these are now `"$GITHUB_WORKSPACE/$p"*.xml` and `*.json`, so the path is quoted and the glob still matches.
- **Extra quoting outside the findings:** I quoted the ZTS package download URL and `echo "$FHIR_RESTORE"`.
- **sudo:** the finding also flagged the sudo `npm install`. I left sudo in place because upstream needs it to install SUSHI globally; only the version argument is now quoted.

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

In action.yml, the step 'Detect FHIR version and determine dependency source' now strips newline characters from fhirVersion (printf '%s' | tr -d '\n\r') before writing FHIR_VERSION to $GITHUB_ENV. The other GITHUB_ENV writes only hold true/false values the script sets itself, so they were left as they were. Not tested; the change was checked by reading the file only.

