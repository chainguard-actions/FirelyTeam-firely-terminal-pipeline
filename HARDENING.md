<!-- markdownlint-disable -->

# Hardening Report: FirelyTeam--firely-terminal-pipeline/v0.8.15

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **FirelyTeam--firely-terminal-pipeline/v0.8.15** was hardened automatically. 1 finding(s) were identified and resolved across 3 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Multiple unquoted shell variable expansions of untrusted input data (rule b violations) in action.yml. Variables sourced from inputs.* are set via env: blocks but then expanded without double-quoting in run: shell commands, allowing shell metacharacter injection by a caller:
1. `fhir login email=$INPUT_SIMPLIFIER_USERNAME password=$INPUT_SIMPLIFIER_PASSWORD` — unquoted expansions of inputs.SIMPLIFIER_USERNAME and inputs.SIMPLIFIER_PASSWORD
2. `sushi $INPUT_SUSHI_OPTIONS` — unquoted expansion of inputs.SUSHI_OPTIONS (two steps)
3. `fhir check $INPUT_PATH_TO_QUALITY_CONTROL_RULES` — unquoted expansion of inputs.PATH_TO_QUALITY_CONTROL_RULES
4. `for p in $INPUT_PATH_TO_CONFORMANCE_RESOURCES` — unquoted expansion of inputs.PATH_TO_CONFORMANCE_RESOURCES (two steps)
5. `java -jar validator_cli.jar ... $INPUT_JAVA_VALIDATION_OPTIONS ...` — unquoted expansion of inputs.JAVA_VALIDATION_OPTIONS (two steps)
6. `wget -q $JAVA_VALIDATOR_DOWNLOAD_LOCATION` — unquoted expansion of inputs.JAVA_VALIDATOR_DOWNLOAD_LOCATION
7. `JAVA_VALIDATOR_DOWNLOAD_LOCATION=$(eval echo "$JAVA_VALIDATOR_DOWNLOAD_LOCATION")` — eval with a user-controlled variable (inputs.JAVA_VALIDATOR_DOWNLOAD_LOCATION) enables arbitrary command execution

Locations:

- `action.yml:155`
- `action.yml:175`
- `action.yml:430`
- `action.yml:490`
- `action.yml:530`
- `action.yml:545`
- `action.yml:560`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection

**Notes:**

Fixed all 7 script injection vulnerabilities in action.yml:
1. fhir login: quoted email and password arguments as "email=$INPUT_SIMPLIFIER_USERNAME" "password=$INPUT_SIMPLIFIER_PASSWORD"
2. sushi $INPUT_SUSHI_OPTIONS (2 steps): replaced with IFS+read -ra array splitting and sushi "${sushi_args[@]}"
3. fhir check $INPUT_PATH_TO_QUALITY_CONTROL_RULES: replaced with ${INPUT_PATH_TO_QUALITY_CONTROL_RULES:+"$INPUT_PATH_TO_QUALITY_CONTROL_RULES"} to safely handle optional arg
4. for p in $INPUT_PATH_TO_CONFORMANCE_RESOURCES (2 steps): replaced with IFS+read -ra array splitting and "${conformance_paths[@]}"
5. java -jar ... $INPUT_JAVA_VALIDATION_OPTIONS (2 steps): replaced with IFS+read -ra array splitting and "${java_validation_opts[@]}"
6. wget -q $JAVA_VALIDATOR_DOWNLOAD_LOCATION: now uses wget -q "$JAVA_VALIDATOR_DOWNLOAD_LOCATION"
7. eval echo "$JAVA_VALIDATOR_DOWNLOAD_LOCATION": replaced with safe bash string substitution "${JAVA_VALIDATOR_DOWNLOAD_LOCATION//'$JAVA_VALIDATOR_VERSION'/$JAVA_VALIDATOR_VERSION}" to expand the version placeholder without using eval

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed all 5 script-injection findings in action.yml: (1) Line 117: Quoted $FIRELY_TERMINAL_VERSION in dotnet tool install command. (2) Line 390: Quoted fsh-sushi@$SUSHI_VERSION in npm install command. (3) Line 468: Quoted $INPUT_EXPECTED_FAILS in echo | grep pipeline in QC checks step. (4) Line 545: Quoted $INPUT_EXPECTED_FAILS in echo | grep; converted LOCAL_IG_PARAMETERS from string to bash array with separate -ig and path elements; converted UNESCPAED_IG_DEPENDENCIES to bash array via IFS read; expanded both with "${array[@]}" in java command. (5) Line 620: Same fixes as #4 but for COMBINED_IG_PARAMETERS in the example resources validation step.

### Iteration 3

**Fixes applied:** script-injection

**Notes:**

Fixed all script injection vulnerabilities in action.yml by replacing bare `if $VARIABLE` patterns with properly quoted `[ "$VARIABLE" = 'true' ]` comparisons. Affected variables: INPUT_DOTNET_VALIDATION_ENABLED, INPUT_TERMINOLOGY_SERVICE_BFARM_ENABLED, INPUT_JAVA_VALIDATION_ENABLED, INPUT_SUSHI_ENABLED, INPUT_SUSHI_USE_CONFIG_DEPENDENCIES, INPUT_JAVA_SNAPSHOT_ENABLED, CLOSE_SLICING_FOR_VALIDATION. All 25 occurrences were fixed across multiple steps including: Check if .NET is installed, Check .NET SDK Version, Install Firely.Terminal, Check Firely Terminal Version, Simplifier login, Detect FHIR version, Validate FHIR version support, Set FHIR specification and restore dependencies, Check if npm is installed, Check npm Version, Install fsh-sushi, Check SUSHI version, Generate conformance resources with SUSHI, Pre-processing FHIR profiles, Configure Firely Terminal validator engine, Run Quality Control checks, Report Success, Remove package cache, Check if Java is installed, Download Java Validator, Validate all conformance resources, Validate all example resources, and Re-run SUSHI.

