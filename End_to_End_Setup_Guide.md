> **Author**: Ken Tsui (CSA), Microsoft
> **Copyright** © 2026 Microsoft. All rights reserved.

# Contoso Smart Workspace — End-to-End Setup Guide

> A streamlined step-by-step guide to build the complete multi-agent system. Follow each section in order.

---

## Prerequisites

- [ ] Microsoft 365 tenant with Copilot Studio license
- [ ] SharePoint Online access
- [ ] Power Automate access
- [ ] Power Apps / Dataverse access
- [ ] Outlook (Exchange Online) enabled

---

## Phase 1: Data & Document Setup

### 1.1 Create SharePoint List: `ClientPolicies`

1. Go to your **SharePoint site** → **New** → **List** → name it `ClientPolicies`
2. Add columns:

| Column Name | Type |
|-------------|------|
| ClientName | Single line of text *(uses the default "Title" column — rename it)* |
| ClientEmail | Single line of text |
| PolicyNumber | Single line of text |
| PolicyType | Choice: `Life`, `Health`, `Travel`, `Critical Illness` |
| CoverageAmount | Currency |
| PolicyStatus | Choice: `Active`, `Pending`, `Expired`, `Under Review` |
| StartDate | Date |
| EndDate | Date |
| AssignedAdvisor | Single line of text |
| LastInteraction | Date |
| Notes | Multiple lines of text |

3. Add sample data:

| ClientName | ClientEmail | PolicyNumber | PolicyType | CoverageAmount | PolicyStatus | StartDate | EndDate | AssignedAdvisor |
|-----------|------------|-------------|-----------|---------------|-------------|-----------|---------|----------------|
| David Chan | david.chan@email.com | CTO-2025-00123 | Life | 1,000,000 | Active | 2024-01-15 | 2034-01-15 | Sarah Wong |
| Emily Lau | emily.lau@email.com | CTO-2025-00456 | Health | 500,000 | Active | 2025-03-01 | 2026-03-01 | Sarah Wong |
| Michael Tam | michael.tam@email.com | CTO-2024-00789 | Critical Illness | 800,000 | Under Review | 2024-06-10 | 2044-06-10 | James Li |
| Karen Ho | karen.ho@email.com | CTO-2025-00234 | Travel | 200,000 | Active | 2025-11-01 | 2026-11-01 | Sarah Wong |
| Alex Yeung | alex.yeung@email.com | CTO-2023-00567 | Life | 2,000,000 | Expired | 2013-09-01 | 2023-09-01 | James Li |

### 1.2 Create SharePoint Document Library: `ProductGuides`

1. **New** → **Document library** → name it `ProductGuides`
2. Upload these 3 documents (create them from the content in Appendix B of the Advanced Demo Guide):
   - `Contoso_Product_FAQ.docx`
   - `Underwriting_Guidelines.docx`
   - `Compliance_Policy.docx`

### 1.3 Create Dataverse Table: `InteractionLog`

1. Open **Power Apps** → **Tables** → **New table** → name it `InteractionLog`
2. Add columns:

| Column | Type |
|--------|------|
| ClientName | Text |
| AdvisorName | Text |
| InteractionDate | Date |
| InteractionType | Choice: `Call`, `Meeting`, `Email`, `Chat` |
| Summary | Multiline Text |
| FollowUpRequired | Yes/No |
| FollowUpDate | Date |

3. Note the **numeric Choice values** for InteractionType (Power Apps → Tables → InteractionLog → Edit column → Choices). Default values are typically:
   - Call = 890,950,000
   - Meeting = 890,950,001
   - Email = 890,950,002
   - Chat = 890,950,003

> ⚠️ Your values may differ. Record them — you'll need them for the Power Automate flow.

---

## Phase 2: Build Agent 1 — Contoso Advisor Hub (Orchestrator)

### 2.1 Create the Agent

1. **Copilot Studio** → **Create** → **New agent**
2. Name: `Contoso Advisor Hub`
3. Description: `Central hub for Contoso financial advisors. Routes to specialized agents for policy lookup, product advice, and scheduling.`

### 2.2 Set Instructions

Paste this into the Instructions field:

```
You are the Contoso Advisor Hub, an AI assistant built for Contoso financial advisors.

Your role is to help advisors work more efficiently by:
- Looking up client policy information
- Providing product recommendations and answering product questions
- Scheduling meetings and sending follow-up emails to clients
- Logging interaction summaries

You coordinate with specialized agents:
- For policy lookups and client data queries → transfer to the Policy Lookup Agent
- For product questions, comparisons, and recommendations → transfer to the Product Advisor Agent
- For scheduling meetings and sending emails → transfer to the Scheduling Assistant Agent

Rules:
- Always confirm the client name or policy number before performing any lookup.
- Protect client confidentiality — never share one client's data with another.
- When you are unsure which agent to use, ask the advisor to clarify their request.
- Use a professional but friendly tone suitable for internal staff.
- If the advisor asks about underwriting or compliance, answer from the knowledge sources provided.
```

### 2.3 Add Knowledge (SharePoint)

1. **Knowledge** → **+ Add knowledge** → **SharePoint**
2. Enter your SharePoint site URL → select `ProductGuides` library
3. Select: `Contoso_Product_FAQ.docx`, `Underwriting_Guidelines.docx`, `Compliance_Policy.docx`
4. Click **Add** → wait for indexing

### 2.4 Enable Generative Orchestration

1. **Settings** → **Generative AI** → enable **Generative orchestration**

---

## Phase 3: Build Agent 2 — Policy Lookup Agent

### 3.1 Create the Agent

1. Name: `Policy Lookup Agent`
2. Description: `Looks up client policy details from SharePoint for financial advisors.`
3. Set Instructions — paste:

```
You are the Policy Lookup Agent. Your job is to help Contoso financial advisors find client policy information.

You can:
- Search for a client by name or policy number
- Return policy details including type, coverage, status, and dates
- Identify policies that are expiring soon or under review

Rules:
- Always confirm the search criteria with the advisor before querying.
- Present results in a clear, structured format.
- If multiple results are found, list them and ask the advisor to select.
- Never modify client data — you are read-only.
- If no results are found, suggest checking the spelling or trying a different search.
```

### 3.2 Create Power Automate Flow: "Search Client Policies"

1. In the agent → **Topics** → open a topic → **+** → **Call an action** → **Create a flow**
2. **Trigger inputs**: `SearchField` (Text), `SearchValue` (Text)
3. **Compose** — "Build Filter Query" (Expression):
   ```
   if(equals(triggerBody()?['text'],'PolicyNumber'),concat('PolicyNumber eq ''',triggerBody()?['text_1'],''''),concat('Title eq ''',triggerBody()?['text_1'],''''))
   ```
4. **SharePoint — Get items**: Site = your URL, List = `ClientPolicies`, Filter Query = Compose output
5. **Compose** — "Get First Result" (Expression): `first(outputs('Get_items')?['body/value'])`
6. **Return values** (10 outputs, all Text type, using Expression tab):

| Output | Expression |
|--------|-----------|
| ClientName | `outputs('Get_First_Result')?['Title']` |
| PolicyNumber | `outputs('Get_First_Result')?['PolicyNumber']` |
| PolicyType | `outputs('Get_First_Result')?['PolicyType']?['Value']` |
| CoverageAmount | `string(outputs('Get_First_Result')?['CoverageAmount'])` |
| PolicyStatus | `outputs('Get_First_Result')?['PolicyStatus']?['Value']` |
| StartDate | `outputs('Get_First_Result')?['StartDate']` |
| EndDate | `outputs('Get_First_Result')?['EndDate']` |
| ClientEmail | `outputs('Get_First_Result')?['ClientEmail']` |
| AssignedAdvisor | `outputs('Get_First_Result')?['AssignedAdvisor']` |
| Notes | `outputs('Get_First_Result')?['Notes']` |

7. Name flow: `Search Client Policies` → **Save**

### 3.3 Create Topic: "Lookup by Client Name"

**Trigger phrases**: `Find client`, `Look up policy`, `Search for client`, `Client information`, `Policy details`

**Build the flow** (see full diagram in Architecture_and_Flow_Diagrams.md):

1. Message: "Sure! Let me search for the client. How would you like to search?"
2. Ask: "Search by:" → Multiple choice (Client Name / Policy Number) → `Topic.SearchMethod`
3. **Condition**: SearchMethod = "Client Name"
   - Ask: "What is the client's name?" → User's entire response → `Topic.ClientName`
   - Set: `Global.SearchFieldValue` = "Title"
   - Call Action: Search Client Policies (SearchField ← Global.SearchFieldValue, SearchValue ← Topic.ClientName)
   - Condition: PolicyNumber is not blank → show results / show "not found"
