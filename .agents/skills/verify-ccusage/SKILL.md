---
name: verify-ccusage
description: Drives the ccusage CLI in an isolated fixture and records terminal plus JSON evidence. Use when proving daily, weekly, session, date-filter, or all-source reports, or when a ccusage change needs a user-level check.
---

# Verify ccusage

The primary surface is the `ccusage` CLI. The VitePress docs site is out of scope. Two runs can proceed together when each sets its own `CCUSAGE_VERIFY_RUN_ID`. Refuse to drive a `HOME` or `CLAUDE_CONFIG_DIR` this run did not create.

Cursor and Claude Code read one tree. The files live in `.agents/skills/verify-ccusage/`. `.cursor/skills/verify-ccusage` is a symlink to that tree. Claude Code receives the same tree at `.claude/skills` from `sync-agent-skills` (`nix/agent-skills.nix`). Do not commit `.claude/skills`.

## Launch

From the repository root:

```sh
export CCUSAGE_VERIFY_RUN_ID=verify-$RANDOM
.agents/skills/verify-ccusage/scripts/verify-ccusage launch
```

Ready when stdout contains a line `ready ccusage <version>` and `<version>` is the `version` field in the root `package.json`. Launch creates `/tmp/ccusage-verify-$CCUSAGE_VERIFY_RUN_ID/` with an empty home and one Claude transcript, and it builds `rust/target/debug/ccusage` when that file is missing. There is no server and no migration.

The seed is verification scaffolding, removed by cleanup:

`projects/project1/session1/chat.jsonl` with timestamp `2025-01-10T10:00:00.000Z`, model `claude-opus-4-6`, 100 input tokens, 250 output tokens, 10 cache-creation tokens, 5 cache-read tokens, and `costUSD` 0.01.

This environment had no Nix dev shell. Launch runs `cargo build --manifest-path rust/Cargo.toml --bin ccusage` instead of `just ccusage::build-rust`, which passes `--release`. When `CCUSAGE_PRICING_JSON_PATH` is unset, launch downloads the LiteLLM snapshot pinned in `flake.lock` to `/tmp/ccusage-litellm-pricing.json` and exports that path. The Nix dev shell sets the variable from the flake input and does not perform that download. A run inside the Nix shell still has to confirm the release binary.

## Doctor

```sh
.agents/skills/verify-ccusage/scripts/verify-ccusage doctor
```

Doctor is read-only. It exits 0 only when the debug binary runs, `ccusage --version` matches root `package.json`, and the seed file is inside this run's scratch directory. There is no listening port and no auth. The report is `/tmp/ccusage-verify-evidence/$CCUSAGE_VERIFY_RUN_ID/doctor.txt`. Run doctor before a drive whenever output looks wrong.

## Drive

The harness is tmux session `ccusage-verify-$CCUSAGE_VERIFY_RUN_ID` at 200 columns. Each report runs under `unshare -r -n` and `env -i`, so the parent process's agent directories and network are not available. Flags on every report: `--offline --mode display --timezone UTC --no-color`. Leave `LOG_LEVEL` unset. `LOG_LEVEL=0` hides the box title.

```sh
.agents/skills/verify-ccusage/scripts/verify-ccusage daily
.agents/skills/verify-ccusage/scripts/verify-ccusage claude-daily
.agents/skills/verify-ccusage/scripts/verify-ccusage weekly
.agents/skills/verify-ccusage/scripts/verify-ccusage session
.agents/skills/verify-ccusage/scripts/verify-ccusage date-filter
```

Read the matching file in `features/` before driving that feature. Bare `ccusage` and `ccusage daily` print the same all-source report. `ccusage claude daily` prints the Claude-only report.

## Evidence

Proof files stay in `/tmp/ccusage-verify-evidence/$CCUSAGE_VERIFY_RUN_ID/`. Each drive writes `<name>.argv`, `<name>.stdout`, `<name>.stderr`, `<name>.exit`, and `<name>.pane.txt`. JSON assertions read the stdout file. Table assertions read both the stdout file and the pane transcript, which must show the report title after the command, not only a later screen. A non-zero exit or a timeout fails the drive. `unshare -r -n` failing on a connect means `--offline` still used the network. Do not treat the flag name as proof.

## Cleanup

```sh
.agents/skills/verify-ccusage/scripts/verify-ccusage cleanup
```

Cleanup kills tmux session `ccusage-verify-$CCUSAGE_VERIFY_RUN_ID` and deletes `/tmp/ccusage-verify-$CCUSAGE_VERIFY_RUN_ID`. It does not delete `/tmp/ccusage-verify-evidence/$CCUSAGE_VERIFY_RUN_ID`.

## Helpers

The only helper is `.agents/skills/verify-ccusage/scripts/verify-ccusage`. It requires `bash`, `cargo`, `jq`, `tmux`, and `unshare`. Invoke it by the path above. Subcommands are `launch`, `doctor`, `daily`, `claude-daily`, `weekly`, `session`, `date-filter`, and `cleanup`.

## Maintenance

When reports or flags change, update this skill with `/maintain-verification-skill`.
