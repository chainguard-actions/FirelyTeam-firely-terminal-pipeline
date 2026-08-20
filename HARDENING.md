<!-- markdownlint-disable -->

# Hardening Report: FirelyTeam--firely-terminal-pipeline/v0.8.8

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **FirelyTeam--firely-terminal-pipeline/v0.8.8** was hardened automatically. 4 finding(s) were identified and resolved across 3 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

All three workflow files use `actions/checkout@v4`, which is a mutable tag reference rather than a pinned 40-character commit SHA. This exposes the workflows to supply-chain attacks if the tag is moved to a malicious commit.

Locations:

- `.github/workflows/update_firelyterminal.yml:15`
- `.github/workflows/update_javavalidator.yml:15`
- `.github/workflows/update_sushi.yml:15`

### permissions (severity: medium)

missing-permissions: None of the three workflow files define a top-level `permissions:` key, and none of the individual jobs define job-level permissions either. Without explicit permissions, workflows inherit the default repository token permissions, which may be overly broad.

Locations:

- `.github/workflows/update_firelyterminal.yml:1`
- `.github/workflows/update_javavalidator.yml:1`
- `.github/workflows/update_sushi.yml:1`

### script-injection (severity: high)

Rule (b) — Multiple `run:` blocks in action.yml expand env vars that hold user-controlled input values without double-quoting, allowing shell metacharacter injection:
1. `dotnet tool install --global Firely.Terminal --version $FIRELY_TERMINAL_VERSION` — FIRELY_TERMINAL_VERSION is sourced from inputs.FIRELY_TERMINAL_VERSION (unquoted).
2. `sudo npm install -g fsh-sushi@$SUSHI_VERSION` — SUSHI_VERSION is sourced from inputs.SUSHI_VERSION (unquoted).
3. `sushi $INPUT_SUSHI_OPTIONS` — INPUT_SUSHI_OPTIONS is sourced from inputs.SUSHI_OPTIONS (unquoted).
4. `fhir check $INPUT_PATH_TO_QUALITY_CONTROL_RULES` and `fhir check --fail $INPUT_PATH_TO_QUALITY_CONTROL_RULES` — from inputs.PATH_TO_QUALITY_CONTROL_RULES (unquoted).
5. `java -jar validator_cli.jar ... $INPUT_JAVA_VALIDATION_OPTIONS ...` — INPUT_JAVA_VALIDATION_OPTIONS from inputs.JAVA_VALIDATION_OPTIONS (unquoted).
6. `wget -q $JAVA_VALIDATOR_DOWNLOAD_LOCATION` — from inputs.JAVA_VALIDATOR_DOWNLOAD_LOCATION (unquoted).
7. `for p in $INPUT_PATH_TO_CONFORMANCE_RESOURCES` — from inputs.PATH_TO_CONFORMANCE_RESOURCES (unquoted, also used in loop body unquoted).
8. `echo $INPUT_EXPECTED_FAILS | grep ...` — from inputs.EXPECTED_FAILS (unquoted).

Locations:

- `action.yml:113`
- `action.yml:399`
- `action.yml:432`
- `action.yml:469`
- `action.yml:471`
- `action.yml:530`
- `action.yml:540`
- `action.yml:556`
- `action.yml:558`
- `action.yml:601`
- `action.yml:609`

### github-env-injection (severity: high)

Multiple `run:` blocks in the workflow files write values derived from external API responses to `$GITHUB_ENV` without sanitization (no `printf '%s' ... | tr -d '\n\r'` applied). A compromised or malicious upstream API could inject newlines to add arbitrary environment variable assignments:
1. `update_firelyterminal.yml`: `echo "FIRELY_TERMINAL_VERSION=$FIRELY_TERMINAL_VERSION" >> $GITHUB_ENV` (value from yq parsing action.yml) and `echo "LATEST_RELEASE=$LATEST_RELEASE" >> $GITHUB_ENV` (value from NuGet API response).
2. `update_javavalidator.yml`: `echo "JAVA_VALIDATOR_VERSION=$JAVA_VALIDATOR_VERSION" >> $GITHUB_ENV` and `echo "LATEST_RELEASE=$LATEST_RELEASE" >> $GITHUB_ENV` (value from GitHub API response).
3. `update_sushi.yml`: `echo "SUSHI_VERSION=$SUSHI_VERSION" >> $GITHUB_ENV` and `echo "LATEST_RELEASE=$LATEST_RELEASE" >> $GITHUB_ENV` (value from GitHub API response).

Locations:

