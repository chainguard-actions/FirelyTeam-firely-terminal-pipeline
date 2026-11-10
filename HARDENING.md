<!-- markdownlint-disable -->

# Hardening Report: FirelyTeam--firely-terminal-pipeline/v0.11.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **FirelyTeam--firely-terminal-pipeline/v0.11.0** was hardened automatically. 24 finding(s) were identified and resolved across 3 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

A composite step references an action by a mutable tag instead of a full 40-character commit SHA: `uses: actions/upload-artifact@v7`. The other uses: lines (actions/cache/restore, actions/cache/save, patrick-werner/validation-outcome-markdown-renderer) are pinned to a SHA.

Locations:

- `action.yml:743`

### github-env-injection (severity: high)

Step 'Determine FHIR package cache key' writes values derived from inputs straight to $GITHUB_OUTPUT with no `printf '%s' ... | tr -d '\n\r'` step first: `echo "key=fhir-packages-${RUNNER_OS}-ft${installedFirelyTerminalVersion:-$FIRELY_TERMINAL_VERSION}-zts${INPUT_TERMINOLOGY_SERVICE_BFARM_ENABLED}-..." >> "$GITHUB_OUTPUT"`. FIRELY_TERMINAL_VERSION and INPUT_TERMINOLOGY_SERVICE_BFARM_ENABLED come from inputs.*, so a newline in either one can inject extra outputs.

Locations:

- `action.yml:391`

### script-injection (severity: high)

Rule (b) – Download Java Validator: the input JAVA_VALIDATOR_DOWNLOAD_LOCATION is run through eval: `JAVA_VALIDATOR_DOWNLOAD_LOCATION=$(eval echo "$JAVA_VALIDATOR_DOWNLOAD_LOCATION")`. Any $(...), backticks or ';' in the input is executed. It is then used unquoted in `wget -q $JAVA_VALIDATOR_DOWNLOAD_LOCATION -O validator_cli.jar`, and `if $INPUT_JAVA_VALIDATION_ENABLED; then` runs the input value as a command.

Locations:

- `action.yml:785`
- `action.yml:788`
- `action.yml:781`

### script-injection (severity: high)

Rule (b) – Check if .NET is installed: `if $INPUT_DOTNET_VALIDATION_ENABLED || $INPUT_TERMINOLOGY_SERVICE_BFARM_ENABLED; then`. These input-derived env vars are expanded unquoted in command position, so the shell runs their values as commands.

Locations:

- `action.yml:106`

### script-injection (severity: high)

Rule (b) – Check .NET SDK Version: `if $INPUT_DOTNET_VALIDATION_ENABLED; then`. The input value is expanded unquoted and run as a command.

Locations:

- `action.yml:122`

### script-injection (severity: high)

Rule (b) – Install Firely.Terminal: `if $INPUT_DOTNET_VALIDATION_ENABLED || $INPUT_TERMINOLOGY_SERVICE_BFARM_ENABLED; then` runs input values as commands, and `dotnet tool install --global Firely.Terminal --version $FIRELY_TERMINAL_VERSION` expands an input unquoted.

Locations:

- `action.yml:133`
- `action.yml:136`

### script-injection (severity: high)

Rule (b) – Check Firely Terminal Version: `if $INPUT_DOTNET_VALIDATION_ENABLED; then`. The input value is expanded unquoted and run as a command.

Locations:

- `action.yml:148`

### script-injection (severity: high)

Rule (b) – Simplifier login: `fhir login email=$INPUT_SIMPLIFIER_USERNAME password=$INPUT_SIMPLIFIER_PASSWORD` expands both inputs unquoted, and `if $INPUT_DOTNET_VALIDATION_ENABLED; then` runs an input value as a command.

Locations:

- `action.yml:159`
- `action.yml:160`

### script-injection (severity: high)

Rule (b) – Detect FHIR version and determine dependency source: `if $INPUT_DOTNET_VALIDATION_ENABLED || $INPUT_JAVA_VALIDATION_ENABLED; then`, `if $INPUT_SUSHI_USE_CONFIG_DEPENDENCIES; then` and `elif $INPUT_SUSHI_ENABLED && ...` expand inputs unquoted in command position.

