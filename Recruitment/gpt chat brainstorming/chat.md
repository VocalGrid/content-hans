Skip to content
You said:
we need to build this

Recruitment Agency - 4 Use Cases I want to show:
1. Lead Reactivation (calling old candidates)
🎯 Goal:
The primary goal is to reactivate "dead leads" for a recruitment agency. These are candidates who previously applied for a job but were not selected. The objective is to find out if they are still in the job market and to update their profile, so they can be quickly matched with new opportunities as they become available.
Method: The main method of reactivation is by calling them. We can also use voicemail, SMS and email as part of the follow-up strategy.
📞 What the Voice AI does:
Calls candidates from old CRM lists (for example, from the last 6–12 months).
Asks if they’re still looking for work or if their situation has changed.
Updates the database with their current status, new experience, or job preferences.
❓Questions the Voice AI Asks:
Their current employment status (e.g., "Are you currently seeking a new job opportunity?").
Their interest in roles similar to the one they previously applied for.
Any new skills or experience they have gained since their last application.
Their current salary expectations.
Their availability or timeline for starting a new position.
Campaign Structure:
Initial Data: The client provides a list of candidates (dead leads), which is uploaded to a database (Airtable).
Scheduled Outbound Calls:
An automated scheduler initiates outbound calls to the candidates between 9 am and 7 pm.
The system has controls to manage the number of simultaneous calls to avoid overloading the system and to ensure they never call the same person twice.


The Call Interaction:
When a candidate answers, the AI agent greets them by name and references the specific job they applied for and the date of their application to personalize the conversation.
The agent's purpose is to inform the candidate they weren't selected for the original role but to ask five specific questions to see if there are other potential job opportunities for them within the company.


Handling No-Answers and Voicemails:
If the initial call goes to voicemail or is not answered, the system waits for 7 minutes.
After the 7-minute wait, the AI agent calls the candidate a second time.


SMS Follow-Up: If the candidate does not pick up on the second call attempt, an automated SMS is sent to them.
Inbound Call Handling (Call Backs):
The system is equipped to handle inbound calls from candidates who missed the initial calls and decided to call back.
When a candidate calls back, an "inbound agent" (another voice AI) takes the call and proceeds with the same objective of asking the qualifying questions. This inbound capability was crucial, accounting for an additional 4% completion rate for their calls.


Data Analysis and Lead Forwarding:
After each call, the transcript is analyzed by a Large Language Model (LLM) like OpenAI to extract relevant information.
If a candidate is identified as a good fit for a new opportunity, their information is forwarded to a human recruitment agent for a follow-up interview and the next steps in the hiring process.


Reporting: The results and analytics of the campaign are displayed on a custom dashboard for the client to review.

💡 Sample Script:
Company: Nova Recruiters
Candidate: Michael Scott

AI: "Hi Michael, this is Ava calling from Nova Recruiters. I'm calling about the Senior Accountant position you applied for with us back on March 12th. I just wanted to follow up and see how things are going! Is now an okay time for a brief chat?"
Candidate: "Hey Ava, sure, I’ve got a minute."
AI: "Great, thanks! I wanted to see if you’re still on the job hunt or if you’ve found something since we last spoke."
Candidate: "Yeah, I’ve been doing some temporary project work for a small construction firm, just to stay busy, but I’m still looking for something full-time."
AI: "Got it, thanks for letting me know. And are you still living in Lakewood, or did you move since you first applied?"
Candidate: "Still in Lakewood."
AI: "Perfect — I’ll keep that the same so we can match you with openings nearby. Are you still mainly interested in staff accounting roles, or have your goals changed a bit?"
Candidate: "Still focused on staff accountant roles, maybe senior accountant positions if the fit’s right."
AI: "Excellent. Have you picked up any new experience or certifications since the last time we talked?"
Candidate: "Yeah, I completed a few extra CPA credits and also got certified in QuickBooks Online."
AI: "That’s fantastic, congratulations! Those definitely open more doors. To make sure we're only sending you opportunities that are a great financial fit, could you share what salary range you're targeting for your next full-time role?"
Candidate: "I'm looking for something in the $70,000 to $80,000 range, depending on the benefits."
AI: "Perfect, thank you for that. Just a couple more quick questions. Are you open to hybrid or remote roles, or do you prefer on-site?"
Candidate: "Hybrid would be great, but I’m flexible."
AI: "Got it, hybrid preferred — noted. And finally, to help our recruiters with timing, how soon would you be looking to start a new position if the right one came along?"
Candidate: "I'd need to give my current project about two weeks' notice, so I could start fairly quickly."
AI: "Great, that's very helpful. And what’s the best way for us to reach you when a good match comes up, by phone or text?"
Candidate: "Text is usually best."
AI: "Perfect. Thanks for the update, Michael! I’ve refreshed your info and marked you as actively looking for accounting roles around Denver. One of our recruiters will reach out as soon as a position opens that fits your profile. It was great catching up with you — I hope we can help you find something long-term soon!"
Candidate: "Thanks, Ava. I appreciate it."
AI: "You’re very welcome, Michael. Have a great rest of your day!"
End call.
—
AI (voicemail):
 “Hi Michael, this is Ava from Nova Recruiters. You applied with us a while back for an accounting role in the Denver area. I just wanted to check if you’re still open to new opportunities.
