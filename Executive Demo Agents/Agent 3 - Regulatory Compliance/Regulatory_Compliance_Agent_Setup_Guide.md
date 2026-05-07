> **Author**: Ken Tsui (CSA), Microsoft
> **Copyright** © 2026 Microsoft. All rights reserved.

# Agent 3: Regulatory & Compliance Agent — End-to-End Setup Guide

> A step-by-step guide to build an agent that helps executives and compliance officers quickly answer regulatory questions, track compliance deadlines, and cross-reference company policies against HK Insurance Authority requirements.

---

## Use Case

Insurance regulation in Hong Kong is complex and constantly evolving. The HK Insurance Authority (IA) issues guidelines, circulars, and codes of conduct that Manulife must comply with. This agent:

- **Answers questions** about IA regulations, guidelines, and circulars
- **Explains compliance requirements** in plain, executive-friendly language
- **Tracks upcoming regulatory deadlines** and flags overdue items
- **Cross-references** company policies against regulatory requirements

### Architecture Overview

```
┌──────────────────────────────────────────────────────────────┐
│  USER (CLO / CCO / CRO / CEO)                                │
│  "What are the ILAS selling requirements under GL15?"        │
│                          │                                   │
│                          ▼                                   │
│  ┌───────────────────────────────────────────┐               │
│  │  🤖 Regulatory & Compliance Agent         │               │
│  │                                           │               │
│  │  Knowledge Sources:                       │               │
│  │  📄 IA_Guideline_GL15_ILAS.docx          │               │
│  │  📄 IA_Guideline_GL8_CoolingOff.docx     │               │
│  │  📄 AML_Guidelines.docx                  │               │
│  │  📄 Manulife_Compliance_Manual.docx      │               │
│  │  📊 SharePoint List: RegulatoryTracker    │               │
│  │                                           │               │
│  │  Topics:                                  │               │
│  │  1. Regulatory Q&A (AI / Knowledge)       │               │
│  │  2. Compliance Deadline Tracker           │               │
│  │  3. Generate Compliance Summary           │               │
│  └────────────────┬──────────────────────────┘               │
│                   │                                          │
│         ┌─────────▼──────────┐                               │
│         │  Power Automate    │                               │
│         │  "Get Regulatory   │                               │
│         │   Deadlines"       │                               │
│         └─────────┬──────────┘                               │
│                   │                                          │
│         ┌─────────▼──────────┐                               │
│         │  SharePoint List   │                               │
│         │  RegulatoryTracker │                               │
│         └────────────────────┘                               │
└──────────────────────────────────────────────────────────────┘
```

---

## Prerequisites

- [ ] Microsoft 365 tenant with Copilot Studio license
- [ ] SharePoint Online access
- [ ] Power Automate access

---

## Phase 1: Data & Document Setup

### 1.1 Create SharePoint Document Library: `RegulatoryDocs`

1. Go to your **SharePoint site** → **New** → **Document library** → name it `RegulatoryDocs`
2. Create and upload the following sample documents:

#### Sample Document 1: `IA_Guideline_GL15_ILAS.docx`