Locations:

- `action.yml:185`
- `action.yml:200`
- `action.yml:219`

### script-injection (severity: high)

Rule (b) – Validate FHIR version support: `if $INPUT_DOTNET_VALIDATION_ENABLED || $INPUT_JAVA_VALIDATION_ENABLED; then`. Input values are expanded unquoted and run as commands.

Locations:

- `action.yml:302`

### script-injection (severity: high)

Rule (b) – Set FHIR specification and restore dependencies: `if $INPUT_DOTNET_VALIDATION_ENABLED; then`. The input value is expanded unquoted and run as a command.

Locations:

- `action.yml:408`

### script-injection (severity: high)

Rule (b) – Check if npm is installed: `if $INPUT_SUSHI_ENABLED; then`. The input value is expanded unquoted and run as a command.

Locations:

- `action.yml:550`

### script-injection (severity: high)

Rule (b) – Check npm Version: `if $INPUT_SUSHI_ENABLED; then`. The input value is expanded unquoted and run as a command.

Locations:

- `action.yml:563`

### script-injection (severity: high)

Rule (b) – Install fsh-sushi: `sudo npm install -g fsh-sushi@$SUSHI_VERSION` expands an input unquoted under sudo, and `if $INPUT_SUSHI_ENABLED; then` runs an input value as a command.

Locations:

- `action.yml:573`
- `action.yml:575`

### script-injection (severity: high)

Rule (b) – Check SUSHI version: `if $INPUT_SUSHI_ENABLED; then`. The input value is expanded unquoted and run as a command.

Locations:

- `action.yml:584`

### script-injection (severity: high)

Rule (b) – Generate conformance resources with SUSHI: `if $INPUT_SUSHI_ENABLED; then` and `sushi $INPUT_SUSHI_OPTIONS` expand inputs unquoted.

Locations:

- `action.yml:593`
- `action.yml:594`

### script-injection (severity: high)

Rule (b) – Pre-processing FHIR profiles (Slicing open-closed): `if $CLOSE_SLICING_FOR_VALIDATION; then`. The input value is expanded unquoted and run as a command.

Locations:

- `action.yml:607`

### script-injection (severity: high)

Rule (b) – Configure Firely Terminal validator engine: `if $INPUT_DOTNET_VALIDATION_ENABLED; then`. The input value is expanded unquoted and run as a command.

Locations:

- `action.yml:662`

### script-injection (severity: high)

Rule (b) – Run Quality Control checks: `if $INPUT_DOTNET_VALIDATION_ENABLED; then`, `if echo $INPUT_EXPECTED_FAILS | grep -w -q ...` and `fhir check $INPUT_PATH_TO_QUALITY_CONTROL_RULES $CHECK_ARGS` all expand input-derived values unquoted.

Locations:

- `action.yml:688`
- `action.yml:710`
- `action.yml:716`

### script-injection (severity: high)

Rule (b) – Remove package cache to enable the Java validator to create snapshots: `if $INPUT_JAVA_SNAPSHOT_ENABLED; then`. The input value is expanded unquoted and run as a command.

Locations:

- `action.yml:758`

### script-injection (severity: high)

Rule (b) – Check if Java is installed: `if $INPUT_JAVA_VALIDATION_ENABLED; then`. The input value is expanded unquoted and run as a command.

Locations:

- `action.yml:768`

### script-injection (severity: high)

Rule (b) – Validate all conformance resources in scope of the repository: `if $INPUT_JAVA_VALIDATION_ENABLED; then`, `for p in $INPUT_PATH_TO_CONFORMANCE_RESOURCES;`, `echo $INPUT_EXPECTED_FAILS | grep ...` and `java -jar validator_cli.jar $GITHUB_WORKSPACE/$p*.xml ... -version $FHIR_VERSION $INPUT_JAVA_VALIDATION_OPTIONS ...` expand input/env-derived values unquoted.

Locations:

- `action.yml:806`
- `action.yml:830`
- `action.yml:845`
- `action.yml:846`

