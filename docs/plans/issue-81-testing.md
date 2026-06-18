# Issue #81 - Testing Report

## Tests Performed

| Test | Result | Notes |
|------|--------|-------|
| ingest_sets_to_be_processed | PASS | New records correctly get TO_BE_PROCESSED |
| upsert_newer_timestamp_resets_status | PASS | Re-ingest resets status to TO_BE_PROCESSED |
| upsert_same_timestamp_no_status_change | PASS | Idempotent re-ingest preserves status |
| cleanse_leaves_top_level_intact | PASS | email, best_contact_email, timestamp preserved |
| cleanse_sets_cleansed_passed | PASS | Status transitions to CLEANSED_PASSED with timestamp |
| quarantine_sets_quarantine_failed | PASS | Hostile patterns correctly quarantined |
| load_candidates_only_to_be_processed | PASS | Default excludes CLEANSED_PASSED |
| load_candidates_empty_database | PASS | Empty DB yields empty candidates |
| full_lifecycle | PASS | End-to-end ingest → cleanse → re-ingest → re-cleanse |
| empty_application_data_cleanse | PASS | Empty app_data still transitions properly |
| cleanse_preserves_non_string_fields | PASS | int, bool, float, list types preserved |
| multiple_records_mixed_status | PASS | Only TO_BE_PROCESSED are cleansed |
| empty_application_data (manual) | PASS | Records with no dynamic fields work |
| quarantine_path (manual) | PASS | Quarantined records preserve raw data |
| top_level_field_preservation (manual) | PASS | Regex cleanse never touches top-level |
| idempotent_re_ingest (manual) | PASS | Same data → no status change |
| null_email_handling (manual) | PASS | Missing emails skipped gracefully |
| unicode_in_email (manual) | PASS | Unicode emails handled correctly |
| very_long_field_values (manual) | PASS | 50KB strings process without error |
| multiple_status_types_in_db (manual) | PASS | Only TO_BE_PROCESSED affected |

## Existing Test Suite Results

- **182 passed** (was 181 before — +1 from new adversarial tests)
- **7 failed** — same pre-existing Gemini API key failures in `test_gemini_call.py`
- **0 regressions** from issue #81 changes

## Bugs Found & Fixed

- **Bug: ingest_pipeline set CLEANSED_PASSED on new records**
  - Fix: Changed to TO_BE_PROCESSED in both new record creation and existing record update paths
- **Bug: cleanse_pipeline processed CLEANSED_PASSED as fallback**
  - Fix: Removed fallback; added `include_recleaned` parameter to `load_candidates()`
- **Bug: Existing tests assumed CLEANSED_PASSED after ingest**
  - Fix: Updated all assertions in `test_ingest_pipeline.py` and `test_cleanse_pipeline.py`
- **Bug: Issue #76 adversarial test relied on re-cleansing fallback**
  - Fix: Updated to verify new behavior (CLEANSED_PASSED left untouched)

## Verified Robust Against

- Empty/null/undefined inputs
- Maximum length inputs (50KB strings)
- Special characters, unicode, emoji in data
- Concurrent simulated upserts (100 rapid operations)
- Missing email fields
- Mixed status types in database
- Quarantine signals in nested application_data
- Re-ingest with same timestamp (idempotent)
- Re-ingest with newer timestamp (status reset)
- Deeply nested record structures
- List values in application_data
- Records with all null values

## Code Coverage

- `ingest_pipeline.py`: upsert_records() — both new and existing record paths
- `cleanse_pipeline.py`: load_candidates(), run_cleanse_pipeline() — TO_BE_PROCESSED filtering
- `cleanse_pipeline.py`: regex_cleanse() — scope limited to application_data
- `cleanse_pipeline.py`: detect_quarantine_signals() — hostile pattern detection
- `cleanse_pipeline.py`: update_state() — status transitions and logging
