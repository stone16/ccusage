# ccusage verification map

This directory is the maintained source for verifying the user-facing behavior of the `ccusage` CLI. Read the index before driving the app, then use the matching feature file as the recipe.

## Baseline preconditions

- Build the debug binary with `verify-ccusage launch` so `rust/target/debug/ccusage` exists.
- Set `CCUSAGE_VERIFY_RUN_ID` and keep it for doctor, drive, and cleanup.
- Launch writes one Claude transcript at `projects/project1/session1/chat.jsonl` inside `/tmp/ccusage-verify-$CCUSAGE_VERIFY_RUN_ID/claude`.
- The transcript timestamp is `2025-01-10T10:00:00.000Z`, model `claude-opus-4-6`, 100 input tokens, 250 output tokens, 10 cache-creation tokens, 5 cache-read tokens, `costUSD` 0.01.
- Home, XDG config, and XDG data directories inside that scratch tree start empty.
- Run `verify-ccusage doctor` and require `doctor=ok` before a drive.
- Never point `CLAUDE_CONFIG_DIR` or `HOME` at a directory this run did not create.

## Driving conventions

- Start every recipe from the baseline state unless its preconditions say otherwise.
- Treat every command as literal. Keep quoted names and flags unchanged.
- Run reports through `.agents/skills/verify-ccusage/scripts/verify-ccusage <feature>`.
- The harness uses tmux session `ccusage-verify-$CCUSAGE_VERIFY_RUN_ID`, `env -i`, and `unshare -r -n`.
- Pass `--offline --mode display --timezone UTC --no-color` and `LOG_LEVEL=0`.
- Restore nothing in the fixture. Cleanup deletes the scratch tree. Do not remove proof artifacts during cleanup.

## Proof and skip reporting

- Capture the user action and the resulting state, not only the final screen.
- CLI proof includes the command, stdout, stderr, the exit code, and the tmux pane transcript.
- JSON proof is the stdout file checked with `jq`.
- Table proof is the pane transcript and must contain the report title.
- Record the feature id and entry point used with every artifact. The `<name>.argv` file is that record.
- Report an unreachable path with the attempted command and the unmet precondition.
- Do not report a skipped entry point as verified through a different path.

## Feature entry contract

Each feature file starts with an H1 title and one paragraph describing the user-visible behavior. It then uses exactly four H2 sections in this order.

1. `Sub-features` lists short IDs with one line for each behavior.
2. `How to get to it (user POV)` lists every user entry point.
3. `Driving it with verify-ccusage` starts with `Preconditions:` and uses labeled bullets that pair each user action with an exact command and observable result.
4. `Gotchas` lists traps that can waste or invalidate a verification run.

## Features

- [Daily report](./daily.md) covers bare `ccusage` and `ccusage daily`.
- [Claude daily report](./claude-daily.md) covers `ccusage claude daily`.
- [Weekly report](./weekly.md) covers `ccusage weekly` and `ccusage claude weekly`.
- [Session report](./session.md) covers `ccusage session` and `ccusage claude session`.
- [Date filter](./date-filter.md) covers inclusive `--since` and `--until` on `ccusage daily`.