```
HONG KONG INSURANCE AUTHORITY
GUIDELINE ON INVESTMENT-LINKED ASSURANCE SCHEMES (ILAS) — GL15

(Sample / Simplified for Demo Purposes)

1. SCOPE
This guideline applies to all authorized insurers offering Investment-Linked 
Assurance Schemes (ILAS) in Hong Kong. It sets out the standards for product 
design, disclosure, selling practices, and ongoing obligations.

2. KEY REQUIREMENTS

2.1 Product Design
- ILAS products must provide genuine insurance coverage, not merely an 
  investment wrapper
- Minimum sum assured must be at least 105% of the total premiums paid 
  at any time during the policy term
- Surrender charge period must not exceed 5 years for regular premium 
  products (reduced from previous 15-18 years)
- Total fees and charges must be clearly disclosed in a standardized 
  Fee Table format

2.2 Disclosure Requirements
- Product Key Facts Statement (KFS) must be provided to the customer 
  before sale
- Illustration must show projected benefits under pessimistic, base, 
  and optimistic scenarios
- All fees (fund management charge, policy admin fee, surrender charge, 
  premium allocation charge) must be itemized
- Break-even period must be clearly stated

2.3 Selling Practices
- Financial Needs Analysis (FNA) must be conducted before recommending 
  any ILAS product
- Suitability assessment must document why the ILAS product is appropriate 
  for the customer's risk tolerance, investment horizon, and financial goals
- Customers aged 65+ require enhanced suitability assessment and cooling-off 
  period of 30 calendar days (instead of standard 21 days)
- Vulnerable customers (low income, limited financial literacy) require 
  supervisor sign-off before sale
- Voice recording of sales process is recommended for ILAS sales exceeding 
  HKD 500,000 annual premium

2.4 Post-Sale Obligations
- Annual benefit statement must be sent within 60 days of policy anniversary
- Fund performance reports must be made available quarterly
- Policyholders must be notified of any fund changes at least 30 days 
  in advance
- Complaint handling: ILAS-related complaints must be escalated to the 
  compliance officer within 2 business days

3. PENALTIES FOR NON-COMPLIANCE
- Regulatory warning for first offence (minor)
- Fine up to HKD 10M for systematic non-compliance
- License conditions or revocation for serious breaches
- Individual accountability — responsible officers may face personal sanctions

4. EFFECTIVE DATE
This guideline took effect on 1 January 2025. All ILAS products sold after 
this date must comply fully.
```

#### Sample Document 2: `IA_Guideline_GL8_CoolingOff.docx`

```
HONG KONG INSURANCE AUTHORITY
GUIDELINE ON COOLING-OFF PERIOD — GL8

(Sample / Simplified for Demo Purposes)

1. PURPOSE
This guideline establishes the minimum cooling-off period during which a 
policyholder may cancel a new insurance policy and receive a full refund 
of premiums paid, less any medical examination costs.

2. STANDARD COOLING-OFF PERIOD
- Duration: 21 calendar days from the date of delivery of the policy 
  or the cooling-off notice, whichever is later
- Applies to: All individual life insurance policies (including ILAS, 
  term life, whole life, endowment, annuity)
- Does NOT apply to: Group policies, general insurance, policies issued 
  under court orders

3. ENHANCED COOLING-OFF PERIOD
- Duration: 30 calendar days
- Applies to: Customers aged 65 and above, ILAS products with annual 
  premium exceeding HKD 100,000
- Rationale: Additional protection for vulnerable or high-value customers

4. REFUND REQUIREMENTS
- Full refund of all premiums paid
- Insurer may deduct: actual medical examination costs incurred
- Insurer must NOT deduct: administrative charges, policy fees, or any 
  other deductions
- Refund must be processed within 14 business days of receiving the 
  cancellation request

5. NOTIFICATION OBLIGATIONS
- A cooling-off notice must be delivered to the policyholder at the time 
  of policy delivery or within 7 calendar days
- The notice must be in the same language as the policy document
- The notice must clearly state: (a) the cooling-off period, (b) how to 
  exercise cancellation, (c) what refund to expect

6. RECORD KEEPING
- Insurers must maintain records of: delivery date of policy, delivery 
  date of cooling-off notice, cancellation requests received, refunds 
  processed
- Records must be retained for 7 years

7. EFFECTIVE DATE
Revised guideline effective 1 July 2024.
```

#### Sample Document 3: `AML_Guidelines.docx`

