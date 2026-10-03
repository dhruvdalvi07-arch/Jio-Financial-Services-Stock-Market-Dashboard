================================================================================
JIO FINANCIAL SERVICES (JIOFIN) – MARKET ANALYSIS DASHBOARD
Power BI Hands-On Assignment Documentation & Implementation Guide
================================================================================

1. PROJECT OVERVIEW
-------------------
• Title: Jio Financial Services – Stock Market Dashboard
• Tool: Microsoft Power BI Desktop
• Dataset: JIO FINANCIAL DATASET.csv (248 Trading Sessions | Jan 2024 - Jan 2025)
• Exchange / Series: NSE Equity Series (EQ)
• Methodology: Raw Dataset -> Power Query Cleaning -> Relational Data Model 
               -> DAX Measures -> Visual Engineering -> Business Insights


2. POWER QUERY DATA CLEANING & TRANSFORMATION STEPS
---------------------------------------------------
1. Ingest Data: Home -> Get Data -> Text/CSV -> JIO FINANCIAL DATASET.csv -> Transform Data.
2. Clean Whitespace in Column Headers:
   Select all columns (Ctrl + A) -> Transform tab -> Format -> Trim.
   (Or in Advanced Editor: Table.TransformColumnNames(Source, Text.Trim))
3. Validate Data Types:
   • Date: Date format (dd-mmm-yy)
   • OPEN, HIGH, LOW, PREV. CLOSE, ltp, close, vwap, 52W H, 52W L: Fixed Decimal Number / Decimal Number
   • VOLUME, No of trades: Whole Number (Integer)
   • Day, Month, Quarter, series: Text
4. Close & Apply.


3. DATE DIMENSION TABLE (CALENDAR DAX)
--------------------------------------
Create a dedicated Date table (Modeling tab -> New Table):

Calendar = 
VAR MinDate = MIN('JIO FINANCIAL DATASET'[Date ])
VAR MaxDate = MAX('JIO FINANCIAL DATASET'[Date ])
RETURN
ADDCOLUMNS(
    CALENDAR(MinDate, MaxDate),
    "Year", YEAR([Date]),
    "Month", FORMAT([Date], "MMMM"),
    "Month Short", FORMAT([Date], "mmm"),
    "MonthNo", MONTH([Date]),
    "Quarter", "Q" & FORMAT([Date], "Q"),
    "QuarterNo", QUARTER([Date]),
    "YearQuarter", YEAR([Date]) & " Q" & FORMAT([Date], "Q"),
    "YearMonth", FORMAT([Date], "yyyy-MM"),
    "YearMonthSort", YEAR([Date]) * 100 + MONTH([Date]),
    "Day", DAY([Date]),
    "DayName", FORMAT([Date], "dddd"),
    "DayOfWeekNo", WEEKDAY([Date], 2),
    "IsWeekend", IF(WEEKDAY([Date], 2) IN {6, 7}, 1, 0)
)

Sort Configuration:
• Sort 'Calendar'[Month] by 'Calendar'[MonthNo]
• Sort 'Calendar'[Month Short] by 'Calendar'[MonthNo]
• Sort 'Calendar'[YearQuarter] by 'Calendar'[YearMonthSort]
Relationship:
• Drag Calendar[Date] to 'JIO FINANCIAL DATASET'[Date ] (1-to-Many, Single filter).


4. COMPLETE DAX MEASURES REPOSITORY
-----------------------------------
Create an empty table (_Measures) via Home -> Enter Data. Create the following:

// 1. Current Price
Latest Closing Price = 
CALCULATE(
    SELECTEDVALUE('JIO FINANCIAL DATASET'[close ]),
    LASTNONBLANK('JIO FINANCIAL DATASET'[Date ], 1)
)

// 2. High & Low Extremes
Highest Closing Price = MAX('JIO FINANCIAL DATASET'[close ])

Lowest Closing Price = MIN('JIO FINANCIAL DATASET'[close ])

// 3. Central Tendencies
Average Closing Price = AVERAGE('JIO FINANCIAL DATASET'[close ])

Average VWAP = AVERAGE('JIO FINANCIAL DATASET'[vwap ])

// 4. Liquidity Aggregates
Total Trading Volume = SUM('JIO FINANCIAL DATASET'[VOLUME ])

Total Number of Trades = SUM('JIO FINANCIAL DATASET'[No of trades ])

