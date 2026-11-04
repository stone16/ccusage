# Date filter

`--since` and `--until` keep all-source daily rows inside an inclusive date window. Both `YYYY-MM-DD` and `YYYYMMDD` are accepted.

## Sub-features

- `date-include` keeps 10 January 2025 when both bounds are that day.
- `date-compact` accepts the same window without dashes.
- `date-exclude` drops the row when `--since` is the next day.

## How to get to it (user POV)

- Run `ccusage daily --since <date> --until <date>`.
- Use either `2025-01-10` or `20250110`.

## Driving it with verify-ccusage

Preconditions:

- `verify-ccusage doctor` printed `doctor=ok`.
- The only daily row is 10 January 2025.
- `CCUSAGE_VERIFY_RUN_ID` is the id from launch.

- **Inclusive window.** Run `ccusage daily --json --since 2025-01-10 --until 2025-01-10 --offline --mode display --timezone UTC --no-color`. The harness command is `.agents/skills/verify-ccusage/scripts/verify-ccusage date-filter`, which runs this argv as `date-include`. Exit code 0. `date-include.stdout` matches the daily JSON in `daily.md`, including `period` `2025-01-10`.
- **Compact window.** Run `ccusage daily --json --since 20250110 --until 20250110 --offline --mode display --timezone UTC --no-color`. The harness runs this argv as `date-compact`. Exit code 0. `date-compact.stdout` matches that same daily JSON.
- **Excluded day.** Run `ccusage daily --json --since 2025-01-11 --offline --mode display --timezone UTC --no-color`. The harness runs this argv as `date-exclude`. Exit code 0. `date-exclude.stdout` has `daily` length 0, `totals.inputTokens` 0, and `totals.totalTokens` 0.
- **Proof.** Keep the three stdout files under `/tmp/ccusage-verify-evidence/$CCUSAGE_VERIFY_RUN_ID/`. An empty list is a successful exclusion, not a skipped entry.

## Gotchas

- A value that is not `YYYY-MM-DD` or `YYYYMMDD`, or that is not a real calendar day, is rejected. Do not expect a silent empty report from `2025-02-31`.
- `--until` is inclusive. `--until 2025-01-10` keeps the fixture.
- This filter selects daily rows. It does not select the weekly `period` `2025-01-06` or the Claude `week` `2025-01-05`.
