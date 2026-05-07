> **Author**: Ken Tsui (CSA), Microsoft
> **Copyright** © 2026 Microsoft. All rights reserved.

# Agent 1: Executive Briefing Agent — End-to-End Setup Guide

> A step-by-step guide to build an agent that helps C-suite executives get instant summaries of board packs, monthly reports, and strategic documents — and generates formatted briefing notes on demand.

---

## Use Case

Manulife HK executives spend hours reading monthly reports, board packs, and strategy documents before key meetings. This agent:

- **Answers questions** about uploaded executive reports and board materials
- **Summarises** key highlights, risks, and action items from documents
- **Generates a formatted Word briefing note** with the executive's chosen focus areas
- **Saves the briefing note** to SharePoint and provides a download link

### Architecture Overview

```
┌──────────────────────────────────────────────────────────────┐
│  USER (CEO / CFO / COO)                                      │
│  "Summarise the April monthly report"                        │
│                          │                                   │
│                          ▼                                   │
│  ┌───────────────────────────────────────────┐               │
│  │  🤖 Executive Briefing Agent              │               │
│  │                                           │               │
│  │  Knowledge Sources:                       │               │
│  │  📄 Monthly_Report_Apr2026.docx          │               │
│  │  📄 Board_Pack_Q1_2026.docx              │               │
│  │  📄 Strategic_Initiatives_2026.docx      │               │
│  │                                           │               │
│  │  Topics:                                  │               │
│  │  1. Summarise Report (AI / Knowledge)     │               │
│  │  2. Generate Briefing Note (Flow)         │               │
│  └────────────────┬──────────────────────────┘               │
│                   │                                          │
│         ┌─────────▼──────────┐                               │
│         │  Power Automate    │                               │
│         │  Flow:             │                               │
│         │  "Generate         │                               │
│         │   Briefing Note"   │                               │
│         └─────┬──────┬───────┘                               │
│               │      │                                       │
│   ┌───────────▼┐  ┌──▼──────────────┐                       │
│   │ OneDrive   │  │ SharePoint      │                       │
│   │ "Convert   │  │ Library:        │                       │
│   │  HTML →    │  │ ExecutiveDocs/  │                       │
│   │  .docx"    │  │ BriefingNotes   │                       │
│   └─────────────┘  └─────────────────┘                       │
└──────────────────────────────────────────────────────────────┘
```

---

## Prerequisites

- [ ] Microsoft 365 tenant with Copilot Studio license
- [ ] SharePoint Online access
- [ ] Power Automate access (for document generation flow)
- [ ] OneDrive for Business (for HTML → Word conversion)

---

## Phase 1: Data & Document Setup

### 1.1 Create SharePoint Document Library: `ExecutiveReports`

This library holds the source documents the agent will use as knowledge.

1. Go to your **SharePoint site** → **New** → **Document library** → name it `ExecutiveReports`
2. Create and upload the following sample documents:

#### Sample Document 1: `Monthly_Report_Apr2026.docx`

Create a Word document with the following content:

