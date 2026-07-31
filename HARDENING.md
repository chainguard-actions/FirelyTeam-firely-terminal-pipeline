<!-- markdownlint-disable -->

# Hardening Report: FirelyTeam--firely-terminal-pipeline/v0.8.17

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **FirelyTeam--firely-terminal-pipeline/v0.8.17** was hardened automatically. 3 finding(s) were identified and resolved across 3 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

All three workflow files use `actions/checkout@v4` — a mutable tag reference instead of a pinned 40-character commit SHA. This exposes the workflow to supply-chain attacks if the tag is moved to a malicious commit.

Locations:

- `.github/workflows/update_firelyterminal.yml:14`
- `.github/workflows/update_javavalidator.yml:14`
- `.github/workflows/update_sushi.yml:14`

### permissions (severity: medium)

None of the three workflow files define a `permissions:` key at the top level or at the job level. Without explicit permissions, workflows inherit the default repository permissions (which may include write access), violating the principle of least privilege.

Locations:

- `.github/workflows/update_firelyterminal.yml:1`
- `.github/workflows/update_javavalidator.yml:1`
- `.github/workflows/update_sushi.yml:1`

### script-injection (severity: high)

Multiple `run:` steps in action.yml expand user-controlled input env vars without double-quoting, violating sub-rule (b). An attacker-supplied input containing shell metacharacters (`;`, `|`, `$()`, etc.) can inject arbitrary commands.

1. `fhir login email=$INPUT_SIMPLIFIER_USERNAME password=$INPUT_SIMPLIFIER_PASSWORD` — both SIMPLIFIER_USERNAME and SIMPLIFIER_PASSWORD are unquoted (sourced from `inputs.SIMPLIFIER_USERNAME` / `inputs.SIMPLIFIER_PASSWORD`).
2. `dotnet tool install --global Firely.Terminal --version $FIRELY_TERMINAL_VERSION` — FIRELY_TERMINAL_VERSION is unquoted (sourced from `inputs.FIRELY_TERMINAL_VERSION`).
3. `sudo npm install -g fsh-sushi@$SUSHI_VERSION` — SUSHI_VERSION is unquoted (sourced from `inputs.SUSHI_VERSION`).
4. `sushi $INPUT_SUSHI_OPTIONS` — INPUT_SUSHI_OPTIONS is unquoted (sourced from `inputs.SUSHI_OPTIONS`); appears in two steps.
5. `fhir check $INPUT_PATH_TO_QUALITY_CONTROL_RULES` — INPUT_PATH_TO_QUALITY_CONTROL_RULES is unquoted (sourced from `inputs.PATH_TO_QUALITY_CONTROL_RULES`).
6. `for p in $INPUT_PATH_TO_CONFORMANCE_RESOURCES` — INPUT_PATH_TO_CONFORMANCE_RESOURCES is unquoted (sourced from `inputs.PATH_TO_CONFORMANCE_RESOURCES`); appears in multiple steps.
7. `java -jar validator_cli.jar ... $INPUT_JAVA_VALIDATION_OPTIONS ...` — INPUT_JAVA_VALIDATION_OPTIONS is unquoted (sourced from `inputs.JAVA_VALIDATION_OPTIONS`); appears in four invocations.
8. `JAVA_VALIDATOR_DOWNLOAD_LOCATION=$(eval echo "$JAVA_VALIDATOR_DOWNLOAD_LOCATION")` — uses `eval` on a value sourced from `inputs.JAVA_VALIDATOR_DOWNLOAD_LOCATION`, allowing arbitrary command execution.

Locations:

- `action.yml:148`
- `action.yml:120`
- `action.yml:185`
- `action.yml:200`
- `action.yml:390`
- `action.yml:444`
- `action.yml:449`
- `action.yml:530`
- `action.yml:535`
- `action.yml:560`
- `action.yml:565`
- `action.yml:610`
- `action.yml:615`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, permissions, script-injection

**Notes:**

Fixed all three finding types: (1) Pinned actions/checkout@v4 to full SHA 11d5960a326750d5838078e36cf38b85af677262 in all three workflow files. (2) Added 'permissions: contents: write / pull-requests: write' blocks to all three workflow files (minimum needed for branch creation and PR creation). (3) Fixed all script injection issues in action.yml: double-quoted SIMPLIFIER_USERNAME/PASSWORD in fhir login, FIRELY_TERMINAL_VERSION in dotnet install, SUSHI_VERSION in npm install; converted SUSHI_OPTIONS to bash array for safe optional expansion; used ${VAR:+"$VAR"} for optional PATH_TO_QUALITY_CONTROL_RULES; used read -ra to safely split PATH_TO_CONFORMANCE_RESOURCES and JAVA_VALIDATION_OPTIONS into arrays; replaced eval echo with safe bash string substitution for JAVA_VALIDATOR_DOWNLOAD_LOCATION.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed all script injection vulnerabilities in hardened/action/action.yml:

1. **Boolean inputs used as shell commands** (18+ locations): Replaced all `if $INPUT_DOTNET_VALIDATION_ENABLED; then`, `if $INPUT_JAVA_VALIDATION_ENABLED; then`, `if $INPUT_SUSHI_ENABLED; then`, `if $INPUT_SUSHI_USE_CONFIG_DEPENDENCIES; then`, `if $INPUT_TERMINOLOGY_SERVICE_BFARM_ENABLED; then`, `if $INPUT_JAVA_SNAPSHOT_ENABLED; then`, `if $CLOSE_SLICING_FOR_VALIDATION; then`, and compound `&&`/`||` combinations with proper POSIX string comparisons: `if [ "$VAR" = 'true' ]; then`. This prevents arbitrary string inputs from being executed as shell commands.

2. **Unquoted $INPUT_EXPECTED_FAILS in echo pipes** (3 locations, lines ~530, ~630, ~700): Changed `echo $INPUT_EXPECTED_FAILS | grep -w -q VALIDATION_*` to `echo "$INPUT_EXPECTED_FAILS" | grep -w -q VALIDATION_*` to prevent word splitting and glob expansion on user-controlled input.

All 28 boolean checks now use proper string comparison syntax, and all 3 echo pipes for EXPECTED_FAILS are properly quoted.

### Iteration 1

**Fixes applied:** script-injection

**Notes:**

Fixed script injection vulnerabilities in two Java validator steps of action.yml:

1. 'Validate all conformance resources in scope of the repository' step:
   - Quoted `$IG_DEPENDENCIES` in `echo "$IG_DEPENDENCIES"` (was unquoted)
   - Replaced unquoted string accumulation `LOCAL_IG_PARAMETERS+=...` with a proper bash array `local_ig_params+=(-ig "$GITHUB_WORKSPACE/$p")`
   - Split `$UNESCPAED_IG_DEPENDENCIES` into an array with `read -ra ig_dep_args` and expanded with `"${ig_dep_args[@]}"`
   - Quoted the path prefix in glob patterns: `"$GITHUB_WORKSPACE/$p"*.xml` and `"$GITHUB_WORKSPACE/$p"*.json`

2. 'Validate all example resources in scope of the repository' step:
   - Same fixes applied: quoted `echo "$IG_DEPENDENCIES"`, replaced `COMBINED_IG_PARAMETERS` string with `combined_ig_params` array, used `ig_dep_args` array for IG dependencies, and quoted path prefixes in glob patterns.

The glob characters (*.xml, *.json) remain outside quotes so they still expand as filesystem globs, but the variable portions are now properly quoted to prevent shell metacharacter injection from attacker-controlled input values.

