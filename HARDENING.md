<!-- markdownlint-disable -->

# Hardening Report: FirelyTeam--firely-terminal-pipeline/v0.10.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **FirelyTeam--firely-terminal-pipeline/v0.10.1** was hardened automatically. 9 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (b): in many steps, boolean inputs are expanded unquoted and run as commands: `if $INPUT_DOTNET_VALIDATION_ENABLED || $INPUT_TERMINOLOGY_SERVICE_BFARM_ENABLED; then`, `if $INPUT_SUSHI_ENABLED; then`, `if $INPUT_JAVA_VALIDATION_ENABLED; then`, `if $INPUT_JAVA_SNAPSHOT_ENABLED; then`, `if $CLOSE_SLICING_FOR_VALIDATION; then`. A caller who sets an input like DOTNET_VALIDATION_ENABLED to an arbitrary string gets that string executed as a shell command. Compare against a quoted literal instead, e.g. `[[ "$INPUT_X" == "true" ]]`.

Locations:

- `action.yml:90`
- `action.yml:106`
- `action.yml:117`
- `action.yml:132`
- `action.yml:143`
- `action.yml:169`
- `action.yml:286`
- `action.yml:356`
- `action.yml:494`
- `action.yml:533`
- `action.yml:583`
- `action.yml:640`
- `action.yml:677`
- `action.yml:756`
- `action.yml:832`

### script-injection (severity: high)

Rule (b): the 'Install Firely.Terminal' step expands an input unquoted: `dotnet tool install --global Firely.Terminal --version $FIRELY_TERMINAL_VERSION` (FIRELY_TERMINAL_VERSION comes from inputs.FIRELY_TERMINAL_VERSION).

Locations:

- `action.yml:120`

### script-injection (severity: high)

Rule (b): the 'Simplifier login' step expands credentials unquoted: `fhir login email=$INPUT_SIMPLIFIER_USERNAME password=$INPUT_SIMPLIFIER_PASSWORD`.

Locations:

- `action.yml:144`

### script-injection (severity: high)

Rule (b): the 'Install fsh-sushi' step expands an input unquoted: `sudo npm install -g fsh-sushi@$SUSHI_VERSION`. It also uses `sudo chown -R $USER`, where USER is inherited from the environment.

Locations:

- `action.yml:510`

### script-injection (severity: high)

Rule (b): the 'Generate conformance resources with SUSHI' and 'Re-run SUSHI' steps expand an input unquoted: `sushi $INPUT_SUSHI_OPTIONS`.

Locations:

- `action.yml:521`
- `action.yml:834`

### script-injection (severity: high)

Rule (b): the 'Run Quality Control checks' step expands inputs unquoted: `echo $INPUT_EXPECTED_FAILS | grep ...`, `fhir check $INPUT_PATH_TO_QUALITY_CONTROL_RULES`, `fhir check --fail $INPUT_PATH_TO_QUALITY_CONTROL_RULES`.

Locations:

- `action.yml:610`

### script-injection (severity: high)

Rule (b): the 'Download Java Validator' step passes an input to eval: `JAVA_VALIDATOR_DOWNLOAD_LOCATION=$(eval echo "$JAVA_VALIDATOR_DOWNLOAD_LOCATION")`, where the value comes from inputs.JAVA_VALIDATOR_DOWNLOAD_LOCATION. Any `$(...)` or `;` in the input is executed. The result is then used unquoted: `wget -q $JAVA_VALIDATOR_DOWNLOAD_LOCATION -O validator_cli.jar`.

Locations:

- `action.yml:661`
- `action.yml:664`

### script-injection (severity: high)

Rule (b): the 'Validate all conformance resources' and 'Validate all example resources' steps expand inputs unquoted: `for p in $INPUT_PATH_TO_CONFORMANCE_RESOURCES;`, `for p in $PATH_TO_EXAMPLES;`, `echo $INPUT_EXPECTED_FAILS | grep ...`, and `java -jar validator_cli.jar ... -version $FHIR_VERSION $INPUT_JAVA_VALIDATION_OPTIONS $UNESCPAED_IG_DEPENDENCIES ...`. The step also runs `rm -rf $fhirCacheLocation` unquoted.

Locations:

- `action.yml:707`
- `action.yml:780`

### github-env-injection (severity: high)

The 'Detect FHIR version and determine dependency source' step writes `echo "FHIR_VERSION=$fhirVersion" >> "$GITHUB_ENV"`. fhirVersion is not a literal computed in the script: it is parsed from package.json or sushi-config.yaml, both of which a pull request can change. It is written without `printf '%s' ... | tr -d '\n\r'`, so a value containing a newline could inject extra environment variables.

Locations:

- `action.yml:272`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection

**Notes:**

Nothing was run or tested. action.yml was rewritten in full because edit_file had corrupted several steps; a search afterwards found none of the flagged patterns left.

Changes:
- **Boolean inputs:** every `if $INPUT_X` became `[[ "$INPUT_X" == "true" ]]`. Behaviour change: an empty boolean input now counts as false (upstream treated it as true), and only lowercase `true` turns a feature on.
- **Single values are quoted:** FIRELY_TERMINAL_VERSION, the Simplifier email and password (`"email=..."`, `"password=..."`), `fsh-sushi@$SUSHI_VERSION`, `-version "$FHIR_VERSION"`, `rm -rf` (now skipped when the cache path is empty), and the ZTS package URL.
- **chown:** now uses `"$(id -un)"` instead of the inherited `$USER`.
- **Optional QC rules path:** passed as `${VAR:+"$VAR"}`, so an empty value adds no argument.
- **EXPECTED_FAILS check:** now `printf '%s\n' "$X" | grep -w -q ...`.
- **List inputs** (SUSHI_OPTIONS, JAVA_VALIDATION_OPTIONS, PATH_TO_CONFORMANCE_RESOURCES, PATH_TO_EXAMPLES) and the IG dependency list taken from package.json: each is split on whitespace into a bash array with `read -r -d '' -a` (guarded for empty, with `|| true`), the same split upstream got from the unquoted variable. Nothing in them is run.
- **-ig arguments:** LOCAL_IG_PARAMETERS and COMBINED_IG_PARAMETERS are now arrays of `-ig "$GITHUB_WORKSPACE/$p"` pairs.
- **Validator file globs:** written as `"$GITHUB_WORKSPACE/$p"*.xml` and `*.json`, so they still match files.
- **Removed eval:** the download URL now has the literal `$JAVA_VALIDATOR_VERSION` and `${JAVA_VALIDATOR_VERSION}` placeholders swapped for the version with bash string replacement, and wget gets the URL quoted. Unlike eval, this only fills in that one placeholder: any other `$VAR` or `$(...)` in a custom URL stays as plain text.
- **FHIR_VERSION to GITHUB_ENV:** newlines and carriage returns are stripped with `printf '%s' | tr -d '\n\r'` before the write.

Outside the findings, I also quoted `echo "$FHIR_RESTORE"` and the supportedVersions array loop.

