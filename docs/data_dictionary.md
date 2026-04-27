
## DATASET SUMMARY

| Item             | Details                                      |
|------------------|----------------------------------------------|
| Dataset name     | Swiggy India Restaurant Dataset              |
| Source           | https://www.kaggle.com/datasets/rrkcoder/swiggy-restaurants-dataset/data    |
| Raw file name    | cleaned_dataset.csv                          |
| Final file name  | Swiggy_Tableau_Final.csv                     
| Total rows       | 10,083                                       |
| Granularity      | One row per restaurant listing per city      |


## COLUMN DEFINITIONS

| Column Name      | Data Type | Description                                              | Example Value        | Used In              | Cleaning Notes                                                                 |
|------------------|-----------|----------------------------------------------------------|----------------------|----------------------|--------------------------------------------------------------------------------|
| restaurant_name  | string    | Name of the restaurant as listed on Swiggy               | Burger King          | EDA, Tableau         | No cleaning needed. Used as label and for COUNT DISTINCT KPI                   |
| cuisine          | string    | Full raw cuisine string, may contain multiple cuisines   | Burgers, American    | EDA only             | Raw multi-value field. Not used directly in charts                             |
| primary_cuisine  | string    | First/main cuisine extracted from the cuisine field      | Burgers              | EDA, Tableau, Stats  | Extracted from cuisine column. Used for all cuisine-level analysis             |
| rating           | string    | Original rating as scraped — includes text values        | 4.2 / NEW / --       | Cleaning only        | Raw field. Not used in analysis. Used only to derive rating_num                |
| rating_num       | float     | Cleaned numeric rating extracted from rating column      | 4.2                  | EDA, KPI, Tableau    | 2,345 nulls where rating was NEW or --. These rows excluded from avg calcs     |
| number_of_ratings| string    | Raw ratings count as scraped — includes K notation      | 10K+ ratings         | Cleaning only        | Raw field. Used only to derive ratings_count                                   |
| ratings_count    | float     | Cleaned numeric ratings count                            | 10000.0              | EDA, Tableau, Stats  | 10K+ capped at 10,000. 2,345 nulls matching rating_num nulls                   |
| average_price    | string    | Raw price as scraped — includes rupee symbol and text    | Rs 350 for two       | Cleaning only        | Raw field. Used only to derive price_num                                       |
| price_num        | float     | Cleaned numeric price for two people in rupees           | 350.0                | EDA, KPI, Tableau    | 25 nulls. Extracted integer from raw string                                    |
| area             | string    | Neighbourhood or area within the city                    | Koramangala          | EDA, Tableau         | No cleaning. Used for hyperlocal drill-down                                    |
| location         | string    | City name                                                | Bangalore            | EDA, KPI, Tableau    | 6 unique values. Used as primary geographic dimension                          |
| pure_veg         | string    | Whether the restaurant is purely vegetarian              | Yes / No             | EDA, Tableau, Stats  | Binary string field. No nulls                                                  |
| is_veg           | integer   | Numeric version of pure_veg                              | 1 / 0                | EDA, Stats           | 1 = pure veg, 0 = non-veg. Derived from pure_veg column                       |
| number_of_offers | string    | Raw offer count as scraped                               | 3                    | Cleaning only        | Raw field. Used only to derive num_offers                                      |
| num_offers       | integer   | Cleaned count of active offers on the restaurant         | 3                    | EDA, Tableau, Stats  | Range 0 to 5. No nulls                                                         |
| offer_name       | string    | Text of all active offers concatenated                   | 60% OFF UPTO Rs 120  | Not used             | EXCLUDED from Tableau CSV. Contains embedded newlines that break CSV parsers   |


## DERIVED COLUMNS

| Derived Column   | Logic                                                         | Business Meaning                                                         |
|------------------|---------------------------------------------------------------|--------------------------------------------------------------------------|
| price_band       | IF price_num < 150 → Budget, < 300 → Mid, < 500 → Premium, ELSE → Luxury | Groups restaurants into pricing tiers for segmentation analysis          |
| rating_tier      | IF rating_num >= 4.3 → Top Rated, >= 4.0 → Good, >= 3.5 → Average, ELSE → Low | Classifies restaurants by performance level relative to platform mean    |
| demand_tier      | IF ratings_count >= 1000 → High Demand, >= 100 → Moderate, >= 10 → Low, ELSE → Very Low | Classifies restaurants by customer engagement level                      |
| value_index      | rating_num / (price_num / 100)                                | Measures rating earned per Rs 100 spent. Higher = better value for money |
| Is Rated         | IF NOT ISNULL(rating) THEN 1 ELSE 0                           | Tableau calculated field. Used for % Rated KPI card                      |


## NULL SUMMARY

| Column        | Null Count | % of Rows | Reason                                      |
|---------------|------------|-----------|---------------------------------------------|
| rating_num    | 2,345      | 23.3%     | Restaurants with NEW or -- as rating        |
| ratings_count | 2,345      | 23.3%     | Same rows as rating_num nulls               |
| price_num     | 25         | 0.2%      | Missing price info on listing               |
| value_index   | 2,362      | 23.4%     | Null when either rating_num or price is null|
| All others    | 0          | 0%        | No nulls                                    |


## DATA QUALITY NOTES

1. EMBEDDED NEWLINES IN OFFER_NAME
   The offer_name column stores multiple offer texts joined by newline characters
   inside a single CSV cell. This causes Tableau and some CSV parsers to read
   44,322 lines instead of 10,083 rows, producing completely wrong averages.
   Fix: offer_name was excluded from Swiggy_Tableau_Final.csv entirely.
   The num_offers column (count only) is used instead.

2. RATING COLUMN HAS MIXED VALUES
   The original rating column contains numeric strings like 4.2, the text NEW
   for recently opened restaurants, and -- for restaurants with no reviews yet.
   These were handled by extracting numeric values into rating_num and setting
   all non-numeric entries to null. Do not use the raw rating column for analysis.

3. RATINGS COUNT USES K NOTATION
   Values like 10K+ were capped at 10,000 during cleaning. This means any
   restaurant with more than 10,000 ratings is treated as 10,000 in all
   calculations. This affects a small number of very high-traffic restaurants
   like McDonald's and Domino's.

4. DUPLICATE RESTAURANT NAMES
   COUNTD(restaurant_name) = 9,388 while total rows = 10,083.
   This is expected — chain restaurants like Burger King and Pizza Hut appear
   in multiple cities and areas, each as a separate row. This is correct.
   Do not deduplicate. Each row represents a unique location listing.

5. PRICE OUTLIERS
   price_num ranges from Rs 20 to Rs 2,000. The scatter plot X axis is capped
   at Rs 1,500 in Tableau to remove extreme luxury outliers that compress the
   main cluster. These outliers are still included in all average calculations.

6. PLATFORM MEAN REFERENCE
   Platform mean rating = 4.03 (AVG of rating_num across all non-null rows).
   This value is used as the reference line on all rating charts in Tableau.
   Any restaurant or cuisine above 4.03 is considered above average.