Feel free to text me back or give me a quick call when you have a minute. Hope you’re doing well!”
—
AI (SMS):
Hey Michael, this is Ava from Nova Recruiters. Just tried giving you a quick call to check if you’re still open to accounting roles around Denver.
You can reply yes if you are, and I’ll give you a quick call back.
—







our techstack 

n8n

vocalgrid ai for voice ai, which is direct replacemenrt of vapi/retell and powered by livekit

we can use supabase or airtable as DB

we can also use ghl workflows, pipelines etc or also upload leads there

need to build demo for youjytube on this usecase
i need to think what table i need to have, what triggers etc, we can upload csv of candidates and dialer starts calling, etc brainstorm with me what should i have where, dont build any voice ai agent prompts for now

ChatGPT said:

Thought for 29s
Awesome use case. Let’s design it like a real product you can demo end-to-end with VocalGrid (LiveKit) + n8n + Supabase (Airtable optional/mirrored) and optional GHL sync.

Below is a tight, build-ready plan: schema, queues, n8n flows, retry logic, inbound handling, LLM extraction, analytics, and a YouTube demo sequence. (Per your note: no agent prompts yet.)

Architecture at a glance
Supabase = Source of truth. Candidates, campaigns, call tasks, attempts, transcripts, extractions, events.

n8n = Orchestrator. Ingestion, schedulers, task leaser, call launcher, webhook handlers, SMS/Email, routing to recruiters.

VocalGrid = Voice. Outbound dialer + inbound agent via LiveKit; posts webhooks to n8n.

(Optional) Airtable/GHL. Client-facing view (Airtable) and/or CRM sync (GHL tags/pipelines).

Data model (Supabase SQL)
Copy/paste in Supabase SQL editor. Names chosen to feel like your existing patterns.

-- 1) Campaigns
create table public.campaigns (
  id uuid primary key default gen_random_uuid(),
  name text not null,
  kind text not null default 'lead_reactivation', -- future-proof: 'screening', 'event', etc.
  timezone text not null default 'America/Toronto',
  call_window jsonb not null default jsonb_build_object('start','09:00','end','19:00'),
  max_concurrent_calls int not null default 5,
  max_attempts int not null default 2,
  retry_minutes int not null default 7,
  dedupe_scope text not null default 'campaign', -- 'campaign' | 'global'
  created_at timestamptz default now()
);

-- 2) Candidates (normalized enough to be useful)
create table public.candidates (
  id uuid primary key default gen_random_uuid(),
  external_ref text,                 -- row id from CSV or ATS id
  full_name text not null,
  phone_e164 text not null,          -- +1XXXXXXXXXX
  email text,
  city text, state text, country text,
  timezone text,                     -- if known (fallback to campaign.tz)
  last_applied_role text,
  last_applied_date date,
  status text not null default 'unknown', -- unknown | active | not_looking | do_not_call | bad_number | etc.
  tags text[] default '{}',
  meta jsonb not null default '{}'::jsonb,  -- arbitrary fields
  created_at timestamptz default now(),
  unique(phone_e164, coalesce(external_ref,''))  -- helps dedupe on repeated imports
);

-- 3) Campaign <-> Candidates mapping (ownership + state)
create table public.campaign_candidates (
  id uuid primary key default gen_random_uuid(),
  campaign_id uuid not null references public.campaigns(id) on delete cascade,
  candidate_id uuid not null references public.candidates(id) on delete cascade,
  enrolled_at timestamptz default now(),
  active boolean not null default true,
  unique (campaign_id, candidate_id)
);

-- 4) Call tasks (work queue)
create table public.call_tasks (
  id bigint generated always as identity primary key,
  campaign_id uuid not null references public.campaigns(id) on delete cascade,
  candidate_id uuid not null references public.candidates(id) on delete cascade,
  attempt_no int not null,          -- 1..max_attempts
  status text not null default 'queued',   -- queued | leased | in_progress | no_answer | voicemail | completed | failed | canceled
  scheduled_at timestamptz not null,
  leased_by text,                   -- worker id
  leased_at timestamptz,
  lock_expires_at timestamptz,
  result jsonb default '{}'::jsonb, -- final outcome metadata
  created_at timestamptz default now(),
  unique (campaign_id, candidate_id, attempt_no)
);

-- 5) Call attempts (actual telephony events per task)
create table public.call_attempts (
  id bigint generated always as identity primary key,
  task_id bigint not null references public.call_tasks(id) on delete cascade,
  vocalgrid_call_id text,           -- from LiveKit/VocalGrid
  started_at timestamptz,
  answered_at timestamptz,
  ended_at timestamptz,
  disposition text,                 -- answered | no_answer | voicemail | busy | failed | canceled
  duration_seconds int,
  recording_url text,
  transcript_url text,
  transcript text,                  -- short transcripts OK; longer goes to object store if needed
  raw_webhook jsonb,
  created_at timestamptz default now()
);

