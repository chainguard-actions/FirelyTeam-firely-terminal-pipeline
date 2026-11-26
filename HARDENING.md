<!-- markdownlint-disable -->

# Hardening Report: FirelyTeam--firely-terminal-pipeline/v0.11.4

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **FirelyTeam--firely-terminal-pipeline/v0.11.4** was hardened automatically. 25 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

action.yml has a uses: reference pinned to a mutable tag rather than a 40-char commit SHA: `uses: actions/upload-artifact@v7` (step 'Upload Quality Control report'). The other uses: refs (actions/cache/restore, actions/cache/save, patrick-werner/validation-outcome-markdown-renderer) are pinned to a SHA.

Locations:

- `action.yml:754`

### github-env-injection (severity: high)

Step 'Determine FHIR package cache key' writes input-derived values to $GITHUB_OUTPUT without the `printf '%s' ... | tr -d '\n\r'` sanitization step: `echo "key=fhir-packages-${RUNNER_OS}-ft${installedFirelyTerminalVersion:-$FIRELY_TERMINAL_VERSION}-zts${INPUT_TERMINOLOGY_SERVICE_BFARM_ENABLED}-..." >> "$GITHUB_OUTPUT"`. FIRELY_TERMINAL_VERSION and INPUT_TERMINOLOGY_SERVICE_BFARM_ENABLED come from inputs.*. A newline in either value can inject extra outputs.

Locations:

- `action.yml:391`

### github-env-injection (severity: high)

Step 'Detect FHIR version and determine dependency source' writes `echo "FHIR_VERSION=$fhirVersion" >> "$GITHUB_ENV"`. fhirVersion is not a literal: it is read from repository-controlled package.json or sushi-config.yaml (including a base64-decoded yq value), and no `printf '%s' ... | tr -d '\n\r'` step runs before the write. A multi-line value can inject arbitrary environment variables into later steps.

Locations:

- `action.yml:288`

### script-injection (severity: high)

Rule (b), step 'Check if .NET is installed': input-derived env vars are expanded unquoted and run as commands: `if $INPUT_DOTNET_VALIDATION_ENABLED || $INPUT_TERMINOLOGY_SERVICE_BFARM_ENABLED; then`. The input value is executed as a shell command (for example `true; malicious`).

Locations:

- `action.yml:106`

### script-injection (severity: high)

Rule (b), step 'Check .NET SDK Version': `if $INPUT_DOTNET_VALIDATION_ENABLED; then` runs an unquoted input value as a command.

Locations:

- `action.yml:122`

### script-injection (severity: high)

Rule (b), step 'Install Firely.Terminal': `if $INPUT_DOTNET_VALIDATION_ENABLED || $INPUT_TERMINOLOGY_SERVICE_BFARM_ENABLED; then` runs input values as commands, and `dotnet tool install --global Firely.Terminal --version $FIRELY_TERMINAL_VERSION` expands an input unquoted.

Locations:

- `action.yml:133`
- `action.yml:136`

### script-injection (severity: high)

Rule (b), step 'Check Firely Terminal Version': `if $INPUT_DOTNET_VALIDATION_ENABLED; then` runs an unquoted input as a command.

Locations:

- `action.yml:148`

### script-injection (severity: high)

Rule (b), step 'Simplifier login': `if $INPUT_DOTNET_VALIDATION_ENABLED; then` and `fhir login email=$INPUT_SIMPLIFIER_USERNAME password=$INPUT_SIMPLIFIER_PASSWORD` expand input-derived values unquoted.

Locations:

- `action.yml:159`
- `action.yml:160`

### script-injection (severity: high)

Rule (b), step 'Detect FHIR version and determine dependency source': unquoted input values are run as commands: `if $INPUT_DOTNET_VALIDATION_ENABLED || $INPUT_JAVA_VALIDATION_ENABLED; then`, `if $INPUT_SUSHI_USE_CONFIG_DEPENDENCIES; then`, `elif $INPUT_SUSHI_ENABLED && ...`.

Locations:

- `action.yml:185`
- `action.yml:200`
- `action.yml:219`

### script-injection (severity: high)

