# n8n Automations — Lead Reactivation

This folder documents the n8n workflows that support the GHL Lead Reactivation system.

- 01 Dispatch Call: receives a webhook from GHL and triggers VocalGrid to place a call.
- 02 Intake Call Webhook: receives post-call events from VocalGrid, runs AI analysis, and updates GHL.
- LLM Extraction Schema: defines exactly what we extract from call transcripts and where it goes in GHL.
- flows.mmd: sequence and data-flow diagrams.

Update the base URL and secrets before deploying.
