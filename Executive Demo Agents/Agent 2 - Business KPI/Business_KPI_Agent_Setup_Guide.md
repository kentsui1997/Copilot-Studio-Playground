> **Author**: Ken Tsui (CSA), Microsoft
> **Copyright** © 2026 Microsoft. All rights reserved.

# Agent 2: Business Performance KPI Agent — End-to-End Setup Guide

> A step-by-step guide to build an agent that gives executives conversational access to business KPIs — new business premium, claims ratio, channel performance, and product breakdowns — without needing dashboards or reports.

---

## Use Case

Manulife HK executives frequently need quick answers to performance questions: *"How are we doing this quarter?"*, *"Which product line is underperforming?"*, *"Compare agency vs bancassurance."* Currently, they wait for reports or ask analysts. This agent provides instant, conversational access to KPI data.

### Architecture Overview

```
┌──────────────────────────────────────────────────────────────┐
│  USER (CEO / CFO / CRO)                                      │
│  "What's our new business premium for Q1?"                   │
│                          │                                   │
│                          ▼                                   │
│  ┌───────────────────────────────────────────┐               │
│  │  🤖 Business Performance KPI Agent        │               │
│  │                                           │               │
│  │  Knowledge Sources:                       │               │
│  │  📊 SharePoint List: BusinessKPIs         │               │
│  │  📊 SharePoint List: ProductPerformance   │               │
│  │  📊 SharePoint List: ChannelPerformance   │               │
│  │                                           │               │
│  │  Topics:                                  │               │
│  │  1. Overall Performance Summary           │               │
│  │  2. Product Breakdown                     │               │
│  │  3. Channel Comparison                    │               │
│  │  4. Year-over-Year Analysis               │               │
│  └────────────────┬──────────────────────────┘               │
│                   │                                          │
│         ┌─────────▼──────────┐                               │
│         │  Power Automate    │                               │
│         │  Flows:            │                               │
│         │  "Get KPI Data"    │                               │
│         │  "Get Product      │                               │
│         │   Performance"     │                               │
│         │  "Get Channel      │                               │
│         │   Performance"     │                               │
│         └─────────┬──────────┘                               │
│                   │                                          │
│         ┌─────────▼──────────┐                               │
│         │  SharePoint Lists  │                               │
│         │  (KPI Data Store)  │                               │
│         └────────────────────┘                               │
└──────────────────────────────────────────────────────────────┘
```

---

## Prerequisites

- [ ] Microsoft 365 tenant with Copilot Studio license
- [ ] SharePoint Online access
- [ ] Power Automate access

---

## Phase 1: Data Setup — SharePoint Lists

### 1.1 Create SharePoint List: `BusinessKPIs`

1. Go to your **SharePoint site** → **New** → **List** → name it `BusinessKPIs`
2. Add columns:

| Column Name | Type |
|---|---|
| MetricName | Single line of text *(rename the default "Title" column)* |
| Period | Single line of text |
| CurrentValue | Number (2 decimal places) |
| PreviousValue | Number (2 decimal places) |
| Target | Number (2 decimal places) |
| Unit | Choice: `HKD_M`, `Percentage`, `Count`, `Days`, `Score` |
| Category | Choice: `Financial`, `Growth`, `Operations`, `People`, `Customer` |
| YoYChange | Single line of text |
| VsTarget | Single line of text |

3. Add sample data:

| MetricName | Period | CurrentValue | PreviousValue | Target | Unit | Category | YoYChange | VsTarget |
|---|---|---|---|---|---|---|---|---|
| New Business Premium | Q1 2026 | 2340 | 2150 | 2200 | HKD_M | Growth | +8.8% | +6.4% |
| New Business Premium | Apr 2026 | 780 | 695 | 720 | HKD_M | Growth | +12.2% | +8.3% |
| Revenue | Q1 2026 | 4200 | 3925 | 4100 | HKD_M | Financial | +7.0% | +2.4% |
| Operating Profit | Q1 2026 | 890 | 848 | 870 | HKD_M | Financial | +5.0% | +2.3% |
| Claims Ratio | Q1 2026 | 62 | 65 | 63 | Percentage | Financial | -3pp | Within range |
| Claims Ratio | Apr 2026 | 61.8 | 63.2 | 63 | Percentage | Financial | -1.4pp | Better than target |
| Lapse Rate 13-month | Q1 2026 | 4.2 | 5.1 | 5.0 | Percentage | Growth | -0.9pp | Better than target |
| Number of New Policies | Q1 2026 | 18500 | 17200 | 18000 | Count | Growth | +7.6% | +2.8% |
| Average Case Size | Q1 2026 | 126 | 124.6 | 125 | HKD_K | Growth | +1.1% | +0.8% |
| Agency Force Size | Q1 2026 | 12800 | 12200 | 12500 | Count | People | +4.9% | +2.4% |
| Agency Force Size | Apr 2026 | 12850 | 12800 | 12500 | Count | People | +50 net | Above target |
| MDRT Qualifiers | Q1 2026 | 1450 | 1320 | 1400 | Count | People | +9.8% | +3.6% |
| Employee Engagement | Q1 2026 | 78 | 75 | 76 | Score | People | +3pts | Above target |
| Customer NPS | Q1 2026 | 72 | 69 | 70 | Score | Customer | +3pts | +2pts |
| Customer NPS | Apr 2026 | 72 | 72 | 70 | Score | Customer | Flat | Above target |
| Policy Issuance SLA | Q1 2026 | 2.1 | 2.5 | 3.0 | Days | Operations | -0.4 days | Better than target |
| Claims Processing Time | Q1 2026 | 5.2 | 5.8 | 7.0 | Days | Operations | -0.6 days | Better than target |
| Digital Adoption Rate | Q1 2026 | 68 | 55 | 60 | Percentage | Operations | +13pp | +8pp |
| Expense Ratio | Q1 2026 | 28.3 | 29.5 | 30 | Percentage | Financial | -1.2pp | Better than target |
| Capital Adequacy Ratio | Q1 2026 | 285 | 278 | 150 | Percentage | Financial | +7pp | Well above min |
| Headcount | Q1 2026 | 3200 | 3080 | 3150 | Count | People | +120 | +50 above plan |
| Voluntary Attrition | Q1 2026 | 8.2 | 9.5 | 10 | Percentage | People | -1.3pp | Better than target |

### 1.2 Create SharePoint List: `ProductPerformance`

1. **New** → **List** → name it `ProductPerformance`
2. Add columns:

| Column Name | Type |
|---|---|
| ProductLine | Single line of text *(rename "Title")* |
| Period | Single line of text |
| Premium_HKD_M | Number |
| YoYChange | Single line of text |
| MarketShare | Number (1 decimal) |
| GrowthDriver | Single line of text |
| Concern | Single line of text |

3. Add sample data:

| ProductLine | Period | Premium_HKD_M | YoYChange | MarketShare | GrowthDriver | Concern |
|---|---|---|---|---|---|---|
| Life Insurance | Q1 2026 | 1260 | +15% | 18.5 | Strong whole life demand | None |
| Life Insurance | Apr 2026 | 420 | +15% | 18.5 | Whole life products | None |
| Health Insurance | Q1 2026 | 555 | +8% | 14.2 | VHIS adoption | Rising medical claims |
| Health Insurance | Apr 2026 | 185 | +8% | 14.2 | VHIS driving growth | Rising medical claims in VHIS tier |
| Critical Illness | Q1 2026 | 330 | +5% | 12.8 | Steady demand | Product refresh needed |
| Critical Illness | Apr 2026 | 110 | +5% | 12.8 | Steady performance | None |
| ILAS | Q1 2026 | 135 | -12% | 8.1 | None | IA regulatory changes |
| ILAS | Apr 2026 | 45 | -12% | 8.1 | None | Regulatory changes impacting sales |
| Retirement/MPF | Q1 2026 | 60 | +3% | 6.5 | Stable contributions | Low growth |
| Retirement/MPF | Apr 2026 | 20 | +3% | 6.5 | Stable | Low growth |

### 1.3 Create SharePoint List: `ChannelPerformance`

1. **New** → **List** → name it `ChannelPerformance`
2. Add columns:

| Column Name | Type |
|---|---|
| Channel | Single line of text *(rename "Title")* |
| Period | Single line of text |
| Premium_HKD_M | Number |
| YoYChange | Single line of text |
| PremiumShare | Number (1 decimal) |
| TopPerformers | Multiple lines of text |
| Notes | Multiple lines of text |

3. Add sample data:

| Channel | Period | Premium_HKD_M | YoYChange | PremiumShare | TopPerformers | Notes |
|---|---|---|---|---|---|---|
| Agency | Q1 2026 | 1740 | +16% | 74.4 | Central, Kowloon East, NT West | Strong recruitment and productivity gains |
| Agency | Apr 2026 | 580 | +16% | 74.4 | Central, Kowloon East, NT West | Top districts driving growth |
| Bancassurance | Q1 2026 | 450 | -3% | 19.2 | DBS, Hang Seng | Seasonal slowdown |
| Bancassurance | Apr 2026 | 150 | -3% | 19.2 | DBS, Hang Seng | Expected recovery in May |
| Digital | Q1 2026 | 150 | +45% | 6.4 | Online term life, VHIS | Fastest growing channel |
| Digital | Apr 2026 | 50 | +45% | 6.4 | Online term life sales | Surging online demand |

