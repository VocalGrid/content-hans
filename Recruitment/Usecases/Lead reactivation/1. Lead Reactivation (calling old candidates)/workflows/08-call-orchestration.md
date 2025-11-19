# Workflow 08 Call Orchestration

- **Purpose**: Fire outbound call webhooks when an Opportunity enters a dialable stage, with time-window guardrails.
- **Model**: GHL triggers stage change, leasing is via tag, n8n launches the call and posts outcomes back.

## Trigger
- **Trigger** Pipeline Stage Changed
  - Pipeline: Recruitment – Lead Reactivation
  - When: Stage changed
  - Stages: To Dial – Attempt 1, To Redial – Attempt 2
  - Re-enrollment: On

## Workflow settings
- **Execution Window**: 09:00–19:00 local
  - Outside window: Queue actions until window opens
  - Timezone: Use Contact timezone if available, else Location timezone
- Optional alternative: add a Wait step to normalize to 09:00, then proceed

## Steps in GHL builder
- **If Else**
  - Conditions
    - DND is disabled
    - `calls_attempt_count` < 2
    - Tags does not include `recruitment.reactivation.leased`

- Branch: Yes (dialable)
  - **Add Contact Tag**
    - Tag: `recruitment.reactivation.leased`
  - **Webhook/Custom Webhook**
    - Method: POST
    - URL: https://your-n8n-endpoint.example.com/webhooks/reactivation/call
    - Headers: (optional) `X-Signature: {{contact.id}}|{{contact.created_at}}` or HMAC
    - Body (JSON):
      ```json
      {
        "contact_id": "{{contact.id}}",
        "phone": "{{contact.phone}}",
        "campaign": "{{contact.reactivation_campaign}}",
        "attempt": "{{contact.calls_attempt_count}}",
        "stage": "{{opportunity.stage_name}}",
        "leased": true
      }
      ```
  - **Add Note**
    - Text: `Call dispatch via webhook at {{now}}`

- Branch: No (not dialable)
  - **Add Note** `Skipped call dispatch due to DND or attempts or lease`

## Fields and tags touched
- **Fields**: calls_attempt_count (read only in this flow)
- **Tags**: recruitment.reactivation.leased

## Outcome
- Webhook fires only inside the execution window, leasing prevents double dials. n8n launches the call and later sets `reactivation_status`, increments `calls_attempt_count`, and removes `recruitment.reactivation.leased`.

## Security and reliability
- **HMAC verify**: sign requests from GHL or verify inbound in n8n
- **Idempotency**: n8n should ignore duplicate leased contacts
- **Retries**: if webhook fails, add a small Wait then Webhook again or handle retries in n8n