```
HONG KONG INSURANCE AUTHORITY
ANTI-MONEY LAUNDERING & COUNTER-TERRORIST FINANCING GUIDELINES

(Sample / Simplified for Demo Purposes)

1. SCOPE
Applies to all authorized insurers and licensed insurance intermediaries 
carrying on or advising on long-term insurance business in Hong Kong.

2. CUSTOMER DUE DILIGENCE (CDD)

2.1 Standard CDD
- Verify identity using reliable, independent source documents
- For individuals: HKID card, passport, or travel document
- For companies: Certificate of Incorporation, business registration, 
  beneficial ownership documentation
- Obtain and verify: full name, date of birth, address, nationality, 
  occupation, source of funds

2.2 Enhanced Due Diligence (EDD)
Required for:
- Politically Exposed Persons (PEPs) — domestic and foreign
- High-value policies (single premium > HKD 1M or annual premium > HKD 500K)
- Customers from high-risk jurisdictions (as per FATF grey/black list)
- Unusual transaction patterns
- Non-face-to-face relationships

EDD measures include:
- Senior management approval for establishing the relationship
- Reasonable measures to establish source of wealth and source of funds
- Enhanced ongoing monitoring

2.3 Ongoing Monitoring
- Regular review of customer profile and transactions
- Risk-based approach: high-risk customers reviewed annually, standard 
  customers reviewed every 3 years
- Transaction monitoring for: large single premiums, frequent policy 
  changes, early surrender patterns, third-party payments

3. SUSPICIOUS TRANSACTION REPORTING (STR)
- File STR with the Joint Financial Intelligence Unit (JFIU) within 
  2 business days of suspicion being formed
- Internal escalation: front-line staff must report to MLRO within 24 hours
- No tipping off — do not inform the customer that a report has been filed
- Maintain confidentiality of the STR and related documentation

4. RECORD KEEPING
- CDD records: retained for at least 6 years after the end of the 
  business relationship
- Transaction records: retained for at least 6 years from the date 
  of the transaction
- STR records: retained indefinitely until instructed otherwise by JFIU

5. TRAINING
- All relevant staff must complete AML/CFT training within 3 months 
  of joining
- Annual refresher training required
- Training records must be maintained and made available to the IA 
  upon request

6. PENALTIES
- Non-compliance with AML/CFT obligations is a criminal offence under 
  the Anti-Money Laundering and Counter-Terrorist Financing Ordinance 
  (AMLO)
- Maximum penalty: fine of HKD 1M and imprisonment for 7 years
- The IA may also impose disciplinary sanctions including licence revocation
```

#### Sample Document 4: `Manulife_Compliance_Manual.docx`

```
MANULIFE HONG KONG — INTERNAL COMPLIANCE MANUAL
(Sample / Simplified for Demo Purposes)

1. COMPLIANCE GOVERNANCE STRUCTURE

Chief Compliance Officer (CCO): Reports directly to the CEO and the Board 
Risk & Compliance Committee.

Compliance Team Structure:
- CCO
  ├── Head of Regulatory Compliance (IA & SFC matters)
  ├── Head of AML/CFT (JFIU liaison, STR filing, CDD oversight)
  ├── Head of Conduct Risk (selling practices, complaints, training)
  └── Compliance Advisory (product approval, marketing review, agent conduct)

2. PRODUCT APPROVAL PROCESS
- All new products and material product changes must be reviewed by 
  Compliance before submission to the IA
- Review checklist includes: GL15 compliance (for ILAS), GL8 compliance 
  (cooling-off), disclosure requirements, FNA requirements, suitability 
  requirements
- Turnaround time: 10 business days for standard products, 20 business 
  days for complex/ILAS products

3. ADVISOR CONDUCT STANDARDS
- All advisors must hold valid IA licence
- Annual compliance training completion required (deadline: 31 March each year)
- Breach reporting: advisors must report any compliance breach to their 
  manager within 24 hours; managers must escalate to Compliance within 
  48 hours
- Mis-selling consequences: verbal warning → written warning → suspension 
  → licence revocation recommendation

4. COMPLAINTS HANDLING
- All complaints must be logged in the CMS within 24 hours of receipt
- Response timeline: acknowledge within 3 business days, substantive 
  response within 30 business days
- IA complaints: must be responded to within 14 business days
- Escalation: any complaint involving potential mis-selling, fraud, or 
  regulatory breach must be escalated to the CCO immediately

5. REGULATORY REPORTING
- Annual returns to the IA: due by 30 June each year
- Quarterly compliance reports to the Board: due within 30 days of 
  quarter end
- STR filing: within 2 business days (see AML guidelines)
- Significant event notification to the IA: within 24 hours

6. CURRENT COMPLIANCE GAPS (as of Q1 2026)
- GL15 compliance: 95% — remaining 5% relates to legacy ILAS products 
  requiring fee restructuring by Q3 2026
- AML training completion: 92% — 8% of new joiners still within the 
  3-month grace period
- Voice recording for ILAS sales: 78% implementation — technology 
  upgrade in progress, full rollout by June 2026
- FNA documentation: 88% audit pass rate — target 95% by year-end
```

