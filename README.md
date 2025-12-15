# Port Authority Bus Terminal Strategic Analytics - Fall_2025_6430_02_Group 1
**Course:** 6430_02 | **Semester:** Fall 2025

## 🚀 Project Overview
This project modernizes the Port Authority’s infrastructure planning by transitioning from fragmented Excel tracking into a centralized SQL Data Warehouse. We utilized a modern BI stack—SSIS for ETL, SQL Server for Warehousing, Python (Prophet/OLS) for advanced modeling, and Power BI for executive storytelling.

## 📊 Key Analytical Insights
- **Recovery Analysis:** 94.6% volume recovery as of July 2025 (413k monthly actual average).
- **Demand Forecasting:** Projected 7.04 million annual passengers by 2030 (23.5% growth).
- **Driver Analysis:** OLS Regression proves commuter timing (Day of Week) is 3x more predictive of terminal load than weather variables.

## 📂 Deliverables & Repository Structure
- **/Database_DWH**: Contains the SQL Server `.bak` file and schema scripts.
- **/ETL_Pipeline**: Contains the SSIS project source code for data consolidation.
- **/Analytics_Models**: Contains Jupyter Notebooks and Markdown reports for OLS and Prophet models.
- **Project Report**: 
- **Dashboard**: 

## 🛠️ Technical Note
All Power BI visuals use **Import Storage Mode**. All historical and forecasted data is embedded within the .pbix file; no external database connection is required for evaluation.
