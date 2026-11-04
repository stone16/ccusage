# Claude daily report

`ccusage claude daily` prints Claude Code usage grouped by day. It does not use the all-source `period` field.

## Sub-features

- `claude-daily-table` prints the Claude daily table.
- `claude-daily-json` prints one JSON day keyed by `date`.

## How to get to it (user POV)

- Run `ccusage claude daily` in a terminal.
- Add `--json` for JSON. Omit it for the table.

## Driving it with verify-ccusage

Preconditions:

- `verify-ccusage doctor` printed `doctor=ok`.
- The seed file is the only Claude transcript.
- `CCUSAGE_VERIFY_RUN_ID` is the id from launch.

- **Table entry.** Run `ccusage claude daily --offline --mode display --timezone UTC --no-color`. The harness command is `.agents/skills/verify-ccusage/scripts/verify-ccusage claude-daily`, which runs this argv as `claude-daily-table`. Exit code 0. `claude-daily-table.stdout` and `claude-daily-table.pane.txt` contain `Claude Code Token Usage Report - Daily` and `2025-01-10`.
- **JSON entry.** Run `ccusage claude daily --json --offline --mode display --timezone UTC --no-color`. The same harness command runs this argv as `claude-daily-json`. Exit code 0. `claude-daily-json.stdout` has one `daily` object whose `date` is `2025-01-10`, with no `agent` field, `inputTokens` 100, `outputTokens` 250, `cacheCreationTokens` 10, `cacheReadTokens` 5, `totalTokens` 365, `modelsUsed` `["claude-opus-4-6"]`, and `totalCost` 0.01.
- **Proof.** Keep `claude-daily-table.pane.txt` and `claude-daily-json.stdout` under `/tmp/ccusage-verify-evidence/$CCUSAGE_VERIFY_RUN_ID/`.

## Gotchas

- A passing `ccusage daily` proof does not prove this command. The JSON keys differ.
- `LOG_LEVEL=0` hides the box title.
- Without `--timezone UTC`, a non-UTC host can bucket this timestamp on another date.
