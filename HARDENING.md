<!-- markdownlint-disable -->

# Hardening Report: FirelyTeam--firely-terminal-pipeline/v0.8.14

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **FirelyTeam--firely-terminal-pipeline/v0.8.14** was hardened automatically. 2 finding(s) were identified and resolved across 3 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (b): Multiple unquoted shell variable expansions of user-controlled inputs allow shell metacharacter injection. The following env vars — all sourced from action inputs — are expanded without double-quoting in run: shell commands:

1. `dotnet tool install --global Firely.Terminal --version $FIRELY_TERMINAL_VERSION` — FIRELY_TERMINAL_VERSION from inputs.FIRELY_TERMINAL_VERSION
2. `sudo npm install -g fsh-sushi@$SUSHI_VERSION` — SUSHI_VERSION from inputs.SUSHI_VERSION
3. `sushi $INPUT_SUSHI_OPTIONS` — INPUT_SUSHI_OPTIONS from inputs.SUSHI_OPTIONS
4. `fhir check $INPUT_PATH_TO_QUALITY_CONTROL_RULES` / `fhir check --fail $INPUT_PATH_TO_QUALITY_CONTROL_RULES` — from inputs.PATH_TO_QUALITY_CONTROL_RULES
5. `echo $INPUT_EXPECTED_FAILS | grep ...` — from inputs.EXPECTED_FAILS (unquoted)
6. `JAVA_VALIDATOR_DOWNLOAD_LOCATION=$(eval echo "$JAVA_VALIDATOR_DOWNLOAD_LOCATION")` — JAVA_VALIDATOR_DOWNLOAD_LOCATION from inputs.JAVA_VALIDATOR_DOWNLOAD_LOCATION; using eval with a user-controlled value enables arbitrary command execution
7. `wget -q $JAVA_VALIDATOR_DOWNLOAD_LOCATION -O validator_cli.jar` — unquoted after eval expansion
8. `java -jar validator_cli.jar $GITHUB_WORKSPACE/$p*.xml ... $INPUT_JAVA_VALIDATION_OPTIONS $UNESCPAED_IG_DEPENDENCIES $LOCAL_IG_PARAMETERS` — INPUT_JAVA_VALIDATION_OPTIONS from inputs.JAVA_VALIDATION_OPTIONS, unquoted (×4 occurrences)
9. `for p in $INPUT_PATH_TO_CONFORMANCE_RESOURCES` — from inputs.PATH_TO_CONFORMANCE_RESOURCES, unquoted (×3 occurrences)
10. `for p in $PATH_TO_EXAMPLES` — from inputs.PATH_TO_EXAMPLES, unquoted
11. `sushi $INPUT_SUSHI_OPTIONS` in post-processing step — same issue repeated

Locations:

- `action.yml:107`
- `action.yml:390`
- `action.yml:406`
- `action.yml:461`
- `action.yml:476`
- `action.yml:484`
- `action.yml:509`
- `action.yml:513`
- `action.yml:548`
- `action.yml:553`
- `action.yml:573`
- `action.yml:578`
- `action.yml:617`
- `action.yml:622`

### github-env-injection (severity: high)

The 'Detect FHIR version and determine dependency source' step writes the value of `fhirVersion` — extracted from repository files (package.json or sushi-config.yaml) without sanitization — to $GITHUB_ENV via `echo "FHIR_VERSION=$fhirVersion" >> "$GITHUB_ENV"`. If a repository file contains a newline in the fhirVersion field, an attacker who controls the repository content could inject arbitrary environment variable definitions into subsequent steps. The required sanitization step (`printf '%s' "$fhirVersion" | tr -d '\n\r'`) is absent before the write.

Locations:

- `action.yml:229`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection

**Notes:**