Rule (b), step 'Validate FHIR version support': `if $INPUT_DOTNET_VALIDATION_ENABLED || $INPUT_JAVA_VALIDATION_ENABLED; then` runs unquoted input values as commands.

Locations:

- `action.yml:302`

### script-injection (severity: high)

Rule (b), step 'Set FHIR specification and restore dependencies': `if $INPUT_DOTNET_VALIDATION_ENABLED; then` runs an unquoted input as a command.

Locations:

- `action.yml:409`

### script-injection (severity: high)

Rule (b), step 'Check if npm is installed': `if $INPUT_SUSHI_ENABLED; then` runs an unquoted input as a command.

Locations:

- `action.yml:551`

### script-injection (severity: high)

Rule (b), step 'Check npm Version': `if $INPUT_SUSHI_ENABLED; then` runs an unquoted input as a command.

Locations:

- `action.yml:564`

### script-injection (severity: high)

Rule (b), step 'Install fsh-sushi': `if $INPUT_SUSHI_ENABLED; then`, `sudo chown -R $USER ...` and `sudo npm install -g fsh-sushi@$SUSHI_VERSION` expand values unquoted (SUSHI_VERSION comes from inputs).

Locations:

- `action.yml:574`
- `action.yml:576`

### script-injection (severity: high)

Rule (b), step 'Check SUSHI version': `if $INPUT_SUSHI_ENABLED; then` runs an unquoted input as a command.

Locations:

- `action.yml:585`

### script-injection (severity: high)

Rule (b), step 'Generate conformance resources with SUSHI': `if $INPUT_SUSHI_ENABLED; then` and `sushi $INPUT_SUSHI_OPTIONS` expand input values unquoted.

Locations:

- `action.yml:594`
- `action.yml:595`

### script-injection (severity: high)

Rule (b), step 'Pre-processing FHIR profiles (Slicing open-closed)': `if $CLOSE_SLICING_FOR_VALIDATION; then` runs an unquoted input as a command.

Locations:

- `action.yml:608`

### script-injection (severity: high)

Rule (b), step 'Configure Firely Terminal validator engine': `if $INPUT_DOTNET_VALIDATION_ENABLED; then` runs an unquoted input as a command.

Locations:

- `action.yml:663`

### script-injection (severity: high)

Rule (b), step 'Run Quality Control checks': unquoted input expansions: `if $INPUT_DOTNET_VALIDATION_ENABLED; then`, `if echo $INPUT_EXPECTED_FAILS | grep ...`, `fhir check $INPUT_PATH_TO_QUALITY_CONTROL_RULES $CHECK_ARGS` (CHECK_ARGS contains the input INPUT_DOTNET_FAIL_ON), and `elif $INPUT_REPORT_RENDERER_ENABLED; then`.

Locations:

- `action.yml:689`
- `action.yml:711`
- `action.yml:717`
- `action.yml:727`

### script-injection (severity: high)

Rule (b), step 'Remove package cache to enable the Java validator to create snapshots': `if $INPUT_JAVA_SNAPSHOT_ENABLED; then` runs an unquoted input as a command, and `rm -rf $fhirCacheLocation` is unquoted.

Locations:

- `action.yml:769`
- `action.yml:771`

### script-injection (severity: high)

Rule (b), step 'Check if Java is installed': `if $INPUT_JAVA_VALIDATION_ENABLED; then` runs an unquoted input as a command.

Locations:

- `action.yml:779`

### script-injection (severity: high)

Rule (b), step 'Download Java Validator': `if $INPUT_JAVA_VALIDATION_ENABLED; then` runs an input as a command. `JAVA_VALIDATOR_DOWNLOAD_LOCATION=$(eval echo "$JAVA_VALIDATOR_DOWNLOAD_LOCATION")` passes the input JAVA_VALIDATOR_DOWNLOAD_LOCATION through eval, so any `$(...)` or `;` in the input is executed. `wget -q $JAVA_VALIDATOR_DOWNLOAD_LOCATION` is also unquoted.

Locations:

- `action.yml:792`
- `action.yml:796`
- `action.yml:799`

### script-injection (severity: high)

