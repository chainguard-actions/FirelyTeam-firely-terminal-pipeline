<!-- markdownlint-disable -->

# Hardening Report: FirelyTeam--firely-terminal-pipeline/v0.8.20

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **FirelyTeam--firely-terminal-pipeline/v0.8.20** was hardened automatically. 4 finding(s) were identified and resolved across 3 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

All three workflow files use `actions/checkout@v4` (a mutable tag reference) instead of a pinned 40-character commit SHA. This exposes the workflow to supply-chain attacks if the tag is moved to a malicious commit.

Locations:

- `.github/workflows/update_firelyterminal.yml:14`
- `.github/workflows/update_javavalidator.yml:14`
- `.github/workflows/update_sushi.yml:14`

### missing-permissions (severity: medium)

None of the three workflow files define a top-level `permissions:` block, and no job-level `permissions:` blocks are present either. Without explicit permissions, workflows inherit the default (often broad) repository permissions, violating the principle of least privilege.

Locations:

- `.github/workflows/update_firelyterminal.yml:1`
- `.github/workflows/update_javavalidator.yml:1`
- `.github/workflows/update_sushi.yml:1`

### script-injection (severity: high)

Multiple unquoted shell variable expansions of input-controlled env vars in action.yml run: blocks (sub-rule b):

1. `eval echo "$JAVA_VALIDATOR_DOWNLOAD_LOCATION"` — `JAVA_VALIDATOR_DOWNLOAD_LOCATION` is set from `inputs.JAVA_VALIDATOR_DOWNLOAD_LOCATION` via env:. Using `eval` with an attacker-controlled value allows arbitrary command execution.

2. `sushi $INPUT_SUSHI_OPTIONS` (unquoted) — `INPUT_SUSHI_OPTIONS` comes from `inputs.SUSHI_OPTIONS`; shell metacharacters in the value are parsed by the shell.

3. `echo $INPUT_EXPECTED_FAILS | grep ...` (unquoted) — `INPUT_EXPECTED_FAILS` comes from `inputs.EXPECTED_FAILS`.

4. `fhir check $INPUT_PATH_TO_QUALITY_CONTROL_RULES` (unquoted) — `INPUT_PATH_TO_QUALITY_CONTROL_RULES` comes from `inputs.PATH_TO_QUALITY_CONTROL_RULES`.

5. `wget -q $JAVA_VALIDATOR_DOWNLOAD_LOCATION` (unquoted) — same input-controlled variable.

6. `for p in $INPUT_PATH_TO_CONFORMANCE_RESOURCES` (unquoted) — `INPUT_PATH_TO_CONFORMANCE_RESOURCES` comes from `inputs.PATH_TO_CONFORMANCE_RESOURCES`.

Locations:

- `action.yml:499`
- `action.yml:501`
- `action.yml:392`
- `action.yml:461`
- `action.yml:463`
- `action.yml:497`

### github-env-injection (severity: high)

Multiple steps write externally-sourced or input-derived values to $GITHUB_ENV and $GITHUB_PATH without the required sanitization (`printf '%s' ... | tr -d '\n\r'`):

1. action.yml: `echo "FHIR_VERSION=$fhirVersion" >> "$GITHUB_ENV"` — `fhirVersion` is extracted from repository files (package.json / sushi-config.yaml) which could contain newlines enabling header injection.

2. action.yml: `echo "NEED_SUSHI_CONVERSION=..." >> "$GITHUB_ENV"`, `echo "USE_SUSHI_CONFIG=..." >> "$GITHUB_ENV"`, `echo "USE_PACKAGE_JSON=..." >> "$GITHUB_ENV"` — values derived from file-system checks written without sanitization.

3. update_firelyterminal.yml: `echo "FIRELY_TERMINAL_VERSION=$FIRELY_TERMINAL_VERSION" >> $GITHUB_ENV` and `echo "LATEST_RELEASE=$LATEST_RELEASE" >> $GITHUB_ENV` — values from yq parsing and external API responses written without sanitization.

4. update_firelyterminal.yml: `echo "~/bin" >> $GITHUB_PATH` — a literal string, but the pattern is consistent with unsanitized writes.

5. update_javavalidator.yml and update_sushi.yml: Same pattern — `$JAVA_VALIDATOR_VERSION`, `$LATEST_RELEASE`, `$SUSHI_VERSION` from external API responses written to $GITHUB_ENV without sanitization.

Locations:

- `action.yml:174`
- `action.yml:175`
- `action.yml:176`
- `action.yml:213`
- `.github/workflows/update_firelyterminal.yml:22`
- `.github/workflows/update_firelyterminal.yml:23`
- `.github/workflows/update_firelyterminal.yml:29`
- `.github/workflows/update_firelyterminal.yml:30`
- `.github/workflows/update_javavalidator.yml:22`
- `.github/workflows/update_javavalidator.yml:23`
- `.github/workflows/update_javavalidator.yml:29`
- `.github/workflows/update_javavalidator.yml:30`
- `.github/workflows/update_sushi.yml:22`
- `.github/workflows/update_sushi.yml:23`
- `.github/workflows/update_sushi.yml:29`
- `.github/workflows/update_sushi.yml:30`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, missing-permissions, script-injection, github-env-injection

**Notes:**

Fixed all four findings across action.yml and three workflow files:

1. unpinned-uses: Pinned actions/checkout@v4 to full SHA 11d5960a326750d5838078e36cf38b85af677262 in all three workflow files.

2. missing-permissions: Added top-level `permissions: contents: write, pull-requests: write` to all three workflow files (minimum needed to create branches and PRs).

3. script-injection in action.yml:
   - Replaced `eval echo "$JAVA_VALIDATOR_DOWNLOAD_LOCATION"` with safe sed-based substitution
   - Quoted `wget -q "$JAVA_VALIDATOR_DOWNLOAD_LOCATION"`
   - Replaced `sushi $INPUT_SUSHI_OPTIONS` (both SUSHI steps) with xargs-tokenized array
   - Quoted `echo "$INPUT_EXPECTED_FAILS" | grep` (was unquoted)
   - Used `${INPUT_PATH_TO_QUALITY_CONTROL_RULES:+"$INPUT_PATH_TO_QUALITY_CONTROL_RULES"}` for optional path arg
   - Replaced `for p in $INPUT_PATH_TO_CONFORMANCE_RESOURCES` and `for p in $PATH_TO_EXAMPLES` with xargs-tokenized arrays
   - Tokenized `$INPUT_JAVA_VALIDATION_OPTIONS` with xargs into arrays

4. github-env-injection: All GITHUB_ENV/GITHUB_PATH writes now sanitize values with `printf '%s' "$var" | tr -d '\n\r'` before writing, in both action.yml and all three workflow files.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed four unquoted shell variable expansions in action.yml:
1. 'Install Firely.Terminal' step: Added quotes around `$FIRELY_TERMINAL_VERSION` → `"$FIRELY_TERMINAL_VERSION"`
2. 'Simplifier login' step: Added quotes around both `$INPUT_SIMPLIFIER_USERNAME` and `$INPUT_SIMPLIFIER_PASSWORD` → `email="$INPUT_SIMPLIFIER_USERNAME" password="$INPUT_SIMPLIFIER_PASSWORD"`
3. 'Install fsh-sushi' step: Added quotes around `fsh-sushi@$SUSHI_VERSION` → `"fsh-sushi@$SUSHI_VERSION"` (npm accepts the package@version form inside a single quoted string)
4. 'Download Java Validator' step: Changed `$JAVA_VALIDATOR_VERSION` to `${JAVA_VALIDATOR_VERSION}` inside the double-quoted sed expression, ensuring the variable is properly delimited and quoted within the sed substitution.

### Iteration 3

**Fixes applied:** script-injection

**Notes:**

Fixed 6 instances of unquoted $p variable (derived from user-controlled inputs PATH_TO_CONFORMANCE_RESOURCES and PATH_TO_EXAMPLES) in action.yml:
1. LOCAL_IG_PARAMETERS+= assignment: changed `-ig $GITHUB_WORKSPACE/$p ` to `-ig \"$GITHUB_WORKSPACE/$p\" ` (properly quoted path)
2. COMBINED_IG_PARAMETERS+= assignment: same fix applied
3-4. Both java validator invocations in 'Validate all conformance resources' step: changed `$GITHUB_WORKSPACE/$p*.xml $GITHUB_WORKSPACE/$p*.json` to `"$GITHUB_WORKSPACE/$p"*.xml "$GITHUB_WORKSPACE/$p"*.json` (glob-safe quoting: path prefix quoted, glob suffix unquoted for proper expansion)
5-6. Both java validator invocations in 'Validate all example resources' step: same glob-safe quoting applied