-- 6) Extraction results (LLM)
create table public.extractions (
  id bigint generated always as identity primary key,
  attempt_id bigint not null references public.call_attempts(id) on delete cascade,
  candidate_id uuid not null references public.candidates(id) on delete cascade,
  schema_version int not null default 1,
  data jsonb not null,              -- normalized answers (see schema below)
  quality text,                     -- good | partial | low_conf
  created_at timestamptz default now()
);

-- 7) Recruiter handoffs
create table public.recruiter_leads (
  id bigint generated always as identity primary key,
  candidate_id uuid not null references public.candidates(id) on delete cascade,
  source_campaign uuid references public.campaigns(id),
  reason text,                      -- "qualified_for_role_match"
  payload jsonb not null,           -- snapshot of extracted data
  status text not null default 'new', -- new | assigned | contacted | closed
  created_at timestamptz default now()
);

-- 8) Messages (SMS/Email log)
create table public.messages (
  id bigint generated always as identity primary key,
  candidate_id uuid not null references public.candidates(id) on delete cascade,
  channel text not null,            -- sms | email | voicemail
  direction text not null,          -- out | in
  template_key text,
  body text,
  provider_message_id text,
  meta jsonb default '{}'::jsonb,
  created_at timestamptz default now()
);

-- 9) Inbound calls
create table public.inbound_calls (
  id bigint generated always as identity primary key,
  candidate_id uuid references public.candidates(id),
  from_e164 text not null,
  vocalgrid_call_id text,
  answered_by text default 'inbound_agent',
  disposition text,
  transcript text,
  transcript_url text,
  created_at timestamptz default now()
);

-- 10) Global guardrails
create table public.suppression_list (
  phone_e164 text primary key,
  reason text,
  created_at timestamptz default now()
);
Helpful views

-- Latest candidate status snapshot
create or replace view public.candidate_last_status_v as
select c.*, 
  (select max(a.ended_at) from call_attempts a 
   join call_tasks t on t.id=a.task_id 
   where t.candidate_id=c.id) as last_called_at
from candidates c;

-- Campaign dashboard aggregates
create or replace view public.campaign_dashboard_v as
select
  c.id as campaign_id,
  c.name,
  count(cc.id) as enrolled,
  count(*) filter (where t.status='completed') as completed,
  count(*) filter (where t.status in ('no_answer','voicemail')) as attempted_not_completed,
  count(distinct t.candidate_id) filter (where t.status='completed') as unique_completed_candidates,
  count(e.id) as extractions,
  count(rl.id) as handoffs
from campaigns c
left join campaign_candidates cc on cc.campaign_id=c.id
left join call_tasks t on t.campaign_id=c.id
left join call_attempts a on a.task_id=t.id
left join extractions e on e.attempt_id=a.id
left join recruiter_leads rl on rl.source_campaign=c.id
group by 1,2;
CSV ingestion & dedupe
Staging: upload CSV to n8n → map fields → upsert into candidates using (phone_e164, external_ref) as dedupe keys.

After upsert, insert campaign_candidates rows (ignore conflicts).

Seed tasks: for each candidate, create attempt_no = 1 in call_tasks with scheduled_at = next valid slot (below).

CSV mapping (example)

full_name,phone_e164,email,city,state,country,timezone,last_applied_role,last_applied_date,external_ref
"Michael Scott","+17205551234","mike@example.com","Lakewood","CO","US","America/Denver","Senior Accountant","2024-03-12","ATS-98765"
Call windowing & scheduling
Respect campaign.call_window (e.g., 09:00–19:00) in candidate’s timezone if present, else campaign tz.

If ingest happens outside window, schedule at next window start.

Never call same person twice: unique (campaign_id, candidate_id, attempt_no) + check suppression/status.

Leased task queue (safe concurrency)
Use SELECT … FOR UPDATE SKIP LOCKED so multiple n8n workers can pull without collisions.

-- RPC function to lease N tasks for a worker
create or replace function public.lease_call_tasks(p_campaign uuid, p_worker text, p_limit int)
returns setof call_tasks
language plpgsql as $$
begin
  return query
  with c as (
    select id from call_tasks
    where campaign_id = p_campaign
      and status = 'queued'
      and scheduled_at <= now()
      and not exists (
        select 1 from suppression_list s 
        join candidates c on c.id = call_tasks.candidate_id
        where s.phone_e164 = c.phone_e164
      )
    order by scheduled_at
    for update skip locked
    limit p_limit
  )
  update call_tasks t
    set status='leased', leased_by=p_worker, leased_at=now(),
        lock_expires_at = now() + interval '10 minutes'
  where t.id in (select id from c)
  returning *;
end $$;
n8n calls lease_call_tasks(campaign_id, "worker-1", campaign.max_concurrent_calls) every minute.

If a worker crashes, lock_expires_at lets others reclaim tasks.

n8n flows (minimal but complete)
Flow A — CSV Importer

