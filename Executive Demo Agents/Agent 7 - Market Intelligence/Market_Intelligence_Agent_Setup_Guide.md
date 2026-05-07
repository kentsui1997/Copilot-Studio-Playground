> **Author**: Ken Tsui (CSA), Microsoft
> **Copyright** © 2026 Microsoft. All rights reserved.

# Agent 7: Market & Competitor Intelligence Agent — End-to-End Setup Guide

> A step-by-step guide to build an agent that gives executives instant access to HK insurance market trends, competitor analysis, and strategic insights from uploaded research reports.

---

## Use Case

Manulife HK's CEO, CMO, and Strategy team need quick intelligence on the competitive landscape: *"What's AIA's latest product launch?", "How does our market share compare?", "What trends should we watch?"*. This agent:

- **Answers questions** about the HK insurance market from uploaded research reports
- **Provides competitor comparisons** (AIA, Prudential, Sun Life, FWD, etc.)
- **Summarises market trends** (digital adoption, regulatory shifts, product innovation)
- **Highlights strategic implications** for Manulife HK

### Architecture Overview

```
┌──────────────────────────────────────────────────────────────┐
│  USER (CEO / CMO / Head of Strategy)                         │
│  "How does our market share compare to AIA?"                 │
│                          │                                   │
│                          ▼                                   │
│  ┌───────────────────────────────────────────┐               │
│  │  🤖 Market & Competitor Intelligence Agent│               │
│  │                                           │               │
│  │  Knowledge Sources:                       │               │
│  │  📄 HK_Insurance_Market_Report_2026.docx │               │
│  │  📄 Competitor_Analysis_Q1_2026.docx     │               │
│  │  📄 Market_Trends_Report.docx            │               │
│  │  📊 SharePoint List: MarketShareData      │               │
│  │                                           │               │
│  │  Topics:                                  │               │
│  │  1. Market Overview (Knowledge Q&A)       │               │
│  │  2. Competitor Comparison                 │               │
│  │  3. Market Share Analysis                 │               │
│  │  4. Trend Analysis                        │               │
│  └────────────────┬──────────────────────────┘               │
│                   │                                          │
│     ┌─────────────▼──────────────┐                           │
│     │  Power Automate Flow       │                           │
│     │  "Get Market Share Data"   │                           │
│     └─────────────┬──────────────┘                           │
│                   │                                          │
│     ┌─────────────▼──────────────┐                           │
│     │  SharePoint List           │                           │
│     │  MarketShareData           │                           │
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

### 1.1 Create SharePoint Document Library: `MarketIntelligence`

1. **New** → **Document library** → name it `MarketIntelligence`
2. Create and upload the following sample documents:

#### Sample Document 1: `HK_Insurance_Market_Report_2026.docx`

```
HONG KONG INSURANCE MARKET — ANNUAL REPORT 2026
(Sample / Based on publicly available industry data patterns)

1. MARKET OVERVIEW

The Hong Kong insurance market recorded total gross premiums of approximately 
HKD 530 billion in 2025, representing a 6.2% year-over-year increase. The 
life insurance segment continues to dominate, accounting for 87% of total 
premiums, while general insurance contributed 13%.

Key market drivers in 2025-2026:
- Return of Mainland Chinese visitor (MCV) business post-COVID, now at 85% 
  of 2019 levels
- VHIS (Voluntary Health Insurance Scheme) continued growth — 1.2M policies 
  in force
- Rising interest rates improving investment returns for insurers
- Regulatory reform under the Insurance Authority driving product innovation
- Digital transformation accelerating across the industry

2. MARKET SIZE BY SEGMENT

| Segment | 2025 Premium (HKD B) | YoY Growth | Market Size |
|---------|---------------------|------------|-------------|
| Individual Life | 380 | +7.1% | Largest segment |
| Group Life | 28 | +3.5% | Steady |
| ILAS | 45 | -8.2% | Declining (regulatory impact) |
| Health & Medical | 42 | +12.5% | Fastest growing |
| VHIS | 18 | +22.0% | Strong government push |
| Annuities/Retirement | 15 | +5.8% | Growing with aging population |
| General Insurance | 68 | +4.2% | Stable |

3. DISTRIBUTION CHANNEL TRENDS

