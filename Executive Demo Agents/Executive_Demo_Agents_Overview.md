> **Author**: Ken Tsui (CSA), Microsoft
> **Copyright** © 2026 Microsoft. All rights reserved.

# Manulife HK — Executive-Level Copilot Studio Demo Agents

> A collection of demo agent concepts designed to showcase the value of Microsoft Copilot Studio to Manulife Hong Kong senior leadership (C-suite, SVPs, Department Heads). Each agent targets a real executive pain point and demonstrates how AI agents can drive strategic decisions, operational efficiency, and competitive advantage.

---

## Why These Demos Matter for Executives

| Executive Concern | What AI Agents Solve |
|---|---|
| "I spend hours reading reports before meetings" | Agents summarise and surface key insights on demand |
| "I can't get real-time data without asking my team" | Agents query data sources and present KPIs conversationally |
| "Compliance requirements keep changing" | Agents monitor regulatory updates and flag action items |
| "I need to respond to escalations faster" | Agents triage, summarise, and recommend actions on escalated issues |
| "Our advisors need better support tools" | Agents demonstrate what's possible for frontline staff |

---

## Demo Agent Portfolio — At a Glance

| # | Agent Name | Target Audience | Key Value Proposition | Complexity |
|---|---|---|---|---|
| 1 | **Executive Briefing Agent** | CEO, COO, CFO | Summarises reports, meeting prep, and key decisions | ⭐⭐ |
| 2 | **Business Performance KPI Agent** | CEO, CFO, CRO | Conversational access to sales, revenue, and operational KPIs | ⭐⭐⭐ |
| 3 | **Regulatory & Compliance Agent** | CLO, CCO, CRO | HK IA regulatory Q&A, compliance tracking, policy gap analysis | ⭐⭐ |
| 4 | **Customer Complaint Escalation Agent** | COO, Head of CX | Triage and summarise escalated complaints with recommended actions | ⭐⭐ |
| 5 | **Advisor Performance Insights Agent** | Head of Distribution, CRO | Advisor productivity, sales leaderboards, coaching recommendations | ⭐⭐⭐ |
| 6 | **HR & Talent Intelligence Agent** | CHRO, Head of HR | Workforce analytics, attrition risk, succession planning Q&A | ⭐⭐ |
| 7 | **Market & Competitor Intelligence Agent** | CEO, CMO, Strategy | HK insurance market trends, competitor moves, strategic insights | ⭐⭐ |
| 8 | **IT Service & Security Posture Agent** | CIO, CISO | IT incident summaries, security posture Q&A, vendor risk | ⭐⭐ |

---

## Agent 1: Executive Briefing Agent

### Scenario
> *"I have a board meeting in 2 hours. Summarise the key points from the last 3 monthly reports and flag anything I need to address."*

### What It Does
- Answers questions about uploaded board packs, monthly reports, and strategy documents
- Generates concise briefing summaries with key metrics and action items
- Highlights risks, decisions needed, and performance outliers
- Produces a formatted briefing note (Word document) on demand

### Copilot Studio Concepts
| Concept | How It's Used |
|---|---|
| **Knowledge** | SharePoint library with board packs, monthly reports, strategy decks |
| **Instructions** | Executive tone, concise answers, structured with bullet points |
| **Topics** | "Generate Briefing Note", "Summarise Report", "What decisions are pending?" |
| **Actions** | Power Automate flow to generate Word briefing doc and save to SharePoint |

### Data Sources (Demo)
- SharePoint Document Library: `ExecutiveReports`
  - Sample monthly performance reports (PDF/DOCX)
  - Sample board pack materials
  - Strategic initiative updates

### Demo Flow
1. User asks: *"What were the key highlights from the April report?"*
2. Agent answers from knowledge base with structured summary
3. User asks: *"Generate a briefing note for tomorrow's board meeting"*
4. Agent collects focus areas → triggers Power Automate → returns Word doc link

---

## Agent 2: Business Performance KPI Agent

### Scenario
> *"What's our new business premium for Q1? How does it compare to last year? Which product line is underperforming?"*

### What It Does
- Queries business performance data (new business, renewals, claims ratio, lapse rate)
- Compares performance across time periods, product lines, and channels
- Provides natural language answers to data questions
- Can break down by region (HK, Macau), channel (agency, bancassurance, digital)

### Copilot Studio Concepts
| Concept | How It's Used |
|---|---|
| **Knowledge** | SharePoint list or Dataverse table with KPI data |
| **Topics** | "Show Q1 performance", "Compare year-over-year", "Product breakdown" |
| **Variables** | Time period, product line, channel, region |
| **Actions** | Power Automate flow querying Dataverse/SharePoint for live KPI data |

### Sample KPI Data Structure

| Metric | Q1 2026 | Q1 2025 | YoY Change |
|---|---|---|---|
| New Business Premium (HKD M) | 2,340 | 2,150 | +8.8% |
| Number of New Policies | 18,500 | 17,200 | +7.6% |
| Claims Ratio | 62% | 65% | -3pp |
| Lapse Rate (13-month) | 4.2% | 5.1% | -0.9pp |
| Agency Force Size | 12,800 | 12,200 | +4.9% |
| MDRT Qualifiers | 1,450 | 1,320 | +9.8% |

