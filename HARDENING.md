<!-- markdownlint-disable -->

# Hardening Report: FirelyTeam--firely-terminal-pipeline/v0.11.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **FirelyTeam--firely-terminal-pipeline/v0.11.1** was hardened automatically. 7 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

action.yml composite step uses a mutable tag instead of a full commit SHA: `uses: actions/upload-artifact@v7` (step 'Upload Quality Control report'). The other uses: lines (actions/cache/restore, actions/cache/save, patrick-werner/validation-outcome-markdown-renderer) are pinned to SHAs.

Locations:

- `action.yml:862`

### github-env-injection (severity: high)

Step 'Determine FHIR package cache key' writes input-derived values to $GITHUB_OUTPUT without sanitization: `echo "key=fhir-packages-${RUNNER_OS}-ft${installedFirelyTerminalVersion:-$FIRELY_TERMINAL_VERSION}-zts${INPUT_TERMINOLOGY_SERVICE_BFARM_ENABLED}-..." >> "$GITHUB_OUTPUT"`. FIRELY_TERMINAL_VERSION and INPUT_TERMINOLOGY_SERVICE_BFARM_ENABLED come from inputs.* and are not passed through `printf '%s' ... | tr -d '\n\r'` before the write, so a newline in either value can inject extra outputs.

Locations:

- `action.yml:426`

### script-injection (severity: high)

Rule (b), eval of input: step 'Download Java Validator' runs `JAVA_VALIDATOR_DOWNLOAD_LOCATION=$(eval echo "$JAVA_VALIDATOR_DOWNLOAD_LOCATION")`. That value comes from inputs.JAVA_VALIDATOR_DOWNLOAD_LOCATION, so eval runs any $(...), backticks or ';' it contains. The next line, `wget -q $JAVA_VALIDATOR_DOWNLOAD_LOCATION -O validator_cli.jar`, is also unquoted.

Locations:

- `action.yml:908`
- `action.yml:911`

### script-injection (severity: high)

Rule (b): many steps run boolean inputs as unquoted commands, e.g. `if $INPUT_DOTNET_VALIDATION_ENABLED || $INPUT_TERMINOLOGY_SERVICE_BFARM_ENABLED; then`, `if $INPUT_SUSHI_ENABLED; then`, `if $INPUT_JAVA_VALIDATION_ENABLED; then`, `if $CLOSE_SLICING_FOR_VALIDATION; then`, `if $INPUT_JAVA_SNAPSHOT_ENABLED; then`, `elif $INPUT_REPORT_RENDERER_ENABLED; then`, `if $INPUT_SUSHI_USE_CONFIG_DEPENDENCIES; then`. Each input value is run as a shell command, so a caller-supplied value runs arbitrary programs. Compare against the literal string instead, e.g. `[[ "$VAR" == "true" ]]`.

Locations:

- `action.yml:106`
- `action.yml:121`
- `action.yml:132`
- `action.yml:147`
- `action.yml:158`

### script-injection (severity: high)

Rule (b): input-derived env vars are expanded unquoted in Firely/SUSHI steps: `dotnet tool install --global Firely.Terminal --version $FIRELY_TERMINAL_VERSION`, `fhir login email=$INPUT_SIMPLIFIER_USERNAME password=$INPUT_SIMPLIFIER_PASSWORD`, `sudo npm install -g fsh-sushi@$SUSHI_VERSION`, `sushi $INPUT_SUSHI_OPTIONS` (in both the generate and re-run steps), `rm -rf $fhirCacheLocation`.

Locations:

- `action.yml:135`
- `action.yml:159`
- `action.yml:677`
- `action.yml:690`
- `action.yml:884`
- `action.yml:1124`

### script-injection (severity: high)

Rule (b): step 'Run Quality Control checks' expands inputs unquoted: `if echo $INPUT_EXPECTED_FAILS | grep -w -q VALIDATION_CONFORMANCE_DOTNET` and `fhir check $INPUT_PATH_TO_QUALITY_CONTROL_RULES $CHECK_ARGS`.

Locations:

- `action.yml:804`
- `action.yml:811`

### script-injection (severity: high)

Rule (b): the Java validation steps ('Validate all conformance resources...' and 'Validate all example resources...') expand workflow-controlled vars unquoted: `for p in $INPUT_PATH_TO_CONFORMANCE_RESOURCES;`, `for p in $PATH_TO_EXAMPLES;`, `echo $INPUT_EXPECTED_FAILS | grep ...`, `echo $IG_DEPENDENCIES`, and `java -jar validator_cli.jar $GITHUB_WORKSPACE/$p*.xml ... -version $FHIR_VERSION $INPUT_JAVA_VALIDATION_OPTIONS $UNESCPAED_IG_DEPENDENCIES $LOCAL_IG_PARAMETERS`.

Locations:

- `action.yml:965`
- `action.yml:980`
- `action.yml:1057`
- `action.yml:1072`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, github-env-injection, script-injection

**Notes:**

All changes are in action.yml; nothing was run or tested. (1) Pinned actions/upload-artifact@v7 to its commit SHA cf430e030ddbb5b0abf93d22962f4752f3646cd9, keeping '# v7' as a comment. (2) In 'Determine FHIR package cache key', the Firely Terminal version and the BfArM flag now have newlines and carriage returns stripped before the key is written to GITHUB_OUTPUT. (3) Every boolean input that was run as a command ('if $INPUT_X; then', about 25 places) is now compared against the literal string with [[ "$VAR" == "true" ]]. Only the exact lowercase value 'true' now enables a feature, so a hand-typed 'True' or '1' counts as off. (4) Removed the eval of JAVA_VALIDATOR_DOWNLOAD_LOCATION. The $JAVA_VALIDATOR_VERSION / ${JAVA_VALIDATOR_VERSION} placeholder is now filled in by bash text replacement (replacement quoted), and the URL is quoted for wget. A custom URL that relied on eval to expand any other variable will no longer have it expanded. (5) Single values are now quoted: the Firely Terminal version, the fhir login email=/password= arguments, fsh-sushi@<version>, $USER, and the folder given to rm -rf. The optional quality-control rules path uses ${VAR:+"$VAR"} so an empty input adds no argument. The fhir check flags are built as an array of separate quoted arguments. EXPECTED_FAILS checks use printf '%s\n' "$VAR" | grep -w. (6) List inputs (SUSHI_OPTIONS, JAVA_VALIDATION_OPTIONS, PATH_TO_CONFORMANCE_RESOURCES, PATH_TO_EXAMPLES, and the IG dependencies read from package.json) are assigned to arrays unquoted, which splits and globs exactly as upstream did, then expanded quoted. In the Java validator calls the workspace and path parts of the *.xml/*.json patterns are quoted while the glob still works, -version is quoted, and the -ig folder arguments are built as quoted arrays, so folder names containing spaces now work.

