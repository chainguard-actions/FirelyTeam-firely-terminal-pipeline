<!-- markdownlint-disable -->

# Hardening Report: FirelyTeam--firely-terminal-pipeline/v0.8.13

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **FirelyTeam--firely-terminal-pipeline/v0.8.13** was hardened automatically. 2 finding(s) were identified and resolved across 3 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (b): Multiple run: blocks expand untrusted input env vars unquoted, allowing shell metacharacter injection. Specific violations:

1. 'fhir login email=$INPUT_SIMPLIFIER_USERNAME password=$INPUT_SIMPLIFIER_PASSWORD' — SIMPLIFIER_USERNAME and SIMPLIFIER_PASSWORD (inputs) are unquoted.
2. 'dotnet tool install --global Firely.Terminal --version $FIRELY_TERMINAL_VERSION' — FIRELY_TERMINAL_VERSION (input) is unquoted.
3. 'sudo npm install -g fsh-sushi@$SUSHI_VERSION' — SUSHI_VERSION (input) is unquoted.
4. 'sushi $INPUT_SUSHI_OPTIONS' (two occurrences) — SUSHI_OPTIONS (input) is unquoted, allowing arbitrary argument injection.
5. 'fhir check $INPUT_PATH_TO_QUALITY_CONTROL_RULES' and 'fhir check --fail $INPUT_PATH_TO_QUALITY_CONTROL_RULES' — PATH_TO_QUALITY_CONTROL_RULES (input) is unquoted.
6. 'echo $INPUT_EXPECTED_FAILS | grep -w -q ...' (three occurrences) — EXPECTED_FAILS (input) is unquoted.
7. 'for p in $INPUT_PATH_TO_CONFORMANCE_RESOURCES' (three occurrences) — PATH_TO_CONFORMANCE_RESOURCES (input) is unquoted.
8. 'wget -q $JAVA_VALIDATOR_DOWNLOAD_LOCATION -O validator_cli.jar' — JAVA_VALIDATOR_DOWNLOAD_LOCATION (input) is unquoted.
9. 'java -jar validator_cli.jar $GITHUB_WORKSPACE/$p*.xml $GITHUB_WORKSPACE/$p*.json -version $FHIR_VERSION $INPUT_JAVA_VALIDATION_OPTIONS $UNESCPAED_IG_DEPENDENCIES ...' (four occurrences) — multiple unquoted variables including JAVA_VALIDATION_OPTIONS (input).
10. 'JAVA_VALIDATOR_DOWNLOAD_LOCATION=$(eval echo "$JAVA_VALIDATOR_DOWNLOAD_LOCATION")' — eval is used with a user-controlled input variable (inputs.JAVA_VALIDATOR_DOWNLOAD_LOCATION), enabling arbitrary command execution.

Locations:

- `action.yml:130`
- `action.yml:112`
- `action.yml:441`
- `action.yml:451`
- `action.yml:519`
- `action.yml:521`
- `action.yml:568`
- `action.yml:571`
- `action.yml:601`
- `action.yml:611`
- `action.yml:614`
- `action.yml:681`
- `action.yml:691`
- `action.yml:694`
- `action.yml:720`

### github-env-injection (severity: high)

The 'Detect FHIR version and determine dependency source' step writes the value of $fhirVersion (extracted from repository files package.json or sushi-config.yaml via jq/yq) to $GITHUB_ENV without sanitization: 'echo "FHIR_VERSION=$fhirVersion" >> "$GITHUB_ENV"'. A malicious repository could embed newlines in the fhirVersion field of package.json or sushi-config.yaml to inject arbitrary environment variables into subsequent steps. The required sanitization step (printf '%s' "$fhirVersion" | tr -d '\n\r') is absent before the write.

Locations:

- `action.yml:248`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection

**Notes:**

Fixed all script-injection vulnerabilities in action.yml: (1) Quoted SIMPLIFIER_USERNAME and SIMPLIFIER_PASSWORD in fhir login command; (2) Quoted FIRELY_TERMINAL_VERSION in dotnet tool install; (3) Quoted SUSHI_VERSION in npm install; (4) Used ${VAR:+"$VAR"} for optional SUSHI_OPTIONS in both sushi invocations; (5) Used ${VAR:+"$VAR"} for optional PATH_TO_QUALITY_CONTROL_RULES in fhir check commands; (6) Quoted INPUT_EXPECTED_FAILS in all echo | grep patterns; (7) Quoted JAVA_VALIDATOR_DOWNLOAD_LOCATION in wget; (8) Quoted FHIR_VERSION, path variables, and used ${VAR:+"$VAR"} for optional JAVA_VALIDATION_OPTIONS in all four java validator invocations; (9) Replaced dangerous eval echo with safe bash parameter substitution for JAVA_VALIDATOR_DOWNLOAD_LOCATION URL template expansion. Fixed github-env-injection by sanitizing fhirVersion with printf '%s' | tr -d '\n\r' before writing to $GITHUB_ENV.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed all three script-injection findings in action.yml by double-quoting the unquoted variable expansions in `for` loops. Changed `for p in $INPUT_PATH_TO_CONFORMANCE_RESOURCES;` to `for p in "$INPUT_PATH_TO_CONFORMANCE_RESOURCES";` in two locations in the 'Validate all conformance resources' step (lines ~549 and ~554), and in one location in the 'Validate all example resources' step (line ~610). Also changed `for p in $PATH_TO_EXAMPLES;` to `for p in "$PATH_TO_EXAMPLES";` in the 'Validate all example resources' step (line ~617). This prevents attacker-controlled values containing shell metacharacters from being word-split and glob-expanded by the shell.

### Iteration 3

**Fixes applied:** script-injection

**Notes:**

Fixed all script injection vulnerabilities in action.yml:

1. **Boolean condition fixes** (if $VAR → if [["$VAR" == "true"]]): Replaced all `if $INPUT_DOTNET_VALIDATION_ENABLED`, `if $INPUT_SUSHI_ENABLED`, `if $INPUT_JAVA_VALIDATION_ENABLED`, `if $INPUT_TERMINOLOGY_SERVICE_BFARM_ENABLED`, `if $INPUT_JAVA_SNAPSHOT_ENABLED`, `if $CLOSE_SLICING_FOR_VALIDATION`, and `if $INPUT_SUSHI_USE_CONFIG_DEPENDENCIES` patterns with proper string comparisons using `[[ "$VAR" == "true" ]]`. This prevents user-controlled input values from being executed as shell commands.

2. **Unquoted word-split variables in java commands**: Replaced unquoted `$UNESCPAED_IG_DEPENDENCIES $LOCAL_IG_PARAMETERS` and `$UNESCPAED_IG_DEPENDENCIES $COMBINED_IG_PARAMETERS` in `java -jar` invocations with properly quoted array expansions. The space-separated argument strings are now split into arrays using `read -ra ARRAY <<< "$VAR"` and expanded as `"${ARRAY[@]}"`, preventing shell injection while preserving correct argument passing.

3. **Additional fix**: Also quoted `$fhirCacheLocation` in `rm -rf` command for safety.

