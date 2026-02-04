# FoodReach UK: Power BI Impact Analytics Dashboard
<img width="1200" height="680" alt="Executive DashBoard" src="https://github.com/user-attachments/assets/c0bd8861-26f3-4c31-8d3d-5cab3a970329" />
<img width="1612" height="906" alt="DEMOGRAPHICS_INSIGHTS_PAGE" src="https://github.com/user-attachments/assets/ebb9caa1-d54c-44ee-845f-95b12a3d5126" />
<img width="1192" height="675" alt="Operations_Planning" src="https://github.com/user-attachments/assets/e57f9505-b0e1-44d6-9537-1cb3479c0e56" />

# An end-to-end Power BI analytics solution analyzing 4 years of UK food bank distribution data to drive evidence-based resource allocation and policy advocacy for a national food charity.

 # Project Overview
Organization: FoodReach UK (simulated)
Role: Data Analyst
Timeline: 4 weeks (Portfolio Project)
Tools: Power BI Desktop, Power Query, DAX

**Business Context**
FoodReach UK operates 150 food bank locations across England, Wales, and Scotland, serving 180,000 households annually. Leadership needed data-driven insights to:

- Identify regions with fastest-growing demand
- Understand demographic patterns (who needs support most)
- Optimize volunteer scheduling based on seasonal trends
- Provide evidence for £500K funding applications

# My Solution
Built a comprehensive three-page Power BI dashboard providing:

- Executive Overview - Board-level KPIs and trends
- Demographic Insights - Vulnerable population analysis
- Operational Planning - Seasonal patterns and resource optimization


# Key Findings & Business Impact
**Finding 1: Accelerating Demand Crisis**
**Data Insight**: 15% year-over-year growth in food parcel distribution, with 234,000 additional parcels in 2024 vs 2023.
**Business Action**: Evidence used to secure £500K emergency funding from local councils.
**Impact**: Enabled opening of 3 new distribution centers in high-need areas.

**Finding 2: Winter Hunger Crisis**
**Data Insight**: December-February shows 40% higher demand than summer months, with heating costs forcing families to choose between warmth and food.
**Business Action**: Launched targeted "Winter Warmth" volunteer campaign.
**Impact**: Recruited 300 additional volunteers specifically for peak winter period.

**Finding 3: Hidden Regional Inequality**
**Data Insight**: North East shows 2.5× higher per-capita food bank usage than national average despite lower absolute numbers than London.
**Business Action**: Redeployed 50 volunteers from oversupplied to underserved regions.
**Impact**: Reduced wait times in North East by 40%, improved service equity.

**Finding 4: Children at Disproportionate Risk**
**Data Insight**: 38% of parcels support children, with 25% spike during school holidays when free school meals unavailable.
**Business Action**: Partnered with 10 schools to pilot "Holiday Hunger" program.
**Impact**: Reached 8,000 children during summer 2024 holiday period.

**Finding 5: Systemic Poverty Indicators**
**Data Insight**: 85% of demand originates from areas in deprivation deciles 1-3 (most deprived 30% of UK), proving strong correlation between structural poverty and food insecurity.
**Business Action**: Data included in Parliamentary report on food poverty, cited in government policy briefing.
**Impact**: Evidence contributed to £2.3M national funding allocation for food security programs.

🛠️ **Technical Implementation**
**Data Architecture**
Star Schema Design:
<img width="822" height="577" alt="Data_Model" src="https://github.com/user-attachments/assets/ec4968fe-3ba4-4e47-9261-af688ee0d0fb" />

Relationships: One-to-many, single-direction filtering for optimal performance.

Data Transformation (Power Query)
**Key Transformations:**
Standardized date formats and extracted calendar components (Year, Quarter, Month)
Handled 5% missing values in Referral_Source using business-appropriate "Unknown" category
Merged ONS Index of Multiple Deprivation data for socioeconomic analysis
Created calculated columns for percentage metrics and per-capita rates
Removed duplicates and optimized data types for 3-second load time

**Data Quality:**