Webhook/Manual → receive CSV (file)

Function → parse CSV → normalize phone to E.164

Supabase Upsert → candidates

Supabase Insert (ignore conflicts) → campaign_candidates

Function → compute scheduled_at per record (window)

Supabase Insert → call_tasks (attempt_no=1)

Flow B — Task Leaser & Dialer (CRON every 1 min)

Supabase Select → get campaign config

Supabase RPC → lease_call_tasks(...) with limit = max_concurrent_calls

Split In Batches → For each task:

HTTP Request → VocalGrid: start outbound call (payload includes candidate info + webhook URL)

Supabase Update: call_tasks.status = 'in_progress'

(store vocalgrid_call_id in a fresh call_attempts row immediately)

Flow C — VocalGrid Webhook (outbound)

Triggered by call lifecycle events: ringing, answered, voicemail, completed, recording/transcript ready.

Switch on event.type

On answered/completed:

Supabase Update call_attempts with disposition/duration/recording/transcript

If answered and required questions completed → Update call_tasks.status = 'completed'

Queue LLM extraction (Flow D)

On no_answer/voicemail/busy/failed:

Update call_tasks.status accordingly

If attempt_no = 1 and disposition in (no_answer,voicemail,busy):

Create attempt_no = 2 task with scheduled_at = now() + retry_minutes

If attempt_no = 2 and not completed:

Send SMS (Flow E) using template

Flow D — Extraction & Profile Update

Queue (from Flow C) with attempt_id

Function (LLM): parse transcript into JSON:

{
  "still_looking": true,
  "current_employment": "Temp project for small construction firm",
  "new_skills": ["CPA credits", "QuickBooks Online"],
  "desired_roles": ["Staff Accountant","Senior Accountant"],
  "salary_expectation": {"min":70000,"max":80000,"currency":"USD"},
  "work_mode": "hybrid",
  "start_timeline": "2 weeks",
  "preferred_contact": "sms",
  "location_confirmed": "Lakewood, CO",
  "notes":""
}
Supabase Insert → extractions

Supabase Update → merge into candidates.meta, update status='active'

If rules say “good fit” → Insert into recruiter_leads with payload snapshot.

Flow E — SMS Fallback

If attempt 2 not answered: send the template SMS → Insert messages row.

Flow F — Inbound Handler (VocalGrid inbound webhook)

Match from_e164 to candidates.phone_e164

Create inbound_calls row; pass call to inbound agent

On end: store disposition + transcript

Run Flow D (extraction) → possibly handoff.

Disposition & retry rules (simple, demo-ready)
Attempt 1:

answered → completed → extract → maybe handoff

voicemail OR no_answer OR busy → schedule Attempt 2 at now()+7m

Attempt 2:

If still not answered → send SMS and stop (respect DNC)

DNC / Opt-Out: if candidate says stop → put number in suppression_list and candidates.status='do_not_call'.

Extraction schema (store versioned)
Keep it flexible and versioned for future prompts.

-- future: add schema_version=2 with slightly different fields
-- already covered in extractions table
Optional Airtable & GHL
Airtable (mirror):

Sync views: Candidates (readonly), Call Attempts, Recruiter Leads, Campaign Dashboard.

Either use Airtable’s sync from Supabase (via n8n) or just push daily snapshots.

GHL (if you want pipeline optics):

Create/Upsert Contact on enrollment.

Add Tags: reactivation, campaign:<name>, stage:<status>.

On recruiter_leads.new → Create Opportunity in pipeline “Reactivation”.

Webhooks from GHL → n8n → update Supabase (recruiter_leads.status).

(Sticky 50/50 logic not needed here unless you split DID pools; if you do, use your existing “sticky” approach keyed by candidate_id mod N.)

Analytics you’ll show on the client dashboard
KPIs (by day & campaign):

Dials, Connect Rate, Voicemail Rate, Completion Rate

% with valid extractions

Qualified → Handoff rate

Handoff SLA (time to first recruiter touch)

Opt-outs, Bad numbers

Inbound call lift (extra % completions from callbacks)

Sample aggregate for Looker Studio:

create or replace view public.campaign_daily_metrics_v as
select
  c.id as campaign_id,
  date(t.created_at at time zone c.timezone) as report_date,
  count(distinct t.id) as dials,
  count(*) filter (where a.disposition='answered') as answers,
  count(*) filter (where t.status='completed') as completed_tasks,
  count(e.id) as extractions,
  count(rl.id) as handoffs,
  round( 100.0 * count(*) filter (where a.disposition='answered') / nullif(count(distinct t.id),0), 2) as connect_rate_pct,
  round( 100.0 * count(e.id) / nullif(count(*) filter (where a.disposition='answered'),0), 2) as extraction_success_pct
from campaigns c
left join call_tasks t on t.campaign_id=c.id
left join call_attempts a on a.task_id=t.id
left join extractions e on e.attempt_id=a.id
left join recruiter_leads rl on rl.source_campaign=c.id
group by 1,2;
Guardrails & compliance (you’ll mention in the video)
Time windows per timezone; holiday list optional.

