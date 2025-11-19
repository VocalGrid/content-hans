# Workflow 07 Bad Number

- **Trigger**: Contact tag added `recruitment.reactivation.bad_number` or `reactivation_status` set to `bad_number`
- **Preconditions**: None

## Steps in GHL builder
- **Trigger** Contact Tag
  - Match: Tag added equals `recruitment.reactivation.bad_number`
  - Re-enrollment: On

- Or: **Trigger** Contact Changed
  - Filters: Field equals `reactivation_status`
  - When: Updated
  - Condition in flow: value equals `bad_number`

- **Create/Update Opportunity**
  - Pipeline: Recruitment – Lead Reactivation
  - Stage: Bad Number

- **Update Contact Field**
  - Field: `next_attempt_at`
  - Value: clear

- **Remove Contact Tag** `recruitment.reactivation.leased`

- **Remove Contact Tag** `recruitment.reactivation.enrolled`

- **Add Note** `Bad number detected; removed from future attempts`

## Fields and tags touched
- **Fields**: next_attempt_at
- **Tags**: recruitment.reactivation.bad_number, recruitment.reactivation.leased, recruitment.reactivation.enrolled

## Outcome
- Record is flagged as undialable and removed from future attempts.
 - Enrollment tag cleared to prevent future batch re-enrollment.
