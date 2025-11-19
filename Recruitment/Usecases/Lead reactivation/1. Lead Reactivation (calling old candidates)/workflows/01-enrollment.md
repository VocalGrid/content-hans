# Workflow 01 Enrollment

- **Trigger**: Contact tag added `recruitment.reactivation.enrolled`
- **Preconditions**: None

## Steps in GHL builder
- **Trigger** Contact Tag
  - Match: Tag added equals `recruitment.reactivation.enrolled`
  - Re-enrollment: Off (prevent duplicates)

- **If Else** condition
  - Group: Contact Field
  - Condition: `calls_attempt_count` is empty

- Branch: Yes
  - **Update Contact Field**
    - Field: `calls_attempt_count`
    - Value: `0`
  - **Update Contact Field**
    - Field: `reactivation_campaign`
    - Value: static campaign label (e.g., `Jannis750k`) or mapped via import
  - **Create/Update Opportunity**
    - Pipeline: Recruitment – Lead Reactivation
    - Stage: Enrolled
    - Title: `Reactivation – {{contact.first_name}} {{contact.last_name}}`
    - Owner: optional
  - **Update Contact Field**
    - Field: `next_attempt_at`
    - Value: now (scheduler enforces call window)
  - **Add Note**
    - Text: `Enrolled in Reactivation on {{now}}`

- Branch: No
  - End (already initialized)

## Fields and tags touched
- **Fields**: calls_attempt_count, reactivation_campaign, next_attempt_at
- **Tags**: recruitment.reactivation.enrolled (trigger only)

## Outcome
- Contact is primed for dialing. Attempt counters and scheduling are initialized.

## Notes
- Guard: only set `calls_attempt_count = 0` when the field is empty, to avoid overwriting any prior attempts.
- Anti-loop: this workflow initializes enrollment only; outcome workflows are responsible for moving stages after calls.