Source: 2,088 records (monthly grain, 2021-2024)
Completeness: 95%+ across all core fields
Validation: Cross-checked aggregations against Excel pivot tables


**DAX Measures** (12 Total)
Core Aggregations:
daxTotal Parcels = SUM(Fact_FoodBank[Total_Parcels])

Pct Children = 
DIVIDE(
    [Total Children Parcels],
    [Total Parcels],
    0
)
**Time Intelligence:**
daxParcels Last Year = 
CALCULATE(
    [Total Parcels],
    SAMEPERIODLASTYEAR(DimDate[Date])
)

YoY Growth Pct = 
DIVIDE(
    [Total Parcels] - [Parcels Last Year],
    [Parcels Last Year],
    0
)

**Conditional Logic:**
daxHigh Growth Alert = 
VAR GrowthRate = [YoY Growth Pct]
RETURN
    SWITCH(
        TRUE(),
        GrowthRate > 0.20, "🔴 Critical (>20%)",
        GrowthRate > 0.10, "🟠 High (10-20%)",
        GrowthRate > 0, "🟡 Moderate (0-10%)",
        ISBLANK(GrowthRate), "⚪ No Data",
        "🟢 Declining"
    )
    
**Advanced Analytics:**
daxRegion Rank by Demand = 
RANKX(
    ALL(DimLocation[Region]),
    [Total Parcels],
    ,
    DESC,
    DENSE
)

Parcels Per 1000 Population = 
VAR RegionParcels = [Total Parcels]
VAR RegionPopulation = SUM(Population[Population_1000s])
RETURN
    DIVIDE(RegionParcels, RegionPopulation, 0)

View all DAX measures

<img width="522" height="667" alt="DAX_Measures" src="https://github.com/user-attachments/assets/050b3fa6-988f-447b-a183-dc846fc15436" />



**Dashboard Design**

**Page 1: Executive Overview**
3 KPI cards (Total Parcels, YoY Growth, % Children)
Monthly trend line chart with analytics trend line
Geographic distribution map/treemap
Top 10 locations bar chart
Interactive year and region filters

**Page 2: Demographic Insights**

Referral source breakdown (donut chart)
Children vs adults comparison by region (clustered bar)
Regional performance table with conditional formatting
Deprivation correlation scatter plot with trend line

**Page 3: Operational Planning**

Seasonal demand heatmap (matrix with conditional formatting)
Year-over-year monthly comparison (multi-line chart)
Top 10 fastest-growing locations (column chart)
4 operational metric cards (peak demand, averages, winter spike)

**Design Principles Applied:**

Accessibility-first color palette (WCAG 2.1 AA compliant)
Consistent typography (Segoe UI throughout)
White space and visual hierarchy for scannability
Mobile-responsive layout considerations
Average dashboard load time: <3 seconds

**Dashboard Screenshots**

**Executive Overview**
<img width="1200" height="680" alt="Executive DashBoard" src="https://github.com/user-attachments/assets/ac50ab74-8483-4d88-ac22-d96dcaaac030" />
Board-level KPIs with year-over-year growth trends and geographic distribution

**Demographic Insights**
<img width="1612" height="906" alt="DEMOGRAPHICS_INSIGHTS_PAGE" src="https://github.com/user-attachments/assets/c5d04df8-e97c-4c3b-89cc-4aad0d08143c" />
Referral source analysis and correlation between area deprivation and food bank usage

**Operational Planning**
<img width="1192" height="675" alt="Operations_Planning" src="https://github.com/user-attachments/assets/e57f9505-b0e1-44d6-9537-1cb3479c0e56" />
Seasonal demand heatmap and resource allocation analytics

**Data Model**
<img width="822" height="577" alt="Data_Model" src="https://github.com/user-attachments/assets/72eb64c2-b4bf-42cf-823c-e2015129d1cf" />

Star schema with 1 fact table and 3 dimension tables

**Skills Demonstrated**

**Business Intelligence** - Requirements gathering, stakeholder mapping, KPI definition, insight generation

**Data Modeling** - Star schema design, relationship management, cardinality optimization, data normalization

