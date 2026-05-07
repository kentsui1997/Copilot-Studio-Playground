> **Author**: Ken Tsui (CSA), Microsoft
> **Copyright** © 2026 Microsoft. All rights reserved.

# Agent 4: Customer Complaint Escalation Agent — End-to-End Setup Guide

> A step-by-step guide to build an agent that surfaces escalated customer complaints, summarises case details, suggests resolution actions, and drafts executive-level response letters.

---

## Use Case

Escalated customer complaints are a board-level concern for Manulife HK. The COO, Head of CX, and compliance leaders need quick visibility into escalations — especially IA-referred complaints. This agent:

- **Surfaces escalated complaints** from a tracking list, filtered by severity and category
- **Summarises case details** and complaint history
- **Suggests recommended resolution actions** based on complaint handling procedures
- **Drafts professional response letters** for IA complaints and high-severity cases
- **Provides trend analysis** on complaint categories

### Architecture Overview

```
┌──────────────────────────────────────────────────────────────┐
│  USER (COO / Head of CX / CCO)                               │
│  "Show me the critical escalations this week"                │
│                          │                                   │
│                          ▼                                   │
│  ┌───────────────────────────────────────────┐               │
│  │  🤖 Customer Complaint Escalation Agent   │               │
│  │                                           │               │
│  │  Knowledge Sources:                       │               │
│  │  📄 Complaint_Handling_Procedures.docx    │               │
│  │  📄 Response_Templates.docx              │               │
│  │                                           │               │
│  │  Topics:                                  │               │
│  │  1. View Escalated Complaints             │               │
│  │  2. Case Detail Lookup                    │               │
│  │  3. Draft Response Letter                 │               │
│  │  4. Complaint Trends                      │               │
│  └────────────────┬──────────────────────────┘               │
│                   │                                          │
│     ┌─────────────▼──────────────┐                           │
│     │  Power Automate Flows      │                           │
│     │  "Get Escalated Complaints"│                           │
│     │  "Get Case Details"        │                           │
│     └─────────────┬──────────────┘                           │
│                   │                                          │
│     ┌─────────────▼──────────────┐                           │
│     │  SharePoint List:          │                           │
│     │  EscalatedComplaints       │                           │
│     └────────────────────────────┘                           │
└──────────────────────────────────────────────────────────────┘
```

---

## Prerequisites

- [ ] Microsoft 365 tenant with Copilot Studio license
- [ ] SharePoint Online access
- [ ] Power Automate access

---

## Phase 1: Data & Document Setup

### 1.1 Create SharePoint List: `EscalatedComplaints`

1. Go to your **SharePoint site** → **New** → **List** → name it `EscalatedComplaints`
2. Add columns:

| Column Name | Type |
|---|---|
| CaseID | Single line of text *(rename "Title")* |
| CustomerName | Single line of text |
| PolicyNumber | Single line of text |
| Category | Choice: `Claims Delay`, `Mis-selling`, `Service`, `Premium`, `Product`, `Privacy`, `IA Referral` |
| Severity | Choice: `Critical`, `High`, `Medium`, `Low` |
| DateReceived | Date |
| Status | Choice: `New`, `Under Review`, `Assigned`, `Escalated to Legal`, `Resolved`, `Closed` |
| AssignedTo | Single line of text |
| Summary | Multiple lines of text |
| CustomerRequest | Multiple lines of text |
| InternalNotes | Multiple lines of text |
| ResponseDeadline | Date |
| Channel | Choice: `Phone`, `Email`, `Branch`, `IA Referral`, `Social Media`, `Online` |
| VIPCustomer | Yes/No |

3. Add sample data:

