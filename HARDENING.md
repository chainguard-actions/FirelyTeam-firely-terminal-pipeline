<!-- markdownlint-disable -->

# Hardening Report: FirelyTeam--firely-terminal-pipeline/v0.8.17

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **FirelyTeam--firely-terminal-pipeline/v0.8.17** was hardened automatically. 4 finding(s) were identified and resolved across 3 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

All three workflow files reference actions/checkout@v4 using a mutable tag instead of a pinned 40-character commit SHA. This exposes the workflow to supply-chain attacks if the tag is moved to a malicious commit.

Locations:

- `.github/workflows/update_firelyterminal.yml:16`
- `.github/workflows/update_javavalidator.yml:16`
- `.github/workflows/update_sushi.yml:16`

### missing-permissions (severity: medium)

None of the three workflow files declare a top-level permissions: key, and no job-level permissions: blocks are present either. Without explicit permissions, workflows inherit the default (often write-all) token permissions, granting unnecessary access.

Locations:

- `.github/workflows/update_firelyterminal.yml:1`
- `.github/workflows/update_javavalidator.yml:1`
- `.github/workflows/update_sushi.yml:1`

### script-injection (severity: high)

Multiple run: blocks in action.yml expand input-derived env vars without double-quoting, violating rule (b). An attacker controlling these inputs can inject shell metacharacters (;, |, &, $(...), etc.).

(1) 'Install Firely.Terminal' step: `dotnet tool install --global Firely.Terminal --version $FIRELY_TERMINAL_VERSION` — $FIRELY_TERMINAL_VERSION (from inputs.FIRELY_TERMINAL_VERSION) is unquoted.

(2) 'Install fsh-sushi' step: `sudo npm install -g fsh-sushi@$SUSHI_VERSION` — $SUSHI_VERSION (from inputs.SUSHI_VERSION) is unquoted.

(3) 'Generate conformance resources with SUSHI' step: `sushi $INPUT_SUSHI_OPTIONS` — $INPUT_SUSHI_OPTIONS (from inputs.SUSHI_OPTIONS) is unquoted.

(4) 'Run Quality Control checks' step: `fhir check $INPUT_PATH_TO_QUALITY_CONTROL_RULES` and `fhir check --fail $INPUT_PATH_TO_QUALITY_CONTROL_RULES` — unquoted path from inputs.PATH_TO_QUALITY_CONTROL_RULES.

(5) 'Download Java Validator' step: `JAVA_VALIDATOR_DOWNLOAD_LOCATION=$(eval echo "$JAVA_VALIDATOR_DOWNLOAD_LOCATION")` followed by `wget -q $JAVA_VALIDATOR_DOWNLOAD_LOCATION` — the use of eval with an input-controlled value (inputs.JAVA_VALIDATOR_DOWNLOAD_LOCATION) allows arbitrary command execution; the subsequent wget is also unquoted.

(6) Java validator steps: `for p in $INPUT_PATH_TO_CONFORMANCE_RESOURCES` — unquoted word-splitting of inputs.PATH_TO_CONFORMANCE_RESOURCES.

(7) Java validator command lines: `java -jar validator_cli.jar $GITHUB_WORKSPACE/$p*.xml ... $INPUT_JAVA_VALIDATION_OPTIONS ...` — $INPUT_JAVA_VALIDATION_OPTIONS (from inputs.JAVA_VALIDATION_OPTIONS) is unquoted.

Locations:

- `action.yml:100`
- `action.yml:362`
- `action.yml:370`
- `action.yml:430`
- `action.yml:471`
- `action.yml:479`
- `action.yml:530`
- `action.yml:560`

### github-env-injection (severity: high)

Workflow files write values sourced from external API responses directly to $GITHUB_ENV without sanitization (no `printf '%s' ... | tr -d '\n\r'` step). A malicious or compromised upstream API could inject newlines to set arbitrary environment variables.

(1) update_firelyterminal.yml: `echo "FIRELY_TERMINAL_VERSION=$FIRELY_TERMINAL_VERSION" >> $GITHUB_ENV` where FIRELY_TERMINAL_VERSION is read from action.yml via yq; and `echo "LATEST_RELEASE=$LATEST_RELEASE" >> $GITHUB_ENV` where LATEST_RELEASE comes from the NuGet API response via wget+jq.

(2) update_javavalidator.yml: `echo "JAVA_VALIDATOR_VERSION=$JAVA_VALIDATOR_VERSION" >> $GITHUB_ENV` (from yq/action.yml) and `echo "LATEST_RELEASE=$LATEST_RELEASE" >> $GITHUB_ENV` (from GitHub API via wget+jq).

