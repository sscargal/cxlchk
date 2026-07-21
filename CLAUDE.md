# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

`cxlchk` is a pure Bash tool that checks a Linux host's Compute Express Link (CXL) configuration for known issues. It has two phases:

- **Collector** (`collector/`) — gathers raw data from the host (command output, `/proc` files, `cxl`/`daxctl` tool output) into a timestamped output directory.
- **Analyzer** (`analyzer/`) — runs independent rule scripts against the collected data and reports PASSED/FAILED/WARNING/CRITICAL/INFO/SKIPPED/UNKNOWN for each.

Entry point is `./cxlchk` (must be run with root privilege when collecting live, since it reads `dmidecode`, `/var/log/messages`, etc.).

## Commands

Run cxlchk (collector + analyzer) against the live host:
```
sudo ./cxlchk
```

Collect only (no analysis):
```
sudo ./cxlchk -C
```

Analyze a previously collected dataset (no root required, no new collection):
```
./cxlchk -A ./cxlchk.<hostname>.<mmdd-HHMM>
```

List available analyzer modules and rules:
```
./cxlchk -l
```

Increase verbosity (stackable): `-v`, `-vv`, `-vvv`.

Lint (this is what CI runs — `.github/workflows/main.yml`):
```
shellcheck cxlchk collector/collector collector/modules/* analyzer/analyzer analyzer/*/* common/common common/debug
```
There is no automated test suite; ShellCheck is the only CI gate. Verify behavior changes by actually running `sudo ./cxlchk` (or `-A` against a saved dataset) and reading the console output/report summary.

## Architecture

Everything is Bash, sourced (`. file`) rather than executed as subprocesses, so all scripts share one global variable/function namespace and process. This is why:

- `cxlchk` sources `common/common` (and `common/debug` when `-v` is used) before doing anything else, defining shared helpers (`passed_msg`, `failed_msg`, `info_msg`, `warn_msg`, `crit_msg`, `debug_msg`, the progress bar functions) and color/string globals (`STR_PASSED`, `green`, etc.) used everywhere downstream.
- Path-to-tool globals (`GREP`, `SED`, `CXLCLI`, `DAXCTL`, `OUTPUT_PATH`, `OPT_VERBOSITY`, ...) are set once in `cxlchk` and read directly by every collector/analyzer script — new modules should use these rather than calling tools directly, to stay consistent with `-c`/verbosity overrides.
- **Collector modules** (`collector/modules/*`): `collector/collector` globs every file under `collector/modules/` and sources it in turn. Each module runs its own commands, writes stdout/stderr to `${OUTPUT_PATH}/<name-with-spaces-as-underscores>` and `<...>.err`, and calls `init_progressbar`/`inc_progressbar` for user feedback. Adding a new data source = adding a new file to `collector/modules/`; no registration step needed.
- **Analyzer rules** (`analyzer/<module>/<rule>`): `analyzer/analyzer` globs every file under `analyzer/` (excluding itself) and sources it. Each rule file:
  - guards against being sourced twice with an `_MODULE_CHECK_..._` sentinel variable,
  - defines one function (e.g. `cxl_find_devices`) that reads a specific collected file from `${OUTPUT_PATH}`,
  - calls that function once at the bottom of the file, passing the collected filename as an argument,
  - reports its verdict via `rule_result <TYPE> "<message>"`, where `<TYPE>` is one of PASSED/FAILED/WARNING/INFO/CRITICAL/SKIPPED (case-insensitive) — `rule_result` in `analyzer/analyzer` dispatches to the right `*_msg` helper and increments the matching `REPORT_COUNT_*` counter.
  - Adding a new check = adding a new file under `analyzer/<module>/`; the module directory name becomes the module namespace shown by `-l`.
- Rules and collector modules are independent and unordered — don't assume execution order between them, and always guard file reads with an existence check (`[[ ! -f ${FNAME} ]]`) followed by `rule_result SKIPPED` — every existing rule follows this pattern for missing data.
- `cxlchk` writes all stdout/stderr to `${OUTPUT_PATH}/cxlchk.log` via `log_stdout_stderr` (using `tee` if available), so anything printed during collection or analysis ends up in that per-run log alongside the raw collected files.
