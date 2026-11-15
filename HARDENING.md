<!-- markdownlint-disable -->

# Hardening Report: FirelyTeam--firely-terminal-pipeline/v0.11.2

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **FirelyTeam--firely-terminal-pipeline/v0.11.2** was hardened automatically. 11 finding(s) were identified and resolved across 3 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

action.yml has a composite step that references an action by a mutable tag instead of a full commit SHA: `uses: actions/upload-artifact@v7` (step 'Upload Quality Control report'). All other uses: refs are pinned to SHAs.

Locations:

- `action.yml:752`

### github-env-injection (severity: high)

Step 'Determine FHIR package cache key' writes input-derived values to $GITHUB_OUTPUT without the printf '%s' | tr -d '\n\r' sanitization step: `echo "key=fhir-packages-${RUNNER_OS}-ft${installedFirelyTerminalVersion:-$FIRELY_TERMINAL_VERSION}-zts${INPUT_TERMINOLOGY_SERVICE_BFARM_ENABLED}-..." >> "$GITHUB_OUTPUT"`. FIRELY_TERMINAL_VERSION and INPUT_TERMINOLOGY_SERVICE_BFARM_ENABLED come from inputs.*, and a newline in either value could add extra step outputs.

Locations:

- `action.yml:400`

### script-injection (severity: high)

Rule (b): the 'Download Java Validator' step runs input data through eval: `JAVA_VALIDATOR_DOWNLOAD_LOCATION=$(eval echo "$JAVA_VALIDATOR_DOWNLOAD_LOCATION")`. The value comes from inputs.JAVA_VALIDATOR_DOWNLOAD_LOCATION, so any $(...) or backticks in it run as shell commands. The result is then expanded unquoted: `wget -q $JAVA_VALIDATOR_DOWNLOAD_LOCATION -O validator_cli.jar`.

Locations:

- `action.yml:790`
- `action.yml:793`

### script-injection (severity: high)

Rule (b): many steps run input-derived booleans as bare unquoted commands, for example `if $INPUT_DOTNET_VALIDATION_ENABLED || $INPUT_TERMINOLOGY_SERVICE_BFARM_ENABLED; then`, `if $INPUT_SUSHI_ENABLED; then`, `if $INPUT_JAVA_VALIDATION_ENABLED; then`, `if $CLOSE_SLICING_FOR_VALIDATION; then`, `if $INPUT_JAVA_SNAPSHOT_ENABLED`, `elif $INPUT_REPORT_RENDERER_ENABLED`. The shell runs each input value as a command, so whatever a caller passes in gets executed. Affected steps include 'Check if .NET is installed', 'Install Firely.Terminal', 'Detect FHIR version...', 'Validate FHIR version support', 'Check if npm is installed', 'Pre-processing FHIR profiles', 'Run Quality Control checks', the Java validator steps and 'Re-run SUSHI'.

Locations:

- `action.yml:106`
- `action.yml:132`
- `action.yml:606`
- `action.yml:972`

### script-injection (severity: high)

Rule (b): step 'Install Firely.Terminal' expands an input unquoted: `dotnet tool install --global Firely.Terminal --version $FIRELY_TERMINAL_VERSION > /dev/null` (from inputs.FIRELY_TERMINAL_VERSION).

Locations:

- `action.yml:135`

### script-injection (severity: high)

Rule (b): step 'Simplifier login' expands credential inputs unquoted: `fhir login email=$INPUT_SIMPLIFIER_USERNAME password=$INPUT_SIMPLIFIER_PASSWORD`.

Locations:

- `action.yml:159`

### script-injection (severity: high)

Rule (b): step 'Install fsh-sushi' expands an input unquoted: `sudo npm install -g fsh-sushi@$SUSHI_VERSION` (inputs.SUSHI_VERSION). The step also uses unquoted `sudo chown -R $USER`.

Locations:

- `action.yml:582`

### script-injection (severity: high)

Rule (b): steps 'Generate conformance resources with SUSHI' and 'Re-run SUSHI...' expand inputs.SUSHI_OPTIONS unquoted: `sushi $INPUT_SUSHI_OPTIONS`.

Locations:

- `action.yml:593`
- `action.yml:974`

### script-injection (severity: high)