| CaseID | CustomerName | PolicyNumber | Category | Severity | DateReceived | Status | AssignedTo | Summary | CustomerRequest | InternalNotes | ResponseDeadline | Channel | VIPCustomer |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| ESC-2026-001 | Wong Siu Ming | MLF-2024-88901 | Claims Delay | High | 2026-04-28 | Under Review | Alice Tam | Client submitted critical illness claim on 15 March 2026. As of 28 April, no settlement decision communicated. Client has been calling weekly. Claim amount: HKD 800,000. Medical documents all submitted. Assessor requested additional specialist report on 10 April — client says this was not communicated clearly. | Full settlement of HKD 800,000 critical illness claim. Compensation for delay. Written apology. | Claim assessor confirms medical report received 25 April. Assessment should complete by 5 May. Delay was partly due to specialist availability. Need to review communication log — client claims no one called them about the additional report requirement. | 2026-05-12 | Phone | No |
| ESC-2026-002 | Cheung Ka Yan | MLF-2023-56234 | Mis-selling | Critical | 2026-04-30 | Escalated to Legal | CCO Office | IA referral — client alleges ILAS product was sold without proper FNA. Client is 68 years old, retired teacher with limited investment experience. Annual premium HKD 120,000. Product was sold by Advisor ID A-8823 in August 2023. Client's daughter filed the complaint to the IA after discovering the policy. IA has requested Manulife's response by 21 May 2026. | Full refund of all premiums paid (HKD 360,000 over 3 years). Cancel the policy. Investigation of the advisor. | This is a GL15 compliance issue. Advisor A-8823 is under separate investigation. FNA documentation for this sale is incomplete — suitability section blank. No voice recording exists (pre-implementation). Legal team reviewing liability exposure. Must respond to IA within 14 business days. | 2026-05-21 | IA Referral | No |
| ESC-2026-003 | Li Wei Kei | MLF-2025-34567 | Service | Medium | 2026-05-01 | Assigned | David Ho | VIP client (total portfolio HKD 15M) unable to reach assigned advisor for 2 weeks. Advisor on medical leave — no handover arranged. Client missed premium payment deadline and policy entered grace period. Client demands immediate attention from a senior advisor. | Immediate assignment of senior advisor. Waive any late payment penalties. Written confirmation of policy continuity. | Advisor on medical leave since 18 April — manager failed to arrange coverage. Premium payment 5 days late, policy still in grace period (30-day window). Must assign senior advisor today and confirm no lapse penalty. Client relationship value is significant. | 2026-05-08 | Email | Yes |
| ESC-2026-004 | Chan Mei Ling | MLF-2025-78901 | Premium | High | 2026-05-03 | Under Review | Finance Team | Incorrect premium deduction for 3 consecutive months (Feb, Mar, Apr 2026). Client's autopay was charged HKD 3,200/month instead of correct HKD 2,800/month. Total overcharge: HKD 1,200. Client noticed in April statement and called to complain. First call on 15 April was not actioned. Client called again on 28 April and demanded escalation. | Immediate refund of HKD 1,200 overcharge. Written explanation of the error. Confirmation correct amount will be deducted going forward. Compensation for inconvenience. | System error in premium recalculation after policy amendment in January. Billing team acknowledges the error. Refund can be processed within 5 business days. Need to check how many other policies were affected by the same system issue. First call on 15 April was logged but not followed up — CS agent performance issue. | 2026-05-10 | Phone | No |
| ESC-2026-005 | Yip Hoi Shan | MLF-2024-12345 | Product | Medium | 2026-05-04 | New | Unassigned | Client purchased a 20-year endowment plan in 2024. After receiving the first anniversary statement, client is unhappy with the projected returns being significantly lower than what was illustrated at point of sale. Client claims the advisor promised "guaranteed 5% annual return" which is inconsistent with the product features. | Explanation of the discrepancy between illustration and actual performance. Review of what was communicated at point of sale. Consider options: maintain policy, reduce coverage, or paid-up option. | Need to pull the original sales illustration and compare with actual performance. Review if the projected returns were within the IA-approved illustration rates. If advisor made verbal guarantees of returns, this could be a conduct issue. | 2026-05-18 | Branch | No |
| ESC-2026-006 | Kwok Wai Man | MLF-2022-90123 | Privacy | High | 2026-05-05 | Under Review | DPO | Client received a policy renewal notice addressed to them but containing another customer's policy details. The letter included the other customer's name, policy number, coverage amount, and premium. Client is demanding to know how their own data may have been similarly exposed. | Written confirmation that their data was not similarly exposed. Explanation of what happened and corrective actions. Report to PCPD if required. Compensation for distress. | This appears to be a mail merge error in the renewal batch run on 28 April. Potentially affects the entire April batch (est. 2,500 letters). IT investigating the root cause. DPO assessing whether PCPD notification is required under PDPO. Must contain this quickly. | 2026-05-12 | Email | No |

