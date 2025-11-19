# Lead Reactivation A→Z (Simple Walkthrough)

This guide shows, step by step, what happens to a lead from upload to handoff. Plain, simple English. You already built GHL + n8n + VocalGrid, so this explains how they work together.

---

## 1) Before you start (once)

- **Custom fields to have**
  - `reactivation_campaign`
  - `reactivation_status`
  - `calls_attempt_count`
  - `next_attempt_at`
  - `last_disposition`
  - `ai_call_summaries`
  - `ai_candidate_experience`
  - `ai_salary_expectation`
  - `ai_availability`
  - `ai_recording_urls`
- **Tags we use**
  - `recruitment.reactivation.enrolled` (starts the process)
  - `recruitment.reactivation.leased` (we are trying to call right now)
  - `recruitment.reactivation.attempt1.done`
  - `recruitment.reactivation.attempt2.done`
  - `recruitment.reactivation.qualified`
  - `recruitment.reactivation.dnc`
  - `recruitment.reactivation.bad_number`
- **Pipeline stages**
  - Enrolled → To Dial Attempt 1 → Attempt 1 Completed → To Redial Attempt 2 → Attempt 2 Completed
  - Side exits: Qualified → Handoff, Not Looking / DNC, Bad Number

---

## 2) Upload your CSV to GHL

- **CSV columns to include** (minimum)
  - First Name, Last Name, Phone, Email (optional but helpful)
- **After import**
  - Select all imported contacts → Bulk Actions → Add Tag → add `recruitment.reactivation.enrolled`.
  - This tag is the ON switch. It kicks off the enrollment workflow.

What happens next (automatic):
- A card is created in your pipeline at stage “Enrolled”.
- `calls_attempt_count` set to 0. `reactivation_status` set to something like “new”.

---

## 3) Getting ready to call (Attempt 1)

- The system moves the card from “Enrolled” to “To Dial – Attempt 1”.
- When it is time to try, we add `recruitment.reactivation.leased` (this prevents double-dials).
- The Call Orchestration workflow sees the card in a dialable stage and checks:
  - Time window 09:00–19:00
  - DND is off
  - `calls_attempt_count < 2`
  - Not already leased
- If OK, GHL sends a webhook to n8n → VocalGrid to place the call.

---

## 4) The call happens (Attempt 1 outcomes)

VocalGrid makes the call and then posts results back to n8n. n8n analyzes the transcript with an LLM and updates GHL.

Common outcomes:
- **Qualified**
  - Stage → “Qualified → Handoff”
  - Tag → `recruitment.reactivation.qualified`
  - Remove `recruitment.reactivation.leased`
- **No answer / voicemail (Attempt 1)**
  - Stage → “Attempt 1 Completed”
  - Tag → `recruitment.reactivation.attempt1.done`
  - Remove `recruitment.reactivation.leased`
  - Set `next_attempt_at = now + 5 hours`
  - Wait 5 hours, then move to “To Redial – Attempt 2”
- **Bad number**
  - Stage → “Bad Number”
  - Tag → `recruitment.reactivation.bad_number`
  - Remove `recruitment.reactivation.leased`
- **DNC / Not looking**
  - Stage → “Not Looking / DNC”
  - Tag → `recruitment.reactivation.dnc` and enable DND
  - Remove `recruitment.reactivation.leased`

Data saved each time:
- `calls_attempt_count` increases on each dial.
- `last_disposition` set (answered, no_answer, voicemail, bad_number, dnc, etc.).
- `ai_*` fields updated with summary, experience, salary, availability, and recording links.

---

## 5) Redial (Attempt 2)

- After the 5-hour wait, the card is in “To Redial – Attempt 2”.
- Call Orchestration checks the same safety rules (time window, DND, attempts).
- VG calls again. When the result comes back:
  - **Qualified** → “Qualified → Handoff”, tag `recruitment.reactivation.qualified`, remove leased.
  - **No answer** → “Attempt 2 Completed”, tag `recruitment.reactivation.attempt2.done`, remove leased.
  - **Bad number / DNC** → go to those stages, set tags, remove leased.
- Optionally, an SMS Fallback workflow can send a friendly text after Attempt 2 Completed.

---

## 6) Where to look in GHL

- **Workflows**
  - 01 Enrollment
  - 02 Lease Start (sets leased state)
  - 03 Qualified Handoff
  - 04 Outcome: No Answer / Voicemail (has the 5-hour wait logic)
  - 05 SMS Fallback
  - 06 DNC Handling
  - 07 Bad Number
  - 08 Call Orchestration (time window + webhook)
- **Pipeline board**
  - Watch cards move across stages in real time.
- **Contact record**
  - Notes show what happened.
  - Custom fields show latest AI summaries and call facts.
  - Tags show the current state.

---

## 7) Simple dummy stories (A→Z)

- **Alex (Qualified on first call)**
  - Import + tag enrolled → To Dial Attempt 1 → Call happens → Alex answers, is a fit
  - Stage goes to “Qualified → Handoff”, tag `recruitment.reactivation.qualified`

- **Bina (No answer first, DNC on second)**
  - Import + tag enrolled → To Dial Attempt 1 → No answer → “Attempt 1 Completed” + tag attempt1.done
  - Wait 5 hours → To Redial Attempt 2 → Bina says “please stop” → “Not Looking / DNC” + tag dnc

- **Carlos (Bad number)**
  - Import + tag enrolled → To Dial Attempt 1 → Carrier error
  - Stage “Bad Number”, tag `recruitment.reactivation.bad_number`

- **Dina (No answer both times → SMS)**
  - Import + tag enrolled → To Dial Attempt 1 → No answer → “Attempt 1 Completed” + tag attempt1.done
  - Wait 5 hours → To Redial Attempt 2 → No answer → “Attempt 2 Completed” + tag attempt2.done
  - SMS Fallback sends a friendly text

---

## 8) Safety rails (why this is safe)

- **Time window**: Calls only dispatch 09:00–19:00.
- **No double-dial**: The leased tag prevents two calls at once.
- **Respect DND**: Opt-outs move to DNC and stop calls.
- **Two tries**: We stop after 2 attempts, then use SMS.

---

## 9) Quick checklist (do this every time you start a batch)

- **Upload CSV** to GHL.
- **Bulk add tag** `recruitment.reactivation.enrolled` to those contacts.
- **Confirm workflows are ON** (especially Call Orchestration and Outcome handling).
- **Watch the board** and Notes to see progress.

That’s it. Add the enrolled tag, and the system does the rest: two safe call tries, smart updates, and clear routing to handoff, DNC, or bad number.
