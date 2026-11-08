# Session report

`ccusage session` lists detected sessions in the all-source shape. `ccusage claude session` lists Claude sessions, and `ccusage claude session --id session1` prints that session's entries instead of the list.

## Sub-features

- `session-table` prints the all-source session table.
- `session-json` prints one all-source session whose `period` is the session id.
- `claude-session-table` prints the Claude session table.
- `claude-session-json` prints one Claude session row.
- `claude-session-id` prints the entries for `session1`.

## How to get to it (user POV)

- Run `ccusage session` in a terminal.
- Run `ccusage claude session` in a terminal.
- Run `ccusage claude session --id session1` in a terminal.
- Add `--json` for JSON. Omit it for the table.

## Driving it with verify-ccusage

Preconditions:

- `verify-ccusage doctor` printed `doctor=ok`.
- The seed file is `projects/project1/session1/chat.jsonl`.
- `CCUSAGE_VERIFY_RUN_ID` is the id from launch.

- **All-source table.** Run `ccusage session --offline --mode display --timezone UTC --no-color`. The harness command is `.agents/skills/verify-ccusage/scripts/verify-ccusage session`, which runs this argv as `session-table`. Exit code 0. `session-table.stdout` contains `Coding (Agent) CLI Usage Report - Session`. `session-table.pane.txt` contains `session1`.
- **All-source JSON.** Run `ccusage session --json --offline --mode display --timezone UTC --no-color`. The harness runs this argv as `session-json`. Exit code 0. `session-json.stdout` has one `session` object whose `period` is `session1`, `agent` is `claude`, `metadata.lastActivity` is `2025-01-10T10:00:00.000Z`, `inputTokens` is 100, `outputTokens` is 250, and `totalTokens` is 365.
- **Claude table.** Run `ccusage claude session --offline --mode display --timezone UTC --no-color`. The harness runs this argv as `claude-session-table`. Exit code 0. `claude-session-table.stdout` contains `Claude Code Token Usage Report - By Session`. `claude-session-table.pane.txt` contains `session1`.
- **Claude JSON.** Run `ccusage claude session --json --offline --mode display --timezone UTC --no-color`. The harness runs this argv as `claude-session-json`. Exit code 0. `claude-session-json.stdout` has one `sessions` object whose `sessionId` is `session1`, `projectPath` is `project1`, `lastActivity` is `2025-01-10T10:00:00.000Z`, `inputTokens` is 100, `outputTokens` is 250, and `totalTokens` is 365.
- **Id entry.** Run `ccusage claude session --id session1 --json --offline --mode display --timezone UTC --no-color`. The harness runs this argv as `claude-session-id-json`. Exit code 0. The JSON has `sessionId` `session1` and one `entries` object whose `model` is `claude-opus-4-6`, `inputTokens` is 100, `outputTokens` is 250, and `costUSD` is 0.01.
- **Proof.** Keep the pane transcripts and the three JSON stdout files under `/tmp/ccusage-verify-evidence/$CCUSAGE_VERIFY_RUN_ID/`.

## Gotchas

- The session id is the directory name `session1`. The seed line has no `sessionId` field.
- `ccusage session --json` uses the key `session`. `ccusage claude session --json` uses `sessions`. `--id` uses `entries` and is not the list shape.
- `projectPath` on the Claude list row is `project1`, not a filesystem path.
