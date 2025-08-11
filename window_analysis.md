# Seasonality Analysis: Step-by-Step Guide with Sample Data

## Overview
This analysis identifies the most intense seasonal periods (3, 4, or 5 consecutive months) for disease cases and rainfall patterns across districts, using multi-year data to find optimal timing windows.

## Step 1: Create District-Month Database

### Sample Raw Data Structure
```
Year | District | Month | Cases_All_Age | Cases_U5 | Rainfall
2018 | District_A | 1 | 45 | 12 | 120
2018 | District_A | 2 | 52 | 15 | 85
2018 | District_A | 3 | 38 | 8 | 45
... | ... | ... | ... | ... | ...
2023 | District_Z | 12 | 67 | 18 | 200
```

### Target Database Structure (One Line per District-Month)
```
District | Month | Cases_All_Age_Mean | Cases_U5_Mean | Rainfall_Mean
District_A | 1 | 42.4 | 11.2 | 115.6
District_A | 2 | 48.8 | 13.4 | 92.3
District_A | 3 | 35.2 | 9.1 | 52.8
... | ... | ... | ... | ...
District_A | 12 | 63.1 | 16.8 | 185.4
```

### Combining Multi-Year Data
**Method: Calculate mean across years 2018, 2019, 2021, 2022, 2023 (excluding 2020)**

For each district and month combination:
- Cases_All_Age_Mean = (2018_value + 2019_value + 2021_value + 2022_value + 2023_value) / 5
- Cases_U5_Mean = Same calculation for U5 cases
- Rainfall_Mean = Same calculation for rainfall

**Example for District_A, January:**
```
Cases_All_Age: (45 + 48 + 41 + 44 + 34) / 5 = 42.4
Cases_U5: (12 + 14 + 10 + 11 + 9) / 5 = 11.2
Rainfall: (120 + 110 + 118 + 115 + 115) / 5 = 115.6
```

## Step 2: Define Sliding Windows (Following Exact Table Structure)

### Window Definitions - 30 Total Windows
Following the provided table structure exactly:

**3-Month Windows (10 windows):**
- Start Month 3: End Month 5 (March, April, May)
- Start Month 4: End Month 6 (April, May, June)
- Start Month 5: End Month 7 (May, June, July)
- Start Month 6: End Month 8 (June, July, August)
- Start Month 7: End Month 9 (July, August, September)
- Start Month 8: End Month 10 (August, September, October)
- Start Month 9: End Month 11 (September, October, November)
- Start Month 10: End Month 12 (October, November, December)
- Start Month 11: End Month 13 (November, December, January)
- Start Month 12: End Month 14 (December, January, February)

**4-Month Windows (10 windows):**
- Start Month 3: End Month 6 (March, April, May, June)
- Start Month 4: End Month 7 (April, May, June, July)
- Start Month 5: End Month 8 (May, June, July, August)
- Start Month 6: End Month 9 (June, July, August, September)
- Start Month 7: End Month 10 (July, August, September, October)
- Start Month 8: End Month 11 (August, September, October, November)
- Start Month 9: End Month 12 (September, October, November, December)
- Start Month 10: End Month 13 (October, November, December, January)
- Start Month 11: End Month 14 (November, December, January, February)
- Start Month 12: End Month 15 (December, January, February, March)

**5-Month Windows (10 windows):**
- Start Month 3: End Month 7 (March, April, May, June, July)
- Start Month 4: End Month 8 (April, May, June, July, August)
- Start Month 5: End Month 9 (May, June, July, August, September)
- Start Month 6: End Month 10 (June, July, August, September, October)
- Start Month 7: End Month 11 (July, August, September, October, November)
- Start Month 8: End Month 12 (August, September, October, November, December)
- Start Month 9: End Month 13 (September, October, November, December, January)
- Start Month 10: End Month 14 (October, November, December, January, February)
- Start Month 11: End Month 15 (November, December, January, February, March)
- Start Month 12: End Month 16 (December, January, February, March, April)

