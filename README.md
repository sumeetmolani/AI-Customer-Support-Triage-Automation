AI Customer Support Triage Automation
AI-powered customer-support classification and routing workflow built with self-hosted n8n. 
Google Forms
Google Sheets
Google Gemini
JavaScript
JSON
1. Project Overview
   This project automates the first stage of customer-support triage.
Customer requests are collected through Google Forms and stored in Google Sheets. A self-hosted n8n workflow reads the requests, standardises the data, validates the input, processes requests individually, sends the request to Google Gemini for AI classification, converts the AI response into structured JSON, and routes the result through a Switch node.
The workflow classifies requests into:
•	Sales
•	Support
•	Billing
•	General
•	Urgent
The AI also determines:
•	Priority
•	Sentiment
•	Whether human intervention is required
•	Reason for the classification
________________________________________
2. Workflow Architecture
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
________________________________________
3. Tools & Technologies
Tool / Technology	          Purpose
n8n 2.25.7	               Self-hosted workflow automation platform
Google Forms	               Collects customer requests
Google Sheets	               Stores customer submissions
Google Gemini	               AI-powered request classification
AI Agent	                    Sends customer information to the AI model according to the classification instructions
JavaScript	               Processes and transforms AI output
JSON	                         Structured data format between workflow nodes
IF	                         Input validation and conditional logic
Loop Over Items	          Processes requests individually
Switch	                    Routes requests to category-specific outputs
________________________________________
4. AI Classification Output
   The AI produces the following structure:
   {
   "category": "sales|support|billing|general|urgent",
   "priority": "low|medium|high|critical",
   "sentiment": "positive|neutral|negative",
   "needs_human": true,
   "reason": "Brief explanation of the classification"
}
Classification categories
Sales
Examples:
•	Pricing questions
•	Purchasing
•	Quotations
•	Implementation proposals
•	Service packages
Support
Examples:
•	API problems
•	Integration errors
•	Technical problems
•	Authentication failures
•	Software issues
Billing
Examples:
•	Incorrect invoices
•	Payment problems
•	Refund request
•	Billing disputes
General
Examples:
•	General service information
•	Questions about how the company works
•	Non-specific informational requests
Urgent
Examples:
•	Production outage
•	Major service interruption
•	Severe operational problem
•	Security incident
________________________________________
5. Example Test Data
The workflow was tested with customer requests representing different business scenarios.
Customer	Request	Expected Category
Ali Khan	Automation services pricing	Sales
Ahmed Raza	API authentication/integration problem	Support
Sara Ali	Incorrect invoice amount	Billing
Usman Ahmed	Available services and process information	General
Hamza Shah	Production system completely down	Urgent
________________________________________
6. Example End-to-End Processing
Example:
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
Expected AI result:
{
  "category": "urgent",
  "priority": "critical",
  "sentiment": "negative",
  "needs_human": true,
  "reason": "The customer's production integration is completely unavailable and requires immediate intervention."
}
________________________________________
7. Key Automation Concepts Demonstrated
This project demonstrates:
•	Self-hosted n8n
•	Workflow orchestration
•	Google Sheets integration
•	Cloud data handling
•	AI/LLM integration
•	Prompt-based classification
•	JSON processing
•	JavaScript scripting
•	Conditional logic
•	Loop/batch processing
•	Deterministic routing
•	API rate-limit awareness
•	Error troubleshooting
•	Data normalisation
________________________________________
8. Project Scope
Implemented
•	Customer data collection
•	Google Sheets integration
•	Data normalisation
•	Input validation
•	Individual item processing
•	AI classification
•	Priority detection
•	Sentiment detection
•	Human-escalation flag
•	JSON transformation
•	Category-based routing
Not implemented in this version
The following were intentionally not added to this workflow:
•	CRM creation
•	Slack notifications
•	Automated email replies
•	Support-ticket creation
•	Human approval interface
•	Customer database
•	Production monitoring dashboard
These can be added as separate automation projects or future extensions.
________________________________________
9. Documentation References
Official documentation used for the technologies in this project:
•	n8n Documentation
•	n8n Google Sheets
•	n8n Edit Fields
•	n8n IF
•	n8n Loop Over Items
•	n8n AI Agent
•	n8n Google Gemini
•	n8n Code
•	n8n Switch
________________________________________
10. Portfolio Description
AI Customer Support Triage Automation
Built a self-hosted n8n automation workflow that collects customer requests from Google Forms/Google Sheets, validates and standardises the data, uses Google Gemini to classify natural-language requests by business category and priority, transforms the AI response into structured JSON using JavaScript, and deterministically routes requests through a Switch node.
Technologies: n8n, Google Forms, Google Sheets, Google Gemini, JavaScript, JSON, AI automation, workflow orchestration.