---

## Phase 2: Create the Agent

### 2.1 Create the Agent

1. Go to **Copilot Studio** → **Create** → **New agent**
2. Name: **"Business Performance KPI Agent"**
3. Description: *"Provides conversational access to Manulife HK business performance data including KPIs, product performance, and channel breakdowns."*

### 2.2 Set Instructions

```
You are the Business Performance KPI Agent for Manulife Hong Kong. You help executives quickly access and understand business performance data.

Your role:
- Answer questions about key business metrics (new business premium, claims ratio, lapse rate, NPS, etc.)
- Break down performance by product line (Life, Health, Critical Illness, ILAS, Retirement)
- Compare channel performance (Agency, Bancassurance, Digital)
- Show year-over-year comparisons and performance vs targets
- Highlight areas of strength and concern

Rules:
- Always present data with context: current value, comparison (YoY or vs target), and brief insight
- Use structured tables when comparing multiple items
- When asked "how are we doing", provide a top-line summary with the 5 most important metrics
- Always specify the time period for any metric you quote
- Use HKD with appropriate units: K (thousands), M (millions), B (billions)
- If a metric is performing worse than target or previous period, flag it clearly
- Do not speculate on causes — state what the data shows and note if further analysis is needed
- If asked about data not available, say "This metric is not currently tracked in the system. You may want to check with [Finance/Operations/HR]."
```

---

## Phase 3: Create Power Automate Flows

### 3.1 Flow: "Get KPI Summary"

1. In Copilot Studio → **Topics** → open a topic → **+** → **Call an action** → **Create a flow**
2. **Trigger input**: `Period` (Text)
3. **SharePoint — Get items**:
   - Site: your SharePoint URL
   - List: `BusinessKPIs`
   - Filter Query (Expression):
     ```
     concat('Period eq ''', triggerBody()?['text'], '''')
     ```
   - Top Count: 50
4. **Select** (Data Operations):
   - From: `value` (from Get items)
   - Map:
     - Metric: `item()?['Title']`
     - Value: `item()?['CurrentValue']`
     - Unit: `item()?['Unit']?['Value']`
     - YoY: `item()?['YoYChange']`
     - VsTarget: `item()?['VsTarget']`
     - Category: `item()?['Category']?['Value']`
5. **Compose** — "Format Results":
   - Expression: `string(body('Select'))`
6. **Return**: `KPIData` (Text) = Outputs from "Format Results"
7. Name: `Get KPI Summary` → **Save**

### 3.2 Flow: "Get Product Performance"

1. **Trigger input**: `Period` (Text)
2. **SharePoint — Get items**:
   - List: `ProductPerformance`
   - Filter Query: `concat('Period eq ''', triggerBody()?['text'], '''')`
3. **Select**:
   - Product: `item()?['Title']`
   - Premium: `item()?['Premium_HKD_M']`
   - YoY: `item()?['YoYChange']`
   - MarketShare: `item()?['MarketShare']`
   - Driver: `item()?['GrowthDriver']`
   - Concern: `item()?['Concern']`
4. **Compose**: `string(body('Select'))`
5. **Return**: `ProductData` (Text)
6. Name: `Get Product Performance` → **Save**

### 3.3 Flow: "Get Channel Performance"

1. **Trigger input**: `Period` (Text)
2. **SharePoint — Get items**:
   - List: `ChannelPerformance`
   - Filter Query: `concat('Period eq ''', triggerBody()?['text'], '''')`
3. **Select**:
   - Channel: `item()?['Title']`
   - Premium: `item()?['Premium_HKD_M']`
   - YoY: `item()?['YoYChange']`
   - Share: `item()?['PremiumShare']`
   - TopPerformers: `item()?['TopPerformers']`
   - Notes: `item()?['Notes']`
4. **Compose**: `string(body('Select'))`
5. **Return**: `ChannelData` (Text)
6. Name: `Get Channel Performance` → **Save**

---

## Phase 4: Create Topics

### 4.1 Topic: "Overall Performance Summary"

**Trigger phrases**: `How are we doing`, `Performance summary`, `KPI overview`, `How's the business`, `Give me the numbers`, `Quarterly update`

**Build the flow:**