(3) update_sushi.yml: `echo "SUSHI_VERSION=$SUSHI_VERSION" >> $GITHUB_ENV` (from yq/action.yml) and `echo "LATEST_RELEASE=$LATEST_RELEASE" >> $GITHUB_ENV` (from GitHub API via wget+jq).

Locations:

- `.github/workflows/update_firelyterminal.yml:26`
- `.github/workflows/update_firelyterminal.yml:33`
- `.github/workflows/update_javavalidator.yml:26`
- `.github/workflows/update_javavalidator.yml:33`
- `.github/workflows/update_sushi.yml:26`
- `.github/workflows/update_sushi.yml:33`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, missing-permissions, script-injection, github-env-injection

**Notes:**

Fixed all four findings:

1. unpinned-uses: Pinned actions/checkout@v4 to full SHA 11d5960a326750d5838078e36cf38b85af677262 in all three workflow files.

2. missing-permissions: Added top-level 'permissions: contents: write / pull-requests: write' to all three workflow files (minimum needed to create branches and PRs).

3. script-injection in action.yml:
   - Quoted $FIRELY_TERMINAL_VERSION in dotnet tool install command
   - Quoted $SUSHI_VERSION in npm install command
   - Replaced 'sushi $INPUT_SUSHI_OPTIONS' with xargs-tokenized array expansion in both SUSHI steps
   - Replaced 'fhir check $INPUT_PATH_TO_QUALITY_CONTROL_RULES' with ${VAR:+"$VAR"} form (optional single path)
   - Replaced 'eval echo "$JAVA_VALIDATOR_DOWNLOAD_LOCATION"' with safe bash string substitution; quoted wget URL
   - Replaced unquoted 'for p in $INPUT_PATH_TO_CONFORMANCE_RESOURCES' with xargs-tokenized array in both Java validator steps
   - Replaced unquoted '$INPUT_JAVA_VALIDATION_OPTIONS' with xargs-tokenized array in both Java validator steps
   - Replaced unquoted '$PATH_TO_EXAMPLES' with xargs-tokenized array in examples step
   - All java command invocations now use array expansion for safe argument passing

4. github-env-injection: All GITHUB_ENV writes in the three workflow files now sanitize values with 'safe=$(printf \'%s\' "$VAR" | tr -d \'\\n\\r\')' before writing.

### Iteration 1

**Fixes applied:** script-injection

**Notes:**

Fixed 4 unquoted shell variable expansions in action.yml:
1. 'Simplifier login' step (line 148): Quoted `$INPUT_SIMPLIFIER_USERNAME` and `$INPUT_SIMPLIFIER_PASSWORD` in the `fhir login` command — changed `email=$INPUT_SIMPLIFIER_USERNAME password=$INPUT_SIMPLIFIER_PASSWORD` to `"email=$INPUT_SIMPLIFIER_USERNAME" "password=$INPUT_SIMPLIFIER_PASSWORD"`.
2. 'Run Quality Control checks' step (line 310): Changed `echo $INPUT_EXPECTED_FAILS` to `echo "$INPUT_EXPECTED_FAILS"`.
3. 'Validate all conformance resources' step (line 390): Changed `echo $INPUT_EXPECTED_FAILS` to `echo "$INPUT_EXPECTED_FAILS"`.
4. 'Validate all example resources' step (line 460): Changed `echo $INPUT_EXPECTED_FAILS` to `echo "$INPUT_EXPECTED_FAILS"`.

All variables were already sourced from the step's `env:` block (not inline `${{ }}` expressions), so quoting the expansions is sufficient to prevent shell metacharacter injection.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed all 21+ instances of script injection in hardened/action/action.yml where boolean inputs (DOTNET_VALIDATION_ENABLED, SUSHI_ENABLED, JAVA_VALIDATION_ENABLED, JAVA_SNAPSHOT_ENABLED, TERMINOLOGY_SERVICE_BFARM_ENABLED, CLOSE_SLICING_FOR_VALIDATION, SUSHI_USE_CONFIG_DEPENDENCIES) were expanded unquoted as shell commands using `if $INPUT_*; then`. Each was replaced with the safe form `if [[ "$INPUT_VAR" == 'true' ]]; then`. Compound conditions like `if $A || $B` and `if $A && $B` were also fixed to `if [[ "$A" == 'true' ]] || [[ "$B" == 'true' ]]`. Verified 0 remaining unquoted boolean patterns and 23 correctly quoted replacements.

