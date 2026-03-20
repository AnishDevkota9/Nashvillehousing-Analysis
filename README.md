This project performs a complete end-to-end data analysis on a real-world Nashville property transactions dataset. The pipeline covers data cleaning, transformation, feature engineering, exploratory data analysis, and visualisation.
The entire pipeline was built twice — once in T-SQL (SQL Server) and once in Python (pandas and matplotlib) — using the exact same dataset and logic. This demonstrates cross-tool competency and shows how the same analytical thinking translates across platforms.

Business Problem
The Nashville real estate dataset contains 56,477 property transactions recorded between 2013 and 2016. In its raw form the data is unusable for business reporting due to multiple quality issues including missing addresses, duplicate records, inconsistent date formats, and unstructured location fields.
This project answers the following business questions:

Which suburbs generate the highest total profit from property sales?
Which property types are sold most frequently and which are most profitable?
What are the top 5 highest-value properties and the 5 worst-performing investments?
How do average property values compare across different cities?
What does the typical property look like for homes built after the year 2000?
Is there a statistical relationship between the year a property was built and its profitability?
How many properties were sold as vacant versus occupied?


Dataset
DetailInfoSourceNashville Housing Dataset — available on KaggleRaw rows56,477Original columns19Final columns21 after cleaning and feature engineeringDate rangeApril 2013 – September 2016
Key fields:
FieldDescriptionParcelIDUnique property identifierSalePriceTransaction sale priceTotalValueAssessed total value of the propertyLandValue / BuildingValueBreakdown of assessed valueLandUseProperty type — Single Family, Duplex, Vacant Land, etc.SoldAsVacantWhether the property was vacant at time of saleYearBuiltYear the property was constructedBedrooms / FullBath / HalfBathProperty profile
Data quality issues found in the raw data:

29 missing PropertyAddress values
31,462 missing OwnerName, OwnerAddress, and financial fields
SaleDate stored as a plain string — needed type conversion
SoldAsVacant had mixed values: 'Y', 'N', 'Yes', 'No'
103–104 exact duplicate rows
PropertyAddress and OwnerAddress stored as single combined strings — needed splitting


Tools Used
ToolPurposeSQL Server — T-SQLData cleaning, transformation, and all EDA queriesPython 3.xParallel cleaning pipeline and visualisationspandasDataFrame operations, groupby, string transformationsNumPyNumeric operations and correlation analysismatplotlibAll data visualisationsJupyter NotebookPython analysis environment

Repository Structure
nashville-housing-analysis/
│
├── SQL/
│   └── nashville_cleaning_eda.sql        # Complete T-SQL script
│
├── Python/
│   └── nashville_analysis.ipynb          # Jupyter notebook
│
├── Data/
│   └── Nashville Data.csv                # Raw source dataset
│
└── README.md

Project Workflow
Nashville Data.csv  (56,477 rows × 19 columns)
          │
          ▼
 ┌──────────────────────┐     ┌──────────────────────┐
 │   SQL Server          │     │   Python / pandas    │
 │   Cleaning pipeline   │     │   Cleaning pipeline  │
 └──────────┬───────────┘     └──────────┬───────────┘
            │                             │
            └────────────┬────────────────┘
                         ▼
              ┌─────────────────────┐
              │  Feature Engineering │   Profit_amount = SalePrice − TotalValue
              └──────────┬──────────┘
                         ▼
              ┌─────────────────────┐
              │  Exploratory         │   Aggregations, rankings,
              │  Data Analysis       │   subqueries, groupby, correlations
              └──────────┬──────────┘
                         ▼
              ┌─────────────────────┐
              │  Visualisations      │   Bar, pie, histogram, scatter, line
              └─────────────────────┘

Data Cleaning and Transformation
SQL Server approach
Step 1 — Date conversion
SaleDate was stored as a plain string. A new column was added with the correct date type.
sqlALTER TABLE NashvilleData ADD Converted_Sales_date DATE;
UPDATE NashvilleData SET Converted_Sales_date = CONVERT(date, SaleDate);
Step 2 — NULL address imputation via self-join
29 rows had a missing PropertyAddress despite having a valid ParcelID. Since PropertyAddress is determined by ParcelID, a self-join was used to fill the NULLs from matching rows with the same ParcelID.
sqlUPDATE a
SET a.PropertyAddress = ISNULL(a.PropertyAddress, b.PropertyAddress)
FROM NashvilleData a
JOIN NashvilleData b
  ON a.ParcelID = b.ParcelID
  AND a.[UniqueID] <> b.[UniqueID];