Suppression list + DNC keyword for SMS replies.

Idempotency on imports (external_ref + phone).

Rate limiting: campaign.max_concurrent_calls.

Recording consent language (in prompt later).

PII access controls (row-level policies in Supabase).

What the YouTube demo will show (sequence)
Upload CSV in your mini portal → n8n log shows X upserts, tasks created.

Live dialer: dashboard shows “Queued → In-Progress”; you place 2–3 real test calls.

Voicemail path: show attempt 1 → voicemail → auto-schedule attempt 2 (7 min) → missed again → SMS sent.

Inbound callback: you call back the demo number → inbound agent handles, transcript lands → extraction triggers.

LLM extraction: show the JSON record (extractions.data) and merged candidate profile.

Recruiter handoff: qualified record appears in Recruiter Leads with status new.

Dashboard: live metrics and recent transcripts/recordings.

Quick n8n/VocalGrid payload notes
Start call (HTTP to VocalGrid)

{
  "to": "+17205551234",
  "from": "+18335550123",
  "session_id": "cam-{{campaign_id}}-cand-{{candidate_id}}-att-{{attempt_no}}",
  "webhook_url": "https://n8n.yourdomain.com/webhooks/vocalgrid-outbound",
  "metadata": {
    "candidate_id": "{{candidate_id}}",
    "campaign_id": "{{campaign_id}}",
    "attempt_no": {{attempt_no}},
    "last_applied_role": "{{last_applied_role}}",
    "last_applied_date": "{{last_applied_date}}"
  }
}
Outbound webhook (what you’ll parse)

{
  "event": "call.completed", 
  "vocalgrid_call_id": "lk_abc123",
  "session_id": "cam-...-att-1",
  "disposition": "answered", 
  "duration_seconds": 312,
  "recording_url": "https://...",
  "transcript_url": "https://...",
  "transcript": "Hi Michael ...",
  "metadata": { "candidate_id":"...", "campaign_id":"...", "attempt_no":1 }
}
SMS templates (store in messages or config)
After 2nd no-answer:

“Hey {{first_name}}, this is Ava from Nova Recruiters. Just tried to reach you to check if you’re still open to roles in {{city}}. Reply YES and I’ll call you back, or text your preferred time.”

(Reply “STOP” → add to suppression_list.)

Why this will demo cleanly
Simple two-attempt rule with 7-min retry and SMS fallback.

Full task leasing so you can throttle/scale live.

Inbound lift visible in metrics.

LLM extraction → Handoff shows tangible recruiter value.

If you want, I can:

Generate RLS policies for these tables,

Provide the n8n flow JSON skeletons (HTTP nodes + Supabase nodes + Function code),

Add a Looker Studio starter query set.

Where do you want to start wiring first: CSV importer, task leaser, or webhook handler?

You said:
what if we use GHL as much as we can, not even rely on supabase alot?

like build the logic with pipelines, workflowss, tags etc, upload csv in ghl
ChatGPT said:

Thought for 45s
Totally doable. Here’s a GHL-first version that keeps Supabase out of the critical path (we’ll still use n8n + VocalGrid for calls + webhooks), and uses Contacts, Custom Fields, Workflows, Pipelines, and Tags in GHL for state + reporting.

High-level design (GHL as source of truth)
GHL: holds the CSV import, contact truth, reactivation pipeline, workflow logic, tags, SMS/email, and recruiter handoffs.

n8n: glue + scheduler. Launches outbound calls via VocalGrid and updates GHL via API. Also handles inbound call webhooks.

VocalGrid (LiveKit): outbound/inbound Voice AI. Posts call lifecycle + transcript to n8n → n8n writes back to GHL.

This keeps the demo inside GHL UI (pipelines, workflows, messages, notes), while still using your voice stack.

GHL objects to set up
1) Pipeline (Board)
Pipeline: “Lead Reactivation”
Stages (left → right):

Enrolled

To Dial – Attempt 1

Attempt 1 Completed

To Redial – Attempt 2

Attempt 2 Completed (No Answer)

Qualified → Handoff

Not Looking / DNC

Bad Number

You’ll move cards between these via workflows or n8n API calls.

2) Contact Custom Fields (all live in GHL; use these instead of DB columns)
reactivation_campaign (text) – e.g., “Q4-Accounting”

reactivation_status (text) – “queued | in_progress | completed | no_answer | voicemail | failed”

attempt_count (number) – 0,1,2

attempt1_time (datetime)

attempt2_time (datetime)

next_attempt_at (datetime) – n8n scheduler looks at this

last_disposition (single select) – answered | no_answer | voicemail | busy | failed

ai_still_looking (checkbox)

ai_current_employment (long text)

ai_desired_roles (long text)

ai_new_skills (long text)

ai_salary_min (number)

ai_salary_max (number)

ai_currency (text, default “USD”)

ai_work_mode (single select: onsite/hybrid/remote)

ai_start_timeline (text)

ai_preferred_contact (single select: sms/phone/email)

ai_location_confirmed (text)

ai_transcript_url (text)

ai_recording_url (text)

