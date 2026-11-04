# Weekly report

`ccusage weekly` groups every detected agent by week. `ccusage claude weekly` groups Claude usage by a different week start. For the 10 January 2025 fixture those keys are `2025-01-06` and `2025-01-05`.

## Sub-features

- `weekly-table` prints the all-source weekly table.
- `weekly-json` prints one all-source week keyed by `period`.
- `claude-weekly-table` prints the Claude weekly table.
- `claude-weekly-json` prints one Claude week keyed by `week`.

## How to get to it (user POV)

- Run `ccusage weekly` in a terminal.
- Run `ccusage claude weekly` in a terminal.
- Add `--json` for JSON. Omit it for the table.

## Driving it with verify-ccusage

Preconditions:

- `verify-ccusage doctor` printed `doctor=ok`.
- The seed file is the only usage transcript.
- `CCUSAGE_VERIFY_RUN_ID` is the id from launch.

- **All-source table.** Run `ccusage weekly --offline --mode display --timezone UTC --no-color`. The harness command is `.agents/skills/verify-ccusage/scripts/verify-ccusage weekly`, which runs this argv as `weekly-table`. Exit code 0. `weekly-table.stdout` and `weekly-table.pane.txt` contain `Coding (Agent) CLI Usage Report - Weekly`. The stdout also contains `2025-01-06`.
- **All-source JSON.** Run `ccusage weekly --json --offline --mode display --timezone UTC --no-color`. The harness runs this argv as `weekly-json`. Exit code 0. `weekly-json.stdout` has one `weekly` object whose `period` is `2025-01-06`, `agent` is `all`, `inputTokens` is 100, `outputTokens` is 250, `totalTokens` is 365, and `totalCost` is 0.01.
- **Claude table.** Run `ccusage claude weekly --offline --mode display --timezone UTC --no-color`. The harness runs this argv as `claude-weekly-table`. Exit code 0. `claude-weekly-table.stdout` and `claude-weekly-table.pane.txt` contain `Claude Code Token Usage Report - Weekly` and `2025-01-05`.
- **Claude JSON.** Run `ccusage claude weekly --json --offline --mode display --timezone UTC --no-color`. The harness runs this argv as `claude-weekly-json`. Exit code 0. `claude-weekly-json.stdout` has one `weekly` object whose `week` is `2025-01-05`, `inputTokens` is 100, `outputTokens` is 250, `totalTokens` is 365, and `totalCost` is 0.01.
- **Proof.** Keep both pane transcripts and both JSON stdout files under `/tmp/ccusage-verify-evidence/$CCUSAGE_VERIFY_RUN_ID/`.

## Gotchas

- The all-source week starts on Monday, so this fixture's `period` is `2025-01-06`.
- `ccusage claude weekly` starts on Sunday unless `--start-of-week` is set, so this fixture's `week` is `2025-01-05`.
- `--since` and `--until` on `ccusage daily` do not select these week keys. See `date-filter.md`.