### 1.2 Create SharePoint List: `RegulatoryTracker`

1. **New** → **List** → name it `RegulatoryTracker`
2. Add columns:

| Column Name | Type |
|---|---|
| RegulatoryItem | Single line of text *(rename "Title")* |
| Source | Choice: `IA Guideline`, `IA Circular`, `AMLO`, `SFC`, `Internal` |
| Reference | Single line of text |
| Description | Multiple lines of text |
| Deadline | Date |
| Status | Choice: `Compliant`, `In Progress`, `Gap Identified`, `Overdue`, `Not Applicable` |
| Owner | Single line of text |
| Priority | Choice: `Critical`, `High`, `Medium`, `Low` |
| Notes | Multiple lines of text |

3. Add sample data:

| RegulatoryItem | Source | Reference | Description | Deadline | Status | Owner | Priority | Notes |
|---|---|---|---|---|---|---|---|---|
| GL15 ILAS Fee Restructuring | IA Guideline | GL15 s2.1 | Legacy ILAS products need fee structure aligned to new caps | 2026-09-30 | In Progress | Head of Products | Critical | 95% complete, remaining 5% legacy products |
| GL8 Enhanced Cooling-Off Implementation | IA Guideline | GL8 s3 | 30-day cooling-off for customers 65+ must be fully automated | 2026-06-30 | In Progress | Head of Operations | High | System changes 80% complete |
| AML Training Completion | AMLO | AMLO s23 | All staff must complete AML refresher training annually | 2026-03-31 | Gap Identified | Head of AML | High | 92% completion — 8% new joiners in grace period |
| ILAS Voice Recording | IA Guideline | GL15 s2.3 | Voice recording for ILAS sales >HKD 500K | 2026-06-30 | In Progress | CIO | High | Technology upgrade in progress, 78% rollout |
| FNA Documentation Standards | IA Guideline | GL15 s2.3 | FNA completion and documentation audit pass rate target 95% | 2026-12-31 | In Progress | Head of Conduct Risk | Medium | Currently at 88% pass rate |
| Annual Returns to IA | IA Guideline | IA Ordinance | Annual regulatory returns submission | 2026-06-30 | Compliant | CCO | Critical | On track for submission |
| Quarterly Compliance Board Report | Internal | Board Charter | Q2 compliance report to Board Risk Committee | 2026-07-30 | Not Applicable | CCO | High | Q1 report submitted on time |
| PEP Screening System Upgrade | AMLO | AMLO s3 | Upgrade PEP screening database to include expanded FATF list | 2026-08-31 | In Progress | Head of AML | Medium | Vendor selected, implementation started |
| Product Marketing Review Process | IA Circular | IA/2026/01 | New IA circular requiring pre-approval of all digital marketing materials | 2026-05-31 | Gap Identified | Head of Marketing | High | Current process only covers print materials |
| Data Privacy — Cross-border Transfer | PDPO | PDPO s33 | Review cross-border data transfer arrangements post new PDPO amendment | 2026-09-30 | In Progress | DPO | Medium | Legal review in progress |

---

## Phase 2: Create the Agent

### 2.1 Create the Agent

1. Go to **Copilot Studio** → **Create** → **New agent**
2. Name: **"Regulatory & Compliance Agent"**
3. Description: *"Helps Manulife HK executives and compliance officers understand regulatory requirements, track compliance deadlines, and assess compliance gaps."*

### 2.2 Set Instructions