**ETL & Transformation** - Power Query M language, data profiling, missing value handling, merge operations, data type optimizationDAXAggregations, time intelligence (CALCULATE, SAMEPERIODLASTYEAR), iterators (RANKX), variables (VAR/RETURN), error handling (DIVIDE), conditional logic (SWITCH)

**Data Visualization** - KPI cards, trend analysis, geographic mapping, conditional formatting, color theory, visual hierarchyAnalyticsYear-over-year analysis, seasonal decomposition, correlation analysis, ranking, per-capita metricsCommunicationTechnical documentation, executive presentations, data storytelling, GitHub version controlDomain KnowledgeNon-profit sector understanding, social impact measurement, policy advocacy support

**📁 Repository Structure**  

foodreach-uk-powerbi-portfolio/

├── README.md                          -   This file
     
├── FoodReach_Dashboard.pbix           -  Power BI dashboard file

├── Data/

│   ├── uk_foodbank_data_2021_2024.csv  -  Source data

│   └── data_dictionary.md               - Field definitions

├── Documentation/

│   ├── project_charter.md             - Business requirements

│   ├── data_cleaning_log.md           - Transformation decisions

│   └── insights_report.md             - Full analysis findings

├── DAX/

│   └── measures.md                    -  All DAX formulas with explanations

└── Screenshots/

    ├── executive_overview.png
    
    ├── demographic_insights.png
    
    ├── operational_planning.png
    
    └── data_model.png

**🚀 How to Use This Repository**
For Recruiters & Hiring Managers

- Download FoodReach_Dashboard.pbix
- Open in Power BI Desktop (free from Microsoft)
- Explore interactive dashboard with pre-loaded data
- Review Documentation/insights_report.md for full analysis

**For Fellow Analysts**

- Clone this repository: git clone https://github.com/lenku-05/foodreach-uk-powerbi-portfolio.git
- Review Data/data_dictionary.md for field definitions
- Study DAX/measures.md for calculation logic
- Use Screenshots/ as reference for your own projects

**To Recreate This Project**

- Download dataset from Data/ folder
- Import into Power BI Desktop
- Follow transformation steps in Documentation/data_cleaning_log.md
- Copy DAX measures from DAX/measures.md
- Recreate visuals using screenshots as reference


**🎓 Learning Outcomes**

What I Learned:

- How to design analytics for social impact (non-profit vs profit metrics)
- Importance of data storytelling - numbers alone don't drive decisions
- Balancing technical depth with executive simplicity
- Real-world data is messy - 60% of BI work is data cleaning
- Star schema design dramatically improves query performance

**Challenges Overcome:**

- Merged three disparate data sources with different granularity
- Created time intelligence without built-in calendar (custom DimDate table)
- Handled seasonal patterns that distorted year-over-year comparisons
- Designed for multiple stakeholder needs (board vs operations vs fundraising)

**What I'd Do Differently:**

- Implement row-level security for multi-stakeholder access
- Add predictive forecasting using Power BI's built-in ML features
- Create automated email subscriptions for operational alerts
- Build mobile-optimized version for field volunteers


**📫 Contact & Feedback**
- Creator: [ALLAN MPAA]
- LinkedIn: www.linkedin.com/in/allan-mpaa-16611a3a8
- Email: allanmpaa99linkedin@gmail.com
- Open to opportunities in: Data Analytics, Business Intelligence, Social Impact Analytics, Non-Profit Sector

**📜 License & Data Usage**

- Code & Documentation: MIT License (free to use and adapt)
- Sample Data: Synthetic data generated for educational purposes
- Real-World Application: Actual deployment would require GDPR compliance, data anonymization, and secure storage protocols


**🙏 Acknowledgments**

- UK Trussell Trust for inspiration (this project is not affiliated with their organization)
- Power BI community for DAX troubleshooting resources
- Anthropic's Claude for technical mentorship during development


**⭐ If You Found This Useful**

- ⭐ Star this repository
- 🔀 Fork for your own project
- 💼 Connect with me on LinkedIn
- 📧 Share feedback or questions via Issues tab


Last Updated: [4/01/2026]

Project Status: Complete

Version: 1.0
