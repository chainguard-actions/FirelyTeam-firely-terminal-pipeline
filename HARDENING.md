<!-- markdownlint-disable -->

# Hardening Report: FirelyTeam--firely-terminal-pipeline/v0.11.3

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **FirelyTeam--firely-terminal-pipeline/v0.11.3** was hardened automatically. 4 finding(s) were identified and resolved across 3 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

action.yml composite step 'Upload Quality Control report' uses a mutable tag instead of a full commit SHA: `uses: actions/upload-artifact@v7`. The other uses: refs (actions/cache/restore, actions/cache/save, patrick-werner/validation-outcome-markdown-renderer) are pinned to a SHA.

Locations:

- `action.yml:800`

### github-env-injection (severity: high)

Step 'Determine FHIR package cache key' writes workflow-controlled input values to $GITHUB_OUTPUT with no `printf '%s' ... | tr -d '\n\r'` step first: `echo "key=fhir-packages-${RUNNER_OS}-ft${installedFirelyTerminalVersion:-$FIRELY_TERMINAL_VERSION}-zts${INPUT_TERMINOLOGY_SERVICE_BFARM_ENABLED}-..." >> "$GITHUB_OUTPUT"`. FIRELY_TERMINAL_VERSION comes from inputs.FIRELY_TERMINAL_VERSION and INPUT_TERMINOLOGY_SERVICE_BFARM_ENABLED comes from inputs.TERMINOLOGY_SERVICE_BFARM_ENABLED. A newline in either value can inject extra step outputs.

Locations:

- `action.yml:445`

### script-injection (severity: high)

Rule (b) plus an eval: step 'Download Java Validator' runs `JAVA_VALIDATOR_DOWNLOAD_LOCATION=$(eval echo "$JAVA_VALIDATOR_DOWNLOAD_LOCATION")`, which evaluates the value of inputs.JAVA_VALIDATOR_DOWNLOAD_LOCATION as shell code, so $(...) or ; in the input runs as a command. It then runs `wget -q $JAVA_VALIDATOR_DOWNLOAD_LOCATION -O validator_cli.jar` with the variable unquoted. `if $INPUT_JAVA_VALIDATION_ENABLED; then` also runs an input value as a command.

Locations:

- `action.yml:857`

### script-injection (severity: high)

Rule (b): many steps expand input-derived env vars unquoted, and several run them directly as commands. Every boolean flag is tested with `if $INPUT_X; then`, which runs the input's value as a shell command. This affects INPUT_DOTNET_VALIDATION_ENABLED, INPUT_TERMINOLOGY_SERVICE_BFARM_ENABLED, INPUT_JAVA_VALIDATION_ENABLED, INPUT_SUSHI_ENABLED, INPUT_SUSHI_USE_CONFIG_DEPENDENCIES, CLOSE_SLICING_FOR_VALIDATION, INPUT_JAVA_SNAPSHOT_ENABLED and INPUT_REPORT_RENDERER_ENABLED in the steps 'Check if .NET is installed', 'Check .NET SDK Version', 'Install Firely.Terminal', 'Check Firely Terminal Version', 'Simplifier login', 'Detect FHIR version...', 'Validate FHIR version support', 'Set FHIR specification...', the npm/SUSHI steps, 'Pre-processing FHIR profiles', 'Configure Firely Terminal validator engine', 'Run Quality Control checks', 'Remove package cache...', 'Check if Java is installed', both Java validation steps and 'Re-run SUSHI...'. Other unquoted expansions: `dotnet tool install --global Firely.Terminal --version $FIRELY_TERMINAL_VERSION`; `fhir login email=$INPUT_SIMPLIFIER_USERNAME password=$INPUT_SIMPLIFIER_PASSWORD`; `sudo npm install -g fsh-sushi@$SUSHI_VERSION`; `sushi $INPUT_SUSHI_OPTIONS` (2 places); `echo $INPUT_EXPECTED_FAILS | grep ...` (3 places); `fhir check $INPUT_PATH_TO_QUALITY_CONTROL_RULES $CHECK_ARGS`; `for p in $INPUT_PATH_TO_CONFORMANCE_RESOURCES` / `for p in $PATH_TO_EXAMPLES`; `java -jar validator_cli.jar ... -version $FHIR_VERSION $INPUT_JAVA_VALIDATION_OPTIONS ...`.

Locations:

- `action.yml:110`
- `action.yml:140`
- `action.yml:163`
- `action.yml:720`
- `action.yml:745`
- `action.yml:790`
- `action.yml:900`
- `action.yml:960`
- `action.yml:1010`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, github-env-injection, script-injection

**Notes:**

All changes are in action.yml.
- unpinned-uses: actions/upload-artifact@v7 is now pinned to cf430e030ddbb5b0abf93d22962f4752f3646cd9 (# v7).
- github-env-injection: the 'Determine FHIR package cache key' step builds the cache path and cache key in variables first, strips CR/LF with printf '%s' | tr -d '\n\r', and then writes them to $GITHUB_OUTPUT. A bare assignment runs the sha256sum command, so a failure there still stops the step.
- script-injection: every `if $INPUT_X; then` style test is now `[ "$X" = "true" ]`. For true/false values it behaves as before, but the input is no longer run as a command. The `eval echo` on JAVA_VALIDATOR_DOWNLOAD_LOCATION is gone. The $JAVA_VALIDATOR_VERSION / ${JAVA_VALIDATOR_VERSION} placeholder is now replaced by plain text substitution, which keeps the default URL working, and wget gets the URL quoted. These are now quoted: the dotnet tool version, the fhir login email=/password= arguments, the npm fsh-sushi@version and $USER in chown, and rm -rf of the cache location. EXPECTED_FAILS is checked with printf '%s\n' "$X" | grep instead of echo $X. The fhir check arguments are now a bash array, and the optional QC rules path uses ${VAR:+"$VAR"}. The list inputs (SUSHI_OPTIONS, PATH_TO_CONFORMANCE_RESOURCES, PATH_TO_EXAMPLES, JAVA_VALIDATION_OPTIONS) are put into arrays with an unquoted assignment, so they split and glob exactly as upstream did, and are passed as "${arr[@]}". The -ig parameter strings are arrays too, and FHIR_VERSION is quoted in the java calls. IG_DEPENDENCIES comes from package.json rather than an input, and is still expanded unquoted as upstream had it.

### Iteration 2

**Fixes applied:** github-env-injection

**Notes:**

Both writes in action.yml now strip newlines first. Not tested.

1. The detected FHIR version (fhirVersion) is cleaned with printf | tr -d '\n\r' just before it is written to GITHUB_ENV.
2. The REPORT path is cleaned the same way into safe_report before it is written to GITHUB_OUTPUT.

No other behaviour changed.

### Iteration 1

**Fixes applied:** publish-gate

**Notes:**

Added '|| true' to the packageHash capture on action.yml:393. Upstream ran sha256sum inside an echo/printf "$(...)", so a failure there didn't stop the step; this change keeps that behaviour. The hash is still captured as before. Nothing else in the file was changed, and I didn't run anything to test it.

