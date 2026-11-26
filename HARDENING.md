<!-- markdownlint-disable -->

# Hardening Report: FirelyTeam--firely-terminal-pipeline/v0.11.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **FirelyTeam--firely-terminal-pipeline/v0.11.0** was hardened automatically. 8 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

action.yml has a composite step that uses a mutable tag instead of a full 40-character commit SHA: `uses: actions/upload-artifact@v7` (step 'Upload Quality Control report'). The other uses: lines (actions/cache/restore, actions/cache/save, patrick-werner/validation-outcome-markdown-renderer) are pinned to SHAs.

Locations:

- `action.yml:770`

### script-injection (severity: high)

Rule (b), step 'Download Java Validator': the input JAVA_VALIDATOR_DOWNLOAD_LOCATION is passed through eval: `JAVA_VALIDATOR_DOWNLOAD_LOCATION=$(eval echo "$JAVA_VALIDATOR_DOWNLOAD_LOCATION")`. Anything in the value such as `$(...)`, backticks or `;` runs as a command. The result is then expanded unquoted in `wget -q $JAVA_VALIDATOR_DOWNLOAD_LOCATION -O validator_cli.jar`. Fix: drop eval, substitute $JAVA_VALIDATOR_VERSION explicitly (for example with ${VAR//\$JAVA_VALIDATOR_VERSION/$JAVA_VALIDATOR_VERSION}) and quote the variable.

Locations:

- `action.yml:806`
- `action.yml:809`

### script-injection (severity: high)

Rule (b): boolean inputs routed through env are expanded unquoted and run as commands, e.g. `if $INPUT_DOTNET_VALIDATION_ENABLED || $INPUT_TERMINOLOGY_SERVICE_BFARM_ENABLED; then`, `if $INPUT_SUSHI_ENABLED; then`, `if $INPUT_JAVA_VALIDATION_ENABLED; then`, `if $INPUT_JAVA_SNAPSHOT_ENABLED; then`, `if $CLOSE_SLICING_FOR_VALIDATION; then`. Whatever the input holds is run as a command. This pattern appears in almost every run: step (Check if .NET is installed, Check .NET SDK Version, Install Firely.Terminal, Check Firely Terminal Version, Simplifier login, Detect FHIR version, Validate FHIR version support, Set FHIR specification, the npm/SUSHI steps, Pre-processing, Configure validator engine, Run Quality Control checks, Remove package cache, Check if Java is installed, Download Java Validator, both Java validation steps, Re-run SUSHI). Fix: compare quoted values, e.g. `if [[ "$INPUT_DOTNET_VALIDATION_ENABLED" == "true" ]]`.

Locations:

- `action.yml:106`
- `action.yml:121`
- `action.yml:132`
- `action.yml:147`
- `action.yml:184`
- `action.yml:636`

### script-injection (severity: high)

Rule (b): unquoted expansion of inputs in command arguments. Examples: `dotnet tool install --global Firely.Terminal --version $FIRELY_TERMINAL_VERSION` (Install Firely.Terminal), `fhir login email=$INPUT_SIMPLIFIER_USERNAME password=$INPUT_SIMPLIFIER_PASSWORD` (Simplifier login), `sudo npm install -g fsh-sushi@$SUSHI_VERSION` (Install fsh-sushi).

Locations:

- `action.yml:135`
- `action.yml:159`
- `action.yml:606`

### script-injection (severity: high)

Rule (b): unquoted input expansion in 'Generate conformance resources with SUSHI' and 'Re-run SUSHI': `sushi $INPUT_SUSHI_OPTIONS`. This is not a shellcheck-annotated array assignment, so the exemption does not apply.

Locations:

- `action.yml:624`
- `action.yml:955`

### script-injection (severity: high)

Rule (b), step 'Run Quality Control checks': inputs are expanded unquoted in `echo $INPUT_EXPECTED_FAILS | grep -w -q VALIDATION_CONFORMANCE_DOTNET` and `fhir check $INPUT_PATH_TO_QUALITY_CONTROL_RULES $CHECK_ARGS`.

Locations:

- `action.yml:748`

### script-injection (severity: high)

Rule (b), steps 'Validate all conformance resources' and 'Validate all example resources': inputs are expanded unquoted. Examples: `for p in $INPUT_PATH_TO_CONFORMANCE_RESOURCES;`, `for p in $PATH_TO_EXAMPLES;`, `echo $INPUT_EXPECTED_FAILS | grep ...` and `java -jar validator_cli.jar $GITHUB_WORKSPACE/$p*.xml $GITHUB_WORKSPACE/$p*.json -version $FHIR_VERSION $INPUT_JAVA_VALIDATION_OPTIONS $UNESCPAED_IG_DEPENDENCIES ...`.

Locations:

- `action.yml:862`
- `action.yml:925`

### github-env-injection (severity: high)

Step 'Determine FHIR package cache key' writes input-derived values to $GITHUB_OUTPUT without the `printf '%s' ... | tr -d '\n\r'` sanitization: `echo "key=fhir-packages-${RUNNER_OS}-ft${installedFirelyTerminalVersion:-$FIRELY_TERMINAL_VERSION}-zts${INPUT_TERMINOLOGY_SERVICE_BFARM_ENABLED}-..." >> "$GITHUB_OUTPUT"`. FIRELY_TERMINAL_VERSION and INPUT_TERMINOLOGY_SERVICE_BFARM_ENABLED come from inputs.*, and RUNNER_OS is inherited process env. A newline in any of them can inject extra step outputs.

Locations:

- `action.yml:399`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, github-env-injection

**Notes:**

All fixes are in hardened/action/action.yml.
- unpinned-uses: actions/upload-artifact@v7 is now pinned to cf430e030ddbb5b0abf93d22962f4752f3646cd9 # v7.
- script-injection (eval): removed `eval echo` from the Java validator download URL. The `$JAVA_VALIDATOR_VERSION` and `${JAVA_VALIDATOR_VERSION}` placeholders are now replaced literally with quoted bash pattern substitution, and the URL is quoted in the wget call.
- script-injection (booleans): every `if $INPUT_X;` (and the `||`/`&&` forms) now compares the quoted value, e.g. `[[ "$X" == "true" ]]`. This covers all the listed steps, including the SUSHI_USE_CONFIG_DEPENDENCIES and SUSHI_ENABLED checks in the FHIR version detection step.
- script-injection (single-value args): quoted the variables in the dotnet tool install `--version` argument, the fhir login `email=`/`password=` arguments, `npm install fsh-sushi@...`, `chown $USER` and `rm -rf` on the cache location. PATH_TO_QUALITY_CONTROL_RULES is passed as ${VAR:+"$VAR"} because it is optional.
- script-injection (list inputs): SUSHI_OPTIONS, JAVA_VALIDATION_OPTIONS, PATH_TO_CONFORMANCE_RESOURCES and PATH_TO_EXAMPLES go through shellcheck-annotated unquoted array assignments. These split and glob exactly as upstream's unquoted $VAR did, and the arrays are then expanded quoted. The internal CHECK_ARGS/NO_FAIL_ARGS, LOCAL_IG_PARAMETERS/COMBINED_IG_PARAMETERS and IG dependency strings are now bash arrays too. The `echo $INPUT_EXPECTED_FAILS | grep -w` checks now use `printf '%s\n' "$INPUT_EXPECTED_FAILS"`. In the Java validator file globs, `"$GITHUB_WORKSPACE/$p"*.xml` keeps the glob unquoted while quoting the path prefix.
- github-env-injection: the FHIR package cache step builds the path and key in variables, strips newlines with `printf '%s' | tr -d '\n\r'`, then writes them to $GITHUB_OUTPUT. Upstream ran the `fhir cache location` and `sha256sum` commands inside echo, so a failure did not stop the step; the separate captures use `|| true` to keep that behaviour.