```
You are the Regulatory & Compliance Agent for Manulife Hong Kong. You help executives, compliance officers, and senior managers understand and navigate regulatory requirements.

Your role:
- Answer questions about HK Insurance Authority (IA) guidelines, circulars, and regulations
- Explain regulatory requirements in clear, non-legal language suitable for executives
- Track compliance deadlines and flag items that are overdue or approaching deadline
- Identify compliance gaps and explain their implications
- Cross-reference company policies against regulatory requirements

Rules:
- Always cite the specific guideline, section, or reference when answering regulatory questions
- NEVER provide legal advice — always include the disclaimer: "This is for informational purposes. Consult the Legal & Compliance team for formal legal guidance."
- When explaining regulations, use plain language first, then provide the technical reference
- If you are not sure about a regulatory requirement, say "I don't have this specific information. Please check with the Compliance team or refer to the IA website at www.ia.org.hk."
- Flag any items that are overdue or have a deadline within the next 30 days
- When summarising compliance status, always include: total items tracked, items compliant, items in progress, gaps identified, overdue items
- Use a professional, precise tone — compliance matters demand accuracy
- Present regulatory timelines and deadlines clearly with specific dates
```

### 2.3 Add Knowledge Sources

1. Go to **Knowledge** → **+ Add knowledge**
2. Upload or connect via SharePoint:
   - `IA_Guideline_GL15_ILAS.docx`
   - `IA_Guideline_GL8_CoolingOff.docx`
   - `AML_Guidelines.docx`
   - `Manulife_Compliance_Manual.docx`
3. Wait for indexing

### 2.4 Test the Knowledge

| Test Query | Expected Behaviour |
|---|---|
| `What is the cooling-off period for ILAS products?` | "21 calendar days standard, 30 days for customers 65+ or annual premium >HKD 100K" (citing GL8 s3) |
| `What are our AML training requirements?` | "Complete within 3 months of joining, annual refresher required" (citing AMLO guidelines) |
| `What are the penalties for GL15 non-compliance?` | "Fine up to HKD 10M, licence conditions, individual sanctions" |
| `What is our current compliance gap in ILAS?` | "95% compliant, remaining 5% legacy products need fee restructuring by Q3 2026" |
| `When must we file STRs?` | "Within 2 business days of suspicion being formed" |

---

## Phase 3: Create Power Automate Flow

### 3.1 Flow: "Get Regulatory Deadlines"

1. **Trigger inputs**: `StatusFilter` (Text), `PriorityFilter` (Text)
2. **SharePoint — Get items**:
   - List: `RegulatoryTracker`
   - Filter Query (Expression):
     ```
     if(
       equals(triggerBody()?['text'], 'All'),
       '',
       concat('Status eq ''', triggerBody()?['text'], '''')
     )
     ```
   - Sort By: `Deadline`
   - Sort Order: Ascending
   - Top Count: 20
3. **Select** (Data Operations):
   - Item: `item()?['Title']`
   - Reference: `item()?['Reference']`
   - Deadline: `item()?['Deadline']`
   - Status: `item()?['Status']?['Value']`
   - Owner: `item()?['Owner']`
   - Priority: `item()?['Priority']?['Value']`
   - Notes: `item()?['Notes']`
4. **Compose**: `string(body('Select'))`
5. **Return**: `TrackerData` (Text)
6. Name: `Get Regulatory Deadlines` → **Save**

---

## Phase 4: Create Topics

### 4.1 Topic: "Regulatory Q&A"

This topic relies entirely on generative answers from the knowledge base.

**Trigger phrases**: `What does the regulation say`, `IA guideline`, `GL15`, `GL8`, `AML requirement`, `Regulatory requirement`, `Compliance question`, `What are the rules for`

**Build the flow:**

1. **Generative answers** node — configured with all knowledge sources linked
2. The agent will automatically answer from the uploaded regulatory documents
3. **Message** (appended after answer): "*Disclaimer: This is for informational purposes. Consult the Legal & Compliance team for formal legal guidance.*"
4. End topic

> 💡 This is the simplest topic — it leverages Copilot Studio's built-in generative answers capability. No flow or complex logic needed.

### 4.2 Topic: "Compliance Deadline Tracker"

