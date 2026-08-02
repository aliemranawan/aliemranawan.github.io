# PL/SQL Performance Tuning Templates

Reusable patterns, diagnostic scripts, and before/after examples for tuning slow PL/SQL batch processes in Oracle EBS and Fusion environments.

This repo generalizes an approach I used to take a nightly manufacturing batch process from **24 hours to under 10 minutes**. No client data, schemas, or business logic are included — every example here uses synthetic table names and generated data so the patterns are reusable on any Oracle instance.

## Why this exists

Most "slow batch job" problems in EBS/Fusion custom PL/SQL come from a small set of repeat offenders:

- Row-by-row processing where a bulk operation would do
- Missing or unusable indexes on join/filter columns
- Uncommitted large transactions holding undo/redo pressure
- Context switching between SQL and PL/SQL inside tight loops
- Interface table scans without partition pruning

This repo is a checklist-plus-code version of how I diagnose and fix those, structured so you can drop the diagnostic scripts into any schema and start from evidence instead of guesswork.

## Structure

```
plsql-performance-tuning-templates/
├── diagnostics/
│   ├── 01_explain_plan_capture.sql       -- capture and pretty-print an execution plan
│   ├── 02_top_sql_by_elapsed.sql         -- find the worst offenders in a batch window
│   ├── 03_index_usage_check.sql          -- unused/missing index candidates
│   └── 04_wait_event_snapshot.sql        -- what the session is actually waiting on
├── patterns/
│   ├── bulk_collect_forall.sql           -- row-by-row -> BULK COLLECT + FORALL
│   ├── cursor_to_set_based.sql           -- cursor loop -> single set-based UPDATE/MERGE
│   ├── partition_pruning_example.sql     -- filtering on a partition key correctly
│   └── commit_batching.sql               -- safe commit-every-N-rows pattern
├── case-study/
│   └── batch_optimization_writeup.md     -- the Ghani Glass batch process, generalized
└── README.md
```

## The case study, in brief

The original process joined a staging interface table against several EBS base tables inside a `FOR ... LOOP` cursor, performing row-level inserts and updates one record at a time, with commits happening far too frequently relative to the transaction cost. Runtime was ~24 hours on a nightly volume in the hundreds of thousands of rows.

The fix, in order of impact:

1. **Replaced the row-by-row cursor loop with `BULK COLLECT ... FORALL`**, moving from thousands of individual context switches between SQL and PL/SQL engines to a handful of bulk operations.
2. **Rewrote two of the row-level lookups as a single `MERGE` statement**, eliminating redundant table scans that were happening once per row.
3. **Added a composite index** on the staging table's join + filter columns, which the original design never exercised because filtering happened in PL/SQL instead of SQL.
4. **Batched commits** every 5,000 rows instead of per-row, cutting redo/undo contention without risking a single giant uncommitted transaction.

Runtime dropped from 24 hours to under 10 minutes. See `case-study/batch_optimization_writeup.md` for the full generalized before/after code.

## How to use this repo

1. Run the scripts in `diagnostics/` against your own batch job first — don't guess at the bottleneck, measure it.
2. Match your bottleneck to a pattern in `patterns/` (they're written as generic templates with placeholder table/column names).
3. Adapt the case study structure in your own tuning writeup — the "diagnose, isolate the top cost, fix in priority order, re-measure" sequence is more valuable than any single script.

## Tech

Oracle PL/SQL, tested against 11g/12c/19c syntax. No third-party dependencies.

## Disclaimer

All table names, column names, and data volumes in this repo are synthetic. No proprietary client code, schema, or business logic from any employer or client engagement is included.
