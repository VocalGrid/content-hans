# CSV Import Guide for Lead Reactivation

This guide shows the recommended CSV headers and how to map each to GHL fields for the Lead Reactivation use case. It also explains best practices so enrollment and dialing start cleanly.

## Recommended CSV columns and GHL mapping

- first_name → GHL standard field First Name
- last_name → GHL standard field Last Name
- email → GHL standard field Email
- phone → GHL standard field Phone (E.164 preferred, e.g., +13035550101)
- city → GHL standard field City
- state → GHL standard field State
- country → GHL standard field Country
- postal_code → GHL standard field Postal Code
- timezone → GHL standard field Timezone (IANA, e.g., America/Denver). If unknown, leave blank.
- full_address → GHL standard field Address (optional; you can also just provide city/state/zip)
- positions_applied → Custom field `positions_applied`
- date_applied → Custom field `date_applied`
- lead_source → Custom field `lead_source`
- applied_job_city → Custom field `applied_job_city`
- reactivation_campaign → Custom field `reactivation_campaign`
- tags → GHL standard Tags (comma-separated). Include `recruitment.reactivation.enrolled` to trigger 01-enrollment.

Notes:
- Leave these fields blank on import so workflows initialize them:
  - `calls_attempt_count` (will be set to 0 by 01-enrollment if empty)
  - `reactivation_status`, `next_attempt_at`, `last_disposition`
  - All AI fields (summaries, transcripts, salaries, etc.)
- You may also include an optional campaign tag like `campaign Jannis750k` in the `tags` column.

## Formatting rules

- Phone: Prefer E.164 (+1XXXXXXXXXX). If not, system normalizes common US formats.
- Timezone: Use IANA identifiers (e.g., America/Detroit). If blank, scheduler will infer or default.
- Date applied: free text is acceptable (e.g., "September 25"); ISO (YYYY-MM-DD) preferred.

## Import steps in GHL

1. Go to Contacts → Import → Upload CSV.
2. Map columns to fields per the list above. For `tags`, use GHL Tags.
3. Confirm import.
4. Verify that contacts got the `recruitment.reactivation.enrolled` tag (either from CSV or applied in bulk post-import).
5. 01-enrollment workflow will initialize `calls_attempt_count` (only if empty), set scheduling fields, and move to the Enrolled stage.

## Avoid anti-patterns

- Do not prefill outcome fields or AI fields in the CSV.
- Do not add terminal tags like qualified/dnc/bad_number; those are applied by workflows.

## Included files

- template.csv — empty CSV with headers only
- sample.csv — example with 5 leads, ready to import
