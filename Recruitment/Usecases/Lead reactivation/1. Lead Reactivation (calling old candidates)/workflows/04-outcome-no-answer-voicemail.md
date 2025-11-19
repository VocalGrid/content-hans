# Workflow 04 Outcome No Answer or Voicemail

- **Trigger**: Custom field changed `reactivation_status`
- **Condition**: value is `no_answer` or `voicemail`

## Steps in GHL builder
- **Trigger** Contact Changed
  - Filters: Field equals `reactivation_status`
  - When: Updated

- **If Else** condition
  - Group: Contact Field
  - Condition: `reactivation_status` includes any of [`no_answer`, `voicemail`]

- Branch: Yes
  - **If Else** guard
    - Group: Opportunity
    - Condition: Stage is any of [To Dial – Attempt 1, To Redial – Attempt 2]
  - Branch: Yes
    - **If Else** condition
      - Group: Contact Field
      - Condition: `calls_attempt_count` equals `1`
    - Branch: Yes
      - **Create/Update Opportunity**
        - Pipeline: Recruitment – Lead Reactivation
        - Stage: Attempt 1 Completed
      - **Add Contact Tag** `recruitment.reactivation.attempt1.done`
    - Branch: No
      - **Create/Update Opportunity**
        - Pipeline: Recruitment – Lead Reactivation
        - Stage: Attempt 2 Completed (No Answer)
      - **Add Contact Tag** `recruitment.reactivation.attempt2.done`
    - **Remove Contact Tag** `recruitment.reactivation.leased`
    - **Add Note** `Outcome {{reactivation_status}} recorded at {{now}}`
  - Branch: No
    - End (handled by other workflows)

- Branch: No
  - End (handled by other workflows)

## Fields and tags touched
- **Fields**: reactivation_status, calls_attempt_count
- **Tags**: recruitment.reactivation.attempt1.done, recruitment.reactivation.attempt2.done, recruitment.reactivation.leased

## Outcome
- Board reflects an attempt outcome and the record is ready for next steps or fallback.