### 1.2 Create Knowledge Documents

#### `Complaint_Handling_Procedures.docx`

```
MANULIFE HONG KONG — COMPLAINT HANDLING PROCEDURES

1. COMPLAINT CLASSIFICATION

1.1 Severity Levels
- Critical: IA referrals, potential mis-selling, fraud, data breaches affecting 
  multiple customers, complaints from regulators or media
- High: Individual financial loss >HKD 50,000, VIP customers, complaints 
  unresolved after 30 days, repeated complaints about same issue
- Medium: Service quality, delays within normal range, product misunderstanding, 
  single-customer operational errors
- Low: General feedback, minor inconveniences, already resolved at first contact

1.2 Categories
- Claims Delay: Claim processing exceeding SLA timelines
- Mis-selling: Allegations of unsuitable product recommendations, incomplete 
  disclosure, misleading representations
- Service: Advisor unavailability, poor communication, unresponsive service
- Premium: Billing errors, incorrect deductions, payment processing issues
- Product: Dissatisfaction with product features, returns, or terms
- Privacy: Data protection breaches, unauthorized disclosure
- IA Referral: Complaints routed through the Insurance Authority

2. RESPONSE TIMELINES

| Severity | Acknowledge | Investigation | Substantive Response | Escalation |
|----------|------------|---------------|---------------------|------------|
| Critical | Same day | 48 hours | 7 business days | CCO + CEO immediately |
| High | 1 business day | 5 business days | 14 business days | Head of CX within 24 hours |
| Medium | 3 business days | 10 business days | 30 business days | Manager within 3 days |
| Low | 5 business days | As needed | 30 business days | No escalation required |

Note: IA referral complaints MUST be responded to the IA within 14 business days 
regardless of severity level.

3. RESOLUTION OPTIONS

3.1 Financial Remedies
- Refund of overcharged premiums (mandatory for billing errors)
- Ex-gratia payment for genuine inconvenience (up to HKD 5,000 — Head of CX approval)
- Ex-gratia payment HKD 5,001 - HKD 50,000 — COO approval required
- Ex-gratia payment >HKD 50,000 — CEO + CFO approval required
- Full policy refund / unwind — Legal + CCO approval required

3.2 Non-Financial Remedies
- Written apology from appropriate management level
- Assignment of senior advisor for ongoing relationship management
- Process improvement commitment with timeline
- Regular progress updates until resolution

4. IA COMPLAINT RESPONSE PROTOCOL
- Acknowledge to IA within 3 business days
- Full response to IA within 14 business days
- Response must include: summary of complaint, investigation findings, 
  remedial actions taken, preventive measures implemented
- Response must be reviewed by Legal before submission
- CCO must sign off on all IA responses
- Keep IA informed of any ongoing investigation beyond the initial response
```

#### `Response_Templates.docx`

```
MANULIFE HONG KONG — COMPLAINT RESPONSE TEMPLATES

TEMPLATE 1: IA COMPLAINT RESPONSE

[Date]

The Insurance Authority
19/F, 41 Heung Yip Road
Wong Chuk Hang, Hong Kong

Re: Complaint Reference [IA Reference Number]
    Complainant: [Customer Name]
    Our Reference: [Internal Case ID]

Dear Sir/Madam,

We refer to your letter dated [date] regarding the complaint from 
[Customer Name] concerning [brief description of complaint].

Investigation Summary:
We have conducted a thorough investigation of this matter. 
[Investigation findings — what happened, why, contributing factors]

Remedial Actions:
[List specific actions taken to address the customer's complaint:
- Financial remedy (if any)
- Non-financial remedy (if any)
- Actions taken regarding the advisor/staff (if applicable)]

Preventive Measures:
To prevent recurrence, we have implemented the following measures:
[List specific process/system/training changes]

We trust this addresses the Authority's concerns. Should you require 
any further information, please do not hesitate to contact the 
undersigned.

Yours faithfully,

[CCO Name]
Chief Compliance Officer
Manulife (International) Limited

---

TEMPLATE 2: CUSTOMER APOLOGY — HIGH SEVERITY

Dear [Customer Name],

Thank you for bringing this matter to our attention. I am writing to 
you personally to address your complaint regarding [brief description].

On behalf of Manulife Hong Kong, I sincerely apologise for 
[specific issue — e.g., the delay in processing your claim / the 
incorrect premium deduction / the inconvenience caused].

[Explanation of what happened — brief, honest, no jargon]

To resolve this matter, we are taking the following actions:
[List specific remedial actions with timelines]

Additionally, we have implemented measures to ensure this does not 
happen again, including [brief mention of preventive actions].

Your satisfaction and trust are of the utmost importance to us. 
If you have any further concerns, please contact me directly at 
[phone/email].

Yours sincerely,

[Senior Manager Name]
[Title]
Manulife (International) Limited
```