```
MANULIFE HONG KONG — MONTHLY PERFORMANCE REPORT
APRIL 2026

EXECUTIVE SUMMARY
April 2026 was a strong month for Manulife HK, with new business premium 
reaching HKD 780M, up 12% year-over-year. Agency channel continued to 
outperform targets, while bancassurance saw a slight dip of 3% due to 
seasonal factors.

KEY HIGHLIGHTS
1. New Business Premium: HKD 780M (target: HKD 720M) — 108% of target
2. Total Active Policies: 1.24M (+2.1% vs March)
3. Claims Ratio: 61.8% (improved from 63.2% in March)
4. Agency Force: 12,850 advisors (+50 net new in April)
5. MDRT Qualifiers YTD: 1,480 (on track for record year)
6. Customer NPS: 72 (up 3 points from Q4 2025)

PRODUCT PERFORMANCE
- Life Insurance: HKD 420M (+15% YoY) — strong demand for whole life products
- Health Insurance: HKD 185M (+8% YoY) — VHIS driving growth
- Critical Illness: HKD 110M (+5% YoY) — steady performance
- ILAS: HKD 45M (-12% YoY) — regulatory changes impacting sales
- Retirement/MPF: HKD 20M (+3% YoY) — stable

CHANNEL PERFORMANCE
- Agency: HKD 580M (+16% YoY) — top districts: Central, Kowloon East, NT West
- Bancassurance: HKD 150M (-3% YoY) — seasonal slowdown, expected to recover in May
- Digital: HKD 50M (+45% YoY) — online term life sales surging

RISKS & CONCERNS
1. ILAS sales decline — need product refresh aligned with new IA guidelines
2. Bancassurance partner renegotiation due in Q3 — commercial terms under review
3. Rising medical claims in VHIS tier — actuarial review initiated
4. Talent competition — AIA and Prudential actively recruiting senior advisors

ACTION ITEMS FOR LEADERSHIP
1. [CEO] Approve ILAS product refresh proposal by May 15
2. [CFO] Review bancassurance partnership financial model by May 20
3. [CRO] Present VHIS claims analysis at June board meeting
4. [COO] Finalise advisor retention package by May 30
5. [CMO] Launch digital campaign for Q2 product push — budget approval needed

UPCOMING MILESTONES
- May 12: Q1 Board Meeting — requires board pack review
- May 20: Bancassurance partner meeting
- June 1: MDRT qualifying deadline reminder to agency force
- June 15: Half-year strategy review with regional leadership
```

#### Sample Document 2: `Board_Pack_Q1_2026.docx`

```
MANULIFE HONG KONG — BOARD PACK
Q1 2026 (JANUARY – MARCH)

AGENDA
1. CEO Opening Remarks
2. Financial Performance Review (CFO)
3. Business Growth Update (CRO)
4. Operations & Technology (COO/CIO)
5. Risk & Compliance (CRO/CCO)
6. People & Culture (CHRO)
7. Strategic Initiatives Update
8. Decisions Required

1. CEO OPENING REMARKS
Q1 2026 has been a strong start to the year. We achieved HKD 2,340M in new 
business premium, exceeding our Q1 target of HKD 2,200M by 6.4%. Our agency 
force continues to grow and productivity per advisor has improved 8% YoY.

Key challenges remain around ILAS regulatory changes, bancassurance partnership 
renewal, and rising medical inflation impacting our health portfolio.

2. FINANCIAL PERFORMANCE (CFO)
- Revenue: HKD 4.2B (+7% YoY)
- Operating Profit: HKD 890M (+5% YoY)
- Expense Ratio: 28.3% (target: <30%)
- Investment Returns: 4.8% (benchmark: 4.5%)
- Capital Adequacy Ratio: 285% (regulatory minimum: 150%)

3. BUSINESS GROWTH (CRO)
- New Business Premium: HKD 2,340M (+8.8% YoY)
- Number of New Policies: 18,500 (+7.6% YoY)
- Persistency (13-month): 95.8% (+0.9pp YoY)
- Average Case Size: HKD 126K (+1.1% YoY)

4. OPERATIONS & TECHNOLOGY (COO/CIO)
- Policy issuance SLA: 2.1 days average (target: <3 days)
- Digital adoption rate: 68% of new policies submitted online
- Claims processing time: 5.2 days average (target: <7 days)
- Cloud migration: 72% complete (target: 85% by year-end)
- Cybersecurity: Zero P1 incidents in Q1

5. RISK & COMPLIANCE (CRO/CCO)
- Claims Ratio: 62% (within acceptable range of 58-65%)
- Lapse Rate: 4.2% (improved from 5.1% in Q1 2025)
- IA Complaints: 3 (down from 5 in Q1 2025)
- AML alerts reviewed: 128 (all cleared, 2 escalated as SAR)
- Regulatory updates: New IA circular on ILAS selling practices — gap analysis in progress

6. PEOPLE & CULTURE (CHRO)
- Headcount: 3,200 staff (+120 YTD)
- Voluntary Attrition: 8.2% annualised (target: <10%)
- Employee Engagement Score: 78/100 (industry avg: 72)
- Agency Force: 12,800 advisors
- MDRT Qualifiers: 1,450 (+9.8% YoY)

7. STRATEGIC INITIATIVES UPDATE
- Digital Self-Service Portal: Phase 2 launch in May — claims tracking, policy management
- AI-Powered Underwriting: Pilot complete, 30% faster processing, full rollout Q3
- Green Insurance Products: ESG-linked retirement product design approved
- Advisor Academy: New training platform launched — 4,200 completions in Q1

8. DECISIONS REQUIRED
Decision 1: Approve HKD 15M budget for ILAS product refresh
  - Recommendation: Approve
  - Sponsor: CRO
  - Deadline: May Board Meeting

Decision 2: Approve bancassurance partnership renewal terms
  - Recommendation: Negotiate revised revenue split (current 60/40 → proposed 55/45)
  - Sponsor: CFO
  - Deadline: Q3 2026

Decision 3: Approve expansion of AI underwriting to all product lines
  - Recommendation: Approve with phased rollout
  - Sponsor: COO
  - Deadline: June Board Meeting

Decision 4: Approve advisor retention incentive package (HKD 8M)
  - Recommendation: Approve — critical to counter competitor poaching
  - Sponsor: CHRO
  - Deadline: May 30
```