Don’t encode timestamps in tags—use fields above. Tags stay human-readable and event-driven.

3) Tags (lightweight state markers)
recruitment.reactivation.enrolled

recruitment.reactivation.leased (set by n8n while a call is in flight; cleared afterward)

recruitment.reactivation.attempt1.done

recruitment.reactivation.attempt2.done

recruitment.reactivation.qualified

recruitment.reactivation.dnc

recruitment.reactivation.bad_number

How the flows work (without Supabase)
A) CSV → GHL (no code)
Client uploads CSV directly into GHL Contacts (or via Import).

You can set default tags during import (e.g., recruitment.reactivation.enrolled) and map fields like last applied role/date.

B) Enrollment workflow (GHL)
Trigger: Tag added = recruitment.reactivation.enrolled
Steps (GHL workflow builder):

If attempt_count empty → set to 0

Set reactivation_campaign (static or from CSV column)

Move Opportunity to Enrolled

Set next_attempt_at = now() (or “next 9am in contact’s timezone”)

(Optional) Add a welcoming Note like: “Enrolled in Reactivation: {{date}}”

This primes the record for n8n to pick up when the window opens.

C) Scheduler + Dialer (n8n owns it, but state lives in GHL)
Every minute (or 5m), n8n:

Query GHL Contacts:
Filters:

Has tag recruitment.reactivation.enrolled

attempt_count < 2

next_attempt_at <= now

Not tagged recruitment.reactivation.leased

Not DND (GHL’s DND field)

Safety/concurrency: pick up to N (your throttle), for each:

Add tag recruitment.reactivation.leased

Move stage → “To Dial – Attempt 1” (if attempt_count = 0) or “To Redial – Attempt 2” (if 1)

Launch call via VocalGrid (HTTP) with metadata (contactId, attempt, last applied role/date)

Create GHL Note: “Dial attempt {{attempt_count+1}} started.”

Using the recruitment.reactivation.leased tag is your lease lock to avoid double dials.

D) Outbound call webhooks (VocalGrid → n8n → GHL)
On call events:

answered/completed with transcript

Set last_disposition = answered

Increment attempt_count

Set reactivation_status = completed

Save ai_* fields from LLM extraction (n8n does the extraction → update fields; also attach transcript/recording URLs)

Move stage →

If Qualified by your rule → Qualified → Handoff, tag recruitment.reactivation.qualified, create/advance Opportunity, assign user.

Else → Attempt 1 Completed or keep where it makes sense for your demo.

Remove tag recruitment.reactivation.leased

Add Note with short call summary (first 2–3 bullet points).

no_answer / voicemail / busy / failed

Set last_disposition accordingly

Increment attempt_count

If this was Attempt 1:

Set attempt1_time = now()

Set next_attempt_at = now() + 7 minutes

Move stage → To Redial – Attempt 2

If this was Attempt 2:

Set attempt2_time = now()

Move stage → Attempt 2 Completed (No Answer)

Trigger SMS fallback (see E)

Remove tag recruitment.reactivation.leased

Add Note summarizing outcome.

DNC / Opt-out detected

Flip GHL DND ON, add tag recruitment.reactivation.dnc, move stage Not Looking / DNC, clear next_attempt_at.

E) SMS fallback (keep it all in GHL)
Create a GHL Workflow:
Trigger: Contact tag added = recruitment.reactivation.attempt2.done OR custom field attempt_count == 2 AND last_disposition != answered.
Action: Send SMS (from GHL)

“Hey {{contact.first_name}}, this is Ava from Nova Recruiters. Tried to reach you to check if you’re still open to roles in {{contact.city}}. Reply YES if you are and I’ll call you back, or text your preferred time.”

Add a Reply STOP → DND rule using GHL’s native SMS compliance.

If you prefer n8n to send SMS, just have webhooks drive a GHL “Add Note” for audit, but it’s cleaner to keep SMS in GHL.

F) Inbound calls (VocalGrid number)
Point your public number to VocalGrid inbound agent.

On inbound call end, VocalGrid → n8n with transcript + from number.

n8n finds Contact by phone in GHL:

Create Note with summary

Update ai_* fields if qualified info captured

Move stage to Qualified → Handoff if meets rule

Create/Update Opportunity, assign user, optionally send a GHL internal notification (Slack/Email step).

GHL-only reporting (what you can show on camera)
Pipeline board counts at each stage (Enrolled, Dialing, Qualified, etc.)

Opportunity value (optional static value per qualified) to visualize “pipeline value”

Contact Smartlists filtered by tags/fields:

“Ready to Dial” (next_attempt_at <= now, attempt_count < 2, no leased)

“Completed w/ Extraction” (ai_still_looking = true)

“DNC / Bad Number”

Conversations tab shows SMS fallback + inbound replies.

Notes show call summaries, transcript links.

Minimal n8n flows you’ll need
1) Poll & Dial (CRON every 1–5 min)

GHL API: search contacts with filters (tag, next_attempt_at, attempt_count, DND=false)

