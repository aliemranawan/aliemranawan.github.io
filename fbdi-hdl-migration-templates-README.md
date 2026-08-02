# FBDI / HDL Migration Templates & Validation Scripts

Reusable templates and pre-load validation scripts for Oracle Fusion data migration using FBDI (File-Based Data Import) and HDL (HCM Data Loader), plus a decision framework for choosing between FBDI, BICC, and HDL depending on the migration scenario.

This repo is built from patterns used across EBS-to-Fusion migration engagements. All sample data is synthetic.

## Why this exists

A large share of failed or delayed Fusion data loads aren't caused by the load process itself — they're caused by malformed staging files that get discovered only after a failed import, when the error message points at a row number instead of the actual root cause. Catching those issues before the file ever reaches Fusion saves a full load-fail-diagnose-retry cycle, which on a real migration can mean hours per iteration.

## Structure

```
fbdi-hdl-migration-templates/
├── decision-framework/
│   └── fbdi-vs-bicc-vs-hdl.md            -- when to use which, with concrete criteria
├── fbdi-templates/
│   ├── gl_journal_import_template.csv    -- sample structure, synthetic values
│   ├── ap_invoice_import_template.csv
│   └── field_mapping_notes.md            -- source (EBS) -> target (Fusion) field notes
├── hdl-templates/
│   ├── worker_load_template.dat          -- sample HCM Data Loader structure
│   └── hdl_business_object_notes.md      -- common business object gotchas
├── validation/
│   ├── validate_fbdi_staging.sql         -- pre-load checks against staging tables
│   ├── validate_hdl_file_structure.py    -- structural validation before HDL load
│   └── common_errors_reference.md        -- error code -> likely root cause mapping
└── README.md
```

## Decision framework (summary)

| Scenario | Recommended | Why |
|---|---|---|
| Bulk historical transactional data, one-time load | **FBDI** | Purpose-built for high-volume initial loads, validates against Fusion interface tables before import |
| Recurring/incremental extract from Fusion for reporting or downstream systems | **BICC** | Designed for extract, not load; efficient for scheduled outbound data movement |
| HR/Payroll master and transactional data (worker, assignment, comp) | **HDL** | Only supported path for most HCM business objects; FBDI doesn't cover the HCM object model |
| Small one-off corrections or low-volume loads | Manual UI entry or ADFdi spreadsheet | FBDI/HDL overhead isn't worth it below a few hundred records |

Full detail with edge cases (e.g. what happens when a legacy EBS interface table has no clean Fusion equivalent) is in `decision-framework/fbdi-vs-bicc-vs-hdl.md`.

## Validation scripts

`validate_fbdi_staging.sql` runs a set of pre-load checks against FBDI interface/staging tables before you kick off the import — required column population, referential integrity against lookup/reference data, date format consistency, and duplicate detection on natural keys. Point it at your staging schema and it flags rows that will fail before you burn a load cycle finding out.

`validate_hdl_file_structure.py` checks a `.dat` file's structure (METADATA/business object hierarchy, delimiter consistency, required attribute presence) against the expected HDL format before upload.

## How to use this repo

1. Start with the decision framework if you're still scoping the migration approach.
2. Copy the relevant template and adapt field mappings to your actual source/target schema.
3. Run the matching validation script against your staging data before the real load attempt.

## Tech

SQL (validated against Oracle Fusion staging table structures), Python 3 for the HDL structural validator (no external dependencies beyond the standard library).

## Disclaimer

All templates use synthetic sample data. No real client business objects, worker data, financial data, or proprietary mapping documents are included.