#### Sample Document 3: `Strategic_Initiatives_2026.docx`

```
MANULIFE HONG KONG — STRATEGIC INITIATIVES TRACKER 2026

INITIATIVE 1: Digital Self-Service Portal (Phase 2)
- Owner: CIO
- Status: 🟢 On Track
- Timeline: May 2026 launch
- Budget: HKD 12M (spent: HKD 9.5M)
- Key Deliverables: Claims tracking, policy management, premium payment, document upload
- Dependencies: API integration with claims system (complete), UX testing (in progress)

INITIATIVE 2: AI-Powered Underwriting
- Owner: COO
- Status: 🟡 Minor Delay
- Timeline: Q3 2026 full rollout (was Q2)
- Budget: HKD 8M (spent: HKD 5.2M)
- Key Deliverables: Auto-decision for standard cases, risk scoring, medical report analysis
- Dependencies: Model validation by actuarial team (pending), regulatory approval for auto-decisions

INITIATIVE 3: Green Insurance Products
- Owner: CRO
- Status: 🟢 On Track
- Timeline: Q4 2026 launch
- Budget: HKD 3M (spent: HKD 1.1M)
- Key Deliverables: ESG-linked retirement product, carbon offset rider, green bond allocation
- Dependencies: Product design (complete), pricing (in progress), IA submission (Q3)

INITIATIVE 4: Advisor Academy Platform
- Owner: CHRO
- Status: 🟢 On Track
- Timeline: Launched Q1 2026
- Budget: HKD 5M (spent: HKD 4.8M)
- Key Deliverables: Online training, certification tracking, gamification, performance dashboards
- Metrics: 4,200 course completions, 92% satisfaction score

INITIATIVE 5: Cybersecurity Enhancement Program
- Owner: CISO
- Status: 🟡 Minor Delay
- Timeline: Year-long program
- Budget: HKD 18M (spent: HKD 7.2M)
- Key Deliverables: Zero trust architecture, SIEM upgrade, employee security training
- Dependencies: Vendor contract finalisation (delayed 3 weeks), cloud migration milestones

INITIATIVE 6: Customer Experience Transformation
- Owner: COO
- Status: 🟢 On Track
- Timeline: Ongoing through 2026
- Budget: HKD 6M (spent: HKD 2.8M)
- Key Deliverables: NPS improvement program, claims experience redesign, advisor feedback loop
- Metrics: NPS improved from 69 to 72 in Q1
```

### 1.2 Create SharePoint Document Library: `BriefingNotes`

1. **New** → **Document library** → name it `BriefingNotes`
2. This library stores the generated briefing notes — no files to upload initially

---

## Phase 2: Create the Agent

### 2.1 Create the Agent

1. Go to **Copilot Studio** → **Create** → **New agent**
2. Name: **"Executive Briefing Agent"**
3. Description: *"Helps Manulife HK executives quickly understand monthly reports, board packs, and strategic documents. Can summarise content, answer questions, and generate formatted briefing notes."*