Rule (b): step 'Run Quality Control checks' expands inputs unquoted: `if echo $INPUT_EXPECTED_FAILS | grep -w -q VALIDATION_CONFORMANCE_DOTNET` and `fhir check $INPUT_PATH_TO_QUALITY_CONTROL_RULES $CHECK_ARGS`. CHECK_ARGS contains the input value INPUT_DOTNET_FAIL_ON, but that value is checked against an allowlist first.

Locations:

- `action.yml:705`
- `action.yml:710`

### script-injection (severity: high)

Rule (b): step 'Remove package cache...' uses an unquoted expansion in a destructive command: `rm -rf $fhirCacheLocation`. The value is command output rather than an input, but splitting or globbing it is unsafe in rm -rf.

Locations:

- `action.yml:770`

### script-injection (severity: high)

Rule (b): steps 'Validate all conformance resources' and 'Validate all example resources' expand workflow-controlled env vars unquoted: `for p in $INPUT_PATH_TO_CONFORMANCE_RESOURCES;`, `for p in $PATH_TO_EXAMPLES;`, `if echo $INPUT_EXPECTED_FAILS | grep ...`, and `java -jar validator_cli.jar $GITHUB_WORKSPACE/$p*.xml ... -version $FHIR_VERSION $INPUT_JAVA_VALIDATION_OPTIONS $UNESCPAED_IG_DEPENDENCIES ...`. INPUT_JAVA_VALIDATION_OPTIONS and FHIR_VERSION (env.*) are workflow-controlled.

Locations:

- `action.yml:838`
- `action.yml:848`
- `action.yml:920`
- `action.yml:928`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, github-env-injection, script-injection

**Notes:**

I hardened action.yml for all the findings; nothing was run or tested. Pinned actions/upload-artifact@v7 to cf430e030ddbb5b0abf93d22962f4752f3646cd9 (resolved by lookup) with a '# v7' comment. The cache-key step now captures each value into a variable first and runs printf '%s' | tr -d '\n\r' on it before writing path= and key= to GITHUB_OUTPUT. In upstream those commands ran inside echo, so a failure didn't stop the step; '|| true' keeps that. Removed the eval in Download Java Validator. Only the documented $JAVA_VALIDATOR_VERSION / ${JAVA_VALIDATOR_VERSION} placeholder is now filled in, using bash string replacement, and wget gets the URL quoted. Every boolean input that used to run as a bare command (if $X, elif $X, && $X) is now compared as a string, e.g. [[ "$X" == "true" ]]; values other than the exact string 'true' now count as false. Quoted FIRELY_TERMINAL_VERSION, the Simplifier login arguments, fsh-sushi@$SUSHI_VERSION, $USER and the supportedVersions loop. rm -rf now gets the cache path quoted, after '--', and only when it isn't empty. SUSHI_OPTIONS, JAVA_VALIDATION_OPTIONS, PATH_TO_CONFORMANCE_RESOURCES and PATH_TO_EXAMPLES are lists. Each one is put into an array with an unquoted assignment, which splits and globs the same way upstream's unquoted expansion did, and the array is then expanded quoted. The -ig parameters are now built as arrays, and the IG dependencies (built from package.json, not an input) are split with read -a. One small behaviour change: a conformance path containing a space now stays one -ig argument, where upstream split it. The file globs ($p*.xml / $p*.json) are kept, with the variable part quoted. In Quality Control, CHECK_ARGS and NO_FAIL_ARGS are now arrays. The optional rules path is passed as ${VAR:+"$VAR"}, so it drops out when empty, and EXPECTED_FAILS is quoted before grep -w.

### Iteration 2

**Fixes applied:** github-env-injection

**Notes:**

In the 'Run Quality Control checks' step (qc) of action.yml, $REPORT now has newlines stripped (printf '%s' | tr -d '\n\r') before it is written to $GITHUB_OUTPUT.

### Iteration 1

**Fixes applied:** publish-gate

**Notes:**

At action.yml lines 871 and 949 I replaced both `read -r -a igDependencyArgs <<< "$UNESCPAED_IG_DEPENDENCIES"` lines with an unquoted array assignment, `igDependencyArgs=($UNESCPAED_IG_DEPENDENCIES)`, and a shellcheck disable comment above each. Upstream expanded this variable unquoted, so it was split on whitespace and each word could match files; the new form does the same. The arrays are still passed as "${igDependencyArgs[@]}". Nothing else was changed. I didn't run the action or re-run the publish checks.