Fixed all script-injection issues in action.yml:
1. Quoted $FIRELY_TERMINAL_VERSION in dotnet tool install command
2. Quoted fsh-sushi@$SUSHI_VERSION in npm install command
3. Used ${INPUT_SUSHI_OPTIONS:+"$INPUT_SUSHI_OPTIONS"} in both sushi invocations (optional positional arg)
4. Used ${INPUT_PATH_TO_QUALITY_CONTROL_RULES:+"$INPUT_PATH_TO_QUALITY_CONTROL_RULES"} in fhir check commands (optional positional arg)
5. Added double quotes around $INPUT_EXPECTED_FAILS in all echo | grep usages
6. Replaced dangerous eval echo with safe bash parameter substitution ${JAVA_VALIDATOR_DOWNLOAD_LOCATION//'$JAVA_VALIDATOR_VERSION'/$JAVA_VALIDATOR_VERSION}
7. Added double quotes around $JAVA_VALIDATOR_DOWNLOAD_LOCATION in wget command
8. Used ${INPUT_JAVA_VALIDATION_OPTIONS:+"$INPUT_JAVA_VALIDATION_OPTIONS"} in all 4 java validator invocations (optional positional arg)
9. Added double quotes around $IG_DEPENDENCIES in echo | tr -d usages
10. Added double quotes around $FHIR_VERSION in java -version arguments

Fixed github-env-injection: Added sanitization step before writing fhirVersion to GITHUB_ENV using printf '%s' "$fhirVersion" | tr -d '\n\r' to strip newlines.

Note: The for loops over $INPUT_PATH_TO_CONFORMANCE_RESOURCES and $PATH_TO_EXAMPLES are intentionally word-split (space-separated path lists) and were left as-is since that is the intended behavior for iterating over multiple paths.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed all four script-injection findings in action.yml:

1. Simplifier login (line 130): Added double-quotes around $INPUT_SIMPLIFIER_USERNAME and $INPUT_SIMPLIFIER_PASSWORD in the fhir login command.

2. Conformance resource path loops (lines 601, 604): Replaced unquoted 'for p in $INPUT_PATH_TO_CONFORMANCE_RESOURCES' with 'while IFS= read -r p; do ... done <<< "$INPUT_PATH_TO_CONFORMANCE_RESOURCES"'. Quoted glob expansions as "$GITHUB_WORKSPACE/${p}"*.xml and "$GITHUB_WORKSPACE/${p}"*.json.

3. Examples path loop (lines 672, 675): Same fix applied to 'for p in $PATH_TO_EXAMPLES' and its glob expansions.

4. Boolean inputs as shell commands (lines 86, 97, 107, 120, 128, 166, 261, 316, 432, 461, 471, 543): Replaced all 'if $INPUT_DOTNET_VALIDATION_ENABLED', 'if $INPUT_JAVA_VALIDATION_ENABLED', 'if $INPUT_SUSHI_ENABLED', 'if $INPUT_TERMINOLOGY_SERVICE_BFARM_ENABLED', 'if $INPUT_JAVA_SNAPSHOT_ENABLED', 'if $CLOSE_SLICING_FOR_VALIDATION', and 'if $INPUT_SUSHI_USE_CONFIG_DEPENDENCIES' patterns with safe string comparisons using [ "$VAR" = "true" ]. Compound conditions with || and && were also properly converted.

### Iteration 3

**Fixes applied:** script-injection

**Notes:**

Fixed two instances of unquoted shell variable expansion in action.yml:

1. 'Validate all conformance resources in scope of the repository' step (lines ~534/536): Converted LOCAL_IG_PARAMETERS from an unquoted string (appended with +="-ig $GITHUB_WORKSPACE/$p ") to a bash array (initialized as LOCAL_IG_PARAMETERS=() and populated with LOCAL_IG_PARAMETERS+=("-ig" "$GITHUB_WORKSPACE/$p")). Replaced UNESCPAED_IG_DEPENDENCIES with IG_DEP_ARRAY built by splitting the IG_DEPENDENCIES string on spaces/newlines into individual array elements. Both arrays are now expanded with "${array[@]}" in the java invocation.

2. 'Validate all example resources in scope of the repository' step (lines ~596/598): Same pattern applied to COMBINED_IG_PARAMETERS and IG_DEP_ARRAY.

This prevents attacker-controlled path values containing shell metacharacters (;, |, &, $(...), etc.) from breaking out of the argument list and executing arbitrary commands.

