# Workflow 05 SMS Fallback

- **Trigger**: Custom field changed or scheduled check
- **Condition**: calls_attempt_count == 2 and last_disposition not answered

## Steps in GHL builder
- **Trigger** Contact Changed
  - Filters: Field equals `calls_attempt_count`
  - When: Updated

- **If Else** condition
  - Group: Contact Field
  - Conditions:
    - `calls_attempt_count` equals `2`
    - AND `last_disposition` does not include `answered`

- Branch: Yes
  - **Send SMS**
    - Message: Outreach text for fallback
  - **Add Note** `SMS fallback sent on {{now}}`

- Branch: No
  - End

## Fields and tags touched
- **Fields**: calls_attempt_count, last_disposition
- **Tags**: none

## Outcome
- A final SMS outreach is sent after two attempts without answer.
