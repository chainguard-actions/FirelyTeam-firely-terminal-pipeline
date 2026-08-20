<!-- markdownlint-disable -->

# Hardening Report: FirelyTeam--firely-terminal-pipeline/v0.8.7

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **FirelyTeam--firely-terminal-pipeline/v0.8.7** was hardened automatically. 3 finding(s) were identified and resolved across 5 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

All three workflow files use `actions/checkout@v4` — a mutable tag reference, not a pinned 40-character commit SHA. A tag can be moved to point to a different (potentially malicious) commit, enabling supply-chain attacks.

Locations:

- `.github/workflows/update_firelyterminal.yml:16`
- `.github/workflows/update_javavalidator.yml:16`
- `.github/workflows/update_sushi.yml:16`

### missing-permissions (severity: medium)

None of the three workflow files declare a top-level `permissions:` block, and no job within them has a `permissions:` block either. Without explicit permissions, workflows inherit the default (often broad) repository token permissions, violating the principle of least privilege.

Locations:

- `.github/workflows/update_firelyterminal.yml:1`
- `.github/workflows/update_javavalidator.yml:1`
- `.github/workflows/update_sushi.yml:1`

### script-injection (severity: high)

Multiple `run:` steps in action.yml expand env vars that hold caller-supplied `inputs.*` values without double-quoting, violating rule (b). An attacker-controlled calling workflow can inject shell metacharacters (`;`, `|`, `$(...)`, etc.) through these inputs.

• `dotnet tool install --global Firely.Terminal --version $FIRELY_TERMINAL_VERSION` — FIRELY_TERMINAL_VERSION is unquoted (from inputs.FIRELY_TERMINAL_VERSION).
• `fhir login email=$INPUT_SIMPLIFIER_USERNAME password=$INPUT_SIMPLIFIER_PASSWORD` — both credential vars are unquoted (from inputs.SIMPLIFIER_USERNAME / inputs.SIMPLIFIER_PASSWORD).
• `sudo npm install -g fsh-sushi@$SUSHI_VERSION` — SUSHI_VERSION is unquoted (from inputs.SUSHI_VERSION).
• `sushi $INPUT_SUSHI_OPTIONS` — SUSHI_OPTIONS is unquoted (from inputs.SUSHI_OPTIONS); allows arbitrary shell injection.
• `fhir check $INPUT_PATH_TO_QUALITY_CONTROL_RULES` — PATH_TO_QUALITY_CONTROL_RULES is unquoted (from inputs.PATH_TO_QUALITY_CONTROL_RULES).
• `for p in $INPUT_PATH_TO_CONFORMANCE_RESOURCES` — PATH_TO_CONFORMANCE_RESOURCES is unquoted (from inputs.PATH_TO_CONFORMANCE_RESOURCES); word-splits on spaces and expands globs.
• `java -jar validator_cli.jar ... $INPUT_JAVA_VALIDATION_OPTIONS ...` — JAVA_VALIDATION_OPTIONS is unquoted (from inputs.JAVA_VALIDATION_OPTIONS).
• `JAVA_VALIDATOR_DOWNLOAD_LOCATION=$(eval echo "$JAVA_VALIDATOR_DOWNLOAD_LOCATION")` followed by `wget -q $JAVA_VALIDATOR_DOWNLOAD_LOCATION` — JAVA_VALIDATOR_DOWNLOAD_LOCATION is set from inputs.JAVA_VALIDATOR_DOWNLOAD_LOCATION; passing it through `eval` allows arbitrary command execution, and the subsequent wget is also unquoted.

Locations:

- `action.yml:107`
- `action.yml:131`
- `action.yml:155`
- `action.yml:390`
- `action.yml:404`
- `action.yml:455`
- `action.yml:503`
- `action.yml:519`
- `action.yml:560`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, missing-permissions, script-injection

**Notes:**

Fixed all three finding types:

1. unpinned-uses: Pinned actions/checkout@v4 to SHA 11d5960a326750d5838078e36cf38b85af677262 in all three workflow files.

2. missing-permissions: Added top-level `permissions: contents: write / pull-requests: write` to all three workflow files (minimum needed to checkout, push branches, and create PRs).

