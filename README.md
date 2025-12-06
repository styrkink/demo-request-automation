# 🚀 Automated Lead Processing & AI Email Engine (n8n + HubSpot + Gmail + Slack + OpenAI)
This repository contains a fully automated lead-processing system built on **n8n**, integrating:

- Tally / Webhook
- HubSpot CRM
- Gmail (AI-generated emails)
- Slack (alerts)
- OpenAI GPT-4.1-mini (email generation)
- Custom logging and error tracking engine

The system processes inbound demo requests, classifies the lead, enriches data, creates CRM objects, sends the first email, tracks replies, generates proposals, logs every step, and notifies the sales team.

---

## 🧩 System Overview

The automation consists of **4 main workflows**:

1. **Demo Request Workflow**  
   Handles new inbound demo requests from Tally.  
   Creates/updates CRM records, classifies the lead, triggers alerts, sends the first email.

2. **AI Email Service**  
   A reusable sub-workflow that generates:
   - First outreach emails
   - Commercial proposals (based on reply text + lead info + conversation memory)

3. **Reply Handler**  
   Parses incoming Gmail replies, fetches the CRM record and previous log entries, generates a proposal, emails it back, and logs the action.

4. **Error Handler**  
   Global workflow triggered on any failure.  
   Extracts context, creates a log entry, and alerts Slack.

---

## 🏆 Business Value

| Problem Before | Solution After |
|----------------|----------------|
| Leads were processed manually, taking 15–45 min each | Automated end-to-end processing in **<30 seconds** |
| Sales manager had to write the first outreach email | AI drafts a high-quality email instantly |
| No consistent qualification or spam filtering | Smart classification via urgency, budget, domain flags |
| No clear SLA and inconsistent response times | SLA tied to lead priority: **2 hours vs 1 business day** |
| Proposal creation required reading client replies and manual drafting | AI generates structured professional proposals using memory + reply text |
| Errors were hard to detect | Clear **error logs + Slack alerts with execution links** |

This automation increases lead-to-meeting conversion, reduces response times, and provides transparency into pipeline operations.

---

## 🧱 Architecture Diagram (Mermaid)

```mermaid
flowchart LR
    A[Tally / Webhook] --> B(Normalize Form)
    B --> C{Classification}
    C -->|Spam| L1[Log + Exit]
    C -->|Valid| D[HubSpot Contact]
    D --> E[HubSpot Deal]
    E --> F{Priority}
    F -->|High| G1[Slack High-P Alert]
    F -->|Normal| G2[Slack Normal-P Alert]
    F --> H[Call AI_Email_Service]
    H --> I[Send First Email]
    I --> J[Log Entry]

    subgraph Reply Handler
        R1[Gmail Trigger] --> R2(Normalize Reply)
        R2 --> R3[Read Log by Email]
        R3 --> R4[Call AI_Email_Service - Proposal]
        R4 --> R5[Send Proposal Email]
        R5 --> R6[Update Log]
    end

    subgraph Error Handler
        ER1[Error Trigger] --> ER2(Prepare Error Log)
        ER2 --> ER3[Insert Log]
        ER3 --> ER4[Slack Error Alert]
    end

Intelligent Lead Classification

Based on:
budget, company size, urgency, email domain, consent flags, UTM parameters
→ reduces junk leads and prioritizes high-value prospects.

🔹 Automatic CRM Enrichment

Each valid lead automatically becomes:

HubSpot Contact

HubSpot Deal (with budget, pipeline, & company)

🔹 AI-Generated Emails

All emails use AI_Email_Service, which supports:

Mode: first_email

Natural, professional outreach

SLA included

Personalized questions

No Markdown, clean text only

Mode: proposal

Summarizes client needs

Structured plan

Budget range aligned with their input

Timeline + next steps

Uses SimpleMemory to maintain conversation history.

🔹 Reply Tracking

Reply handler:

Extracts sender email + thread ID

Fetches previous log

Decides the next step

Sends a tailored proposal

🔹 Logging Engine

All workflows write to a unified n8n Data Table: Leads_log

Each record includes:

email, company, budget

priority & classification

contact_id, deal_id

step, status

error_message

raw_execution (execution ID)

🔹 Error Handling

If any workflow fails:

context is extracted

log entry is saved

Slack alert includes:

workflow name

last executed node

execution URL

full error message

📂 Workflows Included
Demo_request_workflow.json	Main entry point — classifies and processes new leads
AI_Email_Service.json	Reusable AI engine for writing all emails
Reply_handler.json	Generates proposals in reply to client messages
Error_handler.json	Centralized error logging + Slack alerts

All workflow JSON files are available in /workflows/.

📘 Documentation

See /docs/ for:

architecture.md — component diagrams, flow explanation

sequence-demo-request.md — full event-level sequence

sequence-reply-handler.md — how replies are processed

data-schema.md — schema of Leads_log

error-handling.md — all failure scenarios and recovery

mermaid-*.md — diagrams in editable Mermaid format

/docs/diagrams/*.png — exported PNG diagrams

🛠 Technologies Used

n8n (primary automation engine)

HubSpot CRM API

Slack API

Gmail API

OpenAI GPT-4.1-mini

Custom Data Table (n8n Cloud)

📞 Contact

If you're reviewing this as part of a portfolio or Upwork application —
feel free to reach out with questions or request demo access.