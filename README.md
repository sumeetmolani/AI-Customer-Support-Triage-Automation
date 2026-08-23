# AI Customer Support Triage Automation

> AI-powered customer-support classification and routing workflow built with self-hosted n8n, Google Forms, Google Sheets, Google Gemini, JavaScript, and JSON.

## 1. Project Overview

This project automates the first stage of customer-support triage.

Customer requests are collected through Google Forms and stored in Google Sheets. A self-hosted n8n workflow reads the requests, standardises the data, validates the input, processes requests individually, sends the request to Google Gemini for AI classification, converts the AI response into structured JSON, and routes the result through a Switch node.

The workflow classifies requests into:

- **Sales**
- **Support**
- **Billing**
- **General**
- **Urgent**

The AI also determines:

- Priority
- Sentiment
- Whether human intervention is required
- Reason for the classification
- [screenshot]([screenshot/AI Customer Support Automation screenshot.png](https://github.com/sumeetmolani/AI-Customer-Support-Triage-Automation/blob/f639860c281de553d37fe58104d1c04e4bcb94e9/screenshot/AI%20Customer%20Support%20Automation%20screenshot%20.png))

---

## 2. Workflow Architecture

```text
Google Form
     │
     ▼
Google Sheets
     │
     ▼
Edit Fields
     │
     ▼
IF
     │
     ▼
Loop Over Items
     │
     ▼
AI Agent ◄──── Google Gemini Chat Model
     │
     ▼
Code
     │
     ▼
Switch
 ┌───┼────────┬──────────┬─────────┐
 ▼   ▼        ▼          ▼         ▼
Sales Support Billing   General   Urgent
```

---

## 3. Tools & Technologies

| Tool / Technology | Purpose |
|---|---|
| **n8n 2.25.7** | Self-hosted workflow automation platform |
| **Google Forms** | Collects customer requests |
| **Google Sheets** | Stores customer submissions |
| **Google Gemini** | AI-powered request classification |
| **AI Agent** | Sends customer information to the AI model according to the classification instructions |
| **JavaScript** | Processes and transforms AI output |
| **JSON** | Structured data format between workflow nodes |
| **IF** | Input validation and conditional logic |
| **Loop Over Items** | Processes requests individually |
| **Switch** | Routes requests to category-specific outputs |

---

# 4. Node-by-Node Explanation

## 4.1 Google Sheets

### Purpose

Google Sheets is the workflow's input/data source.

Google Form submissions are stored as rows in the spreadsheet. n8n reads those rows and passes the customer information into the automation.

### Example data

```json
{
  "customer_name": "Hamza Shah",
  "customer_email": "hamza@example.com",
  "company": "FinTech Solutions",
  "request_type": "Technical Support",
  "subject": "Production system is down",
  "message": "Our production integration has stopped completely, and customers are unable to submit requests.",
  "urgency": "Critical",
  "contact_method": "Phone"
}
```

### Why we use it

It provides a simple cloud-based data source that integrates directly with n8n and can store multiple customer requests.

### Documentation

[n8n Google Sheets documentation](https://docs.n8n.io/integrations/builtin/app-nodes/n8n-nodes-base.googlesheets/)

---

## 4.2 Edit Fields

### Purpose

The Edit Fields node standardises the incoming Google Sheets data.

For example, long form column names can be mapped to simple internal fields:

```text
Message / Request Details
        ↓
message
```

The workflow uses fields such as:

```text
customer_name
customer_email
company
request_type
subject
message
urgency
contact_method
timestamp
```

### Why we use it

Standardized field names make expressions, AI prompts, JavaScript, and downstream routing easier to maintain.

### Documentation

[n8n Edit Fields documentation](https://docs.n8n.io/integrations/builtin/core-nodes/n8n-nodes-base.set/)

---

## 4.3 IF

### Purpose

The IF node provides conditional validation before the request reaches the AI stage.

For example, required customer information can be checked before processing.

```text
Valid request?
     │
 ┌───┴───┐
 YES     NO
  │       │
  ▼       ▼
Continue Error/other path
```

### Why we use it

It prevents incomplete or invalid data from unnecessarily reaching the AI model.

This is especially useful because sending empty requests to an external AI API wastes requests and can produce meaningless classifications.

### Documentation

[n8n IF documentation](https://docs.n8n.io/integrations/builtin/core-nodes/n8n-nodes-base.if/)

---

## 4.4 Loop Over Items

### Purpose

Loop Over Items processes incoming records in controlled batches.

For this project, the workflow uses:

```text
Batch Size = 1
```

This means each customer request is processed individually.

### Why we use it

The AI API can have request-rate limitations. Processing one request at a time gives the workflow better control over the number and timing of AI requests.

It also makes testing individual customer records easier.

### Documentation

[n8n Loop Over Items documentation](https://docs.n8n.io/flow-logic/looping/)

---

## 4.5 AI Agent

### Purpose

The AI Agent is the intelligence layer of the workflow.

It receives the customer's structured information and follows the classification instructions.

The agent is asked to analyse:

```text
Customer
Company
Request Type
Subject
Message
Urgency
Contact Method
```

and return a structured classification.

### Why we use it

Traditional workflow conditions are good at checking exact values, but they are not designed to understand the meaning of natural-language customer messages.

For example:

> "Our production integration has stopped completely, and customers are unable to submit requests."

The AI can interpret this as an operational outage and classify it as urgent.

### Documentation

[n8n AI Agent documentation](https://docs.n8n.io/integrations/builtin/cluster-nodes/root-nodes/n8n-nodes-langchain.agent/)

---

## 4.6 Google Gemini Chat Model

### Purpose

The Gemini Chat Model provides the language model used by the AI Agent.

The AI Agent handles the agent instructions and workflow integration, while Gemini performs the natural-language analysis.

### Why we use it

Gemini analyses customer messages and produces the classification information required by the workflow.

Example:

```json
{
  "category": "urgent",
  "priority": "critical",
  "sentiment": "negative",
  "needs_human": true,
  "reason": "The customer reports a complete production outage requiring immediate intervention."
}
```

### Documentation

[n8n Google Gemini documentation](https://docs.n8n.io/integrations/builtin/app-nodes/n8n-nodes-langchain.googlegemini/)

---

## 4.7 Code

### Purpose

The Code node processes the AI response using JavaScript.

During testing, Gemini could return JSON wrapped inside Markdown code fences:

```text
```json
{
  "category": "urgent",
  "priority": "critical"
}
```
```

The Code node removes the Markdown wrapper and parses the content into a real JSON object.

### Example

```javascript
const raw = $json.output;

const cleaned = raw
  .replace(/^```json\s*/i, '')
  .replace(/\s*```$/i, '')
  .trim();

const result = JSON.parse(cleaned);

return {
  json: result
};
```

### Why we use it

The Switch node needs structured fields that it can evaluate reliably.

After the Code node, the workflow can use:

```text
$json.category
$json.priority
$json.sentiment
$json.needs_human
```

### Documentation

[n8n Code node documentation](https://docs.n8n.io/code/)

---

## 4.8 Switch

### Purpose

The Switch node performs deterministic routing based on the AI classification.

The workflow checks:

```text
category
```

and routes the item to the appropriate output.

### Routing

```text
sales
  ↓
Sales output

support
  ↓
Support output

billing
  ↓
Billing output

general
  ↓
General output

urgent
  ↓
Urgent output
```

### Why we use it

The AI is responsible for **understanding and classifying** the customer request.

The Switch node is responsible for **deterministic workflow routing**.

This separation makes the automation easier to control and maintain.

### Documentation

[n8n Switch documentation](https://docs.n8n.io/integrations/builtin/core-nodes/n8n-nodes-base.switch/)

---

# 5. AI Classification Output

The AI produces the following structure:

```json
{
  "category": "sales|support|billing|general|urgent",
  "priority": "low|medium|high|critical",
  "sentiment": "positive|neutral|negative",
  "needs_human": true,
  "reason": "Brief explanation of the classification"
}
```

## Classification categories

### Sales

Examples:

- Pricing questions
- Purchasing
- Quotations
- Implementation proposals
- Service packages

### Support

Examples:

- API problems
- Integration errors
- Technical problems
- Authentication failures
- Software issues

### Billing

Examples:

- Incorrect invoices
- Payment problems
- Refund requests
- Billing disputes

### General

Examples:

- General service information
- Questions about how the company works
- Non-specific informational requests

### Urgent

Examples:

- Production outage
- Major service interruption
- Severe operational problem
- Security incident

---

# 6. Example Test Data

The workflow was tested with customer requests representing different business scenarios.

| Customer | Request | Expected Category |
|---|---|---|
| Ali Khan | Automation services pricing | Sales |
| Ahmed Raza | API authentication/integration problem | Support |
| Sara Ali | Incorrect invoice amount | Billing |
| Usman Ahmed | Available services and process information | General |
| Hamza Shah | Production system completely down | Urgent |

---

# 7. Example End-to-End Processing

Example:

```text
Customer request
      ↓
"Production system is down"
      ↓
Google Sheets
      ↓
Edit Fields
      ↓
IF validation
      ↓
Loop Over Items
      ↓
AI Agent
      ↓
Gemini
      ↓
AI classification
      ↓
Code → structured JSON
      ↓
Switch
      ↓
Urgent output
```

Expected AI result:

```json
{
  "category": "urgent",
  "priority": "critical",
  "sentiment": "negative",
  "needs_human": true,
  "reason": "The customer's production integration is completely unavailable and requires immediate intervention."
}
```

---

# 8. Key Automation Concepts Demonstrated

This project demonstrates:

- Self-hosted n8n
- Workflow orchestration
- Google Sheets integration
- Cloud data handling
- AI/LLM integration
- Prompt-based classification
- JSON processing
- JavaScript scripting
- Conditional logic
- Loop/batch processing
- Deterministic routing
- API rate-limit awareness
- Error troubleshooting
- Data normalisation

---

# 9. Project Scope

### Implemented

- Customer data collection
- Google Sheets integration
- Data normalisation
- Input validation
- Individual item processing
- AI classification
- Priority detection
- Sentiment detection
- Human-escalation flag
- JSON transformation
- Category-based routing

### Not implemented in this version

The following were intentionally not added to this workflow:

- CRM creation
- Slack notifications
- Automated email replies
- Support-ticket creation
- Human approval interface
- Customer database
- Production monitoring dashboard

These can be added as separate automation projects or future extensions.

---

# 10. Documentation References

Official documentation used for the technologies in this project:

- [n8n Documentation](https://docs.n8n.io/)
- [n8n Google Sheets](https://docs.n8n.io/integrations/builtin/app-nodes/n8n-nodes-base.googlesheets/)
- [n8n Edit Fields](https://docs.n8n.io/integrations/builtin/core-nodes/n8n-nodes-base.set/)
- [n8n IF](https://docs.n8n.io/integrations/builtin/core-nodes/n8n-nodes-base.if/)
- [n8n Loop Over Items](https://docs.n8n.io/flow-logic/looping/)
- [n8n AI Agent](https://docs.n8n.io/integrations/builtin/cluster-nodes/root-nodes/n8n-nodes-langchain.agent/)
- [n8n Google Gemini](https://docs.n8n.io/integrations/builtin/app-nodes/n8n-nodes-langchain.googlegemini/)
- [n8n Code](https://docs.n8n.io/code/)
- [n8n Switch](https://docs.n8n.io/integrations/builtin/core-nodes/n8n-nodes-base.switch/)

---

# 11. Portfolio Description

**AI Customer Support Triage Automation**

Built a self-hosted n8n automation workflow that collects customer requests from Google Forms/Google Sheets, validates and standardizes the data, uses Google Gemini to classify natural-language requests by business category and priority, transforms the AI response into structured JSON using JavaScript, and deterministically routes requests through a Switch node.

**Technologies:** n8n, Google Forms, Google Sheets, Google Gemini, JavaScript, JSON, AI automation, workflow orchestration.