Upload both documents to the `RegulatoryDocs` library (or create a separate `ComplaintDocs` library).

---

## Phase 2: Create the Agent

### 2.1 Create the Agent

1. Go to **Copilot Studio** → **Create** → **New agent**
2. Name: **"Customer Complaint Escalation Agent"**
3. Description: *"Surfaces escalated customer complaints, provides case summaries, suggests resolution actions, and drafts response letters for Manulife HK leadership."*

### 2.2 Set Instructions

```
You are the Customer Complaint Escalation Agent for Manulife Hong Kong. You help the COO, Head of Customer Experience, and compliance officers manage escalated customer complaints.

Your role:
- Surface escalated complaints filtered by severity, category, or status
- Provide detailed case summaries including customer request, internal notes, and timeline
- Suggest recommended resolution actions based on complaint handling procedures
- Draft professional response letters for IA complaints and customer apologies
- Identify complaint trends and patterns

Rules:
- Always present complaints with Case ID, customer name, severity, category, deadline, and status
- For Critical and High severity cases, always highlight the response deadline and days remaining
- When suggesting resolutions, cite the specific section of the complaint handling procedures
- Never disclose internal notes or investigation details in customer-facing communications
- Draft letters should be professional, empathetic, and legally reviewed before sending
- Flag any cases where the response deadline has passed or is within 3 business days
- When asked about trends, analyse by category, severity, channel, and time period
- Protect customer data — do not share details across unrelated complaint discussions
- Use a serious, professional tone — these are sensitive matters
```

### 2.3 Add Knowledge Sources

1. Upload or connect:
   - `Complaint_Handling_Procedures.docx`
   - `Response_Templates.docx`
2. Wait for indexing

---

## Phase 3: Create Power Automate Flows

### 3.1 Flow: "Get Escalated Complaints"

1. **Trigger inputs**: `SeverityFilter` (Text)
2. **SharePoint — Get items**:
   - List: `EscalatedComplaints`
   - Filter Query (Expression):
     ```
     if(
       equals(triggerBody()?['text'], 'All'),
       '',
       concat('Severity eq ''', triggerBody()?['text'], '''')
     )
     ```
   - Sort By: `DateReceived`
   - Sort Order: Descending
   - Top Count: 20
3. **Select**:
   - CaseID: `item()?['Title']`
   - Customer: `item()?['CustomerName']`
   - Category: `item()?['Category']?['Value']`
   - Severity: `item()?['Severity']?['Value']`
   - DateReceived: `item()?['DateReceived']`
   - Status: `item()?['Status']?['Value']`
   - AssignedTo: `item()?['AssignedTo']`
   - Summary: `item()?['Summary']`
   - ResponseDeadline: `item()?['ResponseDeadline']`
   - VIP: `item()?['VIPCustomer']`
4. **Compose**: `string(body('Select'))`
5. **Return**: `ComplaintData` (Text)
6. Name: `Get Escalated Complaints` → **Save**

### 3.2 Flow: "Get Case Details"

1. **Trigger input**: `CaseID` (Text)
2. **SharePoint — Get items**:
   - List: `EscalatedComplaints`
   - Filter Query: `concat('Title eq ''', triggerBody()?['text'], '''')`