### Complete Window Structure Table
```
Window_Duration | Start_Month | End_Month | Months_Included | Window_ID
3 | 3 | 5 | Mar,Apr,May | 3M_Mar
3 | 4 | 6 | Apr,May,Jun | 3M_Apr
3 | 5 | 7 | May,Jun,Jul | 3M_May
3 | 6 | 8 | Jun,Jul,Aug | 3M_Jun
3 | 7 | 9 | Jul,Aug,Sep | 3M_Jul
3 | 8 | 10 | Aug,Sep,Oct | 3M_Aug
3 | 9 | 11 | Sep,Oct,Nov | 3M_Sep
3 | 10 | 12 | Oct,Nov,Dec | 3M_Oct
3 | 11 | 13 | Nov,Dec,Jan | 3M_Nov
3 | 12 | 14 | Dec,Jan,Feb | 3M_Dec
4 | 3 | 6 | Mar,Apr,May,Jun | 4M_Mar
4 | 4 | 7 | Apr,May,Jun,Jul | 4M_Apr
4 | 5 | 8 | May,Jun,Jul,Aug | 4M_May
4 | 6 | 9 | Jun,Jul,Aug,Sep | 4M_Jun
4 | 7 | 10 | Jul,Aug,Sep,Oct | 4M_Jul
4 | 8 | 11 | Aug,Sep,Oct,Nov | 4M_Aug
4 | 9 | 12 | Sep,Oct,Nov,Dec | 4M_Sep
4 | 10 | 13 | Oct,Nov,Dec,Jan | 4M_Oct
4 | 11 | 14 | Nov,Dec,Jan,Feb | 4M_Nov
4 | 12 | 15 | Dec,Jan,Feb,Mar | 4M_Dec
5 | 3 | 7 | Mar,Apr,May,Jun,Jul | 5M_Mar
5 | 4 | 8 | Apr,May,Jun,Jul,Aug | 5M_Apr
5 | 5 | 9 | May,Jun,Jul,Aug,Sep | 5M_May
5 | 6 | 10 | Jun,Jul,Aug,Sep,Oct | 5M_Jun
5 | 7 | 11 | Jul,Aug,Sep,Oct,Nov | 5M_Jul
5 | 8 | 12 | Aug,Sep,Oct,Nov,Dec | 5M_Aug
5 | 9 | 13 | Sep,Oct,Nov,Dec,Jan | 5M_Sep
5 | 10 | 14 | Oct,Nov,Dec,Jan,Feb | 5M_Oct
5 | 11 | 15 | Nov,Dec,Jan,Feb,Mar | 5M_Nov
5 | 12 | 16 | Dec,Jan,Feb,Mar,Apr | 5M_Dec
```

## Step 3: Calculate Window Sums and Percentages

### For Each Window and Indicator

**Example: District_A, Cases_All_Age, following the exact window structure**

Monthly data for District_A:
```
Month: 1   2   3   4   5   6   7   8   9   10  11  12
Cases: 42.4 48.8 35.2 28.1 45.3 67.2 89.4 92.1 78.6 56.3 51.2 63.1
```

**Step 3a: Calculate Annual Total**
Annual_Total = 42.4 + 48.8 + 35.2 + 28.1 + 45.3 + 67.2 + 89.4 + 92.1 + 78.6 + 56.3 + 51.2 + 63.1 = 697.7

**Step 3b: Calculate Window Sums (Following Table Structure)**
- 3M_Mar (Mar-May): 35.2 + 28.1 + 45.3 = 108.6
- 3M_Apr (Apr-Jun): 28.1 + 45.3 + 67.2 = 140.6
- 3M_May (May-Jul): 45.3 + 67.2 + 89.4 = 201.9
- 3M_Jun (Jun-Aug): 67.2 + 89.4 + 92.1 = 248.7
- 3M_Jul (Jul-Sep): 89.4 + 92.1 + 78.6 = 260.1
- 3M_Aug (Aug-Oct): 92.1 + 78.6 + 56.3 = 227.0
- 3M_Sep (Sep-Nov): 78.6 + 56.3 + 51.2 = 186.1
- 3M_Oct (Oct-Dec): 56.3 + 51.2 + 63.1 = 170.6
- 3M_Nov (Nov-Jan): 51.2 + 63.1 + 42.4 = 156.7
- 3M_Dec (Dec-Feb): 63.1 + 42.4 + 48.8 = 154.3

**Step 3c: Calculate Percentages**
- 3M_Mar: (108.6 / 697.7) × 100 = 15.6%
- 3M_Apr: (140.6 / 697.7) × 100 = 20.1%
- 3M_May: (201.9 / 697.7) × 100 = 28.9%
- 3M_Jun: (248.7 / 697.7) × 100 = 35.6%
- 3M_Jul: (260.1 / 697.7) × 100 = 37.3% ← **Highest for 3-month**
- 3M_Aug: (227.0 / 697.7) × 100 = 32.5%
- 3M_Sep: (186.1 / 697.7) × 100 = 26.7%
- 3M_Oct: (170.6 / 697.7) × 100 = 24.4%
- 3M_Nov: (156.7 / 697.7) × 100 = 22.5%
- 3M_Dec: (154.3 / 697.7) × 100 = 22.1%