For each: add recruitment.reactivation.leased (GHL API), move stage, post to VocalGrid, add Note

2) Outbound Webhook Handler

Receive VocalGrid events

Map disposition → update contact fields/stage/tags; add Note

If answered: run LLM extraction (inside n8n), update ai_*, decide qualification, move stage & create/assign Opportunity

3) Inbound Webhook Handler

Receive inbound; match contact; add Note; update fields; move stage; create/assign Opportunity if qualified

That’s it. No Supabase queue—leasing is done via the recruitment.reactivation.leased tag plus next_attempt_at field.

Guardrails (inside GHL)
Respect DND (don’t schedule if DND true)

Use contact timezone in “Wait” steps if you decide to do some logic inside GHL (e.g., only set next_attempt_at to between 9:00–19:00)

Never rely on tags for timestamps; keep time in custom fields

Keep attempt_count authoritative; n8n increments atomically before/after a call

YouTube demo flow (all GHL-centric)
Import CSV into GHL, auto-tag recruitment.reactivation.enrolled. Show mapped fields.

Show Pipeline filling “Enrolled”.

Toggle to n8n briefly: Poll & Dial flow running (1 minute).

Back to GHL: watch card move to “To Dial – Attempt 1”, Note added “Dial started”.

Answer one call → after it ends, show:

Contact Custom Fields updated (ai_*, salary, work mode, etc.)

Note with summary + transcript link

Card jumps to Qualified → Handoff, Opportunity assigned.

Let another contact miss both attempts → show SMS fallback fired from GHL.

Do an Inbound callback to the number → show Note + field updates; possibly qualify and move stage.

What you’ll need from GHL (one-time setup checklist)
Pipeline + Stages (names above)

Custom fields (list above)

Tags (list above)

Workflows:

Enrollment initializer

SMS fallback (triggered by fields/tags)

(Optional) recruiter notification on Qualified → Handoff

API keys/locations for n8n (sub-account level)

A dedicated tracking number in GHL (optional, you can still store call activity via Notes), but inbound is better directly to VocalGrid.

Sample n8n → GHL updates (pseudo)
Add Tag

POST /contacts/{id}/tags → { "tags": ["recruitment.reactivation.leased"] }

Update Custom Fields

PUT /contacts/{id} with the custom field IDs → set attempt_count, next_attempt_at, last_disposition, etc.

Move Opportunity Stage

PUT /opportunities/{opportunityId} → { "pipelineId": "...", "stageId": "..." }

Add Note

POST /notes → { "contactId": "...", "body": "Attempt 1 completed – disposition: answered\nKey takeaways: ...\nRecording: ...\nTranscript: ..." }

(Exact endpoints vary by your GHL API wrapper; you’ve used these patterns before.)

Why this version is nice
Single source of truth = GHL (contacts, pipeline, messaging, handoffs).

No external DB to maintain for the demo.

n8n stays thin: polling, leasing via tags, call launch, webhooks, and field updates.

Everything is visible in GHL UI for your video.

If you want, I can spit out:

A field/property checklist you can paste into your GHL custom-field creation,

n8n flow skeletons (HTTP nodes + simple Function nodes) with placeholder GHL endpoints,

A stage/ID map template so your flows are copy/paste ready.

You said:
just some tweaks

we can set GHL triggers on new contacts, or certain tag or certain pipeline stage

and then start triggering calls from there

after each call, n8n will receive these from VG , the transcriopt of call etc, and do processing to analyze and exgtract info, and then put it in certain custgom fields, you already thought about this in the plan right ? with ai* custom fields?

ChatGPT said:

Thought for 13s
Yep—perfect. We can make it GHL-triggered → VocalGrid call → n8n processes → write back to GHL ai_* fields. Here’s the tightened wiring so it’s rock solid and demo-friendly.

Triggers (GHL → n8n)
Use GHL Workflows with a Webhook action to kick off dialing. You can trigger on any of:

New Contact created

Tag added (e.g., reactivation.start)

Pipeline stage changed (e.g., “To Dial – Attempt 1” or “To Redial – Attempt 2”)

When the trigger fires, GHL sends a webhook to n8n with:

contactId, phone, name, stage, tags (include what you need via workflow merge fields)

Why event-driven (vs polling): instant, predictable, visible in GHL. We’ll still keep a 1-minute “sweeper” in n8n as a safety net if you want.

Outbound call kickoff (n8n)
Webhook A /ghl/ready-to-dial (from GHL):

GET Contact (GHL API) → read fields (attempt_count, DND, next_attempt_at).

Guardrails: skip if DND, outside call window, or already “leased”.

Lease: add tag recruitment.reactivation.leased, bump attempt_count, set reactivation_status = in_progress, stamp attempt1_time/attempt2_time if needed.

Move stage:

attempt 1 → “To Dial – Attempt 1” → optionally to “Dialing”

attempt 2 → “To Redial – Attempt 2” → optionally to “Dialing”

Start call (VocalGrid) with metadata: {contactId, attempt, campaign, last_applied_role/date}.

Add Note: “Dial attempt {{attempt}} started… (session_id…)”.