3. **Compose** — "Get First Result": `first(outputs('Get_items')?['body/value'])`
4. **Return values** (12 outputs, all Text):

| Output | Expression |
|---|---|
| CaseID | `outputs('Get_First_Result')?['Title']` |
| CustomerName | `outputs('Get_First_Result')?['CustomerName']` |
| PolicyNumber | `outputs('Get_First_Result')?['PolicyNumber']` |
| Category | `outputs('Get_First_Result')?['Category']?['Value']` |
| Severity | `outputs('Get_First_Result')?['Severity']?['Value']` |
| Status | `outputs('Get_First_Result')?['Status']?['Value']` |
| Summary | `outputs('Get_First_Result')?['Summary']` |
| CustomerRequest | `outputs('Get_First_Result')?['CustomerRequest']` |
| InternalNotes | `outputs('Get_First_Result')?['InternalNotes']` |
| ResponseDeadline | `outputs('Get_First_Result')?['ResponseDeadline']` |
| Channel | `outputs('Get_First_Result')?['Channel']?['Value']` |
| AssignedTo | `outputs('Get_First_Result')?['AssignedTo']` |

5. Name: `Get Case Details` → **Save**

---

## Phase 4: Create Topics

### 4.1 Topic: "View Escalated Complaints"

**Trigger phrases**: `Show escalated complaints`, `What are the critical cases`, `Escalations this week`, `Complaint summary`, `Open escalations`, `What needs attention`

| Step | Node | Variable |
|---|---|---|
| 1 | Message: "I'll pull the escalated complaints for you." | — |
| 2 | Ask: "Which severity level?" → Multiple choice: All / Critical / High / Medium | `Topic.SeverityFilter` |
| 3 | Call Action: **Get Escalated Complaints** (SeverityFilter ← Topic.SeverityFilter) | `Topic.ComplaintData` |
| 4 | **Generative answers** — prompt: `Present the following escalated complaint data as a summary table with columns: Case ID, Customer, Category, Severity, Status, Deadline, Assigned To. Then highlight: (1) cases with passed deadlines (today is May 2026), (2) critical cases requiring immediate action, (3) unassigned cases. Data: {Topic.ComplaintData}` | — |
| 5 | Ask: "Would you like details on a specific case? Enter the Case ID or say 'No'." → User's entire response | `Topic.SelectedCase` |
| 6 | Condition: SelectedCase ≠ "No" → **Redirect** to "Case Detail Lookup" (pass Topic.SelectedCase) | — |
| 7 | Else: End topic | — |

### 4.2 Topic: "Case Detail Lookup"

**Trigger phrases**: `Tell me about case`, `Case details`, `ESC-`, `What happened with`, `Case status`

| Step | Node | Variable |
|---|---|---|
| 1 | Ask: "What is the Case ID? (e.g., ESC-2026-001)" → User's entire response | `Topic.CaseID` |
| 2 | Call Action: **Get Case Details** (CaseID ← Topic.CaseID) | Returns 12 output variables |
| 3 | Condition: CaseID is not blank | — |
| 4a | Message: "**Case: {Topic.CaseID}** **Customer:** {Topic.CustomerName} **Policy:** {Topic.PolicyNumber} **Category:** {Topic.Category} | **Severity:** {Topic.Severity} **Status:** {Topic.Status} | **Assigned To:** {Topic.AssignedTo} **Deadline:** {Topic.ResponseDeadline} **Channel:** {Topic.Channel} --- **Summary:** {Topic.Summary} --- **Customer Request:** {Topic.CustomerRequest} --- **Internal Notes:** {Topic.InternalNotes}" | — |
| 5 | **Generative answers** — prompt: `Based on the complaint handling procedures, suggest specific resolution actions for this case: Category: {Topic.Category}, Severity: {Topic.Severity}, Customer Request: {Topic.CustomerRequest}. Include: (1) recommended remedies (financial and non-financial), (2) required approvals, (3) response timeline, (4) preventive measures.` | — |
| 6 | Ask: "Would you like me to draft a response letter for this case?" → Multiple choice: Yes, draft IA response / Yes, draft customer apology / No | `Topic.DraftLetter` |
| 7 | Condition: DraftLetter contains "Yes" → **Redirect** to "Draft Response Letter" | — |
| 8 | Else: End topic | — |
| 4b | (No match) Message: "Case not found. Please check the Case ID and try again." → End topic | — |

