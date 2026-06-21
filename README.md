# 📞 Call Center Performance Dashboard

> Excel data analysis project analyzing 5,000 call records using Power Query, Power Pivot, and DAX, featuring an interactive multi-page dashboard covering call performance, resolution, agent efficiency, and customer satisfaction.

An end-to-end Excel data analysis project that uncovers where calls are being lost, why issues go unresolved, and how individual agents compare in speed, satisfaction, and workload.

The entire pipeline — from raw data to interactive dashboard — was built inside Microsoft Excel using:

- **Power Query** for data ingestion, cleaning, and transformation
- **Power Pivot** for building a relational Data Model
- **DAX** for calculated KPIs and measures across call, topic, and agent performance
- **PivotCharts + Slicers** for interactive visualization

**Key business outcomes:**

- Identified that 19% of incoming calls go unanswered, with Monday carrying the highest call volume of the week
- Found that 41%+ of calls fall into the "Long Delay" handling-speed category, directly impacting satisfaction
- Pinpointed Technical Support and Streaming as the topics with the highest unresolved rates
- Surfaced clear performance gaps between agents — from satisfaction to handle time to call volume

## 📋 Table of Contents

- [Dataset Description](#-dataset-description)
- [Data Model — Star Schema](#️-data-model--star-schema)
- [ETL Process — Power Query](#️-etl-process--power-query)
- [DAX Measures](#-dax-measures)
- [Dashboard Pages & Analysis](#-dashboard-pages--analysis)
- [Key Insights](#-key-insights)
- [Recommendations](#-recommendations)
- [Tools & Technologies](#️-tools--technologies)
- [File Structure](#-file-structure)

## 📦 Dataset Description

The raw data contains one record per call, and includes the following fields:

| Field | Description |
|---|---|
| Call Id | Unique identifier for each call |
| Agent | Agent who handled the call |
| Date | Date the call took place |
| Time / Hour | Time and hour of day the call occurred |
| Topic | Reason for the call (Streaming, Technical Support, Payment related, Admin Support, Contract related) |
| Answered (Y/N) | Whether the call was answered |
| Resolved | Whether the customer's issue was resolved |
| Speed of Answer | Time taken to answer the call, in seconds |
| Satisfaction Rating | Customer satisfaction score (1–5) |
| AvgTalkDuration | Talk duration of the call |
| DurationTalk Band | Duration grouped into bands (e.g. 121–150s) |

**Calculated columns added:** Handling Speed Category (Within Target / Acceptable / Moderate / Long / Critical Delay), Weekday, Month

## 🗂️ Data Model — Star Schema

The data is modeled in Power Pivot, with one central fact table connected to three dimension tables via surrogate keys.

[![Star Schema](https://github.com/Huda-Talaat/Call-Center-Analysis-PWC/raw/main/StarSchema.jpg)](https://github.com/Huda-Talaat/Call-Center-Analysis-PWC/raw/main/StarSchema.jpg)

**Tables:**

**PWC CallCenter** (fact table)
- Contains all transactional fields: Call Id, Date, Time, Hour, Answered (Y/N), Resolved, Speed of answer in..., Satisfaction rating, AvgTalkDuration, DurationTalk Band, Custom
- Foreign keys: Agent Key, Topic Key

**Dim Agent** — Agent Key → Agent
**Dim Topic** — Topic Key → Topic, TotalCalls
**Dim Calender** — Date, Month, Month Name, Day Name — drives weekday and monthly trend analysis

**Why this model?**

- Enables fast aggregations via PivotTables across calls, topics, and agents simultaneously
- Clean separation between descriptive attributes (dimensions) and call-level facts
- Makes DAX measures simpler and more readable
- Allows slicers (Call Status, Weekday, Resolved, Topic, Month, Agent) to filter every visual at once

## ⚙️ ETL Process — Power Query

Before loading data into the model, Power Query was used to clean and prepare the data:

1. **Data type enforcement** — dates and times parsed correctly, numeric fields validated
2. **Handle nulls** — blank Topic values isolated and excluded from topic-level analysis
3. **Surrogate key generation** — integer keys added to Agent and Topic dimension tables
4. **Dimension table extraction** — unique Agent and Topic values extracted into separate dimension tables
5. **Calculated columns added:**
   - `Handling Speed Category` — Within Target, Acceptable Delay, Moderate Delay, Long Delay, Critical Delay (based on Speed of Answer)
   - `Duration Band` — calls grouped into 30-second talk-duration bands
   - `Weekday` / `Month Name` — extracted from Date for trend analysis

## 📐 DAX Measures

Key calculated measures used across the dashboard:

```
-- Total Calls
Total Calls = COUNTROWS('PWC CallCenter')

-- Answered Calls
Answered Calls = CALCULATE([Total Calls], 'PWC CallCenter'[Answered (Y/N)] = "Yes")

-- Answered Rate
Answered Rate = DIVIDE([Answered Calls], [Total Calls])

-- Not-Answered Calls
Not-Answered Calls = [Total Calls] - [Answered Calls]

-- Resolved Calls
Resolved Calls = CALCULATE([Answered Calls], 'PWC CallCenter'[Resolved] = "Yes")

-- Resolved Rate
Resolved Rate = DIVIDE([Resolved Calls], [Answered Calls])

-- AVG Talk Duration (Minutes)
AVG TalkDuration-Minutes = AVERAGE('PWC CallCenter'[AvgTalkDuration]) / 60

-- ASA (Average Speed of Answer)
ASA-Seconds = AVERAGE('PWC CallCenter'[Speed of answer in...])

-- AVG Satisfaction
AVG Satisfaction = AVERAGE('PWC CallCenter'[Satisfaction rating])

-- AVG Calls per Agent
AVG No.of Calls/Agent = DIVIDE([Total Calls], DISTINCTCOUNT('PWC CallCenter'[Agent Key]))
```

## 📊 Dashboard Pages & Analysis

All pages share the same **Call Status**, **Weekday**, **Resolved**, **Topic**, **Month**, and **Agent** slicers, so every visual updates together when a filter is applied.

### Home Page

Navigation hub for the five dashboard pages.

[![Home Page](https://github.com/Huda-Talaat/Call-Center-Analysis-PWC/raw/main/Home%20Page.jpg)](https://github.com/Huda-Talaat/Call-Center-Analysis-PWC/raw/main/Home%20Page.jpg)

### Overview

High-level summary of call volume, service level, and satisfaction.

[![Overview](https://github.com/Huda-Talaat/Call-Center-Analysis-PWC/raw/main/Overview.jpg)](https://github.com/Huda-Talaat/Call-Center-Analysis-PWC/raw/main/Overview.jpg)

**KPIs:** Total Calls (5,000) — Service Level (26%) — AVG Satisfaction (3.40) — ASA (68 seconds)

**Total Calls by Topic** — Streaming leads at 1,022, followed closely by Technical Support (1,019), Payment related (1,007), and Admin Support / Contract related (976 each).

**Total Calls by Weekday** — Monday is the busiest day (770 calls), followed by Saturday (768); Wednesday is the quietest (679).

### Call Analysis

Deep dive into answer rate, resolution rate, and handling speed.

[![Call Analysis](https://github.com/Huda-Talaat/Call-Center-Analysis-PWC/raw/main/Call%20Analysis.jpg)](https://github.com/Huda-Talaat/Call-Center-Analysis-PWC/raw/main/Call%20Analysis.jpg)

**Purpose:** Understand how many calls are actually being answered and resolved, and how fast.

**Answered Rate** — 81% of calls were answered (4,054 calls); 19% were not answered (946 calls).

**Resolved Rate** — Of answered calls, 73% were resolved (3,646) and 27% remained unresolved (1,354).

**Handling Speed Category** — 2,082 calls (the largest share) fell into "Long Delay," followed by Moderate Delay (724) and Acceptable Delay (698). Only 374 calls were "Within Target," and 176 fell into "Critical Delay."

### More Details

Breakdown of call status by month, hour, and duration band.

[![More Details](https://github.com/Huda-Talaat/Call-Center-Analysis-PWC/raw/main/More%20Details.jpg)](https://github.com/Huda-Talaat/Call-Center-Analysis-PWC/raw/main/More%20Details.jpg)

**AHT (Average Handle Time)** — 3.75 minutes overall (225 seconds).

**Call Status by Month** — Answered calls trend slightly downward across the quarter (1,455 in Jan → 1,301 in Mar), while unanswered calls stay fairly flat (~310–320/month).

**Call Status by Hour** — Call volume peaks between 11 AM–1 PM and tapers off sharply after 5 PM.

### Agent

Compares all 8 agents across satisfaction, handle time, call volume, and resolution.

[![Agent](https://github.com/Huda-Talaat/Call-Center-Analysis-PWC/raw/main/Agent.jpg)](https://github.com/Huda-Talaat/Call-Center-Analysis-PWC/raw/main/Agent.jpg)

**Purpose:** Identify which agents are excelling and which need support.

**Satisfaction Rate by Agent** — Martha leads at 3.47, followed by Dan (3.45); Joe is lowest at 3.33.

**AHT by Agent** — Dan has the highest average handle time (231s), while Diane is the most efficient (219s).

**Call Volume by Agent** — Jim handles the most calls (666), Stewart the fewest (582); average is 625 calls per agent.

**Call Resolved by Agent** — Resolution rates are fairly consistent across agents, ranging from 73% to 76%.

### Insights & Recommendations

Key findings and actionable recommendations derived from the full analysis.

[![Report](https://github.com/Huda-Talaat/Call-Center-Analysis-PWC/raw/main/Report.jpg)](https://github.com/Huda-Talaat/Call-Center-Analysis-PWC/raw/main/Report.jpg)

## 🔍 Key Insights

1. **81% of calls were answered** (4,054 calls); the remaining 19% (946 calls) were abandoned — a relatively high miss rate.
2. **Monday carries the highest call volume** of the week (770 calls), while agents are evenly distributed across all weekdays — suggesting the unanswered rate is a staffing/scheduling issue, not an agent-performance issue.
3. **Of answered calls, only 73% were resolved**, leaving 27% of customers without a solution.
4. **Technical Support and Streaming have the highest unresolved rates** among all topics, pointing to a knowledge or training gap specific to these two areas.
5. **41%+ of all calls fall into "Long Delay"** handling speed, the single largest category — and a likely driver of the relatively low overall satisfaction score (3.40/5).
6. **Average Speed of Answer is 67.5 seconds** — over a minute before a customer is even connected to an agent.
7. **Martha has the highest satisfaction score (3.47)** with a reasonable handle time, while **Dan is the fastest in AHT** without sacrificing satisfaction — proving speed and quality aren't mutually exclusive.
8. **Joe has the lowest satisfaction score (3.33)** among all agents, flagging a need for coaching or support.

## 💡 Recommendations

1. **Reduce Unanswered Calls** — Re-evaluate staffing and shift distribution, particularly around Monday's peak volume; consider optimizing the IVR system to reduce wait-driven abandonment.
2. **Cut Down Long Delays** — Target average Speed of Answer to under 40 seconds by increasing staffing during peak hours (11 AM–1 PM).
3. **Close the Resolution Gap** — Build focused training tracks for Technical Support and Streaming, the two topics with the highest unresolved rates.
4. **Support Lower-Performing Agents** — Provide targeted coaching for agents with lower satisfaction scores, using Martha and Dan's approach as a benchmark.
5. **Balance Workload** — Review the gap between Jim's call volume (666) and Stewart's (582) to ensure fair and sustainable distribution.
6. **Run Continuous Follow-up** — Conduct weekly reviews of top- and bottom-performing agents, and re-measure resolution and satisfaction rates monthly to track improvement.

## 🛠️ Tools & Technologies

| Tool | Purpose |
|---|---|
| Microsoft Excel | Primary development environment |
| Power Query | ETL — data cleaning & transformation |
| Power Pivot | In-memory data model, relationships |
| DAX | Calculated measures and KPIs across calls, topics, and agents |
| PivotCharts | Interactive dashboard visuals |

## 📁 File Structure

```
Call-Center-Analysis-PWC/
│
├── PWC_Analysis.xlsm            # Main Excel file (dashboard + data model)
└── README.md                    # Project documentation
```
