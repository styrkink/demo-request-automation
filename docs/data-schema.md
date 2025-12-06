# Data Schema – `lead_logs` Table

The system uses a single main Data Table in n8n called `lead_logs`.  
It acts as an **append-only event log** for all lead-related actions.

Each step for each lead is written as a separate row, preserving complete history.

---

## 1. Table: `lead_logs`

| Column         | Type     | Description                                                                 |
|----------------|----------|-----------------------------------------------------------------------------|
| `id`           | integer  | Primary key (auto-increment)                                               |
| `email`        | string   | Lead email                                                                 |
| `company`      | string   | Company name                                                               |
| `priority`     | string   | Lead priority (`high` / `normal`, can be extended)                         |
| `classification` | string | Lead classification (`valid` / `spam`)                                     |
| `source`       | string   | Acquisition source (LinkedIn, Telegram, YouTube, etc.)                     |
| `budget`       | number   | Numeric budget value (e.g. USD)                                           |
| `urgency`      | string   | Urgency level (`High`, `Medium`, `Low`)                                   |
| `use_case`     | string   | Free-text description of the problem / request                             |
| `contact_id`   | string   | HubSpot contact ID (vid)                                                   |
| `deal_id`      | string   | HubSpot deal ID                                                            |
| `step`         | string   | Logical step in the pipeline (e.g. `classification`, `crm_done`, `first_email_sent`, `proposal_sent`, `spam_detected`, `error`) |
| `status`       | string   | Result of this step (`ok`, `received_valid`, `received_spam`, `error`, etc.) |
| `error_message`| string   | Error description if the step failed or was processed by Error_handler     |
| `raw_execution`| string   | n8n execution ID that produced this log entry                              |
| `createdAt`    | datetime | Timestamp when the row was created (auto-managed by n8n Data Tables)       |
| `updatedAt`    | datetime | Timestamp when the row was last updated (auto-managed)                     |

> Note: although the table has `updatedAt`, the design is **append-only**; we treat each row as immutable and add new rows instead of updating existing steps.

---

## 2. Example Rows

### 2.1. Valid lead received

```json
{
  "email": "john@example.com",
  "company": "Acme Inc.",
  "priority": "high",
  "classification": "valid",
  "source": "LinkedIn",
  "budget": 1500,
  "urgency": "High",
  "use_case": "Automate lead handling and proposals.",
  "contact_id": null,
  "deal_id": null,
  "step": "classification",
  "status": "received_valid",
  "error_message": "",
  "raw_execution": "123",
  "createdAt": "2025-11-30T12:00:00.000Z"
}

### 2.2. CRM created (contact + deal)

{
  "email": "john@example.com",
  "company": "Acme Inc.",
  "priority": "high",
  "classification": "valid",
  "source": "LinkedIn",
  "budget": 1500,
  "urgency": "High",
  "use_case": "Automate lead handling and proposals.",
  "contact_id": "327570212546",
  "deal_id": "123456789",
  "step": "crm_done",
  "status": "ok",
  "error_message": "",
  "raw_execution": "123",
  "createdAt": "2025-11-30T12:00:05.000Z"
}

### 2.3. First email sent

{
  "email": "john@example.com",
  "company": "Acme Inc.",
  "priority": "high",
  "classification": "valid",
  "source": "LinkedIn",
  "budget": 1500,
  "urgency": "High",
  "use_case": "Automate lead handling and proposals.",
  "contact_id": "327570212546",
  "deal_id": "123456789",
  "step": "first_email_sent",
  "status": "ok",
  "error_message": "",
  "raw_execution": "123",
  "createdAt": "2025-11-30T12:00:07.000Z"
}

### 2.4. Proposal sent (via Reply_handler)

{
  "email": "john@example.com",
  "company": "Acme Inc.",
  "priority": "high",
  "classification": "valid",
  "source": "LinkedIn",
  "budget": 1500,
  "urgency": "High",
  "use_case": "Automate lead handling and proposals.",
  "contact_id": "327570212546",
  "deal_id": "123456789",
  "step": "proposal_sent",
  "status": "ok",
  "error_message": "",
  "raw_execution": "456",
  "createdAt": "2025-12-01T09:15:00.000Z"
}

### 2.5. Error

{
  "email": null,
  "company": null,
  "priority": null,
  "classification": null,
  "source": null,
  "budget": null,
  "urgency": null,
  "use_case": null,
  "contact_id": null,
  "deal_id": null,
  "step": "error",
  "status": "error",
  "error_message": "[Lead Intake Main] node \"Create a deal\": HubSpot API 500",
  "raw_execution": "789",
  "createdAt": "2025-12-01T10:00:00.000Z"
}