| Channel | Premium Share 2025 | Premium Share 2023 | Trend |
|---------|-------------------|-------------------|-------|
| Agency | 62% | 68% | Declining share but growing absolute |
| Bancassurance | 25% | 23% | Growing, driven by bank partnerships |
| Broker | 8% | 7% | Slight increase |
| Digital / Direct | 5% | 2% | Fastest growing channel |

Key observation: Digital channel has more than doubled its market share 
in 2 years. While still small (5%), the growth trajectory suggests it 
could reach 10-15% by 2028.

4. MAINLAND CHINESE VISITOR (MCV) BUSINESS

MCV business was a major growth engine pre-COVID. Current status:
- 2019 peak: HKD 43.4B (25% of new business)
- 2025 actual: HKD 36.9B (recovery to ~85% of 2019 levels)
- Top products purchased by MCV: Whole life, critical illness, savings plans
- Key competitors for MCV business: AIA, Prudential, Manulife, FWD
- Regulatory requirements: In-person signing at HK office required

5. REGULATORY ENVIRONMENT

Key regulatory developments:
- IA continues to tighten ILAS regulation (GL15 updates)
- Risk-Based Capital (RBC) regime implementation ongoing — effective 2025
- Green insurance product guidelines expected in 2026
- Open API framework for insurance industry under consultation
- Policyholder protection fund discussions ongoing

6. TECHNOLOGY & DIGITAL TRENDS

- 72% of insurers have increased IT spending in 2025
- AI/ML adoption: underwriting (45% of insurers), claims (38%), customer 
  service (55%)
- Blockchain pilots: 3 consortium initiatives for medical claims
- Insurtech investment in HK: HKD 2.8B in 2025 (+15% YoY)
- Key areas: digital distribution, automated underwriting, claims AI, 
  customer engagement platforms
```

#### Sample Document 2: `Competitor_Analysis_Q1_2026.docx`

```
MANULIFE HK — COMPETITOR INTELLIGENCE REPORT Q1 2026
(Sample / For Demo Purposes)

1. COMPETITOR OVERVIEW

| Company | Market Rank | Estimated Market Share | Key Strength | Strategy Focus |
|---------|------------|----------------------|-------------|----------------|
| AIA | #1 | 22.5% | Strongest brand, largest agency force | Digital health ecosystem, wellness rewards |
| Manulife | #2 | 18.5% | Strong agency, trusted brand, MCV business | AI adoption, advisor productivity |
| Prudential | #3 | 15.8% | Pulse app ecosystem, bancassurance | Digital-first, health focus |
| Sun Life | #4 | 8.2% | MPF market leader, retirement expertise | Wealth management, retirement solutions |
| FWD | #5 | 7.5% | Disruptor brand, digital-native, aggressive | Simpler products, digital distribution |
| China Life (Overseas) | #6 | 6.8% | MCV specialist, mainland connections | Cross-border products |
| HSBC Life | #7 | 5.5% | Bancassurance through HSBC network | Leveraging banking relationships |
| BOC Life | #8 | 4.2% | Bank of China network | MCV and local Chinese market |

2. AIA — DETAILED ANALYSIS

Recent moves (Q1 2026):
- Launched "AIA One" — an integrated financial wellness platform combining 
  insurance, health tracking, and investment in one app
- Expanded Vitality wellness program — now covering 450,000 members in HK
- Introduced AI-powered claims assessment — average processing time reduced 
  to 3 days
- Recruited 500+ advisors in Q1 (aggressive talent acquisition)
- New product: "AIA Lifetime Protect" — guaranteed acceptance whole life 
  with simplified underwriting

Implications for Manulife:
- AIA's digital ecosystem is creating customer lock-in through wellness 
  engagement
- Their AI claims capability is setting new industry standards
- Aggressive advisor recruitment directly targets our senior advisors
- Need to respond with own digital ecosystem and advisor retention strategy

3. PRUDENTIAL — DETAILED ANALYSIS

Recent moves (Q1 2026):
- Pulse app reached 1.5M users in HK (up from 1.2M in Q4 2025)
- Partnered with ZA Bank for digital insurance distribution
- Launched micro-insurance products (HKD 10/month cancer cover)
- Expanding ESG-linked investment products
- MDRT count increased 12% YoY