### 2.2 Set Instructions

Paste this into the Instructions field:

```
You are the Executive Briefing Agent for Manulife Hong Kong. You help senior executives (CEO, CFO, COO, CRO, and other C-suite leaders) quickly understand key business information.

Your role:
- Summarise monthly performance reports, board packs, and strategic initiative updates
- Answer specific questions about business metrics, risks, action items, and deadlines
- Generate formatted briefing notes when requested
- Highlight items that require executive decisions or immediate attention

Rules:
- Always structure your answers with clear headings, bullet points, and key metrics
- When summarising, prioritise: (1) headline numbers, (2) items requiring decisions, (3) risks and concerns, (4) upcoming milestones
- When citing metrics, include both the current value and the comparison (YoY, vs target, vs last month)
- If the user asks about something not covered in the documents, say "This information is not available in the current reports. You may want to check with [relevant department]."
- Use a concise, executive-appropriate tone — no filler, no unnecessary context
- When generating a briefing note, always confirm the focus areas with the executive before generating
- Flag any overdue action items or missed deadlines
- Present financial figures in HKD with appropriate units (K, M, B)
```

### 2.3 Add Knowledge Sources

#### Option A: Upload Directly (Quick Demo Setup)

1. Go to **Knowledge** → **+ Add knowledge**
2. Select **"Files"**
3. Upload:
   - `Monthly_Report_Apr2026.docx`
   - `Board_Pack_Q1_2026.docx`
   - `Strategic_Initiatives_2026.docx`
4. Wait for indexing to complete

#### Option B: Connect to SharePoint (Enterprise Setup)

1. Go to **Knowledge** → **+ Add knowledge**
2. Select **"SharePoint"**
3. Enter your SharePoint site URL → select `ExecutiveReports` library
4. Select all three documents
5. Click **Add** → wait for indexing

### 2.4 Test the Knowledge

Before building Topics, verify the agent can answer from the knowledge base:

| Test Query | Expected Behaviour |
|---|---|
| `What was our new business premium in April?` | "HKD 780M, up 12% YoY, 108% of target" |
| `What are the key risks mentioned in the April report?` | Lists 4 risks from the report |
| `What decisions are pending for the board?` | Lists 4 decisions from the board pack |
| `What's the status of the AI underwriting initiative?` | "Minor Delay — Q3 rollout (was Q2)" |
| `What's our claims ratio?` | "61.8% in April, improved from 63.2% in March" |
| `Who are the MDRT qualifiers?` | "1,480 YTD, on track for record year" |

---

## Phase 3: Create the Power Automate Flow — "Generate Briefing Note"

This flow takes the executive's focus areas, builds a styled HTML briefing document, converts it to Word, and saves it to SharePoint.

### 3.1 Open Power Automate from Copilot Studio

1. Open the **Executive Briefing Agent** in Copilot Studio
2. Go to **Topics** → click **+ Add a topic** → **From blank**
3. Click **+** → **"Call an action"** → **"Create a flow"**

### 3.2 Configure the Trigger

Click on **"When Copilot Studio calls a flow"** and add these **six** text inputs:

| Input # | Type | Name |
|---|---|---|
| 1 | Text | `ExecutiveName` |
| 2 | Text | `MeetingName` |
| 3 | Text | `MeetingDate` |
| 4 | Text | `FocusAreas` |
| 5 | Text | `KeyHighlights` |
| 6 | Text | `ActionItems` |

> ⚠️ **Trigger input internal names** (hover over each input to confirm):
>
> | Display Name | Internal Name |
> |---|---|
> | ExecutiveName | `text` |
> | MeetingName | `text_1` |
> | MeetingDate | `text_2` |
> | FocusAreas | `text_3` |
> | KeyHighlights | `text_4` |
> | ActionItems | `text_5` |

### 3.3 Add Compose Step — "Generate Date"

1. **+ New step** → **Compose** → rename to `Generate Date`
2. Expression: `formatDateTime(utcNow(), 'dd/MM/yyyy HH:mm')`