### Demo Flow
1. User asks: *"How are we doing this quarter?"*
2. Agent returns top-line KPI summary
3. User asks: *"Which product line has the highest growth?"*
4. Agent breaks down by Life, Health, ILAS, Retirement
5. User asks: *"Show me the agency channel vs bancassurance"*
6. Agent provides channel comparison

---

## Agent 3: Regulatory & Compliance Agent

### Scenario
> *"What are the latest IA guidelines on ILAS products? Are we compliant with the new cooling-off period requirements?"*

### What It Does
- Answers questions about HK Insurance Authority (IA) regulations
- Explains compliance requirements in plain language
- Flags upcoming regulatory changes and deadlines
- Cross-references company policies against regulatory requirements

### Copilot Studio Concepts
| Concept | How It's Used |
|---|---|
| **Knowledge** | Uploaded regulatory documents (IA guidelines, GL circulars, compliance manuals) |
| **Instructions** | Accurate, cite sources, never provide legal advice, flag uncertainty |
| **Topics** | "Regulatory Q&A", "Compliance checklist", "Upcoming changes" |

### Sample Knowledge Documents
- IA Guideline on ILAS (GL15)
- Guideline on Cooling-off Period (GL8)
- Anti-Money Laundering Guidelines
- Manulife Internal Compliance Manual (sample)
- Regulatory Change Tracker (sample spreadsheet)

### Demo Flow
1. User asks: *"What is the cooling-off period for ILAS products?"*
2. Agent responds with accurate answer citing GL8
3. User asks: *"Are there any regulatory changes coming in Q3?"*
4. Agent surfaces upcoming deadlines from tracker
5. User asks: *"Summarise our AML obligations for the board"*
6. Agent generates structured AML compliance summary

---

## Agent 4: Customer Complaint Escalation Agent

### Scenario
> *"Show me the top escalated complaints this week. What are the common themes? Draft a response for the IA complaint."*

### What It Does
- Surfaces escalated customer complaints from a tracking list
- Categorises complaints by type (claims, service, product, mis-selling)
- Summarises complaint details and history
- Suggests recommended resolution actions
- Drafts executive-level response letters

### Copilot Studio Concepts
| Concept | How It's Used |
|---|---|
| **Knowledge** | Complaint handling procedures, response templates |
| **Topics** | "Show escalated complaints", "Draft response", "Complaint trends" |
| **Variables** | Complaint ID, category, severity, date range |
| **Actions** | Power Automate querying SharePoint/Dataverse complaint tracker |
| **AI Builder** | Prompt to draft professional response letters |

### Sample Data: Escalated Complaints

| ID | Date | Category | Severity | Summary | Status |
|---|---|---|---|---|---|
| ESC-001 | 2026-04-28 | Claims Delay | High | Client waiting 45+ days for critical illness claim settlement | Under Review |
| ESC-002 | 2026-04-30 | Mis-selling | Critical | IA referral — client alleges ILAS product misrepresentation | Escalated to Legal |
| ESC-003 | 2026-05-01 | Service | Medium | VIP client unable to reach assigned advisor for 2 weeks | Assigned |
| ESC-004 | 2026-05-03 | Premium | High | Incorrect premium deduction for 3 consecutive months | Investigating |

### Demo Flow
1. User asks: *"What are the critical escalations this week?"*
2. Agent returns filtered list with summaries
3. User asks: *"Tell me more about ESC-002"*
4. Agent provides full details and recommended actions
5. User asks: *"Draft a response to the IA for this case"*
6. Agent generates professional response letter

---

## Agent 5: Advisor Performance Insights Agent

### Scenario
> *"Who are our top 10 advisors this month? What's the average case size for the agency channel? Which district is underperforming?"*

### What It Does
- Queries advisor performance data (cases, premium, persistency, activity)
- Provides leaderboards and rankings
- Identifies underperformers and suggests coaching focus areas
- Breaks down by district, unit, product, and time period

### Copilot Studio Concepts
| Concept | How It's Used |
|---|---|
| **Topics** | "Top performers", "District comparison", "Advisor profile" |
| **Variables** | District, time period, metric type, advisor code |
| **Actions** | Power Automate querying Dataverse/SQL with advisor performance data |

### Sample Data

| Rank | Advisor | District | Cases (MTD) | Premium (HKD K) | Persistency |
|---|---|---|---|---|---|
| 1 | Alice Wong | Central | 28 | 1,850 | 96% |
| 2 | David Chan | Kowloon East | 25 | 1,720 | 94% |
| 3 | Sarah Lee | NT West | 23 | 1,680 | 97% |
| ... | ... | ... | ... | ... | ... |

---

## Agent 6: HR & Talent Intelligence Agent

### Scenario
> *"What's our attrition rate for the tech team? Who are the succession candidates for the Head of Claims role? How does our headcount compare to budget?"*

### What It Does
- Answers workforce analytics questions (headcount, attrition, hiring pipeline)
- Provides succession planning information
- Compares actual vs budgeted headcount
- Surfaces employee engagement trends