- `.github/workflows/update_firelyterminal.yml:28`
- `.github/workflows/update_firelyterminal.yml:35`
- `.github/workflows/update_javavalidator.yml:28`
- `.github/workflows/update_javavalidator.yml:34`
- `.github/workflows/update_sushi.yml:27`
- `.github/workflows/update_sushi.yml:33`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, permissions, script-injection, github-env-injection

**Notes:**

Fixed all four findings:

1. **unpinned-uses**: Pinned `actions/checkout@v4` to `actions/checkout@11d5960a326750d5838078e36cf38b85af677262 # v4` in all three workflow files.

2. **permissions**: Added `permissions: { contents: write, pull-requests: write }` top-level block to all three workflow files (minimum needed for branch creation and PR creation).

3. **script-injection** in action.yml: (a) Quoted `$FIRELY_TERMINAL_VERSION` in dotnet install; (b) Quoted `$SUSHI_VERSION` in npm install; (c) Tokenized `INPUT_SUSHI_OPTIONS` with xargs into an array for both SUSHI invocations; (d) Used `${VAR:+"$VAR"}` for optional `INPUT_PATH_TO_QUALITY_CONTROL_RULES`; (e) Quoted `$INPUT_EXPECTED_FAILS` in all echo/grep pipes; (f) Quoted `$JAVA_VALIDATOR_DOWNLOAD_LOCATION` in wget; (g) Tokenized `INPUT_PATH_TO_CONFORMANCE_RESOURCES` and `PATH_TO_EXAMPLES` with xargs into arrays for Java validator loops; (h) Tokenized `INPUT_JAVA_VALIDATION_OPTIONS` with xargs into an array for both Java validator steps.

4. **github-env-injection**: All six GITHUB_ENV writes in the three workflow files now sanitize values with `safe=$(printf '%s' "$VAR" | tr -d '\n\r')` before writing to `"$GITHUB_ENV"`.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed two categories of script injection vulnerabilities in action.yml:

1. **eval injection (line 531)**: Replaced `eval echo "$JAVA_VALIDATOR_DOWNLOAD_LOCATION"` with safe bash parameter substitution `${JAVA_VALIDATOR_DOWNLOAD_LOCATION//'$JAVA_VALIDATOR_VERSION'/$JAVA_VALIDATOR_VERSION}`. This safely expands the literal string `$JAVA_VALIDATOR_VERSION` within the URL without executing any shell commands, preventing command injection via malicious input values like `$(malicious_command)`.

2. **Variable-as-command injection (multiple lines)**: Replaced all `if $INPUT_VAR` and `if $CLOSE_SLICING_FOR_VALIDATION` patterns with proper string comparisons `if [ "$INPUT_VAR" = "true" ]`. This affects 20+ steps across the workflow including: Check if .NET is installed, Check .NET SDK Version, Install Firely.Terminal, Check Firely Terminal Version, Simplifier login, Detect FHIR version (including `$INPUT_SUSHI_USE_CONFIG_DEPENDENCIES` and `$INPUT_SUSHI_ENABLED`), Validate FHIR version support, Set FHIR specification, all SUSHI steps, Pre-processing FHIR profiles, Configure Firely Terminal validator engine, Run Quality Control checks, Remove package cache, Check if Java is installed, Download Java Validator, Validate all conformance/example resources, and Re-run SUSHI. The `yq eval` occurrences are the yq tool's subcommand and are not shell eval — they were left unchanged.

### Iteration 3

**Fixes applied:** script-injection

**Notes:**

Fixed both script-injection findings in action.yml:

1. 'Validate all conformance resources' step (~line 556): Converted unquoted string variables LOCAL_IG_PARAMETERS and UNESCPAED_IG_DEPENDENCIES to properly-quoted bash arrays (local_ig_params and unescaped_ig_deps). Fixed unquoted $p in glob patterns by quoting the path prefix: "$GITHUB_WORKSPACE/$p"*.xml. Arrays are expanded with "${array[@]}" syntax.

2. 'Validate all example resources' step (~line 634): Same fix pattern — COMBINED_IG_PARAMETERS → combined_ig_params array, UNESCPAED_IG_DEPENDENCIES → unescaped_ig_deps array, and glob patterns properly quoted as "$GITHUB_WORKSPACE/$p"*.xml and "$GITHUB_WORKSPACE/$p"*.json.

The xargs-based tokenization with NUL delimiters is used for IG_DEPENDENCIES (which is a space-separated list of -ig flags from jq output), guarded by an emptiness check to avoid xargs running with no input.

