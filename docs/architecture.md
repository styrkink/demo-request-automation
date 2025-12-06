# Architecture

This document describes the architecture of the lead automation system built on n8n.

The solution consists of four main workflows:

1. `Demo_request_workflow` — main intake + CRM + first email + logging.
2. `AI_Email_Service` — sub-workflow (microservice) for generating emails.
3. `Reply_handler` — handles incoming replies from Gmail and sends proposals.
4. `Error_handler` — global error handling and alerting.

---

## 1. Components

### 1.1. Intake & CRM (Demo_request_workflow)

**Responsibility:**  
Process new demo requests from a Tally form, normalize data, classify leads, sync with CRM, send the first email, notify the team, and log events.

**Key blocks:**

- Webhook (Tally → n8n)
- Normalize form (Set/Function)
- Classification (spam/valid + priority)
- HubSpot:
  - Create/Update Contact
  - Create Deal (and associate to contact)
- Priority handling (high/normal)
- Slack notifications
- Call to `AI_Email_Service` for first email
- Logging into `lead_logs` (Data Table)

---

### 1.2. AI Email Microservice (AI_Email_Service)

**Responsibility:**  
Reusable email generator, called from other workflows via `Execute Workflow`.

**Inputs:**

- `mode` — `"first_email"` or `"proposal"`
- `lead` — normalized lead object
- `priority` — `"high"` / `"normal"`
- `contactVid` — HubSpot contact ID (optional)
- `dealId` — HubSpot deal ID (optional)
- `clientReply` — raw textual reply from the client (used in `proposal` mode)

**Logic:**

- Normalize input
- AI Agent (OpenAI via n8n) with Simple Memory
- Build `subject`, `body`, and `toEmail`
- Send email via Gmail

---

### 1.3. Reply Handler (Reply_handler)

**Responsibility:**  
Process replies from Gmail, restore lead context from logs, generate a proposal with AI, send it, and log the event.

**Key blocks:**

- Gmail Trigger (on new message)
- Normalize reply (fromEmail, replyBody, subject, threadId)
- Read last log by email from `lead_logs`
- Build `lead` object + CRM IDs
- Add `clientReply`
- Call `AI_Email_Service` with `mode = "proposal"`
- Send proposal via Gmail
- Log `proposal_sent` in `lead_logs`

---

### 1.4. Error Handling (Error_handler)

**Responsibility:**  
Catch any failure in the system, create an error log entry, and notify engineers via Slack.

**Workflow:**

- Error Trigger (n8n built-in)
- Prepare error log (Function)
- Insert log into `lead_logs` with `step = error` and `status = error`
- Send Slack message with:
  - workflow name
  - last node
  - error message
  - execution URL

---

## 2. Data Flow Overview

In simplified form:

1. **Lead intake**  
   Tally → Webhook → Normalize → Classification → CRM → AI first email → Logging

2. **Reply cycle**  
   Gmail → Normalize reply → Read previous log → Rebuild lead → AI proposal → Gmail → Logging

3. **Error handling**  
   Any workflow fails → Error Trigger → Prepare error log → Insert log → Slack alert

See detailed Mermaid diagrams in:

- `docs/mermaid-architecture.md`
- `docs/mermaid-dataflow.md`
- `docs/mermaid-sequences.md`

---

## 3. Design Principles

- **Append-only logging** — every significant step produces a log entry, no updates; full history is preserved.
- **Microservice pattern** — `AI_Email_Service` is decoupled and reused for multiple scenarios (`first_email`, `proposal`).
- **Error isolation** — dedicated `Error_handler` workflow centralizes error reporting.
- **SLA-aware logic** — lead priority influences both messaging and internal handling (Slack channels, copy, timing).
- **Replayability** — execution IDs (`raw_execution`) and logs allow reproducing and analyzing specific runs.