Implications for Manulife:
- Pulse app engagement is driving cross-sell — average 1.8 products per 
  Pulse user vs industry average of 1.3
- Micro-insurance is reaching younger demographics (25-35) we're not capturing
- Their bancassurance partnership with ZA Bank is innovative — explores 
  new distribution models

4. FWD — DETAILED ANALYSIS

Recent moves (Q1 2026):
- Simplified product range to 12 core products (vs industry average of 40+)
- 80% of new business via digital channel (highest in HK market)
- Launched "FWD MAX" — modular insurance where customers build their own plan
- Targeted marketing to millennials and Gen Z
- Fastest-growing insurer in HK by percentage growth (+28% premium YoY)

Implications for Manulife:
- FWD is proving that simple products + digital distribution can work in HK
- Their modular approach is attracting younger customers who want flexibility
- We need a digital-first product strategy for the under-35 segment
- Their marketing is fresh and non-traditional — consider similar approach 
  for our digital channel

5. COMPETITIVE THREATS & OPPORTUNITIES

Top threats:
1. AIA's wellness ecosystem creating customer lock-in
2. FWD's digital growth disrupting traditional distribution
3. Prudential's Pulse app driving engagement and cross-sell
4. Aggressive advisor poaching by AIA and Prudential
5. Regulatory tightening of ILAS reducing a traditional Manulife strength

Top opportunities:
1. AI/automation — Manulife can leapfrog with AI underwriting and claims
2. MCV business recovery — Manulife well-positioned with mainland brand recognition
3. ESG/green products — first-mover advantage opportunity
4. Advisory-led digital hybrid model — combine advisor strength with digital tools
5. Health ecosystem — partner with existing health platforms rather than build
```

#### Sample Document 3: `Market_Trends_Report.docx`

```
HK INSURANCE MARKET — KEY TRENDS TO WATCH 2026-2028

TREND 1: AI TRANSFORMATION
- 65% of HK insurers will have production AI/ML systems by end of 2026
- Key applications: automated underwriting (30% faster), AI claims 
  triage (40% reduction in touch time), conversational AI for customer 
  service, predictive analytics for lapse prevention
- Emerging: Generative AI for sales support, document processing, 
  compliance review
- Manulife position: Above average — AI underwriting pilot complete, 
  Copilot Studio deployment in progress

TREND 2: HEALTH ECOSYSTEM CONVERGENCE
- Insurance + wellness + healthcare delivery converging into platforms
- AIA Vitality and Prudential Pulse are early examples
- Next wave: integration with telemedicine, wearable data, EHR
- By 2028: health data may influence real-time pricing
- Manulife position: Lagging — no proprietary health platform

TREND 3: DIGITAL DISTRIBUTION ACCELERATION
- Digital channel expected to reach 10-15% of premium by 2028 (from 5% today)
- Key driver: younger customers prefer online research and purchase
- Hybrid model emerging: online research → advisor-assisted closing
- Manulife position: Growing (45% YoY) but from small base

TREND 4: ESG & SUSTAINABLE INSURANCE
- IA expected to issue green insurance guidelines in 2026
- Customer demand: 42% of affluent HK customers interested in ESG-linked products
- Product innovation: green bonds as underlying assets, carbon offset riders, 
  sustainability-linked premium discounts
- Manulife position: On track — ESG retirement product approved for Q4 launch

TREND 5: CROSS-BORDER INSURANCE (GBA)
- Greater Bay Area (GBA) insurance opportunities growing
- Pilot schemes for cross-border motor insurance and health insurance
- HK-based insurers can serve GBA customers under certain conditions
- Regulatory frameworks still being developed
- Manulife position: Monitoring — not yet active participant