// 5. 52-Week Bounds
52 Week High = 
CALCULATE(
    SELECTEDVALUE('JIO FINANCIAL DATASET'[52W H ]),
    LASTNONBLANK('JIO FINANCIAL DATASET'[Date ], 1)
)

52 Week Low = 
CALCULATE(
    SELECTEDVALUE('JIO FINANCIAL DATASET'[52W L ]),
    LASTNONBLANK('JIO FINANCIAL DATASET'[Date ], 1)
)

// 6. Absolute Price Change
Price Change = 
IF(
    HASONEVALUE('JIO FINANCIAL DATASET'[Date ]),
    SELECTEDVALUE('JIO FINANCIAL DATASET'[close ]) - SELECTEDVALUE('JIO FINANCIAL DATASET'[PREV. CLOSE ]),
    VAR FirstDate = MIN('JIO FINANCIAL DATASET'[Date ])
    VAR LastDate = MAX('JIO FINANCIAL DATASET'[Date ])
    VAR StartBase = CALCULATE(SELECTEDVALUE('JIO FINANCIAL DATASET'[PREV. CLOSE ]), 'JIO FINANCIAL DATASET'[Date ] = FirstDate)
    VAR EndClose = CALCULATE(SELECTEDVALUE('JIO FINANCIAL DATASET'[close ]), 'JIO FINANCIAL DATASET'[Date ] = LastDate)
    RETURN EndClose - StartBase
)

// 7. Daily Price Change %
Daily Price Change % = 
IF(
    HASONEVALUE('JIO FINANCIAL DATASET'[Date ]),
    DIVIDE(
        SELECTEDVALUE('JIO FINANCIAL DATASET'[close ]) - SELECTEDVALUE('JIO FINANCIAL DATASET'[PREV. CLOSE ]),
        SELECTEDVALUE('JIO FINANCIAL DATASET'[PREV. CLOSE ]),
        0
    ),
    VAR FirstDate = MIN('JIO FINANCIAL DATASET'[Date ])
    VAR LastDate = MAX('JIO FINANCIAL DATASET'[Date ])
    VAR StartBase = CALCULATE(SELECTEDVALUE('JIO FINANCIAL DATASET'[PREV. CLOSE ]), 'JIO FINANCIAL DATASET'[Date ] = FirstDate)
    VAR EndClose = CALCULATE(SELECTEDVALUE('JIO FINANCIAL DATASET'[close ]), 'JIO FINANCIAL DATASET'[Date ] = LastDate)
    RETURN DIVIDE(EndClose - StartBase, StartBase, 0)
)

// 8. KPI Reference Deltas
Close vs VWAP Delta = [Average Closing Price] - [Average VWAP]

52W Range Spread = [52 Week High] - [52 Week Low]

52W Low Premium % = DIVIDE([Latest Closing Price] - [52 Week Low], [52 Week Low], 0)


5. DASHBOARD LAYOUT & FIELD MAPPING GUIDE
-----------------------------------------
Canvas Settings: 16:9 (1280 x 720 px), Background: #F8FAFC (0% transparency).

TOP HEADER & SLICERS:
• Title: "Jio Financial Services – Stock Market Dashboard"
• Subtitle: "NSE: JIOFIN | Market Analysis & Executive Performance Review | Range: 29-Jan-2024 to 24-Jan-2025"
• Slicers:
  - Year: Calendar[Year] (Tile format)
  - Quarter: Calendar[Quarter] (Tile format)
  - Month: Calendar[Month] (Dropdown format)

ROW 1: 9 KPI CARDS (Side-by-side strip, Font 18 pt, Decimals: 2):
1. Latest Closing Price: [Latest Closing Price] -> ₹ 244.45
2. Highest Closing Price: [Highest Closing Price] -> ₹ 387.95
3. Lowest Closing Price: [Lowest Closing Price] -> ₹ 244.45
4. Average VWAP: [Average VWAP] -> ₹ 331.90
5. Average Closing Price: [Average Closing Price] -> ₹ 331.26
6. Total Trading Volume: [Total Trading Volume] -> 6.21 B (Display units: Billions)
7. Total Number of Trades: [Total Number of Trades] -> 52.54 M (Display units: Millions)
8. 52 Week High: [52 Week High] -> ₹ 394.70
9. 52 Week Low: [52 Week Low] -> ₹ 237.10

ROW 2: PERFORMANCE & INTRADAY PANELS:
• Panel 1 (Middle-Left) | Close vs VWAP Trend:
  - Visual: Line Chart
  - X-axis: Calendar[Date] (Set to Date, NOT Date Hierarchy)
  - Y-axis: [Average Closing Price] (Solid Navy line), [Average VWAP] (Dashed Amber line)
