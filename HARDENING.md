<!-- markdownlint-disable -->

# Hardening Report: FirelyTeam--firely-terminal-pipeline/v0.11.4

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **FirelyTeam--firely-terminal-pipeline/v0.11.4** was hardened automatically. 11 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

action.yml has a composite step that refers to an action by a mutable tag instead of a full 40-character commit SHA: `uses: actions/upload-artifact@v7` (step 'Upload Quality Control report'). The other uses: references (actions/cache/restore, actions/cache/save, patrick-werner/validation-outcome-markdown-renderer) are pinned to SHAs.

Locations:

- `action.yml:750`

### github-env-injection (severity: high)

Step 'Determine FHIR package cache key' writes values taken from workflow inputs to $GITHUB_OUTPUT without first running `printf '%s' ... | tr -d '\n\r'`: `echo "key=fhir-packages-${RUNNER_OS}-ft${installedFirelyTerminalVersion:-$FIRELY_TERMINAL_VERSION}-zts${INPUT_TERMINOLOGY_SERVICE_BFARM_ENABLED}-..." >> "$GITHUB_OUTPUT"`. FIRELY_TERMINAL_VERSION comes from inputs.FIRELY_TERMINAL_VERSION and INPUT_TERMINOLOGY_SERVICE_BFARM_ENABLED comes from inputs.TERMINOLOGY_SERVICE_BFARM_ENABLED. A newline in either value can add extra outputs.

Locations:

- `action.yml:412`

### script-injection (severity: high)

Rule (b): the step 'Download Java Validator' passes workflow-controlled input to eval: `JAVA_VALIDATOR_DOWNLOAD_LOCATION=$(eval echo "$JAVA_VALIDATOR_DOWNLOAD_LOCATION")`. JAVA_VALIDATOR_DOWNLOAD_LOCATION comes from inputs.JAVA_VALIDATOR_DOWNLOAD_LOCATION, and eval re-parses the value, so any $(...), backticks or ';' in it runs as a command. The next line, `wget -q $JAVA_VALIDATOR_DOWNLOAD_LOCATION -O validator_cli.jar`, also expands the variable unquoted.

Locations:

- `action.yml:790`
- `action.yml:793`

### script-injection (severity: high)

Rule (b): boolean inputs are expanded unquoted and run as commands in almost every step, e.g. `if $INPUT_DOTNET_VALIDATION_ENABLED || $INPUT_TERMINOLOGY_SERVICE_BFARM_ENABLED; then`, `if $INPUT_JAVA_VALIDATION_ENABLED; then`, `if $INPUT_SUSHI_ENABLED; then`, `if $CLOSE_SLICING_FOR_VALIDATION; then`, `if $INPUT_JAVA_SNAPSHOT_ENABLED; then`, `elif $INPUT_REPORT_RENDERER_ENABLED; then`, `if $INPUT_SUSHI_USE_CONFIG_DEPENDENCIES; then`. These values come from inputs.*. Expanding a variable as a command runs whatever program its value names, with arguments, so a value other than true/false is executed. The safe form is a quoted comparison such as `[[ "$VAR" == "true" ]]`.

Locations:

- `action.yml:120`
- `action.yml:614`
- `action.yml:955`

### script-injection (severity: high)

Rule (b): step 'Install Firely.Terminal' uses `dotnet tool install --global Firely.Terminal --version $FIRELY_TERMINAL_VERSION`. FIRELY_TERMINAL_VERSION comes from inputs.FIRELY_TERMINAL_VERSION and is expanded unquoted.

Locations:

- `action.yml:150`

### script-injection (severity: high)

Rule (b): step 'Simplifier login' uses `fhir login email=$INPUT_SIMPLIFIER_USERNAME password=$INPUT_SIMPLIFIER_PASSWORD`. Both values come from inputs and are expanded unquoted.

Locations:

- `action.yml:171`

### script-injection (severity: high)

Rule (b): step 'Install fsh-sushi' uses `sudo npm install -g fsh-sushi@$SUSHI_VERSION`. SUSHI_VERSION comes from inputs.SUSHI_VERSION and is expanded unquoted.