-- Result: 35 rows filled
Step 3 — Address splitting
PropertyAddress was stored as a single combined string. PARSENAME was used to split it into separate street and city columns.
sqlALTER TABLE NashvilleData
ADD PropertyAddress_Split NVARCHAR(100), PropertyCity NVARCHAR(100);

UPDATE NashvilleData SET
  PropertyAddress_Split = PARSENAME(REPLACE(PropertyAddress, ',', '.'), 2),
  PropertyCity = PARSENAME(REPLACE(PropertyAddress, ',', '.'), 1);
The same approach was applied to OwnerAddress to extract street, city, and state into three separate columns.
Step 4 — Standardise SoldAsVacant
The column had a mix of 'Y', 'N', 'Yes', and 'No' values. These were standardised to 'Yes' and 'No' using CASE WHEN.
sqlUPDATE NashvilleData
SET SoldAsVacant = CASE
  WHEN SoldAsVacant = 'Y' THEN 'Yes'
  WHEN SoldAsVacant = 'N' THEN 'No'
  ELSE SoldAsVacant
END;
Step 5 — Deduplication using window functions
sqlWITH duplicatecte AS (
  SELECT *,
    ROW_NUMBER() OVER (
      PARTITION BY ParcelID, PropertyAddress, SalePrice, SaleDate, LegalReference
      ORDER BY [UniqueID]
    ) AS row_count
  FROM NashvilleData
)
DELETE FROM duplicatecte WHERE row_count > 1;
-- Result: 104 duplicate rows removed
Step 6 — Drop unused columns and add profit
sqlALTER TABLE NashvilleData
DROP COLUMN PropertyAddress, SaleDate, OwnerAddress, TaxDistrict;

ALTER TABLE NashvilleData
ADD Profit_amount AS (SalePrice - TotalValue);

Python approach
Loading the data
pythonnashville_data = pd.read_csv(r'Nashville Data.csv')
nashville_data.info()
# 56,477 rows × 19 columns
# PropertyAddress: 56,448 non-null (29 missing)
# OwnerName: 25,261 non-null
# TotalValue: 26,015 non-null
Date conversion
pythonnashville_data['SaleDate'] = pd.to_datetime(nashville_data['SaleDate'])
NULL address imputation
The data was sorted by ParcelID so that rows sharing the same ParcelID sit adjacent, then forward fill was applied to propagate the known address into the NULL row.
pythonnashville_data = nashville_data.sort_values(by='ParcelID', ascending=True)
nashville_data['PropertyAddress'] = nashville_data['PropertyAddress'].fillna(method='ffill')

nashville_data['PropertyAddress'].isna().sum()
# Output: 0
Address splitting
pythonnashville_data[['address_of_property', 'city_of_property']] = (
    nashville_data['PropertyAddress'].str.split(',', expand=True)
)

nashville_data[['address_of_owner', 'city_of_owner', 'state_of_owner']] = (
    nashville_data['OwnerAddress'].str.split(',', expand=True)
)
Deduplication
pythonnashville_data.duplicated().groupby(nashville_data.duplicated()).count()
# False    56,374
# True       103
# Result: 103 duplicates found

nashville_data = nashville_data.drop_duplicates()
Drop columns and add profit
pythonnashville_data.drop(columns=['PropertyAddress', 'OwnerAddress', 'TaxDistrict'], inplace=True)

nashville_data['SalePrice'] = pd.to_numeric(nashville_data['SalePrice'], errors='coerce')
nashville_data['Profit_amount'] = nashville_data['SalePrice'] - nashville_data['TotalValue']

Data Cleaning Summary
StepIssueSQL ApproachPython ApproachDate conversionSaleDate stored as stringALTER TABLE + CONVERT(date, SaleDate)pd.to_datetime()NULL address imputation29 rows missing PropertyAddressSelf-join on ParcelID using ISNULL()sort_values + fillna(method='ffill')Address splittingStreet and city in one columnPARSENAME(REPLACE(col, ',', '.'), n)str.split(',', expand=True)Owner address splittingStreet, city, state in one columnPARSENAME — 3 partsstr.split(',', expand=True)Standardise SoldAsVacantMixed Y/N/Yes/No valuesCASE WHEN col='Y' THEN 'Yes'.replace({'Y':'Yes', 'N':'No'})Deduplication103–104 duplicate rowsROW_NUMBER() OVER (PARTITION BY ...) → DELETE.drop_duplicates()Drop unused columnsSeveral columns not neededALTER TABLE DROP COLUMN.drop(columns=[...])Feature engineeringNo profitability metricADD Profit_amount AS (SalePrice - TotalValue)Direct column assignment