Rule (b), step 'Validate all conformance resources in scope of the repository': unquoted input expansions: `if $INPUT_JAVA_VALIDATION_ENABLED; then`, `for p in $INPUT_PATH_TO_CONFORMANCE_RESOURCES;`, `if echo $INPUT_EXPECTED_FAILS | grep ...`, `java -jar validator_cli.jar $GITHUB_WORKSPACE/$p*.xml ... -version $FHIR_VERSION $INPUT_JAVA_VALIDATION_OPTIONS ...`.

Locations:

- `action.yml:817`
- `action.yml:841`
- `action.yml:856`
- `action.yml:857`

### script-injection (severity: high)

Rule (b), step 'Validate all example resources in scope of the repository': unquoted input expansions: `if $INPUT_JAVA_VALIDATION_ENABLED; then`, `for p in $INPUT_PATH_TO_CONFORMANCE_RESOURCES;`, `for p in $PATH_TO_EXAMPLES;`, `if echo $INPUT_EXPECTED_FAILS | grep ...`, `java -jar validator_cli.jar ... -version $FHIR_VERSION $INPUT_JAVA_VALIDATION_OPTIONS ...`.

Locations:

- `action.yml:892`
- `action.yml:910`
- `action.yml:915`
- `action.yml:925`
- `action.yml:926`

### script-injection (severity: high)

Rule (b), step 'Re-run SUSHI to restore conformance resources after slicing modification': `if $CLOSE_SLICING_FOR_VALIDATION && $INPUT_SUSHI_ENABLED; then` runs unquoted inputs as commands, and `sushi $INPUT_SUSHI_OPTIONS` is unquoted.

Locations:

- `action.yml:956`
- `action.yml:958`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, github-env-injection, script-injection

**Notes:**

I fixed all three finding types in action.yml. I only edited it, never ran it, and there are no tests in the action directory.

**unpinned-uses**
- `actions/upload-artifact@v7` now points at the full commit SHA cf430e030ddbb5b0abf93d22962f4752f3646cd9, with `# v7` kept as a comment.

**github-env-injection**
- `FHIR_VERSION` has newlines stripped before it is written to $GITHUB_ENV.
- In the cache-key step, the Firely Terminal version and the TERMINOLOGY_SERVICE_BFARM_ENABLED (ZTS) flag also have newlines stripped before going into $GITHUB_OUTPUT. The package.json hash is now captured first with `|| true`, so a failing `sha256sum` still doesn't stop the step, as before.

**script-injection**
- **Boolean checks:** every `if $INPUT_X;` style check (including the `||` / `&&` / `elif` forms) is now `[ "$X" = "true" ]`, so an input is never run as a command. Behaviour change: only lowercase `true` counts as enabled. Before, other values just ran as a command; one like `True` failed with "command not found".
- **Single values are now quoted:** the dotnet tool version, the Simplifier login email and password, `chown "$USER"`, `fsh-sushi@version`, `rm -rf` on the cache path, `wget` on the download URL and `-version` for the FHIR version.
- **Dropped `eval` on JAVA_VALIDATOR_DOWNLOAD_LOCATION:** it now just replaces the documented `$JAVA_VALIDATOR_VERSION` / `${JAVA_VALIDATOR_VERSION}` placeholder. Any other `$...` in a custom URL is now left as literal text instead of being expanded.
- **EXPECTED_FAILS checks:** they now use `printf '%s\n' "$INPUT_EXPECTED_FAILS" | grep -w`.
- **`fhir check`:** the arguments are built as a bash array. The optional rules path uses `${VAR:+"$VAR"}`, so it drops out when empty.
- **List inputs:** SUSHI_OPTIONS, JAVA_VALIDATION_OPTIONS, PATH_TO_CONFORMANCE_RESOURCES and PATH_TO_EXAMPLES, plus the dependency list read from package.json, are turned into arrays with unquoted assignment. That splits on whitespace and expands globs exactly like upstream's unquoted use. The `-ig` parameter lists are built as arrays too.
- **Validator file globs:** `"$GITHUB_WORKSPACE/$p"*.xml` and `*.json` keep the glob outside the quotes, so file matching works as before.