### 3.4 Add Compose Step — "Build File Name"

1. **+ New step** → **Compose** → rename to `Build File Name`
2. Expression:
   ```
   concat('Briefing_Note_', triggerBody()?['text'], '_', formatDateTime(utcNow(), 'yyyyMMdd_HHmmss'))
   ```

### 3.5 Add Compose Step — "Build HTML Document"

1. **+ New step** → **Compose** → rename to `Build HTML Document`
2. In the **Inputs** field, paste the following HTML (replace `[DYNAMIC: ...]` placeholders with Dynamic Content):

```html
<html>
<head>
<style>
  body { font-family: Calibri, Arial, sans-serif; margin: 40px; color: #333; }
  h1 { text-align: center; color: #00703C; margin-bottom: 5px; }
  h2 { text-align: center; color: #555; font-weight: normal; margin-top: 0; font-size: 16px; }
  .meta-table { width: 100%; margin-bottom: 20px; }
  .meta-table td { padding: 5px 10px; border: none; }
  .meta-table td:first-child { font-weight: bold; width: 180px; color: #00703C; }
  .section-header { background-color: #00703C; color: white; padding: 8px 15px;
    margin-top: 25px; margin-bottom: 10px; font-size: 14px; font-weight: bold; }
  .content-box { padding: 10px 15px; background-color: #f9f9f9; border-left: 4px solid #00703C;
    margin-bottom: 15px; white-space: pre-wrap; }
  .footer { margin-top: 30px; padding-top: 15px; border-top: 2px solid #00703C;
    font-size: 11px; color: #777; text-align: center; }
  .confidential { text-align: center; color: #cc0000; font-weight: bold; font-size: 12px;
    margin-bottom: 20px; }
</style>
</head>
<body>

<h1>MANULIFE HONG KONG</h1>
<h2>Executive Briefing Note</h2>
<div class="confidential">CONFIDENTIAL — FOR INTERNAL USE ONLY</div>

<table class="meta-table">
  <tr><td>Prepared For</td><td>[DYNAMIC: ExecutiveName]</td></tr>
  <tr><td>Meeting / Purpose</td><td>[DYNAMIC: MeetingName]</td></tr>
  <tr><td>Meeting Date</td><td>[DYNAMIC: MeetingDate]</td></tr>
  <tr><td>Generated On</td><td>[DYNAMIC: GenerateDate]</td></tr>
</table>

<div class="section-header">FOCUS AREAS</div>
<div class="content-box">[DYNAMIC: FocusAreas]</div>

<div class="section-header">KEY HIGHLIGHTS &amp; METRICS</div>
<div class="content-box">[DYNAMIC: KeyHighlights]</div>

<div class="section-header">ACTION ITEMS &amp; DECISIONS REQUIRED</div>
<div class="content-box">[DYNAMIC: ActionItems]</div>

<div class="footer">
  This briefing note was generated by the Manulife Executive Briefing Agent.<br>
  Source documents: Monthly Reports, Board Packs, Strategic Initiatives Tracker.<br>
  For questions, contact the Strategy & Planning team.
</div>

</body>
</html>
```

> 📝 **Dynamic Content mapping:**
>
> | Placeholder | Dynamic Content Source |
> |---|---|
> | `[DYNAMIC: ExecutiveName]` | **ExecutiveName** from trigger |
> | `[DYNAMIC: MeetingName]` | **MeetingName** from trigger |
> | `[DYNAMIC: MeetingDate]` | **MeetingDate** from trigger |
> | `[DYNAMIC: GenerateDate]` | **Outputs** from "Generate Date" Compose step |
> | `[DYNAMIC: FocusAreas]` | **FocusAreas** from trigger |
> | `[DYNAMIC: KeyHighlights]` | **KeyHighlights** from trigger |
> | `[DYNAMIC: ActionItems]` | **ActionItems** from trigger |

### 3.6 Add "OneDrive for Business — Create File" (Temporary HTML)