### Complete Results Table for District_A (Following Exact Window Structure)
```
Window_ID | Cases_All_Age_Sum | Cases_All_Age_% | Cases_U5_Sum | Cases_U5_% | Rainfall_Sum | Rainfall_%
3M_Mar | 108.6 | 15.6% | 28.4 | 15.7% | 198.1 | 16.8%
3M_Apr | 140.6 | 20.1% | 35.2 | 19.5% | 245.7 | 20.8%
3M_May | 201.9 | 28.9% | 48.3 | 26.8% | 310.2 | 26.3%
3M_Jun | 248.7 | 35.6% | 58.7 | 32.5% | 385.4 | 32.7%
3M_Jul | 260.1 | 37.3% | 62.1 | 34.4% | 405.8 | 34.4%
3M_Aug | 227.0 | 32.5% | 54.8 | 30.4% | 370.2 | 31.4%
3M_Sep | 186.1 | 26.7% | 45.6 | 25.3% | 315.8 | 26.8%
3M_Oct | 170.6 | 24.4% | 42.1 | 23.3% | 295.4 | 25.1%
3M_Nov | 156.7 | 22.5% | 38.9 | 21.6% | 270.6 | 22.9%
3M_Dec | 154.3 | 22.1% | 37.2 | 20.6% | 265.3 | 22.5%
4M_Mar | 148.7 | 21.3% | 36.3 | 20.1% | 268.8 | 22.8%
4M_Apr | 208.8 | 29.9% | 48.9 | 27.1% | 335.9 | 28.5%
4M_May | 269.1 | 38.6% | 63.6 | 35.2% | 420.6 | 35.7%
4M_Jun | 340.2 | 48.8% | 77.4 | 42.9% | 510.2 | 43.3%
4M_Jul | 327.1 | 46.9% | 75.8 | 42.0% | 495.8 | 42.1%
4M_Aug | 283.4 | 40.6% | 67.2 | 37.2% | 441.6 | 37.5%
4M_Sep | 242.4 | 34.7% | 58.9 | 32.6% | 386.2 | 32.8%
4M_Oct | 227.0 | 32.5% | 55.4 | 30.7% | 361.0 | 30.6%
4M_Nov | 212.1 | 30.4% | 51.8 | 28.7% | 335.9 | 28.5%
4M_Dec | 189.5 | 27.2% | 46.5 | 25.8% | 313.4 | 26.6%
5M_Mar | 176.8 | 25.3% | 42.6 | 23.6% | 313.1 | 26.6%
5M_Apr | 253.1 | 36.3% | 58.2 | 32.2% | 401.2 | 34.0%
5M_May | 336.3 | 48.2% | 76.5 | 42.4% | 510.0 | 43.3%
5M_Jun | 407.4 | 58.4% | 90.6 | 50.2% | 615.6 | 52.2%
5M_Jul | 383.4 | 55.0% | 88.9 | 49.3% | 586.2 | 49.7%
5M_Aug | 339.7 | 48.7% | 80.1 | 44.4% | 531.8 | 45.1%
5M_Sep | 298.7 | 42.8% | 71.0 | 39.3% | 477.4 | 40.5%
5M_Oct | 283.4 | 40.6% | 68.5 | 38.0% | 451.4 | 38.3%
5M_Nov | 268.4 | 38.5% | 64.9 | 36.0% | 426.5 | 36.2%
5M_Dec | 224.7 | 32.2% | 55.6 | 30.8% | 378.7 | 32.1%
```

## Step 4: Find Optimal Windows

### For Each Window Duration and Indicator

**Example: Cases_All_Age for District_A (Following Exact Window Structure)**

**3-month windows:**
- Highest %: 3M_Jul (Jul-Sep) = 37.3%
- Optimal start month: 7 (July)

**4-month windows:**
- Highest %: 4M_Jun (Jun-Sep) = 48.8%
- Optimal start month: 6 (June)

**5-month windows:**
- Highest %: 5M_Jun (Jun-Oct) = 58.4%
- Optimal start month: 6 (June)

### Summary Results for District_A (Following Window Table)
```
Indicator | Window_Duration | Optimal_Start_Month | Max_Percentage
Cases_All_Age | 3 | 7 (July) | 37.3%
Cases_All_Age | 4 | 6 (June) | 48.8%
Cases_All_Age | 5 | 6 (June) | 58.4%
Cases_U5 | 3 | 7 (July) | 34.4%
Cases_U5 | 4 | 6 (June) | 42.9%
Cases_U5 | 5 | 6 (June) | 50.2%
Rainfall | 3 | 7 (July) | 34.4%
Rainfall | 4 | 6 (June) | 43.3%
Rainfall | 5 | 6 (June) | 52.2%
```

## Step 5: Create Maps

### Map Data Structure
For each of the 6 maps, create a dataset with:
```
District | Value_to_Map
District_A | [percentage or start_month]
District_B | [percentage or start_month]
... | ...
```

### Six Maps to Create:

1. **3-Month Window - Maximum Percentage**
   - Value: Max percentage for 3-month windows per district
   - Example: District_A = 23.4%

2. **3-Month Window - Optimal Start Month**
   - Value: Start month that gives maximum percentage
   - Example: District_A = 7 (July)

3. **4-Month Window - Maximum Percentage**
   - Value: Max percentage for 4-month windows per district
   - Example: District_A = 31.2%

4. **4-Month Window - Optimal Start Month**
   - Value: Start month that gives maximum percentage
   - Example: District_A = 6 (June)

5. **5-Month Window - Maximum Percentage**
   - Value: Max percentage for 5-month windows per district
   - Example: District_A = 58.4%

6. **5-Month Window - Optimal Start Month**
   - Value: Start month that gives maximum percentage
   - Example: District_A = 6 (June)

### Map Color Schemes
- **Percentage maps**: Use gradient from light to dark (higher % = darker)
- **Start month maps**: Use 12-color categorical scheme (Jan=1, Feb=2, ..., Dec=12)

