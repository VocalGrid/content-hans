
# 🚨 CRITICAL: Scaling to 10-15 attempts

**Your current system is hardcoded for 2 attempts** in:
1. GHL workflow 02-lease-start: `IF calls_attempt_count == 0` → A1, `ELSE` → A2
2. GHL workflow 04-outcome: `IF calls_attempt_count == 1` → A1 Completed, `ELSE` → A2 Completed
3. n8n End Call Analysis Routing Plan: hardcoded stage IDs for `A1_COMPLETED`, `A2_COMPLETED_NO_ANSWER`
4. Pipeline: only 2 attempt stages

## 🎯 Solution: Configuration-driven, generic stages

### Changes needed:

#### 1. **Pipeline redesign** (collapse attempts into generic stages)

Replace:
- To Dial – Attempt 1
- Attempt 1 Completed
- To Redial – Attempt 2  
- Attempt 2 Completed (No Answer)

With:
- **Enrolled**
- **To Dial** (generic, reused for all attempts)
- **Attempt Completed** (generic, stores current attempt in field)
- **Qualified → Handoff** (terminal)
- **Not Looking / DNC** (terminal)
- **Bad Number** (terminal)

#### 2. **Add campaign config field**
- `max_attempts` (number, default 2, per campaign or global)
- `hours_between_attempts` (number, default 5)

#### 3. **GHL workflow updates**

**02-lease-start**: Remove attempt-specific branching
```
- Add tag leased
- Set reactivation_status = in_progress
- Move stage → To Dial (generic)
- Add Note "Dial attempt {{calls_attempt_count + 1}} starting"
```

**04-outcome-no-answer**: Make generic
```
- IF reactivation_status in [no_answer, voicemail]
  - Move stage → Attempt Completed
  - Add tag recruitment.reactivation.attemptX.done (use formula {{calls_attempt_count}})
  - Remove leased
  - IF calls_attempt_count < max_attempts
    - Set next_attempt_at = now + hours_between_attempts
    - Wait {{hours_between_attempts}} hours
    - Move stage → To Dial (ready for next attempt)
  - ELSE (max reached)
    - Trigger SMS fallback
    - Remove enrolled
```

#### 4. **n8n Routing Plan update**

Replace hardcoded A1/A2 logic with:
```js
const attemptCurrent = parseInt(wb.custom_data?.attempt_number ?? '1', 10);
const maxAttempts = parseInt(wb.custom_data?.max_attempts ?? '2', 10);

// Config
const STAGES = {
  ENROLLED: '...',
  TO_DIAL: '...',        // single generic stage
  ATTEMPT_COMPLETED: '...',  // single generic stage
  QUALIFIED_HANDOFF: '...',
  DNC: '...',
  BAD_NUMBER: '...',
};

// Routing
if (qualified) {
  stageId = STAGES.QUALIFIED_HANDOFF;
  addTags = [TAGS.QUALIFIED];
  removeTags = [TAGS.LEASED, TAGS.ENROLLED];
} else if (dnc) {
  stageId = STAGES.DNC;
  addTags = [TAGS.DNC];
  removeTags = [TAGS.LEASED, TAGS.ENROLLED];
} else if (badNumber) {
  stageId = STAGES.BAD_NUMBER;
  addTags = [TAGS.BAD_NUMBER];
  removeTags = [TAGS.LEASED, TAGS.ENROLLED];
} else if (noAnswer || voicemail) {
  stageId = STAGES.ATTEMPT_COMPLETED;
  addTags = [`recruitment.reactivation.attempt${attemptCurrent}.done`];
  removeTags = [TAGS.LEASED];
  
  if (attemptCurrent < maxAttempts) {
    nextAttemptAt = isoPlusHours(HOURS_BETWEEN_ATTEMPTS);
    // GHL workflow will handle wait + move back to TO_DIAL
  } else {
    // max reached, keep in ATTEMPT_COMPLETED, trigger SMS
    removeTags.push(TAGS.ENROLLED);
  }
}
```

---

## Summary doc I'll create

I'll add a new doc: **Scaling to N Attempts.md** with exact steps to make the system support 2-15 attempts via config, not code.

**Want me to create this scaling guide now?**