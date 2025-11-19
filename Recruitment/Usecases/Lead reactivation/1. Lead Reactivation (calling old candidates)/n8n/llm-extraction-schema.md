# LLM Extraction Schema — Lead Reactivation

This document defines exactly what we extract from call transcripts and how we update GHL.

## Goals
- Decide routing for the record: qualified, no answer attempt 1 or 2, bad number, DNC, completed not qualified.
- Capture short human-readable summaries and key facts.
- Update the standardized GHL custom fields and tags.

## Required output JSON (from LLM)
The LLM must return a single JSON object with the following shape. Keep values concise and factual; avoid hallucinations.

```json
{
  "last_disposition": "answered|no_answer|voicemail|bad_number|dnc|completed",
  "route": "qualified|no_answer_attempt1|no_answer_attempt2|bad_number|dnc|completed_not_qualified",
  "ai_call_summaries": "short bullet style summary up to 800 chars",
  "ai_candidate_experience": "concise summary of role type, years, tools, relevant notes up to 1200 chars",
  "ai_salary_expectation": "plain text, e.g., 70k to 80k, negotiable",
  "ai_availability": "plain text, e.g., immediate, two weeks, specific date",
  "tags_to_add": ["recruitment.reactivation.attempt1.done"],
  "schedule_retry": false,
  "next_attempt_at": null,
  "qualified_reason": "optional short reason if route is qualified",
  "recording_url": null,
  "transcript_url": null
}
```

Notes:
- `route` drives stage moves and tag updates (see mapping below).
- If `schedule_retry` is true and `next_attempt_at` is null, n8n computes next 09 00 in contact timezone.
- If `recording_url` or `transcript_url` are present, they will be appended to storage and surfaced in notes.

## Mapping to GHL

- Custom fields to update:
  - `last_disposition` ← `last_disposition`
  - `reactivation_status` ← if route is qualified then `qualified` else from `last_disposition` or `completed`
  - `ai_call_summaries` ← `ai_call_summaries`
  - `ai_candidate_experience` ← `ai_candidate_experience`
  - `ai_salary_expectation` ← `ai_salary_expectation`
  - `ai_availability` ← `ai_availability`
  - `ai_recording_urls` ← append `recording_url` when present
  - `next_attempt_at` ← `next_attempt_at` (or cleared if no further attempts)

- Tags to add (examples):
  - Attempt outcomes:
    - Attempt 1 done → `recruitment.reactivation.attempt1.done`
    - Attempt 2 done → `recruitment.reactivation.attempt2.done`
  - Qualified → `recruitment.reactivation.qualified`
  - DNC → `recruitment.reactivation.dnc`
  - Bad number → `recruitment.reactivation.bad_number`

- Stage moves per `route`:
  - `qualified` → Stage: "Qualified → Handoff"
  - `no_answer_attempt1` → Stage: "Attempt 1 Completed"
  - `no_answer_attempt2` → Stage: "Attempt 2 Completed"
  - `bad_number` → Stage: "Bad Number"
  - `dnc` → Stage: "Not Looking / DNC" and enable DND
  - `completed_not_qualified` → keep at appropriate attempt completed stage

- Lease cleanup:
  - On any terminal route, remove tag `recruitment.reactivation.leased`.

## LLM Prompt outline
Provide the transcript text and minimal context:

```
You are a careful analyst. Extract structured facts from a recruitment reactivation call transcript.
Return ONLY valid JSON matching the schema below. Do not include commentary.

Determine: last_disposition, route, key bullet summary, experience highlights, salary expectation, availability, needed tags, scheduling instruction.
- last_disposition: answered, no_answer, voicemail, bad_number, dnc, completed
- route: qualified, no_answer_attempt1, no_answer_attempt2, bad_number, dnc, completed_not_qualified
- If the human said stop or opt out, route=dnc.
- If number invalid, route=bad_number.
- If clearly interested and suitable, route=qualified and include qualified_reason.
- Limit each text field to its char budget.

Schema:
{ last_disposition, route, ai_call_summaries, ai_candidate_experience, ai_salary_expectation, ai_availability, tags_to_add, schedule_retry, next_attempt_at, qualified_reason, recording_url, transcript_url }
```

## Example LLM output
```json
{
  "last_disposition": "answered",
  "route": "qualified",
  "ai_call_summaries": "Actively looking. 5y staff accounting. Open to hybrid Denver. Target 70-80k. Can start in 2 weeks.",
  "ai_candidate_experience": "5 years staff accounting; QuickBooks Online; CPA credits; prefers hybrid within Denver.",
  "ai_salary_expectation": "70k to 80k",
  "ai_availability": "two weeks",
  "tags_to_add": ["recruitment.reactivation.qualified"],
  "schedule_retry": false,
  "next_attempt_at": null,
  "qualified_reason": "Experience and salary fit current requisitions.",
  "recording_url": "https://vg/rec/123.mp3",
  "transcript_url": "https://vg/trans/123.json"
}
```