### script-injection (severity: high)

Rule (b) – Validate all example resources in scope of the repository: `if $INPUT_JAVA_VALIDATION_ENABLED; then`, `for p in $INPUT_PATH_TO_CONFORMANCE_RESOURCES;`, `for p in $PATH_TO_EXAMPLES;`, `echo $INPUT_EXPECTED_FAILS | grep ...` and `java -jar validator_cli.jar ... -version $FHIR_VERSION $INPUT_JAVA_VALIDATION_OPTIONS ...` expand input/env-derived values unquoted.

Locations:

- `action.yml:881`
- `action.yml:899`
- `action.yml:904`
- `action.yml:914`
- `action.yml:915`

### script-injection (severity: high)

Rule (b) – Re-run SUSHI to restore conformance resources after slicing modification: `if $CLOSE_SLICING_FOR_VALIDATION && $INPUT_SUSHI_ENABLED; then` and `sushi $INPUT_SUSHI_OPTIONS` expand inputs unquoted.

Locations:

- `action.yml:945`
- `action.yml:947`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, github-env-injection, script-injection

**Notes:**

All findings are fixed in action.yml. Nothing has been run or tested.

- **unpinned-uses:** actions/upload-artifact@v7 is now pinned to cf430e030ddbb5b0abf93d22962f4752f3646cd9 with a `# v7` comment.
- **github-env-injection:** the cache key is now built in a variable and passed through `printf '%s' | tr -d '\n\r'` before it goes to $GITHUB_OUTPUT. The sha256sum result is saved first, so a failure there still stops the step.
- **script-injection, true/false flags:** every check like `if $INPUT_X; then` (also the `||`, `&&` and `elif` forms) is now `[ "$X" = "true" ]`. Upstream ran the value as a command, so only `true` ever passed; the new check behaves the same way.
- **script-injection, single values:** the Firely Terminal version, the Simplifier email and password (passed as `"email=..."` and `"password=..."`), the SUSHI version (`"fsh-sushi@$SUSHI_VERSION"`), the download URL, `-version "$FHIR_VERSION"` and the cache path given to `rm -rf` are now quoted. The optional quality-control rules path uses `${VAR:+"$VAR"}` so it drops out when empty.
- **script-injection, lists:** `fhir check` arguments are now built as a bash array, which keeps the report path as one argument. `SUSHI_OPTIONS`, `JAVA_VALIDATION_OPTIONS`, `PATH_TO_CONFORMANCE_RESOURCES`, `PATH_TO_EXAMPLES` and the `-ig` dependency lists from package.json are each split into an array with an unquoted assignment (`arr=($VAR)`). That splits and expands wildcards exactly as upstream's unquoted use did. The `-ig` workspace arguments are now built as arrays too. File patterns are written `"$GITHUB_WORKSPACE/$p"*.xml`, so the folder part is quoted and the `*` still matches files.
- **script-injection, EXPECTED_FAILS:** the check now uses `printf '%s\n' "$INPUT_EXPECTED_FAILS" | grep -w -q` instead of an unquoted echo.
- **script-injection, eval:** `eval echo` on JAVA_VALIDATOR_DOWNLOAD_LOCATION is gone. The script now replaces only the literal `$JAVA_VALIDATOR_VERSION` or `${JAVA_VALIDATOR_VERSION}` placeholder with the version input. The default URL resolves the same as before. A custom URL that relied on other shell variables or commands being expanded will no longer have them expanded.

### Iteration 2

**Fixes applied:** github-env-injection

**Notes:**

In the qc step of action.yml, REPORT is now passed through printf '%s' | tr -d '\n\r' (stored as safeReport) before it is written to $GITHUB_OUTPUT. Nothing else in the step changed. The rc capture and the set +e/-e handling stay as they were.

### Iteration 1

**Fixes applied:** publish-gate

**Notes:**

action.yml line 390: changed to 'packageJsonHash=$(sha256sum package.json | cut -c1-16) || true' so a sha256sum failure doesn't abort the step, matching upstream's echo-embedded behaviour. Nothing else changed.

