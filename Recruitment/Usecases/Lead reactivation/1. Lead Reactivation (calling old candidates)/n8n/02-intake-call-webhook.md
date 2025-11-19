# n8n 02 Intake Call Webhook

- Purpose: Receive end of call events from VocalGrid, analyze the transcript with an LLM, and update GHL fields, tags, and pipeline stage.
- Trigger: HTTP Webhook (VocalGrid events)

## Expected inbound payload from VocalGrid
```json
{
  "session_id": "abc-123",
  "contact_id": "ghl-456",
  "phone": "+15551234567",
  "attempt": 1,
  "started_at": "2025-11-05T18:15:00Z",
  "ended_at": "2025-11-05T18:20:21Z",
  "duration_sec": 321,
  "disposition": "answered|no_answer|voicemail|bad_number|dnc|qualified|completed",
  "recording_url": "https://vg.example.com/rec/abc-123.mp3",
  "transcript_url": "https://vg.example.com/trans/abc-123.json",
  "transcript_text": "optional inline transcript text...",
  "metadata": { "campaign": "Jannis750k" }
}
```

## High-level steps (nodes)
- Webhook In
- Validate session and contact
- Fetch Contact from GHL
- Download transcript text if only a URL is provided
- LLM Analysis (structured JSON)
- Map results to GHL updates
- Stage and Tag updates
- Cleanup: remove lease tag when terminal
- Respond 200 with applied changes

## LLM analysis: request
- Model prompt: summarize and classify the call for recruitment reactivation.
- Return the JSON object defined in LLM Extraction Schema (see `llm-extraction-schema.md`).

## GHL updates performed here
- Update Contact Field: `last_disposition` = result.last_disposition
- Update Contact Field: `reactivation_status` = result.reactivation_status
- Update Contact Field: `ai_call_summaries` = result.ai_call_summaries
- Update Contact Field: `ai_candidate_experience` = result.ai_candidate_experience
- Update Contact Field: `ai_salary_expectation` = result.ai_salary_expectation
- Update Contact Field: `ai_availability` = result.ai_availability
- Update Contact Field: `ai_recording_urls` append recording_url
- Optional: Update Contact Field: `next_attempt_at` when scheduling a retry

## Stage moves and tags
- If result.route == "qualified":
  - Create/Update Opportunity → Stage "Qualified → Handoff"
  - Add Tag `recruitment.reactivation.qualified`
  - Remove Tag `recruitment.reactivation.leased`
- Else if result.route == "no_answer_attempt1":
  - Create/Update Opportunity → Stage "Attempt 1 Completed"
  - Add Tag `recruitment.reactivation.attempt1.done`
  - Remove Tag `recruitment.reactivation.leased`
- Else if result.route == "no_answer_attempt2":
  - Create/Update Opportunity → Stage "Attempt 2 Completed"
  - Add Tag `recruitment.reactivation.attempt2.done`
  - Remove Tag `recruitment.reactivation.leased`
- Else if result.route == "bad_number":
  - Create/Update Opportunity → Stage "Bad Number"
  - Add Tag `recruitment.reactivation.bad_number`
  - Remove Tag `recruitment.reactivation.leased`
- Else if result.route == "dnc":
  - Enable DND
  - Create/Update Opportunity → Stage "Not Looking / DNC"
  - Add Tag `recruitment.reactivation.dnc`
  - Remove Tag `recruitment.reactivation.leased`
- Else if result.route == "completed_not_qualified":
  - Keep current attempt stage or move to Attempt 1 Completed as appropriate
  - Remove Tag `recruitment.reactivation.leased`

## next_attempt_at policy
- If route requires a retry and current attempt == 1:
  - next_attempt_at = next weekday 09:00 in contact timezone
- If route requires no further dial:
  - Clear next_attempt_at

## Notes
- Add GHL Note with top bullets from `ai_call_summaries` and the final classification.
- Attach links to `recording_url` and `transcript_url` when available.

## Error handling
- If the contact has DND enabled, do not update, add a Note and return 200.
- If payload missing `session_id`, write a Note and return 400.
- If GHL update fails, retry with exponential backoff up to N times.
