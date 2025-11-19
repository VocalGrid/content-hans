# Workflow 04b Outcome Answered but Not Qualified (Not Looking)

- Trigger: Custom field changed `reactivation_status`
- Condition: `reactivation_status = answered` AND `still_looking = false`
- Purpose: Handle answered calls where the candidate is not looking. Close out the attempt cleanly and remove from future batches.

## Steps in GHL builder
- Trigger Contact Changed
  - Filters: Field equals `reactivation_status`
  - When: Updated

- If Else condition (status)
  - Group: Contact Field
  - Condition: `reactivation_status` equals `answered`

- Branch: Yes
  - If Else condition (still_looking)
    - Group: Contact Field
    - Condition: `still_looking` equals `false`
  - Branch: Yes
    - If Else guard (current stage is dialable)
      - Group: Opportunity
      - Condition: Stage is any of [To Dial – Attempt 1, To Redial – Attempt 2]
    - Branch: Yes
      - If Else condition (attempt split)
        - Group: Contact Field
        - Condition: `calls_attempt_count` equals `1`
      - Branch: Yes
        - Create/Update Opportunity
          - Pipeline: Recruitment – Lead Reactivation
          - Stage: Attempt 1 Completed
      - Branch: No
        - Create/Update Opportunity
          - Pipeline: Recruitment – Lead Reactivation
          - Stage: Attempt 2 Completed (No Answer)
      - Remove Contact Tag `recruitment.reactivation.leased`
      - Remove Contact Tag `recruitment.reactivation.enrolled`
      - Add Note `Answered – not looking on {{now}}`
    - Branch: No
      - End (already moved elsewhere)
  - Branch: No
    - End (handled by other workflows)

- Branch: No
  - End

## Fields and tags touched
- Fields: reactivation_status, still_looking, calls_attempt_count (read)
- Tags: recruitment.reactivation.leased (removed), recruitment.reactivation.enrolled (removed)

## Outcome
- Answered but not looking is treated as terminal; card is moved to the relevant Attempt Completed stage, lease is cleared, and the record is excluded from future batch enrollments.
