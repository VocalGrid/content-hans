# Workflow 06 DNC Handling

- **Trigger**: Contact tag added `recruitment.reactivation.dnc` or DND turned on
- **Preconditions**: None

## Steps in GHL builder
- **Trigger** Contact Tag
  - Match: Tag added equals `recruitment.reactivation.dnc`
  - Re-enrollment: On

- Or: **Trigger** Contact Do Not Disturb (DND)
  - When: Turned on

- **Disable/Enable DND**
  - Mode: Enable DND (respect opt out)

- **Create/Update Opportunity**
  - Pipeline: Recruitment – Lead Reactivation
  - Stage: Not Looking / DNC

- **Update Contact Field**
  - Field: `next_attempt_at`
  - Value: clear

- **Remove Contact Tag** `recruitment.reactivation.leased`

- **Remove Contact Tag** `recruitment.reactivation.enrolled`

- **Add Note** `DNC applied on {{now}}`

## Fields and tags touched
- **Fields**: DND, next_attempt_at
- **Tags**: recruitment.reactivation.dnc, recruitment.reactivation.leased, recruitment.reactivation.enrolled

## Outcome
- Record is taken out of dialing and marked as do not contact.
 - Enrollment tag cleared to avoid future batch re-enrollment.
