# Workflow 03b Qualified (Field → Tag bridge)

- Trigger: Custom field changed `reactivation_status`
- Purpose: Normalize field-driven qualification to the tag-driven handoff (Workflow 03).
- Default: Disabled (optional variant). Enable only if you expect `reactivation_status = qualified` to arrive from external systems.

## Steps in GHL builder
- Trigger Contact Changed
  - Filters: Field equals `reactivation_status`
  - When: Updated

- If Else condition
  - Group: Contact Field
  - Condition: `reactivation_status` equals `qualified`

- Branch: Yes
  - Add Contact Tag `recruitment.reactivation.qualified`
  - Add Note `Qualified via status; normalized to tag on {{now}}`
  - End (Workflow 03 will handle the stage move and notifications)

- Branch: No
  - End

## Fields and tags touched
- Fields: reactivation_status (read only)
- Tags: recruitment.reactivation.qualified (added)

## Outcome
- Qualification consistently flows through Workflow 03 (tag-driven mover).
