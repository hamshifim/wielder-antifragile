---
description: Raw Data Ingestion Tiering and Provenance Discipline
---
# Data Ingestion Guidelines

Raw ingestion should preserve information while keeping reusable, transient, and measured facts separate.

## Three Raw Tiers

1. **Protocol**
   - Reusable method/assay definition.
   - Assign a stable `protocol_id`.
   - Compute a deterministic `protocol_hash` over normalized stable protocol content.
   - Reuse an existing protocol when the hash matches.
   - If the same nominal protocol family yields a new hash, create a new protocol row and warn the user about possible protocol drift.

2. **Experiment Metadata**
   - Transient run/workbook/sample context.
   - Own the `metadata_id` referenced by raw measurements.
   - Store source file, study/run identifiers, study dates, sample grouping, compound/batch, matrix/species, and the linked `protocol_id`.
   - Do not copy this context into every measurement row.

3. **Empirical**
   - Row-level observed measurements only.
   - Include the trailing `metadata_id` join key.
   - Do not repeat source file, protocol, study, compound, matrix, sample-name, or other recurring context fields.

## Audit Leftovers

Keep an audit table for non-empty cells or records not confidently assigned to the three raw tiers. This is not a fourth data tier; it is ingestion accounting. Derived report outputs can live here during raw ingestion until a later derived-data pass explicitly owns them.

## Column Order

Column order should optimize human inspection before storage purity.

- Put the most scientifically or operationally discriminating dimension first.
- Put supporting descriptors next.
- Put technical/provenance fields last.
- IDs, hashes, source paths, scope labels, and bookkeeping columns should usually trail unless the table exists specifically to inspect provenance.

Example: an experiment metadata table should lead with dimensions such as `sample_name`, `compound_id`, `species`, `strain`, `matrix`, and `mw`, then supporting run/study descriptors, and end with `metadata_scope`, `source_file`, `protocol_id`, and `metadata_id`.