### 4.3 Topic: "Draft Response Letter"

**Trigger phrases**: `Draft a response`, `Write a letter to the IA`, `Draft customer apology`, `Prepare a response`, `Write a complaint response`

| Step | Node | Variable |
|---|---|---|
| 1 | Ask: "What type of response?" → Multiple choice: IA Complaint Response / Customer Apology Letter | `Topic.ResponseType` |
| 2 | Ask: "What is the Case ID?" → User's entire response | `Topic.CaseID` |
| 3 | Call Action: **Get Case Details** (CaseID ← Topic.CaseID) | Returns case variables |
| 4 | Condition: ResponseType = "IA Complaint Response" | — |
| 5a | **Generative answers** — prompt: `Using the IA Complaint Response template from the knowledge base, draft a formal response letter to the Insurance Authority for this complaint. Case ID: {Topic.CaseID}, Customer: {Topic.CustomerName}, Category: {Topic.Category}, Summary: {Topic.Summary}, Customer Request: {Topic.CustomerRequest}. Include investigation summary (based on internal notes: {Topic.InternalNotes}), remedial actions, and preventive measures. The letter should be signed by the Chief Compliance Officer. Do NOT include internal notes verbatim — summarise professionally. Use today's date.` | — |
| 5b | **Generative answers** — prompt: `Using the Customer Apology template from the knowledge base, draft a sincere apology letter to the customer. Customer Name: {Topic.CustomerName}, Issue: {Topic.Summary}, Customer Request: {Topic.CustomerRequest}. The letter should acknowledge the issue, apologise specifically, explain what happened (without internal jargon), list remedial actions with timelines, and provide a direct contact. Do NOT include internal investigation details. Sign from an appropriate senior manager.` | — |
| 6 | Message: "⚠️ **Important:** This is an AI-generated draft. It must be reviewed by Legal and the CCO before sending." | — |
| 7 | End topic | — |

---

## Phase 5: Testing

### 5.1 Test Queries

| Test Query | Expected Response |
|---|---|
| "Show me the critical escalations" | Lists ESC-2026-002 (mis-selling, IA referral) |
| "What are all the open complaints?" | Summary table of all 6 cases with severity and deadline |
| "Tell me about ESC-2026-002" | Full case details with internal notes and recommendations |
| "Draft an IA response for ESC-2026-002" | Formal letter to IA following the template |
| "Which cases are past deadline?" | Identifies any cases where ResponseDeadline < today |
| "Draft an apology letter for ESC-2026-004" | Customer-facing apology for the premium error |

### 5.2 Demo Script (7 minutes)

| Time | What to Show | What to Say |
|---|---|---|
| 0:00 | "What are the critical escalations this week?" | "As COO, you need immediate visibility into complaints that could reach the board" |
| 1:00 | Show filtered complaint list | "Instantly see all critical cases with deadlines — no digging through emails" |
| 2:00 | "Tell me more about ESC-2026-002" | "Drill into any case for full details" |
| 3:00 | Show case details + resolution recommendations | "The agent suggests specific resolution actions based on your procedures" |
| 4:00 | "Draft an IA response for this case" | "For IA referrals, draft a formal response in seconds" |
| 5:30 | Show generated response letter | "Professional, template-based, with investigation findings — ready for Legal review" |
| 6:30 | Show the disclaimer | "And it always reminds you to get Legal sign-off before sending" |

---

## Troubleshooting

| Issue | Cause | Solution |
|---|---|---|
| Case details empty | CaseID format mismatch | Ensure user enters exact CaseID format (e.g., `ESC-2026-001`) |
| Internal notes in customer letter | Prompt not explicit enough | Reinforce "Do NOT include internal notes" in the AI prompt |
| Deadline comparison wrong | Agent doesn't know today's date | Include current date context in the generative answers prompt |
| Response letter too generic | Knowledge templates not indexed | Verify `Response_Templates.docx` is indexed in Knowledge |