TREND 6: AGING POPULATION & RETIREMENT
- HK population aged 65+: 22% by 2028 (from 20% today)
- Growing demand: annuities, long-term care, medical expense protection
- Government incentives: tax deductions for qualifying annuity premiums
- Manulife position: Strong — Sun Life is primary competitor in retirement
```

### 1.2 Create SharePoint List: `MarketShareData`

1. **New** → **List** → name it `MarketShareData`
2. Add columns:

| Column Name | Type |
|---|---|
| Company | Single line of text *(rename "Title")* |
| Period | Single line of text |
| MarketShare | Number (1 decimal) |
| EstimatedPremium_B | Number (1 decimal) |
| YoYGrowth | Single line of text |
| MarketRank | Number |
| KeyStrength | Single line of text |
| RecentMove | Multiple lines of text |

3. Add sample data:

| Company | Period | MarketShare | EstimatedPremium_B | YoYGrowth | MarketRank | KeyStrength | RecentMove |
|---|---|---|---|---|---|---|---|
| AIA | Q1 2026 | 22.5 | 119.3 | +7.5% | 1 | Largest agency, wellness ecosystem | AIA One platform launch, AI claims |
| Manulife | Q1 2026 | 18.5 | 98.1 | +8.8% | 2 | Trusted brand, MCV business | AI underwriting pilot, advisor academy |
| Prudential | Q1 2026 | 15.8 | 83.7 | +9.2% | 3 | Pulse app, digital engagement | 1.5M Pulse users, ZA Bank partnership |
| Sun Life | Q1 2026 | 8.2 | 43.5 | +4.5% | 4 | MPF leader, retirement | Wealth management expansion |
| FWD | Q1 2026 | 7.5 | 39.8 | +28.0% | 5 | Digital-native, simple products | FWD MAX modular product, 80% digital sales |
| China Life Overseas | Q1 2026 | 6.8 | 36.0 | +5.2% | 6 | MCV specialist | Cross-border product expansion |
| HSBC Life | Q1 2026 | 5.5 | 29.2 | +6.1% | 7 | HSBC network | Bancassurance integration |
| BOC Life | Q1 2026 | 4.2 | 22.3 | +3.8% | 8 | BOC network | MCV and local market focus |

---

## Phase 2: Create the Agent

### 2.1 Create the Agent

1. Go to **Copilot Studio** → **Create** → **New agent**
2. Name: **"Market & Competitor Intelligence Agent"**
3. Description: *"Provides Manulife HK executives with instant access to market intelligence, competitor analysis, and strategic trend insights for the Hong Kong insurance market."*

### 2.2 Set Instructions

```
You are the Market & Competitor Intelligence Agent for Manulife Hong Kong. You help the CEO, CMO, and Strategy team understand the competitive landscape and market dynamics.

Your role:
- Answer questions about the HK insurance market using the uploaded research reports
- Provide competitor comparisons with market share, strengths, and recent strategic moves
- Summarise market trends and their implications for Manulife
- Highlight competitive threats and opportunities
- Provide Manulife's positioning relative to competitors on key dimensions