**Trigger phrases**: `Compliance deadlines`, `What's overdue`, `Upcoming deadlines`, `Regulatory tracker`, `Compliance status`, `What do we need to do`, `Gap analysis`

| Step | Node | Variable |
|---|---|---|
| 1 | Message: "I'll check the regulatory compliance tracker for you." | — |
| 2 | Ask: "What would you like to see?" → Multiple choice: All items / Overdue items / In Progress / Gaps Identified / Critical & High priority only | `Topic.Filter` |
| 3 | Call Action: **Get Regulatory Deadlines** (StatusFilter ← Topic.Filter) | `Topic.TrackerData` |
| 4 | **Generative answers** — prompt: `Analyse the following regulatory compliance tracker data and present it as: (1) Summary: total items, compliant count, in progress, gaps, overdue. (2) Table of items sorted by deadline. (3) Items needing immediate attention (overdue or deadline within 30 days). (4) Recommendations. Data: {Topic.TrackerData}. Today's date is May 2026.` | — |
| 5 | Message: "Would you like details on any specific regulatory item?" | — |
| 6 | End topic | — |

### 4.3 Topic: "Generate Compliance Summary"

**Trigger phrases**: `Compliance summary for the board`, `Prepare compliance report`, `Summarise our compliance position`, `Compliance overview`

| Step | Node | Variable |
|---|---|---|
| 1 | Message: "I'll prepare a compliance summary." | — |
| 2 | Call Action: **Get Regulatory Deadlines** (StatusFilter = "All") | `Topic.AllData` |
| 3 | **Generative answers** — prompt: `Generate a board-level compliance summary based on the following data and the compliance manual knowledge. Include: (1) Executive summary (2-3 sentences), (2) Overall compliance score (% of items compliant), (3) Key risks and gaps, (4) Upcoming deadlines in next 60 days, (5) Recommended actions for the board. Data: {Topic.AllData}` | — |
| 4 | Message: "*This summary is auto-generated. Please review with the CCO before presenting to the board.*" | — |
| 5 | End topic | — |

---

## Phase 5: Testing

### 5.1 Test Queries

| Test Query | Expected Response |
|---|---|
| "What are the ILAS selling requirements?" | Detailed answer from GL15 with sections on FNA, suitability, voice recording |
| "What's overdue in our compliance tracker?" | Lists items past deadline with details |
| "What are our AML obligations in summary?" | Structured summary from AML guidelines |
| "When is the next IA filing deadline?" | "Annual returns due 30 June 2026" |
| "What compliance gaps do we have?" | Lists GL15 legacy, AML training, voice recording, FNA gaps |
| "Prepare a compliance summary for the board" | Generates executive-level compliance overview |

### 5.2 Demo Script (7 minutes)

| Time | What to Show | What to Say |
|---|---|---|
| 0:00 | "What is the cooling-off period for ILAS products?" | "Compliance officers and executives can get instant answers to regulatory questions" |
| 1:30 | Show answer with citations | "It cites the specific guideline — GL8, GL15 — not generic answers" |
| 2:30 | "What compliance deadlines are coming up?" | "Now let's look at our compliance tracker" |
| 3:30 | Show deadline summary with flags | "Instantly see what's overdue, what's approaching, and who owns each item" |
| 4:30 | "What are our current compliance gaps?" | "For a board meeting, you need to know where the gaps are" |
| 5:30 | "Prepare a compliance summary for the board" | "Generate a board-ready summary in seconds" |
| 6:30 | Show build experience | "All knowledge from your actual regulatory documents — single source of truth" |

---

## Troubleshooting

| Issue | Cause | Solution |
|---|---|---|
| Agent gives vague regulatory answers | Documents not detailed enough | Add more specific regulatory content to knowledge base |
| Tracker data not filtering correctly | Status values don't match filter | Ensure SharePoint choice values exactly match topic choices |
| Agent provides legal advice | Instructions not strict enough | Reinforce "never provide legal advice" in Instructions |
| Missing disclaimer | Generative answers bypass the message node | Add disclaimer text directly in the generative answers prompt |