3. script-injection in action.yml:
   - Quoted FIRELY_TERMINAL_VERSION in dotnet tool install command
   - Quoted SIMPLIFIER_USERNAME and SIMPLIFIER_PASSWORD in fhir login command
   - Quoted SUSHI_VERSION in npm install command
   - Replaced unquoted `sushi $INPUT_SUSHI_OPTIONS` with xargs-tokenized array in both SUSHI steps
   - Replaced unquoted `fhir check $INPUT_PATH_TO_QUALITY_CONTROL_RULES` with ${VAR:+"$VAR"} form
   - Replaced `for p in $INPUT_PATH_TO_CONFORMANCE_RESOURCES` with xargs-tokenized array in both Java validator steps
   - Replaced unquoted `$INPUT_JAVA_VALIDATION_OPTIONS` with xargs-tokenized array in both Java validator steps
   - Removed dangerous `eval echo "$JAVA_VALIDATOR_DOWNLOAD_LOCATION"` and replaced with safe bash parameter substitution; quoted the wget URL

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed all 5 script injection vulnerabilities in hardened/action/action.yml:
1. Quoted `$INPUT_EXPECTED_FAILS` in all three `echo ... | grep` usages (lines ~705, ~851, ~961) to prevent shell metacharacter injection from the caller-controlled EXPECTED_FAILS input.
2. Converted `LOCAL_IG_PARAMETERS` string variable to a bash array `local_ig_params` in the 'Validate all conformance resources' step, building it with `local_ig_params+=("-ig" "$GITHUB_WORKSPACE/$p")` and expanding it safely as `"${local_ig_params[@]}"` in both java invocations.
3. Converted `COMBINED_IG_PARAMETERS` string variable to a bash array `combined_ig_params` in the 'Validate all example resources' step with the same pattern.
These changes prevent command injection via attacker-controlled path values containing shell metacharacters (`;`, `|`, `&`, `$(...)`, etc.).

### Iteration 3

**Fixes applied:** script-injection

**Notes:**

Fixed two script injection vulnerabilities in action.yml:
1. In the 'Validate all conformance resources in scope of the repository' step (line ~710): Changed `$GITHUB_WORKSPACE/$p*.xml $GITHUB_WORKSPACE/$p*.json` to `"$GITHUB_WORKSPACE/$p"*.xml "$GITHUB_WORKSPACE/$p"*.json` in both branches of the if/else.
2. In the 'Validate all example resources in scope of the repository' step (line ~800): Applied the same fix to both branches of the if/else.
The glob patterns (*.xml, *.json) are kept outside the quotes so they still expand via shell globbing, while the directory path containing the user-controlled `$p` variable is now properly double-quoted to prevent shell metacharacter injection.

### Iteration 4

**Fixes applied:** script-injection

**Notes:**

Fixed all 22 script injection locations in hardened/action/action.yml. Replaced every unsafe `if $VARIABLE; then` pattern (where VARIABLE holds a user-controlled boolean input) with the safe `if [[ "$VARIABLE" == 'true' ]]; then` form. This prevents bash from evaluating the variable's value as a shell command. Variables fixed: INPUT_DOTNET_VALIDATION_ENABLED, INPUT_TERMINOLOGY_SERVICE_BFARM_ENABLED, INPUT_JAVA_VALIDATION_ENABLED, INPUT_SUSHI_ENABLED, INPUT_SUSHI_USE_CONFIG_DEPENDENCIES, INPUT_JAVA_SNAPSHOT_ENABLED, and CLOSE_SLICING_FOR_VALIDATION. Also fixed compound conditions like `if $A || $B` and `if $A && $B` to use the safe `[[ ]]` form for each operand.

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Fixed github-env-injection in all three workflow files:
1. update_firelyterminal.yml (lines 31, 37): Added `safe=$(printf '%s' "$VAR" | tr -d '\n\r')` sanitization before writing FIRELY_TERMINAL_VERSION and LATEST_RELEASE to $GITHUB_ENV.
2. update_javavalidator.yml (lines 31, 37): Added same sanitization before writing JAVA_VALIDATOR_VERSION and LATEST_RELEASE to $GITHUB_ENV.
3. update_sushi.yml (lines 30, 36): Added same sanitization before writing SUSHI_VERSION and LATEST_RELEASE to $GITHUB_ENV.
Also fixed unquoted $GITHUB_ENV references to use quoted "$GITHUB_ENV" for consistency.

