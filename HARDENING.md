<!-- markdownlint-disable -->

# Hardening Report: FirelyTeam--firely-terminal-pipeline/v0.11.2

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **FirelyTeam--firely-terminal-pipeline/v0.11.2** was hardened automatically. 8 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

action.yml has a composite step that uses a mutable tag instead of a full commit SHA: `uses: actions/upload-artifact@v7` (step 'Upload Quality Control report'). The other uses: references (actions/cache/restore, actions/cache/save, patrick-werner/validation-outcome-markdown-renderer) are pinned to 40-character SHAs.

Locations:

- `action.yml:752`

### script-injection (severity: high)

Rule (b): in the 'Download Java Validator' step, the input JAVA_VALIDATOR_DOWNLOAD_LOCATION (inputs.JAVA_VALIDATOR_DOWNLOAD_LOCATION, controlled by the calling workflow) goes through eval: `JAVA_VALIDATOR_DOWNLOAD_LOCATION=$(eval echo "$JAVA_VALIDATOR_DOWNLOAD_LOCATION")`. Any `;`, `$(...)` or backticks in the input run as shell commands. The value is then expanded unquoted: `wget -q $JAVA_VALIDATOR_DOWNLOAD_LOCATION -O validator_cli.jar`.

Locations:

- `action.yml:794`
- `action.yml:797`

### script-injection (severity: high)

Rule (b): many steps expand boolean inputs unquoted as the shell command itself, e.g. `if $INPUT_DOTNET_VALIDATION_ENABLED || $INPUT_TERMINOLOGY_SERVICE_BFARM_ENABLED; then`, `if $INPUT_SUSHI_ENABLED; then`, `if $INPUT_JAVA_VALIDATION_ENABLED; then`, `if $INPUT_SUSHI_USE_CONFIG_DEPENDENCIES; then`, `if $CLOSE_SLICING_FOR_VALIDATION; then`, `if $INPUT_JAVA_SNAPSHOT_ENABLED; then`, `elif $INPUT_REPORT_RENDERER_ENABLED; then`. The input value runs as a command, so a value like `true; malicious` executes code. Affected steps include 'Check if .NET is installed', 'Check .NET SDK Version', 'Install Firely.Terminal', 'Check Firely Terminal Version', 'Simplifier login', 'Detect FHIR version...', 'Validate FHIR version support', 'Set FHIR specification...', the npm/SUSHI steps, 'Pre-processing FHIR profiles', 'Configure Firely Terminal validator engine', 'Run Quality Control checks', the Java steps and 'Re-run SUSHI'.

Locations:

- `action.yml:106`
- `action.yml:121`
- `action.yml:132`
- `action.yml:147`
- `action.yml:184`
- `action.yml:199`
- `action.yml:301`
- `action.yml:408`
- `action.yml:549`
- `action.yml:606`
- `action.yml:661`
- `action.yml:687`
- `action.yml:767`
- `action.yml:790`

### script-injection (severity: high)

Rule (b): input-derived env vars are expanded unquoted in command arguments: `dotnet tool install --global Firely.Terminal --version $FIRELY_TERMINAL_VERSION` (Install Firely.Terminal); `fhir login email=$INPUT_SIMPLIFIER_USERNAME password=$INPUT_SIMPLIFIER_PASSWORD` (Simplifier login); `sudo npm install -g fsh-sushi@$SUSHI_VERSION` (Install fsh-sushi); `sushi $INPUT_SUSHI_OPTIONS` (Generate conformance resources with SUSHI, and again in Re-run SUSHI).

Locations:

- `action.yml:135`
- `action.yml:159`
- `action.yml:574`
- `action.yml:593`

### script-injection (severity: high)

Rule (b): the 'Run Quality Control checks' step expands inputs unquoted: `if echo $INPUT_EXPECTED_FAILS | grep -w -q VALIDATION_CONFORMANCE_DOTNET` and `fhir check $INPUT_PATH_TO_QUALITY_CONTROL_RULES $CHECK_ARGS`, where CHECK_ARGS contains $INPUT_DOTNET_FAIL_ON.

Locations:

- `action.yml:709`
- `action.yml:715`

### script-injection (severity: high)