4. **Condition**: SearchMethod = "Policy Number"
   - Ask: "What is the policy number?" → User's entire response → `Topic.PolicyNumber1`
   - Set: `Global.SearchFieldValue` = "PolicyNumber"
   - Call Action: Search Client Policies (SearchField ← Global.SearchFieldValue, SearchValue ← Topic.PolicyNumber1)
   - Condition: ReturnedClientName is not blank → show results / show "not found"
5. End both branches with "End current topic"
6. **Save**

---

## Phase 4: Build Agent 3 — Product Advisor Agent

### 4.1 Create the Agent

1. Name: `Product Advisor Agent`
2. Description: `Provides product information, comparisons, and recommendations for Contoso financial advisors.`
3. Set Instructions — paste:

```
You are the Product Advisor Agent. You help Contoso financial advisors understand products and make recommendations to their clients.

You can:
- Answer detailed questions about Life, Health, Travel, and Critical Illness insurance products
- Compare products side by side
- Recommend products based on a client's age, family situation, and needs
- Explain underwriting requirements and exclusions

Rules:
- Base all answers on the knowledge sources provided to you. Do not make up product details.
- When recommending products, always explain the reasoning.
- Present comparisons in a table format for easy reading.
- Always note that final product suitability is determined by the advisor and the client.
- Flag any underwriting concerns (e.g., pre-existing conditions, age limits).
```

### 4.2 Add Knowledge (SharePoint)

1. **Knowledge** → **+ Add knowledge** → **SharePoint**
2. Select `ProductGuides` → `Contoso_Product_FAQ.docx`, `Underwriting_Guidelines.docx`
3. **Add** → wait for indexing

### 4.3 Create Topic: "Product Recommendation"

**Trigger phrases**: `Recommend a product`, `What should I suggest`, `Best product for`, `Product recommendation`, `What plan fits`

**Build the flow**:

1. Message: "I'll help you find the right product for your client. Let me gather some details."
2. Ask: "What is the client's age?" → Number → `Topic.ClientAge`
3. Ask: "What is the client's primary concern?" → Multiple choice (Income protection for family / Medical expense coverage / Critical illness lump-sum benefit / Travel protection / Retirement planning) → `Topic.PrimaryConcern`
4. Ask: "Does the client have any pre-existing medical conditions?" → Multiple choice (Yes / No / Not sure) → `Topic.PreExisting`
5. Ask: "What is the client's approximate monthly budget for insurance?" → Multiple choice (Under HKD 500 / HKD 500-1500 / HKD 1500-3000 / Above HKD 3000) → `Topic.Budget`
6. **Create generative answers** node — Input prompt:
   ```
   Based on the following client profile, recommend the most suitable Contoso product(s) with reasoning:
   - Age: {Topic.ClientAge}
   - Primary concern: {Topic.PrimaryConcern}
   - Pre-existing conditions: {Topic.PreExisting}
   - Budget: {Topic.Budget}
   Use the product knowledge base to provide accurate details.
   ```
   → Link to SharePoint knowledge docs in the node
7. Message: "Would you like me to compare specific plans side by side, or would you like to schedule a meeting with this client to discuss?"
8. **Save**

---

## Phase 5: Build Agent 4 — Scheduling Assistant

### 5.1 Create the Agent

1. Name: `Scheduling Assistant`
2. Description: `Schedules client meetings via Outlook and sends follow-up emails for Contoso advisors.`
3. Set Instructions — paste:

```
You are the Scheduling Assistant for Contoso financial advisors.

You can:
- Schedule meetings with clients via Outlook calendar
- Send follow-up emails to clients after consultations
- Log interactions to the system

Rules:
- Always confirm the meeting details (date, time, attendees, subject) before creating the event.
- Default meeting duration is 30 minutes unless the advisor specifies otherwise.
- Include a professional Contoso email signature in all outgoing emails.
- Time zone is Hong Kong (UTC+8).
- Never send emails without the advisor's explicit confirmation.
```

### 5.2 Create Power Automate Flow: "Schedule Client Meeting"

