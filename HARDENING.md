<!-- markdownlint-disable -->

# Hardening Report: FirelyTeam--firely-terminal-pipeline/v0.8.19

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **FirelyTeam--firely-terminal-pipeline/v0.8.19** was hardened automatically. 3 finding(s) were identified and resolved across 3 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Multiple unquoted shell variable expansions of input-derived env vars in action.yml run blocks allow shell metacharacter injection (sub-rule b):

1. `sushi $INPUT_SUSHI_OPTIONS` — INPUT_SUSHI_OPTIONS is set from `${{ inputs.SUSHI_OPTIONS }}` and used unquoted, allowing shell injection.
2. `fhir check $INPUT_PATH_TO_QUALITY_CONTROL_RULES` and `fhir check --fail $INPUT_PATH_TO_QUALITY_CONTROL_RULES` — INPUT_PATH_TO_QUALITY_CONTROL_RULES from `${{ inputs.PATH_TO_QUALITY_CONTROL_RULES }}` used unquoted.
3. `for p in $INPUT_PATH_TO_CONFORMANCE_RESOURCES` — INPUT_PATH_TO_CONFORMANCE_RESOURCES from `${{ inputs.PATH_TO_CONFORMANCE_RESOURCES }}` used unquoted in loop (multiple steps).
4. `JAVA_VALIDATOR_DOWNLOAD_LOCATION=$(eval echo "$JAVA_VALIDATOR_DOWNLOAD_LOCATION")` — uses shell `eval` on a value sourced from `${{ inputs.JAVA_VALIDATOR_DOWNLOAD_LOCATION }}`, enabling arbitrary command execution.
5. `wget -q $JAVA_VALIDATOR_DOWNLOAD_LOCATION` — unquoted expansion of input-derived value.
6. `java -jar validator_cli.jar ... $INPUT_JAVA_VALIDATION_OPTIONS ...` — INPUT_JAVA_VALIDATION_OPTIONS from `${{ inputs.JAVA_VALIDATION_OPTIONS }}` used unquoted in multiple steps.
7. `echo $INPUT_EXPECTED_FAILS | grep ...` — INPUT_EXPECTED_FAILS from `${{ inputs.EXPECTED_FAILS }}` used unquoted in multiple steps.
8. `for p in $PATH_TO_EXAMPLES` — PATH_TO_EXAMPLES from `${{ inputs.PATH_TO_EXAMPLES }}` used unquoted.

Locations:

- `action.yml:447`
- `action.yml:530`
- `action.yml:531`
- `action.yml:571`
- `action.yml:573`
- `action.yml:590`
- `action.yml:591`
- `action.yml:620`
- `action.yml:621`
- `action.yml:625`
- `action.yml:626`
- `action.yml:680`
- `action.yml:681`
- `action.yml:685`
- `action.yml:686`
- `action.yml:730`
- `action.yml:735`

### unpinned-uses (severity: high)

All three workflow files reference `actions/checkout@v4`, which is a mutable tag reference rather than a pinned 40-character commit SHA. This means the action could be silently updated to a different (potentially malicious) version without any change to the workflow file. All failing references: `uses: actions/checkout@v4`.

Locations:

- `.github/workflows/update_firelyterminal.yml:16`
- `.github/workflows/update_javavalidator.yml:16`
- `.github/workflows/update_sushi.yml:16`

### missing-permissions (severity: medium)

None of the three workflow files define a top-level `permissions:` key, and none of the jobs within them define job-level `permissions:` blocks. Without explicit permissions, workflows run with the default (potentially broad) GITHUB_TOKEN permissions, which can include write access to repository contents and pull requests.

Locations:

- `.github/workflows/update_firelyterminal.yml:1`
- `.github/workflows/update_javavalidator.yml:1`
- `.github/workflows/update_sushi.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, unpinned-uses, missing-permissions

**Notes:**

Fixed all three finding categories:

1. script-injection (action.yml): (a) Replaced `sushi $INPUT_SUSHI_OPTIONS` with xargs-tokenized array expansion in both SUSHI steps. (b) Fixed `fhir check $INPUT_PATH_TO_QUALITY_CONTROL_RULES` with `${VAR:+"$VAR"}` pattern for optional single path. (c) Quoted `echo "$INPUT_EXPECTED_FAILS"` in all grep usages. (d) Replaced `for p in $INPUT_PATH_TO_CONFORMANCE_RESOURCES` and `for p in $PATH_TO_EXAMPLES` with xargs-tokenized arrays in both Java validation steps. (e) Replaced `eval echo "$JAVA_VALIDATOR_DOWNLOAD_LOCATION"` with safe bash parameter substitution `${JAVA_VALIDATOR_DOWNLOAD_LOCATION/\$JAVA_VALIDATOR_VERSION/$JAVA_VALIDATOR_VERSION}`. (f) Quoted `wget -q "$JAVA_VALIDATOR_DOWNLOAD_LOCATION"`. (g) Replaced unquoted `$INPUT_JAVA_VALIDATION_OPTIONS` and IG_DEPENDENCIES with xargs-tokenized arrays in both Java validation steps.

2. unpinned-uses: Pinned `actions/checkout@v4` to `actions/checkout@11d5960a326750d5838078e36cf38b85af677262 # v4` in all three workflow files.

3. missing-permissions: Added top-level `permissions: contents: write / pull-requests: write` to all three workflow files (needed because they create PRs via `gh pr create` and push branches).

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed three script-injection findings in hardened/action/action.yml:
1. Line 113: Quoted `$FIRELY_TERMINAL_VERSION` → `"$FIRELY_TERMINAL_VERSION"` in `dotnet tool install --global Firely.Terminal --version` command.
2. Line 430: Quoted `fsh-sushi@$SUSHI_VERSION` → `"fsh-sushi@$SUSHI_VERSION"` in `sudo npm install -g` command.
3. Line 556: Sanitized `$JAVA_VALIDATOR_VERSION` before use in bash parameter expansion by stripping non-alphanumeric/dot/hyphen characters into `SAFE_JAVA_VALIDATOR_VERSION`, then using that sanitized variable in the URL substitution to prevent injection via special bash characters like `/` or `*`.

### Iteration 1

**Fixes applied:** script-injection

**Notes:**

Fixed all script injection vulnerabilities in action.yml where boolean input variables were expanded unquoted in `if` conditions (e.g., `if $INPUT_DOTNET_VALIDATION_ENABLED; then`). In bash, `if $VAR; then` executes the variable's value as a shell command, allowing injection. All occurrences were replaced with safe string comparisons using `if [[ "$VAR" == 'true' ]]; then`. Variables fixed: INPUT_DOTNET_VALIDATION_ENABLED (7 occurrences), INPUT_TERMINOLOGY_SERVICE_BFARM_ENABLED (2 occurrences), INPUT_JAVA_VALIDATION_ENABLED (4 occurrences), INPUT_SUSHI_ENABLED (6 occurrences), INPUT_SUSHI_USE_CONFIG_DEPENDENCIES (1 occurrence), INPUT_JAVA_SNAPSHOT_ENABLED (1 occurrence), CLOSE_SLICING_FOR_VALIDATION (2 occurrences). Total: 23 fixes applied across the file.