Rule (b): the 'Validate all conformance resources' and 'Validate all example resources' steps expand inputs unquoted: `for p in $INPUT_PATH_TO_CONFORMANCE_RESOURCES;`, `for p in $PATH_TO_EXAMPLES;`, `echo $INPUT_EXPECTED_FAILS | grep ...`, and `java -jar validator_cli.jar $GITHUB_WORKSPACE/$p*.xml ... -version $FHIR_VERSION $INPUT_JAVA_VALIDATION_OPTIONS $UNESCPAED_IG_DEPENDENCIES ...`.

Locations:

- `action.yml:805`
- `action.yml:870`

### github-env-injection (severity: high)

The 'Determine FHIR package cache key' step writes values from inputs to $GITHUB_OUTPUT without the `printf '%s' ... | tr -d '\n\r'` sanitization step: `echo "key=fhir-packages-${RUNNER_OS}-ft${installedFirelyTerminalVersion:-$FIRELY_TERMINAL_VERSION}-zts${INPUT_TERMINOLOGY_SERVICE_BFARM_ENABLED}-..." >> "$GITHUB_OUTPUT"`. FIRELY_TERMINAL_VERSION and INPUT_TERMINOLOGY_SERVICE_BFARM_ENABLED come from inputs.*.

Locations:

- `action.yml:390`

### github-env-injection (severity: high)

The 'Detect FHIR version and determine dependency source' step writes `echo "FHIR_VERSION=$fhirVersion" >> "$GITHUB_ENV"` without sanitization. fhirVersion is not a literal computed in the script. It is read from repository-controlled package.json or sushi-config.yaml (through jq, yq, or a base64 round-trip), so a value with embedded newlines could inject arbitrary environment variables such as BASH_ENV into later steps.

Locations:

- `action.yml:287`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, github-env-injection

**Notes:**

Everything is in action.yml. Nothing was run or tested.

- unpinned-uses: actions/upload-artifact@v7 is now pinned to its commit SHA, cf430e030ddbb5b0abf93d22962f4752f3646cd9, with a '# v7' comment.
- script-injection, boolean inputs: inputs are no longer run as commands. Each `if $X` test is now a string comparison, `[ "$X" = "true" ]`. Upstream behaved the same way, since any value other than 'true' made the test fail.
- script-injection, download URL: JAVA_VALIDATOR_DOWNLOAD_LOCATION no longer goes through `eval`. Bash string substitution fills in the default's `$JAVA_VALIDATOR_VERSION` placeholder (or `${JAVA_VALIDATOR_VERSION}`), and the URL is passed to wget quoted. Any other `$` in a custom URL now stays as literal text instead of being expanded.
- script-injection, single values: these are now double-quoted: --version for dotnet tool install, the email=/password= arguments to `fhir login`, `fsh-sushi@$SUSHI_VERSION`, `$USER`, the cache path passed to `rm -rf`, and -version for the Java validator. The EXPECTED_FAILS check now uses `printf '%s\n' "$INPUT_EXPECTED_FAILS" | grep`.
- script-injection, Quality Control step: CHECK_ARGS and NO_FAIL_ARGS are now bash arrays, so the report path stays one argument even if it contains spaces. The optional rules path is passed as `${VAR:+"$VAR"}`, so it drops out when empty.
- script-injection, list inputs: SUSHI_OPTIONS, PATH_TO_CONFORMANCE_RESOURCES, PATH_TO_EXAMPLES, JAVA_VALIDATION_OPTIONS and the IG dependencies read from package.json are lists. Each is turned into an array with an unquoted assignment (`arr=($VAR)`), which splits and globs exactly as upstream's unquoted use did, then passed as `"${arr[@]}"`. The `-ig` parameters are now built as array items. The file globs are written as `"$GITHUB_WORKSPACE/$p"*.xml` so only the `*` suffix is a wildcard; a wildcard inside a path input now stays literal there.
- github-env-injection: the FHIR package cache key is built into a variable and stripped of newlines with `printf '%s' | tr -d '\n\r'` before it goes to GITHUB_OUTPUT. The detected FHIR version gets the same treatment before it goes to GITHUB_ENV. The sha256sum call now runs on its own line rather than inside the echo.

### Iteration 1

**Fixes applied:** publish-gate

**Notes:**

action.yml:391: added '|| true' to the packageJsonHash assignment ('packageJsonHash=$(sha256sum package.json | cut -c1-16) || true'). This restores upstream's behaviour of hiding a failed sha256sum inside echo's arguments. The output is still captured and written to GITHUB_OUTPUT as before. Nothing else was changed.