1. **Trigger inputs** (5 Text inputs): `Subject`, `StartDateTime`, `Duration`, `AttendeeEmail`, `Body`
2. **Compose** — "Calculate End Time" (Expression):
   ```
   addMinutes(triggerBody()?['text_1'], if(equals(triggerBody()?['text_2'], '30 minutes'), 30, if(equals(triggerBody()?['text_2'], '45 minutes'), 45, 60)))
   ```
3. **Office 365 Outlook — Create event (V4)**:
   - Calendar id: Calendar
   - Subject: `Subject` (dynamic)
   - Start time: `StartDateTime` (dynamic)
   - End time: `Calculate End Time` output (dynamic)
   - Time zone: UTC+08:00 Hong Kong
   - Required attendees: `AttendeeEmail` (dynamic)
   - Body: `Body` (dynamic)
4. **Return**: `Confirmation` (Text) = `Meeting created successfully`
5. Name: `Schedule Client Meeting` → **Save**

### 5.3 Create Power Automate Flow: "Send Follow-Up Email"

1. **Trigger inputs** (3 Text inputs): `ToEmail`, `Subject`, `EmailBody`
2. **Office 365 Outlook — Send an email (V2)**: To = `ToEmail`, Subject = `Subject`, Body = `EmailBody`
3. **Return**: `Confirmation` (Text) = `Email sent successfully`
4. Name: `Send Follow-Up Email` → **Save**

### 5.4 Create Power Automate Flow: "Log Interaction to Dataverse"

1. **Trigger inputs** (4 Text inputs): `ClientName`, `InteractionType`, `Summary`, `FollowUpDate`
2. **Compose** — "Map InteractionType to Choice Value" (Expression):
   ```
   if(equals(triggerBody()?['text_1'],'Call'),890950000,if(equals(triggerBody()?['text_1'],'Meeting'),890950001,if(equals(triggerBody()?['text_1'],'Email'),890950002,if(equals(triggerBody()?['text_1'],'Chat'),890950003,890950001))))
   ```
   > ⚠️ Replace numeric values with your actual Dataverse choice values from Phase 1.3
3. **Dataverse — Add a new row**: Table = `InteractionLog`
   - ClientName ← dynamic `ClientName`
   - InteractionType ← click "Enter custom value" → dynamic `Map InteractionType` output
   - Summary ← dynamic `Summary`
   - FollowUpRequired = Yes
   - FollowUpDate ← dynamic `FollowUpDate`
4. **Return**: `Confirmation` (Text) = `Interaction logged`
5. Name: `Log Interaction to Dataverse` → **Save**

### 5.5 Create Topic: "Schedule Client Meeting"

**Trigger phrases**: `Schedule a meeting`, `Book an appointment`, `Set up a call`, `Arrange meeting with client`, `Calendar`

**Build the flow** (21 steps — see full detail in Advanced Demo Guide Section 3.4.5):

| Step | Node | Variable |
|------|------|----------|
| 4 | Message: "Let's get that meeting scheduled!" | — |
| 5 | Ask: "What is the client's full name?" → User's entire response | `Topic.ClientName` (string) |
| 6 | Ask: "What is the client's email address?" → Email | `Topic.ClientEmail` (string) |
| 7 | Ask: "What date and time?" → Date and time | `Topic.MeetingDateTime` (datetime) |
| 8 | Ask: "How long?" → Multiple choice (30 min / 45 min / 1 hour) | `Topic.Duration` (choice) |
| 9 | Ask: "What is the purpose?" → Multiple choice (5 options) | `Topic.MeetingPurpose` (choice) |
| 10 | Message: summary with all variables via {x} | — |
| 11 | Ask: "Confirm?" → Multiple choice (Yes, create it / No) | `Topic.Confirmation` (choice) |
| 12 | Condition: Confirmation = "Yes, create it" | — |
| 13a | Set Variable: `MeetingSubject` = `Concatenate("Contoso - ", Text(Topic.MeetingPurpose), " with ", Topic.ClientName)` | `Topic.MeetingSubject` (string) |
| 13b | Set Variable: `MeetingBody` = `Concatenate("Dear ", Topic.ClientName, ", This meeting has been scheduled to discuss: ", Text(Topic.MeetingPurpose), ". Best regards, Contoso Financial Advisory Team")` | `Topic.MeetingBody` (string) |
| 14a | Set Variable: `MeetingDateTimeText` = `Text(Topic.MeetingDateTime)` | `Topic.MeetingDateTimeText` (string) |
| 14b | Set Variable: `DurationText` = `Text(Topic.Duration)` | `Topic.DurationText` (string) |
| 15 | Call Action: Schedule Client Meeting flow (map 5 inputs) | `Topic.FlowConfirmation` (string) for output |
| 16 | Message: "✅ Meeting created! Invitation sent to {Topic.ClientEmail}." | — |
| 17 | Ask: "Log this interaction?" → Multiple choice (Yes / No) | `Topic.LogInteraction` (choice) |
| 18 | Condition: LogInteraction = "Yes" | — |
| 19 | Call Action: Log Interaction to Dataverse (ClientName, "Meeting", MeetingPurpose, MeetingDateTime) | — |
| 20 | Message: "Interaction logged successfully! ✅" | — |
| 21 | End all branches with "End current topic" | — |