Locations:

- `action.yml:590`

### script-injection (severity: high)

Rule (b): steps 'Generate conformance resources with SUSHI' and 'Re-run SUSHI to restore conformance resources after slicing modification' both use `sushi $INPUT_SUSHI_OPTIONS`. INPUT_SUSHI_OPTIONS comes from inputs.SUSHI_OPTIONS and is expanded unquoted.

Locations:

- `action.yml:605`
- `action.yml:957`

### script-injection (severity: high)

Rule (b): step 'Run Quality Control checks' uses `if echo $INPUT_EXPECTED_FAILS | grep ...` and `fhir check $INPUT_PATH_TO_QUALITY_CONTROL_RULES $CHECK_ARGS`. CHECK_ARGS is built from inputs.DOTNET_FAIL_ON, and all three are expanded unquoted.

Locations:

- `action.yml:712`
- `action.yml:717`

### script-injection (severity: high)

Rule (b): step 'Remove package cache to enable the Java validator to create snapshots' runs `rm -rf $fhirCacheLocation`, a command-substitution result expanded unquoted. The step is also gated by the unquoted `if $INPUT_JAVA_SNAPSHOT_ENABLED`.

Locations:

- `action.yml:768`

### script-injection (severity: high)

Rule (b): steps 'Validate all conformance resources in scope of the repository' and 'Validate all example resources in scope of the repository' expand input-derived variables unquoted: `for p in $INPUT_PATH_TO_CONFORMANCE_RESOURCES;`, `for p in $PATH_TO_EXAMPLES;`, `echo $INPUT_EXPECTED_FAILS | grep ...`, and `java -jar validator_cli.jar $GITHUB_WORKSPACE/$p*.xml ... -version $FHIR_VERSION $INPUT_JAVA_VALIDATION_OPTIONS $UNESCPAED_IG_DEPENDENCIES $LOCAL_IG_PARAMETERS`. These are plain unquoted expansions, not the exempt array-assignment form with a shellcheck directive.

Locations:

- `action.yml:838`
- `action.yml:855`
- `action.yml:905`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, github-env-injection, script-injection

**Notes:**

Pinned actions/upload-artifact@v7 to cf430e030ddbb5b0abf93d22962f4752f3646cd9. Cache key step: captured fhir cache path and sha256 hash separately (with '|| true', since upstream ran them inside echo's arguments), then stripped \n\r from the path and the key with printf|tr before writing to GITHUB_OUTPUT. Boolean inputs that were run as commands ('if $VAR') are now quoted [[ "$VAR" == "true" ]] comparisons. Replaced eval of JAVA_VALIDATOR_DOWNLOAD_LOCATION with a literal bash substitution of the $JAVA_VALIDATOR_VERSION / ${JAVA_VALIDATOR_VERSION} placeholder, and the wget URL is now quoted. Quoted the Firely Terminal version, the fsh-sushi@version and the fhir login email=/password= arguments. SUSHI_OPTIONS, JAVA_VALIDATION_OPTIONS, PATH_TO_CONFORMANCE_RESOURCES and PATH_TO_EXAMPLES are lists, so they now go through unquoted array assignment with shellcheck directives. That splits and globs them exactly as upstream did, and the arrays are expanded quoted. IG dependency and -ig parameter lists are arrays too. QC check arguments are a bash array, and the optional rules path uses ${VAR:+"$VAR"}. EXPECTED_FAILS is checked with printf '%s\n' "$VAR" | grep. rm -rf of the cache location is quoted and only runs when the location is non-empty. Java file globs keep their glob behaviour with a quoted prefix ("$GITHUB_WORKSPACE/$p"*.xml).

### Iteration 2

**Fixes applied:** github-env-injection

**Notes:**

In action.yml, the 'Detect FHIR version and determine dependency source' step now strips newlines from fhirVersion before writing it to $GITHUB_ENV, using safeFhirVersion=$(printf '%s' "$fhirVersion" | tr -d '\n\r'). The other writes to $GITHUB_ENV and $GITHUB_OUTPUT are either fixed literals or already have newlines stripped. Nothing was run to test the change.