• Panel 2 (Middle-Center) | Intraday Distribution:
  - Visual: Clustered Column Chart or Line/Area Chart
  - X-axis: Calendar[Date] (Date format)
  - Y-axis: HIGH (Avg), LOW (Avg), OPEN (Avg), close (Avg)
• Panel 3 (Middle-Right) | Daily Price Change % Volatility:
  - Visual: Clustered Column Chart
  - X-axis: Calendar[Date] (Date format)
  - Y-axis: [Daily Price Change %]
  - Conditional Color (fx): >= 0% Green (#059669), < 0% Red (#DC2626)

ROW 3: STRUCTURE & TURNOVER FOOTPRINT:
• Panel 4 (Bottom-Left) | Periodic Performance Matrix:
  - Visual: Matrix
  - Rows: Calendar[Year], Calendar[Quarter], Calendar[Month]
  - Values: [Daily Price Change %], [Average Closing Price], [Total Trading Volume]
• Panel 5 (Bottom-Center) | Trading Volume vs Trades:
  - Visual: Line and Clustered Column Chart
  - Shared X-axis: Calendar[Date] (Date format)
  - Column Y-axis: [Total Trading Volume] (or VOLUME set to SUM)
  - Line Y-axis: [Total Number of Trades] (or No of trades set to SUM)
• Panel 6 (Bottom-Right) | 52-Week Range Positioning:
  - Visual: Gauge
  - Value: [Latest Closing Price] (₹ 244.45)
  - Minimum: [52 Week Low] (₹ 237.10)
  - Maximum: [52 Week High] (₹ 394.70)
  - Target: [Average Closing Price] (₹ 331.26)


6. BUSINESS INSIGHTS & ANSWERS TO PROFESSOR QUESTIONS
-----------------------------------------------------
1. Period with Highest Stock Price:
   • Peak Closing Date: 23-Apr-2024 at ₹ 387.95 (Intraday peak: ₹ 394.70).
   • Peak Period: April 2024 / Q2 2024 (Average Close: ₹ 371.20).

2. Period with Lowest Stock Price:
   • Lowest Closing Date: 24-Jan-2025 at ₹ 244.45 (Intraday floor: ₹ 237.10).
   • Lowest Period: January 2025 (Average Close: ₹ 280.39), representing a -36.99% retracement from peak.

3. Best Performing Day (Highest Price Increase):
   • Date: 05-Feb-2024
   • Return: +13.91% (+₹ 35.30 per share), closing at ₹ 289.05 from ₹ 253.75.

4. Worst Performing Day (Highest Price Decrease):
   • Date: 13-Mar-2024
   • Return: -9.30% (-₹ 33.65 per share), dropping from ₹ 361.70 to ₹ 328.05.

5. Highest & Lowest Trading Volume Days:
   • Highest Volume Day: 23-Feb-2024 with 270,077,122 shares traded across 1,305,409 trades (Liquidity Climax).
   • Lowest Volume Day: 18-May-2024 with 4,311,952 shares traded across 45,083 trades (Special weekend session).

6. Closing Price vs VWAP Behavior:
   • Bearish Institutional Dominance: Closing price closed BELOW daily VWAP on 160 of 248 trading sessions (64.52%).
   • The stock closed ABOVE VWAP on only 87 sessions (35.08%), reflecting sustained institutional distribution in H2 2024.

7. Best Performing Month & Quarter:
   • Best Quarter: Q1 2024 (+47.49% net capital appreciation).
   • Best Month: February 2024 (+24.87%), followed by March 2024 (+14.11%).
   • Worst Periods: Q4 2024 (-14.80%) and January 2025 (-18.16%).

8. Proximity to 52-Week High & Low:
   • Latest Close (24-Jan-2025): ₹ 244.45
   • Distance from 52-Week Low (₹ 237.10): Only +₹ 7.35 (+3.10%) above the floor.
   • Distance from 52-Week High (₹ 394.70): Down -38.07% from annual peak.


7. GITHUB REPOSITORY PUSH COMMANDS
----------------------------------
Inside your project folder (containing README.md, Jio_Data_<Name>.pbix, dataset, assets/):

git init
git add .
git commit -m "feat: complete Jio Financial Services Power BI dashboard and documentation"
git branch -M main
git remote add origin https://github.com/<your-username>/jio-financial-market-analysis.git
git push -u origin main
================================================================================