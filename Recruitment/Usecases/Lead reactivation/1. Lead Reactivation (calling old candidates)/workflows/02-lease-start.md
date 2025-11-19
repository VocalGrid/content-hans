# Workflow 02 Lease Start Dial Prep

- **Trigger**: Contact tag added `recruitment.reactivation.leased`
- **Preconditions**: DND is off

## Steps in GHL builder
- **Trigger** Contact Tag
  - Match: Tag added equals `recruitment.reactivation.leased`
  - Re-enrollment: On (already enabled, allows leasing again later)

- **Update Contact Field**
  - Field: `reactivation_status`
  - Value: `in_progress`

- **If Else** condition
  - Group: Contact Field
  - Condition: `calls_attempt_count` equals `0`

- Branch: Yes
  - **Create/Update Opportunity**
    - Pipeline: Recruitment – Lead Reactivation
    - Stage: To Dial – Attempt 1
    - Title: keep existing
  - **Add Note** `Dial attempt 1 started.`

- Branch: No
  - **Create/Update Opportunity**
    - Pipeline: Recruitment – Lead Reactivation
    - Stage: To Redial – Attempt 2
  - **Add Note** `Dial attempt 2 started.`

## Fields and tags touched
- **Fields**: reactivation_status
- **Tags**: recruitment.reactivation.leased (kept until cleared by n8n)

## Outcome
- Record is clearly in flight and visible on the board at the right column.

## Notes
- Do not add any outcome tags here. Outcome workflows (04/04b/06/07) own result tagging and lease cleanup.
- This workflow only marks in-progress, places the record into the correct dialable stage, and adds a note.
