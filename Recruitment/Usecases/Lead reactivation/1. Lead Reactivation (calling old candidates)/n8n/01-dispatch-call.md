# n8n 01 Dispatch Call

- Purpose: Receive a webhook from GHL when a contact reaches a dialable stage and instruct VocalGrid to place a call, with guardrails.
- Trigger: HTTP Webhook (from GHL workflow Call Orchestration)

## Expected inbound payload from GHL
```json
{
  "contact_id": "{{contact.id}}",
  "phone": "{{contact.phone}}",
  "campaign": "{{contact.reactivation_campaign}}",
  "attempt": "{{contact.calls_attempt_count}}",
  "stage": "{{opportunity.stage_name}}",
  "leased": true,
  "signature": "optional-hmac"
}
```

## High-level steps (nodes)
- Webhook In
- Verify Signature (optional)
- Fetch Contact from GHL API
- Guardrails If/Else
  - DND must be OFF
  - Tag `recruitment.reactivation.leased` not already present
  - Time window 09:00–19:00 in contact timezone
- If outside window
  - Compute next 09:00 and set `next_attempt_at`
  - Respond 202 queued
- If inside window
  - Increment `calls_attempt_count` by 1
  - Set `reactivation_status = in_progress`
  - Add tag `recruitment.reactivation.leased` (idempotent)
  - Compose `session_id = {contact_id}-{calls_attempt_count}-{YYYYMMDDHHmm}`
  - POST to VocalGrid Start Call API
  - Add GHL Note: "Dial attempt {{calls_attempt_count}} dispatched session {{session_id}}"
  - Respond 200 with session_id

## GHL updates performed here
- Update Contact Field: `calls_attempt_count` ++
- Update Contact Field: `reactivation_status = in_progress`
- Update Contact Field: `next_attempt_at` (only when queued)
- Add Contact Tag: `recruitment.reactivation.leased`
- Add Note: dispatch info

## VocalGrid Start Call API (example)
- Method: POST
- URL: {VG_BASE_URL}/api/calls/start
- Headers: Authorization: Bearer {VG_API_KEY}
- Body:
```json
{
  "session_id": "{session_id}",
  "contact_id": "{contact_id}",
  "phone": "+15551234567",
  "campaign": "Jannis750k",
  "attempt": 1,
  "metadata": {
    "source": "ghl",
    "stage": "To Dial Attempt 1"
  },
  "webhook_url": "{N8N_PUBLIC_BASE}/webhook/vg/intake"
}
```

## Error handling
- If phone invalid from VG/Twilio
  - Set `last_disposition = bad_number`
  - Move stage to Bad Number
  - Remove tag `recruitment.reactivation.leased`
  - Add Note with error details

## Security
- Require a shared secret or HMAC on inbound GHL → n8n webhooks.
- Use HTTPS only. Mask tokens in n8n credentials.