| Field | Value |
|---|---|
| **Folder Path** | `/` or `/TempFiles` |
| **File Name** | Expression: `concat(outputs('Build_File_Name'), '.html')` |
| **File Content** | Dynamic content → **Outputs** from "Build HTML Document" |

### 3.7 Add "OneDrive for Business — Convert File"

| Field | Value |
|---|---|
| **File** | Dynamic content → **Id** from "Create file" (OneDrive) |
| **Target type** | `docx` |

### 3.8 Add "SharePoint — Create File" (Final Document)

| Field | Value |
|---|---|
| **Site Address** | Your SharePoint site URL |
| **Folder Path** | `/BriefingNotes` |
| **File Name** | Expression: `concat(outputs('Build_File_Name'), '.docx')` |
| **File Content** | Dynamic content → **File content** from "Convert file" |

### 3.9 Add "OneDrive for Business — Delete File" (Cleanup)

| Field | Value |
|---|---|
| **File** | Dynamic content → **Id** from "Create file" (the OneDrive step, NOT SharePoint) |

### 3.10 Add Compose Step — "Build Document URL"

Expression:
```
concat('https://YOUR_TENANT.sharepoint.com/sites/YOUR_SITE/BriefingNotes/', replace(concat(outputs('Build_File_Name'), '.docx'), ' ', '%20'))
```

> ⚠️ Replace `YOUR_TENANT` and `YOUR_SITE` with your actual SharePoint URL.

### 3.11 Add Return Step

| Output Name | Type | Value |
|---|---|---|
| `Confirmation` | Text | `Briefing note generated successfully` |
| `DocumentLink` | Text | Dynamic content → **Outputs** from "Build Document URL" |

### 3.12 Save

Name the flow: **`Generate Briefing Note`** → **Save**

---

## Phase 4: Create Topics

### 4.1 Topic: "Summarise Report"

This topic uses generative answers from the knowledge base — no Power Automate flow needed.

**Trigger phrases**: `Summarise the report`, `What are the key highlights`, `Monthly report summary`, `Brief me on`, `What do I need to know`, `Board pack summary`

**Build the flow:**

1. **Message**: "I'll summarise the key information for you. What would you like me to focus on?"
2. **Ask**: "Focus area:" → Multiple choice:
   - Full summary
   - Financial performance only
   - Risks and concerns
   - Action items and decisions
   - Strategic initiatives status
   → Save to: `Topic.FocusArea`
3. **Generative answers** node — Input:
   ```
   Based on the executive reports, provide a concise summary focused on: {Topic.FocusArea}
   
   Structure your response with:
   - Headline metrics with comparisons (YoY, vs target)
   - Key highlights (max 5 bullet points)
   - Items requiring attention or decisions
   - Upcoming deadlines
   ```