This uses the recruitment.reactivation.leased tag as the lock to prevent double dials.

After each call (VocalGrid → n8n → GHL)
Webhook B /vocalgrid/events:

Parse event: answered | no_answer | voicemail | busy | failed | completed.

Update GHL:

last_disposition

attempt_count += 1 (only once per attempt)

reactivation_status (completed, no_answer, etc.)

Clear recruitment.reactivation.leased tag

Set ai_recording_url, ai_transcript_url if provided

If answered/completed:

LLM extraction (in n8n) → produce structured JSON.

Write back to GHL custom fields (ai_*):

ai_still_looking (checkbox true/false)

ai_current_employment (long text)

ai_desired_roles (comma list)

ai_new_skills (comma list)

ai_salary_min, ai_salary_max, ai_currency

ai_work_mode (onsite/hybrid/remote)

ai_start_timeline (text)

ai_preferred_contact (sms/phone/email)

ai_location_confirmed (text)

Stage logic:

If qualified by your rule → move to “Qualified → Handoff”, add tag recruitment.reactivation.qualified, create/update Opportunity, assign user, send internal notification.

Else → move to “Attempt 1 Completed” (or keep simple “Completed”).

If no_answer/voicemail/busy/failed:

If this was Attempt 1:

Set next_attempt_at = now() + 7 minutes

Move to “To Redial – Attempt 2”

If this was Attempt 2:

Move to “Attempt 2 Completed (No Answer)”

Add tag recruitment.reactivation.attempt2.done (for SMS fallback workflow)

Add a concise Note with disposition and timing.

DNC/Opt-out (detected by agent or SMS):

Flip GHL DND on, add tag recruitment.reactivation.dnc, move to “Not Looking / DNC”, clear next_attempt_at.

Inbound callbacks (VocalGrid → n8n → GHL)
Webhook C /vocalgrid/inbound:

Match from phone → Contact in GHL.

After call ends: store transcript/recording; run same LLM extraction; write ai_* fields.

If qualified → move to “Qualified → Handoff”, create/assign Opportunity, notify recruiter.

Add Note summarizing key points.

GHL workflows you’ll set once
Ready to Dial
Trigger: New Contact / Tag reactivation.start / Stage “To Dial – Attempt 1”
Action: Webhook → /ghl/ready-to-dial (n8n)

SMS Fallback (GHL sends SMS natively)
Trigger: Tag recruitment.reactivation.attempt2.done AND last_disposition != answered
Action: Send SMS:

“Hey {{first_name}}, this is Ava from Nova Recruiters. Tried to reach you to check if you’re still open to roles in {{city}}. Reply YES if you are, or text a good time.”

STOP handling
Trigger: SMS reply contains STOP → Set DND = ON, add tag recruitment.reactivation.dnc.

Qualified notification
Trigger: Stage moved to “Qualified → Handoff”
Actions: Create/assign Opportunity (if not exists), notify user/Slack/email.

Small but important implementation notes
Idempotency: use session_id = {contactId}-{attempt}-{yyyyMMddHHmm} when starting calls. If VG retries webhooks, de-dupe by vocalgrid_call_id or session_id.

Time windows: enforce in n8n before dialing (e.g., 09:00–19:00 local). If outside, set next_attempt_at to next window start and do not add recruitment.reactivation.leased.

Race safety: always add recruitment.reactivation.leased before dialing; clear it on any terminal event.

Custom field IDs: grab them once from GHL API and keep a small ID map in n8n env/JSON so updates are easy.

Notes: keep them short; link to recording/transcript; recruiters will love the quick skim.

Example payloads
GHL → n8n (kickoff)

{
  "contactId": "abc123",
  "phone": "+17205551234",
  "firstName": "Michael",
  "stage": "To Dial – Attempt 1",
  "tags": ["reactivation.start"],
  "campaign": "Q4-Accounting"
}
n8n → VocalGrid (start outbound)

{
  "to": "+17205551234",
  "from": "+18335550123",
  "session_id": "abc123-1-20251105T1730",
  "webhook_url": "https://n8n.yourdomain.com/webhooks/vocalgrid/events",
  "metadata": {
    "contactId": "abc123",
    "attempt": 1,
    "campaign": "Q4-Accounting",
    "last_applied_role": "Senior Accountant",
    "last_applied_date": "2024-03-12"
  }
}
VocalGrid → n8n (completed) → GHL field updates

{
  "event": "call.completed",
  "disposition": "answered",
  "duration_seconds": 312,
  "recording_url": "https://.../rec.mp3",
  "transcript": "Hi Michael ...",
  "metadata": { "contactId":"abc123", "attempt":1 }
}
→ Extracted JSON → write to ai_* fields as above.

If you want, I’ll draft:

A compact custom field checklist (names + types) for quick creation in GHL.

n8n flow skeletons (3 webhooks + 1 outbound start) with placeholder URLs/headers.

A tiny field-ID map JSON you can paste into n8n for updates.





No file chosenNo file chosen
ChatGPT can make mistakes. Check important info. See Cookie Preferences.