### Copilot Studio Concepts
| Concept | How It's Used |
|---|---|
| **Knowledge** | HR policy documents, org charts, succession plans (sample) |
| **Topics** | "Headcount summary", "Attrition analysis", "Succession candidates" |
| **Variables** | Department, role level, time period |
| **Actions** | Power Automate querying HR data from Dataverse/SharePoint |

### Demo Flow
1. User asks: *"What's the overall attrition rate this year?"*
2. Agent: *"Year-to-date voluntary attrition is 8.2%, down from 9.5% same period last year..."*
3. User asks: *"Which departments have the highest turnover?"*
4. Agent breaks down by department with analysis
5. User asks: *"Who are the succession candidates for Head of Claims?"*
6. Agent surfaces succession pipeline with readiness levels

---

## Agent 7: Market & Competitor Intelligence Agent

### Scenario
> *"What's the latest on AIA's new product launch? How does our market share compare? What are the key trends in the HK insurance market?"*

### What It Does
- Answers questions about the HK insurance market landscape
- Provides competitor analysis (AIA, Prudential, Sun Life, FWD, etc.)
- Summarises market trends (digital adoption, regulatory shifts, product innovation)
- Surfaces insights from uploaded market research reports

### Copilot Studio Concepts
| Concept | How It's Used |
|---|---|
| **Knowledge** | Market research reports, competitor analysis docs, IA market statistics |
| **Instructions** | Objective tone, cite sources, distinguish facts from analysis |
| **Topics** | "Market overview", "Competitor comparison", "Trend analysis" |

### Sample Knowledge Documents
- HK IA Annual Statistics Report (sample)
- Internal Competitor Intelligence Report (sample)
- Market Trend Analysis Q1 2026 (sample)
- Product Comparison Matrix (sample spreadsheet)

---

## Agent 8: IT Service & Security Posture Agent

### Scenario
> *"How many P1 incidents did we have this month? What's the status of our cloud migration? Are there any open security vulnerabilities?"*

### What It Does
- Summarises IT service health and incident trends
- Answers questions about ongoing IT projects and initiatives
- Provides security posture overview (vulnerability counts, compliance status)
- Surfaces vendor risk assessments

### Copilot Studio Concepts
| Concept | How It's Used |
|---|---|
| **Knowledge** | IT status reports, security dashboards (exported), project plans |
| **Topics** | "IT incidents summary", "Project status", "Security posture" |
| **Actions** | Power Automate querying ServiceNow/Dataverse for incident data |

---

## Recommended Demo Strategy for Executive Audience

### Demo Order (30-minute executive session)

| Order | Agent | Time | Why This Order |
|---|---|---|---|
| 1 | **Executive Briefing Agent** | 8 min | Most relatable — every exec reads reports. Instant "wow" moment |
| 2 | **Business Performance KPI Agent** | 8 min | Shows data access without dashboards. Strategic value |
| 3 | **Customer Complaint Escalation Agent** | 7 min | Operational risk management. Board-level concern |
| 4 | **Regulatory & Compliance Agent** | 7 min | Critical for insurance. Demonstrates knowledge/RAG capability |

### Key Messages for Executives

1. **No code required** — Business users can build and maintain these agents
2. **Enterprise-grade security** — Runs on Microsoft 365, respects existing permissions
3. **Rapid time-to-value** — Each agent can be built in hours, not months
4. **Composable** — Agents can be orchestrated together (multi-agent pattern)
5. **Channel-flexible** — Deploy to Teams, web, mobile, or internal portals

### Tips for Executive Demos

- **Start with their pain point**, not the technology
- **Use realistic data** — executives spot fake data instantly
- **Show the build experience** — briefly show how simple it is to create an agent (2 min)
- **Emphasise governance** — who controls what the agent can access and say
- **End with a vision** — paint the picture of 10+ agents across the organisation working together

---

## Next Steps

1. Pick 2-3 agents from this portfolio to build as full demos
2. Create sample data sets in SharePoint / Dataverse
3. Build the agents in Copilot Studio (follow the patterns from the existing hackathon guides)
4. Prepare a 30-minute demo script with talking points
5. Schedule an executive briefing session

---

## Folder Structure

```
Executive Demo Agents/
├── Executive_Demo_Agents_Overview.md          ← This file
├── Agent 1 - Executive Briefing/
│   └── (setup guide, sample data, demo script)
├── Agent 2 - Business KPI/
│   └── (setup guide, sample data, demo script)
├── Agent 3 - Regulatory Compliance/
│   └── (setup guide, sample data, demo script)
├── Agent 4 - Complaint Escalation/
│   └── (setup guide, sample data, demo script)
├── Agent 5 - Advisor Performance/
│   └── (setup guide, sample data, demo script)
├── Agent 6 - HR Talent Intelligence/
│   └── (setup guide, sample data, demo script)
├── Agent 7 - Market Intelligence/
│   └── (setup guide, sample data, demo script)
└── Agent 8 - IT Security Posture/
    └── (setup guide, sample data, demo script)
```