4. **Message**: "Would you like me to generate a formatted briefing note document?"
5. **Ask**: → Multiple choice: (Yes, generate a briefing note / No, that's all) → `Topic.GenerateDoc`
6. **Condition**: GenerateDoc = "Yes, generate a briefing note"
   - **Redirect** to Topic: "Generate Briefing Note"
7. **Else**: Message "Got it. Let me know if you need anything else." → End topic
8. **Save**

### 4.2 Topic: "Generate Briefing Note"

**Trigger phrases**: `Generate a briefing note`, `Create a briefing document`, `Prepare a briefing`, `I need a briefing note`, `Make me a summary document`

**Build the flow:**

| Step | Node | Variable |
|---|---|---|
| 1 | Message: "I'll prepare a briefing note for you. Let me collect the details." | — |
| 2 | Ask: "What is your name?" → User's entire response | `Topic.ExecutiveName` |
| 3 | Ask: "What meeting or purpose is this for?" → User's entire response | `Topic.MeetingName` |
| 4 | Ask: "When is the meeting?" → User's entire response | `Topic.MeetingDate` |
| 5 | Ask: "What areas should I focus on?" → User's entire response | `Topic.FocusAreas` |
| 6 | **Generative answers** node — prompt: `Based on the executive reports, summarise the key highlights and metrics related to: {Topic.FocusAreas}. Present as concise bullet points with numbers.` | Saved to `Topic.KeyHighlights` |
| 7 | **Generative answers** node — prompt: `Based on the executive reports, list all action items, pending decisions, and upcoming deadlines related to: {Topic.FocusAreas}. Include owners and deadlines.` | Saved to `Topic.ActionItems` |
| 8 | Message: "Here's what I'll include in your briefing note: **Prepared for:** {Topic.ExecutiveName} **Meeting:** {Topic.MeetingName} ({Topic.MeetingDate}) **Focus Areas:** {Topic.FocusAreas} **Key Highlights:** {Topic.KeyHighlights} **Action Items:** {Topic.ActionItems}" | — |
| 9 | Ask: "Shall I generate the document?" → Multiple choice (Yes, generate it / No, let me adjust) | `Topic.Confirm` |
| 10 | Condition: Confirm = "Yes, generate it" | — |
| 11 | Call Action: **Generate Briefing Note** flow (map all 6 inputs) | `Topic.FlowConfirmation`, `Topic.DocLink` |
| 12 | Message: "✅ Your briefing note has been generated! [Download your briefing note]({Topic.DocLink})" | — |
| 13 | Else: Message "No problem. Tell me what you'd like to change and I'll adjust." → End topic | — |

> ⚠️ **Key rules:**
> - For step 6 and 7, use the **Generative answers** node with **"Save response as"** turned on to capture the AI output into a variable
> - If your Copilot Studio version doesn't support saving generative answer output to a variable, replace with an **AI Builder Custom Prompt** action
> - Ensure all variables passed to the flow are string type — use `Text()` wrapper if needed

**Save** the topic.

---

## Phase 5: Testing

### 5.1 Test Knowledge Q&A

| Test Query | Expected Response |
|---|---|
| "What was our revenue in Q1?" | "HKD 4.2B, up 7% YoY" |
| "What are the decisions needed for the next board meeting?" | Lists 4 decisions with sponsors and deadlines |
| "What's the status of the digital portal?" | "On Track — Phase 2 launch in May, HKD 9.5M of HKD 12M spent" |
| "What's our NPS?" | "72, up 3 points from Q4 2025" |
| "What are the main risks?" | Lists ILAS decline, bancassurance renegotiation, medical claims, talent competition |

### 5.2 Test Document Generation

1. Say: *"Generate a briefing note for the board meeting"*
2. Agent should ask for name, meeting, date, focus areas
3. Agent should generate highlights and action items from knowledge
4. Agent should show preview and ask for confirmation
5. On confirmation, flow runs and returns a download link
6. Click link → verify Word document opens with formatted content

### 5.3 Demo Script (8 minutes)

| Time | What to Show | What to Say |
|---|---|---|
| 0:00 | Open agent, type "Summarise the April report" | "Imagine you have a board meeting in 2 hours. Instead of reading a 10-page report..." |
| 1:30 | Show the structured summary response | "The agent instantly surfaces the key metrics, risks, and action items" |
| 2:30 | Ask "What decisions are pending?" | "You can drill into any area — the agent knows your documents" |
| 3:30 | Ask "Generate a briefing note for the board meeting" | "Now let's generate a formatted document you can bring to the meeting" |
| 5:00 | Fill in details, confirm generation | "It pulls the highlights from the reports and formats them professionally" |
| 6:30 | Show the generated Word document | "A branded, formatted briefing note — saved to SharePoint, ready to print" |
| 7:30 | Briefly show the Copilot Studio builder | "And this was built without writing a single line of code" |

---

## Troubleshooting

| Issue | Cause | Solution |
|---|---|---|
| Agent gives generic answers | Knowledge not indexed | Check Knowledge page → ensure documents show ✅ status |
| "I don't have information about that" | Question outside document scope | Rephrase or add more documents to knowledge |
| Flow fails at Convert file | OneDrive connector not authenticated | Re-authenticate the OneDrive connection in Power Automate |
| Document link not clickable | Spaces in file name not encoded | Ensure `Build Document URL` uses `replace(..., ' ', '%20')` |
| Generative answers too verbose | Instructions not guiding conciseness | Update Instructions to emphasise "concise, bullet points, executive tone" |
