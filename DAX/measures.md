# My DAX Measures - FoodReach UK Dashboard
**Author**: [ALLAN MPAA]
**Date**: [05/2/2026]
**Project**: FoodReach UK Impact Dashboard

**What This Document Is**

This is where I keep all my DAX formulas with notes about what they do and why I built them. If you're looking at my Power BI file and wondering "what does this measure do?", this is your answer key.

**The Basics (Counting & Adding Stuff)**

**Total Parcels**

What it does: Adds up all the food parcels we gave out.
daxTotal Parcels = SUM(Fact_FoodBank[Total_Parcels])
Why I need it: This is the main number everything else is based on. Every dashboard needs a "total" measure.
Where I use it: Everywhere! KPI cards, charts, calculations.

**Total Children Parcels**

What it does: Counts just the parcels that went to kids.
daxTotal Children Parcels = SUM(Fact_FoodBank[Children_Parcels])
Why I need it: We need to track how many children we're supporting separately.

**Total Adult Parcels**

What it does: Counts parcels for adults.

daxTotal Adult Parcels = SUM(Fact_FoodBank[Adult_Parcels])

**Pct Children**

What it does: Shows what percentage of parcels went to children.
daxPct Children = 
DIVIDE(
    [Total Children Parcels],
    [Total Parcels],
    0
)
Why DIVIDE instead of just "/": If there are zero parcels somewhere, regular division crashes with an error. DIVIDE just returns 0 instead. Learned this the hard way!
Shows as: 38.2% (meaning 38.2% of parcels support kids)

**Total Locations**

What it does: Counts how many different local authorities we operate in.
daxTotal Locations = DISTINCTCOUNT(DimLocation[Local_Authority])
Why DISTINCTCOUNT: Regular COUNT would count duplicates. DISTINCTCOUNT only counts each location once.
Current number: 44 locations

**Avg Parcels Per Location**

What it does: Average parcels per location (total ÷ number of locations).
daxAvg Parcels Per Location = 
DIVIDE(
    [Total Parcels],
    [Total Locations],
    0
)
Why it matters: Helps spot locations that are unusually busy or quiet.

**Time dax (Comparing This Year vs Last Year)**

This is where DAX gets interesting. These measures let me compare "this January" to "last January" automatically.

**Parcels Last Year**

What it does: Shows me the number from the same period last year.
    daxParcels Last Year = 
    CALCULATE(
        [Total Parcels],
        SAMEPERIODLASTYEAR(DimDate[Date])
    )
    
How it works:

CALCULATE changes what dates we're looking at
SAMEPERIODLASTYEAR shifts everything back 12 months
So if I'm viewing Jan 2024, this shows Jan 2023

First time I tried this: Spent 2 hours figuring out why it wasn't working. Turned out I hadn't marked my date table as a date table! 🤦

**YoY Growth**

What it does: Shows the difference between this year and last year (in actual numbers).
daxYoY Growth = [Total Parcels] - [Parcels Last Year]
Example: If we had 100,000 last year and 112,000 this year, growth = 12,000

**YoY Growth Pct**

What it does: Shows growth as a percentage.
daxYoY Growth Pct = 
DIVIDE(
    [YoY Growth],
    [Parcels Last Year],
    0
)
Example: 12,000 ÷ 100,000 = 12% growth
Format: Percentage with 1 decimal (shows as 12.5%)
Why this matters: Executives want percentages, not raw numbers. "Up 12%" is clearer than "up 12,000."

**The Smart One (Alert System)
High Growth Alert**

What it does: Looks at the growth rate and tells me if it's critical, high, moderate, or declining.

**daxHigh Growth Alert = 
VAR GrowthRate = [YoY Growth Pct]
RETURN
    SWITCH(
        TRUE(),
        GrowthRate > 0.20, "🔴 Critical (>20%)",
        GrowthRate > 0.10, "🟠 High (10-20%)",
        GrowthRate > 0, "🟡 Moderate (0-10%)",
        ISBLANK(GrowthRate), "⚪ No Data",
        "🟢 Declining"
    )**
    
What VAR means: It stores the growth rate so I don't have to calculate it multiple times. Like saving a result to use later.

**Why I built this:**

Originally I just had percentages. But when presenting to stakeholders, they don't want to do mental math. They want: "Is this red, amber, or green?" This answers that instantly.

Fun mistake: First time I wrote this, I put GrowthRate > 20 instead of GrowthRate > 0.20. Everything showed as declining and I was SO confused! Percentages in DAX are decimals (0.20 = 20%, not 20). Took me 20 minutes to figure that out. 😅
The thresholds:

- 🔴 Over 20% growth = CRITICAL (need immediate resources)
- 🟠 10-20% growth = HIGH (start planning)
- 🟡 0-10% growth = MODERATE (normal)
- ⚪ No data = First year or missing data
- 🟢 Negative = Declining (demand going down - good news!)


**Ranking & Comparisons**

**Region Rank by Demand**

What it does: Ranks regions from highest demand (#1) to lowest.

daxRegion Rank by Demand = 
RANKX(
    ALL(DimLocation[Region]),
    [Total Parcels],
    ,
    DESC,
    DENSE
)

Why ALL() is important:

Without it, if you filter to one region, it would always be rank #1 (because you're ranking it against... itself). ALL says "rank this region against EVERY region, even if they're filtered out."
Debugging this was painful: It kept showing "1" for every region when I applied filters. Took an hour of Googling to figure out I needed ALL().
DESC means: Highest number gets rank 1 (descending order)

**Parcels Per 1000 Population**

What it does: Shows parcels per 1,000 residents (normalizes by population size).
daxParcels Per 1000 Population = 
VAR RegionParcels = [Total Parcels]
VAR RegionPopulation = SUM(Population[Population_1000s])
RETURN
    DIVIDE(RegionParcels, RegionPopulation, 0)
    
Why this matters:

London has way more parcels than Newcastle in raw numbers. BUT Newcastle has a smaller population. Per-capita shows Newcastle actually has HIGHER need (107 parcels per 1,000 people vs London's 28).
This is the measure that changes the story! Without it, we'd just focus on big cities. With it, we see smaller areas in real crisis.
Example:

- Newcastle: 285,000 parcels ÷ 2.65 million people = 107.5 per 1,000
- London: 250,000 parcels ÷ 8.98 million people = 27.8 per 1,000
- Newcastle has 4× higher need per capita!


**How These Connect**

Some measures build on other measures. Here's the family tree:
Total Parcels (the parent measure)
  ↓
  ├─ Pct Children (uses Total Parcels)
  ├─ Avg Parcels Per Location (uses Total Parcels)
  ├─ Parcels Last Year (uses Total Parcels but for last year)
  │   ↓
  │   └─ YoY Growth (uses Parcels Last Year)
  │       ↓
  │       └─ YoY Growth Pct (uses YoY Growth)
  │           ↓
  │           └─ High Growth Alert (uses YoY Growth Pct)
  │
  ├─ Region Rank (uses Total Parcels)
  └─ Parcels Per 1000 Pop (uses Total Parcels)
Why this matters: If Total Parcels breaks, everything breaks. But if one child measure breaks, the others still work.

Testing Notes
How I tested these:

- ✅ Created a small table with known numbers
- ✅ Manually calculated what the answer should be
- ✅ Checked if my measure matched
- ✅ Tested edge cases (what if data is blank? Zero? Negative?)

What could go wrong:

- ⚠️ Time measures return blank for 2021 (no 2020 data to compare to) - this is expected
- ⚠️ Per-capita uses 2019 population numbers (not updated annually) - limitation
- ⚠️ Ranking doesn't handle ties perfectly (first alphabetically wins) - minor issue


**What I Learned Building These**

Hardest concept: Filter context (CALCULATE)

Most useful function: DIVIDE (saved me from so many errors)

Biggest "aha!" moment: Realizing measures > calculated columns

Most debugging time: 2 hours on SAMEPERIODLASTYEAR (forgot to mark date table)

Favorite measure: High Growth Alert (feels smart!)

If I built this again:

- I'd test each measure immediately instead of writing 5 then debugging
- I'd add comments in complex formulas (future me will forget why I did something)
- I'd create a test dataset first to validate logic


Still Learning
Comfortable with:

Basic aggregations (SUM, COUNT, etc.)
Safe division (DIVIDE)
Time intelligence (SAMEPERIODLASTYEAR, DATEADD)
Variables (VAR/RETURN)
Simple conditional logic (SWITCH, IF)

Still figuring out:

Complex filter combinations (multiple CALCULATE filters)
Iterator functions (SUMX, AVERAGEX) - used them but don't fully get when they're better
Performance optimization (my dataset is small, so haven't needed to worry yet)


Last updated: [05/02/2026]
Status: Working measures, ready to explain!

Note: These measures work in my specific data model. If you copy them to a different model, you'll need to update table/column names.
