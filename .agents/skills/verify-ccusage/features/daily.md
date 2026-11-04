# Daily report

Bare `ccusage` and `ccusage daily` print every detected agent for one day. On the verification fixture the only agent is Claude, and the row is 10 January 2025.

## Sub-features

- `daily-table` prints the all-source daily table, including the box title.
- `daily-json` prints one JSON day keyed by `period`.
- `bare-json` accepts `ccusage` with no subcommand and returns the same JSON.

## How to get to it (user POV)

- Run `ccusage` with no subcommand.
- Run `ccusage daily` in a terminal.
- Add `--json` for JSON. Omit it for the table.

## Driving it with verify-ccusage

Preconditions:

- `verify-ccusage doctor` printed `doctor=ok`.
- The seed file is the only usage transcript in the scratch home.
- `CCUSAGE_VERIFY_RUN_ID` is the id from launch.

- **Table entry.** Run `ccusage daily --offline --mode display --timezone UTC --no-color`. The harness command is `.agents/skills/verify-ccusage/scripts/verify-ccusage daily`, which runs this argv as `daily-table`. Exit code 0. `daily-table.stdout` and `daily-table.pane.txt` contain `Coding (Agent) CLI Usage Report - Daily`, `2025-01-10`, and `opus-4-6`.
- **JSON entry.** Run `ccusage daily --json --offline --mode display --timezone UTC --no-color`. The same harness command runs this argv as `daily-json`. Exit code 0. `daily-json.stdout` has one `daily` object whose `period` is `2025-01-10`, `agent` is `all`, `metadata.agents` is `["claude"]`, `inputTokens` is 100, `outputTokens` is 250, `cacheCreationTokens` is 10, `cacheReadTokens` is 5, `totalTokens` is 365, `modelsUsed` is `["claude-opus-4-6"]`, and `totalCost` is 0.01.
- **No-subcommand entry.** Run `ccusage --json --offline --mode display --timezone UTC --no-color`. The harness runs this argv as `bare-json`. Exit code 0. The stdout file is byte-identical to `daily-json.stdout`.
- **Proof.** Keep `daily-table.pane.txt`, `daily-json.stdout`, and `bare-json.stdout` under `/tmp/ccusage-verify-evidence/$CCUSAGE_VERIFY_RUN_ID/`.

## Gotchas

- `ccusage claude daily` is a different report. Its JSON uses `date` and has no `agent` field. See `claude-daily.md`.
- The table shortens the model to `opus-4-6`. The JSON `modelsUsed` value stays `claude-opus-4-6`.
- `LOG_LEVEL=0` hides the box title. Leave it unset when the title is part of the proof.
- `--mode calculate` prices tokens from the catalog and will not keep `totalCost` at 0.01.