| Step | Node | Variable |
|---|---|---|
| 1 | Message: "I'll pull the latest performance data for you." | — |
| 2 | Ask: "Which period would you like to see?" → Multiple choice: Q1 2026 / Apr 2026 | `Topic.Period` |
| 3 | Call Action: **Get KPI Summary** (Period ← Topic.Period) | `Topic.KPIData` |
| 4 | **Generative answers** node — prompt: `Based on the following KPI data, provide a concise executive summary highlighting the top 5 metrics, any areas of concern, and overall assessment. Data: {Topic.KPIData}. Format with bullet points and include the metric value, comparison, and a brief insight for each.` | — |
| 5 | Message: "Would you like to drill into product performance, channel breakdown, or another area?" | — |
| 6 | Ask: → Multiple choice: Product breakdown / Channel comparison / That's all | `Topic.DrillDown` |
| 7 | Condition: DrillDown = "Product breakdown" → **Redirect** to "Product Breakdown" topic | — |
| 8 | Condition: DrillDown = "Channel comparison" → **Redirect** to "Channel Comparison" topic | — |
| 9 | Else: Message "Got it. Let me know if you need anything else." → End topic | — |

### 4.2 Topic: "Product Breakdown"

**Trigger phrases**: `Product performance`, `Which product is growing`, `Product breakdown`, `How is Life doing`, `ILAS performance`, `Product comparison`

| Step | Node | Variable |
|---|---|---|
| 1 | Message: "Let me pull the product performance data." | — |
| 2 | Ask: "Which period?" → Multiple choice: Q1 2026 / Apr 2026 | `Topic.Period` |
| 3 | Call Action: **Get Product Performance** (Period ← Topic.Period) | `Topic.ProductData` |
| 4 | **Generative answers** — prompt: `Present the following product performance data in a clear comparison table, then highlight: (1) best performing product, (2) product of concern, (3) key insight. Data: {Topic.ProductData}` | — |
| 5 | Message: "Would you like to compare channels or see overall KPIs?" | — |
| 6 | End topic | — |

### 4.3 Topic: "Channel Comparison"

**Trigger phrases**: `Channel performance`, `Agency vs bancassurance`, `Channel comparison`, `How is the agency doing`, `Digital channel`, `Compare channels`

| Step | Node | Variable |
|---|---|---|
| 1 | Message: "Let me get the channel performance data." | — |
| 2 | Ask: "Which period?" → Multiple choice: Q1 2026 / Apr 2026 | `Topic.Period` |
| 3 | Call Action: **Get Channel Performance** (Period ← Topic.Period) | `Topic.ChannelData` |
| 4 | **Generative answers** — prompt: `Present the following channel performance data in a comparison table, then highlight: (1) strongest channel, (2) channel needing attention, (3) fastest growing channel, (4) top performing districts/partners. Data: {Topic.ChannelData}` | — |
| 5 | End topic | — |

---

## Phase 5: Testing

### 5.1 Test Queries

| Test Query | Expected Response |
|---|---|
| "How are we doing this quarter?" | Top-line KPI summary for Q1 2026 with key metrics |
| "What's our new business premium?" | "HKD 2,340M for Q1 2026, +8.8% YoY, 6.4% above target" |
| "Which product is underperforming?" | "ILAS is down 12% YoY due to IA regulatory changes" |
| "Compare agency and bancassurance" | Table comparing the three channels with insights |
| "What's the digital channel doing?" | "HKD 150M in Q1, up 45% YoY — fastest growing channel" |
| "What's our claims ratio?" | "62% for Q1 2026, improved 3pp YoY, within target range" |

### 5.2 Demo Script (8 minutes)

| Time | What to Show | What to Say |
|---|---|---|
| 0:00 | Type: "How are we doing this quarter?" | "Instead of opening a dashboard or waiting for a report, just ask" |
| 1:30 | Show KPI summary with insights | "Instant access to all your key metrics with context" |
| 2:30 | Type: "Which product is underperforming?" | "You can ask natural language questions about the data" |
| 3:30 | Show product breakdown table | "ILAS flagged immediately — down 12% due to regulatory changes" |
| 4:30 | Type: "Compare agency vs bancassurance" | "Compare any dimensions conversationally" |
| 5:30 | Show channel comparison | "Agency leading at 74%, digital growing fastest at 45% YoY" |
| 6:30 | Type: "What's our NPS?" | "Quick lookups for any specific metric" |
| 7:30 | Show build experience briefly | "All connected to SharePoint — data team updates the lists, agent always current" |

---

## Troubleshooting

| Issue | Cause | Solution |
|---|---|---|
| Flow returns empty data | Filter query not matching period text | Ensure Period values in SharePoint exactly match choices (e.g., `Q1 2026` not `Q1-2026`) |
| Agent shows raw JSON | Generative answers node not processing data | Check that the data is passed correctly in the prompt; add formatting instructions |
| Wrong period data returned | User typed period differently | Use multiple choice questions instead of free text for period selection |
| "Select" action empty | Get items returned no rows | Verify SharePoint list has data for the selected period |
