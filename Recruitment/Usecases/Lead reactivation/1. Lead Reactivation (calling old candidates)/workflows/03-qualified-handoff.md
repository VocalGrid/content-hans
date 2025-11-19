# Workflow 03 Qualified Handoff

- **Trigger**: Contact tag added `recruitment.reactivation.qualified`
- **Preconditions**: None

## Steps in GHL builder
- **Trigger** Contact Tag
  - Match: Tag added equals `recruitment.reactivation.qualified`

- **Create/Update Opportunity**
  - Pipeline: Recruitment – Lead Reactivation
  - Stage: Qualified → Handoff
  - Title: keep existing

- **Assign to User**
  - User: Recruiter owner or round robin

- **Send Internal Notification** (optional)
  - To: Assigned user / team
  - Channel: Email/SMS/Slack (if integrated)
  - Message: Candidate qualified from Reactivation

- **Add Note**
  - Text: `Qualified; Handoff created on {{now}}`

- **Remove Contact Tag** `recruitment.reactivation.enrolled`

## Fields and tags touched
- **Fields**: none required
- **Tags**: recruitment.reactivation.qualified (trigger only), recruitment.reactivation.enrolled

## Outcome
- Qualified candidates are surfaced to recruiters with a clear handoff.
 - Enrollment tag is cleared to prevent future batch re-enrollment.