Rules:
- Always distinguish between facts (from reports) and analysis/interpretation
- When comparing competitors, use structured tables
- Include Manulife's position and strategic implications when discussing competitors or trends
- Source data is based on industry reports and internal estimates — note this when presenting market share figures
- Do not speculate on competitor financials beyond what is in the reports
- Present market share data with the caveat: "Based on industry estimates; actual figures may vary"
- Use an objective, analytical tone — avoid disparaging competitors
- When discussing trends, always include: what it means for Manulife and suggested actions
- If asked about something not in the reports, say "This specific intelligence is not available in the current reports. I recommend checking with the Strategy team or requesting an updated competitor brief."
```

### 2.3 Add Knowledge Sources

Upload or connect:
- `HK_Insurance_Market_Report_2026.docx`
- `Competitor_Analysis_Q1_2026.docx`
- `Market_Trends_Report.docx`

---

## Phase 3: Create Power Automate Flow

### 3.1 Flow: "Get Market Share Data"

1. **Trigger input**: `Company` (Text)
2. **SharePoint — Get items**:
   - List: `MarketShareData`
   - Filter Query:
     ```
     if(equals(triggerBody()?['text'], 'All'), '', concat('Title eq ''', triggerBody()?['text'], ''''))
     ```
   - Sort By: `MarketRank`
   - Sort Order: Ascending
3. **Select**:
   - Company: `item()?['Title']`
   - MarketShare: `item()?['MarketShare']`
   - Premium_B: `item()?['EstimatedPremium_B']`
   - YoYGrowth: `item()?['YoYGrowth']`
   - Rank: `item()?['MarketRank']`
   - Strength: `item()?['KeyStrength']`
   - RecentMove: `item()?['RecentMove']`
4. **Compose**: `string(body('Select'))`
5. **Return**: `MarketData` (Text)
6. Name: `Get Market Share Data` → **Save**

---

## Phase 4: Create Topics

### 4.1 Topic: "Market Overview"

Uses generative answers from knowledge — minimal flow logic.

**Trigger phrases**: `Market overview`, `How is the HK insurance market`, `Market size`, `Industry update`, `Insurance market trends`

1. **Generative answers** node linked to all three knowledge documents
2. End topic

### 4.2 Topic: "Competitor Comparison"

**Trigger phrases**: `Compare competitors`, `How does AIA compare`, `Competitor analysis`, `What is Prudential doing`, `FWD strategy`, `Market share comparison`

| Step | Node | Variable |
|---|---|---|
| 1 | Call Action: **Get Market Share Data** (Company = "All") | `Topic.MarketData` |
| 2 | **Generative answers** — prompt: `Using the market share data and competitor analysis knowledge, present a comprehensive competitor comparison. Include: (1) Market share ranking table with premium and growth, (2) Key strength of each competitor, (3) Recent strategic moves, (4) Manulife's competitive position (strengths and gaps), (5) Top 3 threats and top 3 opportunities. Market data: {Topic.MarketData}` | — |
| 3 | Message: "*Market share figures are based on industry estimates.*" | — |
| 4 | End topic | — |

### 4.3 Topic: "Market Share Analysis"

**Trigger phrases**: `Market share`, `Our market position`, `Where do we rank`, `How much market share do we have`

| Step | Node | Variable |
|---|---|---|
| 1 | Ask: "Which competitor would you like to compare against? Or 'All' for the full market." → Multiple choice: All / AIA / Prudential / Sun Life / FWD | `Topic.Competitor` |
| 2 | Call Action: **Get Market Share Data** (Company ← Topic.Competitor) | `Topic.MarketData` |
| 3 | Condition: Competitor = "All" | — |
| 4a | **Generative answers** — prompt: `Present the full market share data as a ranked table. Highlight Manulife's position. Show the gap between Manulife and #1 (AIA). Note the fastest growing competitor. Data: {Topic.MarketData}` | — |
| 4b | **Generative answers** — prompt: `Compare Manulife directly against {Topic.Competitor} across all available metrics. Show market share, premium, growth, strengths, and recent moves for both. Highlight where Manulife leads and where {Topic.Competitor} leads. Suggest actions to close any gaps. Data: {Topic.MarketData}` | — |
| 5 | End topic | — |

### 4.4 Topic: "Trend Analysis"

**Trigger phrases**: `Market trends`, `What trends should we watch`, `What's changing in insurance`, `Digital trends`, `Industry disruption`, `Future of insurance in HK`

1. **Generative answers** — configured to answer from `Market_Trends_Report.docx` knowledge source
2. End topic

---

## Phase 5: Testing

| Test Query | Expected Response |
|---|---|
| "What's our market share?" | "18.5%, ranked #2 behind AIA (22.5%)" |
| "Compare us to AIA" | Side-by-side comparison with strengths, gaps, and recommendations |
| "What is FWD doing?" | Detailed analysis of FWD's digital strategy and implications |
| "What are the key market trends?" | 6 trends with Manulife positioning and suggested actions |
| "How is the digital channel growing?" | "5% of market premium, doubled in 2 years, expected 10-15% by 2028" |
| "What threats should I worry about?" | Top 5 threats from competitor analysis with recommendations |

### Demo Script (7 minutes)

| Time | What to Show | What to Say |
|---|---|---|
| 0:00 | "How does our market share compare?" | "Executives need competitive intelligence on demand" |
| 1:30 | Show full market ranking table | "We're #2 at 18.5% — 4pp gap to AIA. FWD growing fastest at 28%" |
| 2:30 | "What is AIA doing that we should worry about?" | "Drill into any competitor for detailed analysis" |
| 3:30 | Show AIA analysis with implications | "Their wellness ecosystem is creating customer lock-in" |
| 4:30 | "What market trends should we watch?" | "Strategic planning requires market foresight" |
| 5:30 | Show trend analysis with Manulife positioning | "Each trend includes where we stand and what to do about it" |
| 6:30 | Wrap up | "All from uploaded research reports — update the docs, agent updates instantly" |