Key Findings
Total Profit by City
CityTotal Profit ($)Nashville1,145,007,482Antioch30,989,820Hermitage27,450,569Madison21,368,918Old Hickory13,654,081Brentwood9,623,924Goodlettsville6,898,191Nolensville1,314,090Whites Creek935,100Mount Juliet330,200Joelton72,831Bellevue12,600
Nashville accounts for over 97% of total profit across all cities combined.

Average Total Property Value by City
CityAvg Total Value ($)Nolensville1,921,700.00Brentwood434,475.98Nashville256,264.52Mount Juliet213,200.00Old Hickory146,235.25Goodlettsville147,648.20Hermitage131,214.44Antioch111,825.66Whites Creek94,670.83Bellevue12,400.00
Nolensville's high average is driven by a small number of very high-value properties — a good example of why averages need to be interpreted carefully alongside sample sizes.

Sales Volume by Property Type
Land UseCountSingle Family34,197
Single Family homes represent approximately 60.5% of all transactions in the dataset.

Top 5 Worst Performing Properties
AddressCitySale Price ($)Profit ($)4225 Franklin PikeNashville905,000-5,497,6004225 Franklin PikeNashville1,100,000-5,302,6004321 Chickering LnNashville1,800,000-2,655,2001011 Grassland LnNashville650,000-1,755,800956 Tyne BlvdNashville850,000-1,656,000
All 5 worst-performing properties were sold while their assessed TotalValue significantly exceeded the transaction price — likely sold during or before construction completion.

Properties with Profit Above $200,000

1,483 properties exceeded this threshold
Minimum qualifying profit: $200,200
Highest single profit: $12,300,000 — 2812 McGavock Pike, Nashville (Vacant Residential Land sold for $12,350,000)


Sold as Vacant Breakdown
StatusCountShareNot vacant51,80291.7%Vacant4,6758.3%

Average Property Profile — Homes Built After 2000
Year BuiltAvg BedroomsAvg Full BathsAvg Half Baths2001320200443120074312010321201443120164312017441
Properties built from 2004 onwards consistently show larger profiles compared to early-2000s builds.

Nashville Total Value — Properties Built After 1999
Year BuiltTotal Value ($)200022,288,800200543,209,900201016,200,8002013124,552,7002014217,190,7002015343,430,8002016298,923,500201710,704,700
A sharp increase from 2013–2015 reflects Nashville's well-documented property boom, peaking in 2015.

Profit by Land Use Type
Land UseTotal Profit ($)Single Family846,820,716Vacant Residential Land232,158,099Duplex90,259,272Vacant Commercial Land48,810,700Zero Lot Line19,767,814Parking Lot12,479,000Triplex8,026,926Mobile Home-286,350Vacant Res Land-41,483,308
Two land use categories showed negative aggregate profit — consistently assessed above their sale prices across all transactions.

Year Built vs Profitability — Correlation
Pearson Correlation (YearBuilt vs Profit_amount):  r = -0.1532
Weak negative correlation — newer properties do not generate higher profit margins. This is because newer builds carry higher assessed TotalValue, which compresses the profit margin even when sold at higher prices. The scatter plot confirms wide variance with no strong linear trend.

Visualisations Produced
ChartVariablesKey InsightHorizontal barSales count by cityNashville leads in volume; all suburbs far lowerBar chartAvg sale price by year built post-2010Newer builds trend toward higher average sale pricesPie chartSoldAsVacant breakdown91.7% of all properties sold as occupiedHistogram — 50 binsSalePrice distributionHeavily right-skewed; most sales under $500K with a long upper tailLine chartVacant property count by year built post-2010Peaks at 177 properties in 2015, drops sharply afterScatter plotProfit_amount vs YearBuiltWide variance; weak negative trend confirms r = -0.15

How to Run
SQL Server:

Import Nashville Data.csv into SQL Server as a table named NashvilleData inside a database called NashvilleHousing
Open nashville_cleaning_eda.sql in SSMS
Run sections in order — data cleaning first, then EDA queries

Python:
bashpip install pandas numpy matplotlib jupyter
jupyter notebook nashville_analysis.ipynb
Update the file path in pd.read_csv(r'...') to point to your local copy of Nashville Data.csv.

Skills Demonstrated

Real-world data quality assessment and cleaning on a 56K-row dataset
NULL imputation using relational self-join logic in SQL and forward fill in Python
Deduplication using ROW_NUMBER window functions and drop_duplicates
String parsing and column extraction using PARSENAME and str.split
Aggregation and ranking with GROUP BY, HAVING, subqueries, and groupby
Feature engineering — derived profitability metric from existing columns
Statistical correlation analysis using Pearson r
Data visualisation across 6 different chart types
Cross-tool competency — identical analytical pipeline built in both SQL Server and Python