> ⚠️ **Key rules for this topic:**
> - Use `Text()` wrapper on `choice` and `datetime` variables in `Concatenate()` formulas
> - Create separate string variables for the flow (MeetingDateTimeText, DurationText) to avoid type errors
> - Do NOT map flow output `Confirmation` to `Topic.Confirmation` (type mismatch — create `FlowConfirmation` instead)

**Save** the topic.

### 5.6 Create Topic: "Send Follow-Up Email"

**Trigger phrases**: `Send a follow-up`, `Email the client`, `Send an email`, `Follow up with client`

| Step | Node | Variable |
|------|------|----------|
| 3 | Ask: "What is the client's email address?" → Email | `Topic.ClientEmail` (string) |
| 4 | Ask: "What is the client's name?" → User's entire response | `Topic.ClientName` (string) |
| 5 | Ask: "What would you like the email to say?" → User's entire response | `Topic.EmailContent` (string) |
| 6 | Set Variable: `EmailSubject` = `Concatenate("Follow-up from Contoso - ", Topic.ClientName)` | `Topic.EmailSubject` (string) |
| 7 | Call an action → **Create a prompt** (AI Builder Custom Prompt) | — |
| — | Prompt instructions: `Compose the body of a professional follow-up email from a Contoso financial advisor to their client. Client name: {ClientName}. The advisor's key points to include: {EmailContent}.` | — |
| — | Add Text inputs: `ClientName`, `EmailContent` → map to Topic variables | — |
| — | Output: keep default `predictionOutput` (record type) | `Topic.predictionOutput` (record) |
| 8 | Message: "Here's the draft email: {Topic.predictionOutput.text} Confirm send?" | — |
| 9 | Ask: "Confirm send?" → Multiple choice (Yes, send it / Edit and resend / Cancel) | `Topic.EmailConfirm` (choice) |
| 10 | Condition: EmailConfirm = "Yes, send it" | — |
| 11 | Call Action: Send Follow-Up Email (ToEmail ← ClientEmail, Subject ← EmailSubject, EmailBody ← predictionOutput.text) | `Topic.EmailFlowConfirmation` (string) |
| 12 | Message: "✅ Email sent to {Topic.ClientEmail}!" | — |
| 13 | Else: Message "No problem. The email was not sent." → End topic | — |

**Save** the topic.

---

## Phase 6: Connect Sub-Agents to Hub

### 6.1 Register Sub-Agents

1. Open **Contoso Advisor Hub** → **Agents** (left nav) → **+ Add agent**
2. Add each:

| Agent | Description for Orchestrator |
|-------|------------------------------|
| Policy Lookup Agent | Use this agent when the advisor wants to find client policy details, search by client name or policy number, or check for expiring policies. |
| Product Advisor Agent | Use this agent when the advisor asks about product details, comparisons, recommendations, or underwriting guidelines. |
| Scheduling Assistant | Use this agent when the advisor wants to schedule a meeting, send an email, or log an interaction with a client. |

### 6.2 Enable Slot-Filling (Topic Input Variables)

For each sub-agent topic, open **Variables** panel and enable **"Receive values from other topics"**:

| Agent | Topic | Variables to Enable | Variables to SKIP |
|-------|-------|--------------------|--------------------|
| Policy Lookup Agent | Lookup by Client Name | `Topic.ClientName`, `Topic.SearchMethod` | — |
| Product Advisor Agent | Product Recommendation | `Topic.ClientAge`, `Topic.PrimaryConcern`, `Topic.PreExisting`, `Topic.Budget` | — |
| Scheduling Assistant | Schedule Client Meeting | `Topic.ClientName`, `Topic.ClientEmail` | ⚠️ `Topic.MeetingPurpose` (choice variable — breaks Concatenate formulas if slot-filled) |
| Scheduling Assistant | Send Follow-Up Email | `Topic.ClientEmail`, `Topic.ClientName`, `Topic.EmailContent` | — |

**Save** each topic after enabling.

---

## Phase 7: Test

### 7.1 Test Each Sub-Agent Individually

Open each sub-agent's **Test your agent** panel and verify:

| Agent | Test Input | Expected |
|-------|-----------|----------|
| Policy Lookup | "Find client David Chan" | Returns policy CTO-2025-00123 details from SharePoint |
| Policy Lookup | "Search by policy number CTO-2025-00456" | Returns Emily Lau's health policy |
| Product Advisor | "Recommend a product" → age 42, critical illness, no pre-existing, HKD 1500-3000 | AI recommendation from knowledge base |
| Scheduling Assistant | "Schedule a meeting" → walk through all fields → confirm | Calendar event appears in Outlook |
| Scheduling Assistant | "Send a follow-up email" → provide details → confirm | Email appears in Outlook Sent Items |

### 7.2 Test End-to-End via Hub Agent

Open **Contoso Advisor Hub** → **Test your agent**:

1. Type: `I need to check the policy for David Chan`
   - Should transfer to Policy Lookup Agent
   - Should skip asking for client name (slot-filled)
   - Should return policy details from SharePoint

2. Type: `I have a new client, age 42, mainly concerned about critical illness protection, no pre-existing conditions, budget around HKD 1500/month`
   - Should transfer to Product Advisor Agent
   - Should extract variables from message
   - Should generate AI recommendation

3. Type: `Let's schedule a meeting with David Chan to discuss his policy renewal`
   - Should transfer to Scheduling Assistant
   - Should slot-fill ClientName
   - Should ask for remaining details (email, date, duration, purpose via buttons)
   - Confirm → calendar event created

4. Type: `Send a follow-up email to david.chan@email.com`
   - Should transfer to Scheduling Assistant
   - Should generate AI email draft
   - Confirm → email sent

### 7.3 Verify in External Systems

- [ ] **Outlook Calendar**: Event with correct subject, time, attendee
- [ ] **Outlook Sent Items**: Follow-up email with AI-generated body
- [ ] **Power Apps → InteractionLog table**: New row with correct data

---

## Phase 8: Publish

1. Open **Contoso Advisor Hub** → click **Publish** (top right)
2. Open each sub-agent → click **Publish**
3. All 4 agents must be published for orchestration to work

---

## Quick Reference: All Flows & Topics

| Agent | Topics | Power Automate Flows |
|-------|--------|---------------------|
| Contoso Advisor Hub | *(uses generative orchestration — no custom topics needed)* | — |
| Policy Lookup Agent | Lookup by Client Name | Search Client Policies |
| Product Advisor Agent | Product Recommendation | — |
| Scheduling Assistant | Schedule Client Meeting | Schedule Client Meeting, Log Interaction to Dataverse |
| Scheduling Assistant | Send Follow-Up Email | Send Follow-Up Email |

---

## Quick Reference: Key Gotchas

| Issue | Solution |
|-------|----------|
| SharePoint column "ClientName" has internal name `Title` | Use `Title` in filter queries, not `ClientName` |
| SharePoint Get Items returns array | Use `first()` in a Compose step before returning values |
| Choice columns return objects | Use `?['Value']` to get text; use `string()` for Currency |
| `choice`/`datetime` variables in `Concatenate()` | Wrap with `Text()`: e.g., `Text(Topic.MeetingPurpose)` |
| Flow output `Confirmation` conflicts with topic variable | Create a separate `FlowConfirmation` variable (string) |
| `predictionOutput` is record type | Access text via `Topic.predictionOutput.text` |
| MeetingPurpose slot-filling breaks subject/body | Do NOT enable "Receive values from other topics" on it |
| Connector auth expires before demo | Test all flows in Power Automate → re-authenticate before demo |
