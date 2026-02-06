# Portfolio-Project-SQL-Airbnb
# 🏠 Airbnb NYC 2019 Exploratory Data Analysis

---

## 📌 Project Overview
This project presents an **SQL-based EDA** of the Airbnb NYC 2019 dataset. The analysis explores **listings, pricing, hosts, reviews, availability, and geographic patterns** across New York City neighbourhoods.

The project demonstrates the process of **data cleaning, transformation, validation, and analysis** using PostgreSQL, producing actionable insights for Airbnb hosts, property managers, and analysts.

---

## 🛠 Key SQL & Data Analysis Techniques Demonstrated

This project showcases a range of SQL and analytical skills:

✔ Database setup and table creation in **PostgreSQL**  
✔ Data cleaning and type casting using `ALTER TABLE` and `CAST`  
✔ Handling missing values and invalid numeric entries  
✔ Creating analysis-ready tables for EDA  
✔ Aggregations, grouping, and window functions  
✔ Calculating descriptive statistics (average, median, correlations)  
✔ Data validation with conditional flags and duplicate checks  

---

## 🎯 Project Objectives
The primary goals of this project are to:

- **Demonstrate professional SQL EDA workflow for a portfolio project**  
- **Generate cleaned fact and dimension tables for further analysis with Tableau**

- Explore the distribution and characteristics of Airbnb listings across NYC  
- Identify pricing patterns and outliers  
- Evaluate host activity and total revenue potential  
- Analyse reviews, popularity, and booking patterns  
- Investigate availability trends and seasonal effects  
- Examine geographic distribution and potential clusters of listings  

---

# 📊 EDA Sections
The following sections include the SQL code block queries and the results.
<details>
<summary>##0. Data Quality Checks</summary>

**Purpose:** Assess the quality of the data including NULL values, duplicates and invalid values

### a. Missing or NULL values per column

```sql
SELECT
    COUNT(*) AS total_rows,
    COUNT(id) AS id_count,
    COUNT(name) AS name_count,
    COUNT(host_id) AS host_id_count,
    COUNT(host_name) AS host_name_count,
    COUNT(neighbourhood_group) AS neighbourhood_group_count,
    COUNT(neighbourhood) AS neighbourhood_count,
    COUNT(latitude) AS latitude_count,
    COUNT(longitude) AS longitude_count,
    COUNT(room_type) AS room_type_count,
    COUNT(price) AS price_count,
    COUNT(minimum_nights) AS minimum_nights_count,
    COUNT(number_of_reviews) AS number_of_reviews_count,
    COUNT(last_review) AS last_review_count,
    COUNT(reviews_per_month) AS reviews_per_month_count,
    COUNT(calculated_host_listings_count) AS calculated_host_listings_count_count,
    COUNT(availability_365) AS availability_365_count
FROM listings;
```
| Column                               | Count |
| ------------------------------------ | ----- |
| total_rows                           | 48895 |
| id_count                             | 48895 |
| name_count                           | 48879 |
| host_id_count                        | 48895 |
| host_name_count                      | 48874 |
| neighbourhood_group_count            | 48895 |
| neighbourhood_count                  | 48895 |
| latitude_count                       | 48895 |
| longitude_count                      | 48895 |
| room_type_count                      | 48895 |
| price_count                          | 48895 |
| minimum_nights_count                 | 48895 |
| number_of_reviews_count              | 48895 |
| last_review_count                    | 38843 |
| reviews_per_month_count              | 38843 |
| calculated_host_listings_count_count | 48895 |
| availability_365_count               | 48895 |
---

Therefore, columns with missing data include:

| column                  | count | reasoning                                                      |
| ----------------------- | ----- | -------------------------------------------------------------- |
| name_count              | 48879 | irrelevant to solve as this doesnt impact numerical stats      |
| host_name_count         | 48874 | irrelevant to solve as this doesnt impact numerical stats      |
| last_review_count       | 38843 | means 0 reviews to date hence equal to reviews_per_month_count |
| reviews_per_month_count | 38843 | means 0 reviews hence equal to last_review_count               |

None of the `NULL` values indicate critically missing data therefore the chosen solution is to replace `NULL` with 0 for the `reviews_per_month` column as this is suitable for quick analysis purposes however for the purpose of EDA demonstration, a cleaned table with this change permanently integrated is generated.

```sql
CREATE TABLE listings_clean AS
SELECT *,
       COALESCE(reviews_per_month::NUMERIC, 0) AS reviews_per_month_clean
FROM listings;
```

### b. Check for invalid numeric values

Due to the process in which the table was imported into the database, all datatypes were initially set to `TEXT`, this was then later corrected using `ALTER TABLE` and `ALTER COLUMN` e.g.

```sql
ALTER TABLE listings
ALTER COLUMN id TYPE INT USING id::INT;
```
Therefore, invalid characters in the numeric columns were removed in this process.
Data which was deemed unreasonable such as minimum nights available being 0 or negative was flagged, this was added to `listings_clean` as the `numeric_flag` column. 

```sql
ALTER TABLE listings_clean
ADD COLUMN numeric_flag TEXT DEFAULT 'OK';

UPDATE listings_clean
SET numeric_flag = CASE
           WHEN price <= 0 THEN 'INVALID'
           WHEN price > 10000 THEN 'INVALID'
           WHEN minimum_nights <= 0 THEN 'INVALID'
           WHEN minimum_nights > 365 THEN 'INVALID'
    ELSE 'OK'
END;
```

### c. Check for duplicates

In regards to the listings `id` and the `host_id` a duplicate check was done, no duplicate listing ids were identified however some `host_id`s were found – these corresponded with different listings so are therefore legitimate e.g. 

```sql 
SELECT host_id, name, COUNT(*)
FROM listings
	GROUP BY host_id, name
	HAVING COUNT(*) > 1;
```
   
</details>

<details>
<summary>##1. General Overview</summary>

**Purpose:** Provide high-level statistics and counts for listings, boroughs, room types, and review dates.

---

### a. Count total listings

```sql
SELECT COUNT(*) AS total_listings 
FROM listings_clean;
```
| total_listings |
| -------------- |
| 48895          |

### b. Count listings per borough 

```sql
SELECT neighbourhood_group, COUNT(*) AS total_listings
FROM listings_clean
GROUP BY neighbourhood_group
ORDER BY total_listings DESC;
```
| neighbourhood_group | total_listings |
| ------------------- | -------------- |
| Manhattan           | 21661          |
| Brooklyn            | 20104          |
| Queens              | 5666           |
| Bronx               | 1091           |
| Staten Island       | 373            |

### c. Count listings per room type

```sql
SELECT room_type,
       COUNT(*) AS total_listings
FROM listings_clean
GROUP BY room_type
ORDER BY total_listings DESC;
```
| room_type       | total_listings |
| --------------- | -------------- |
| Entire home/apt | 25409          |
| Private room    | 22326          |
| Shared room     | 1160           |

### d. Date range of last_review

```sql
SELECT 
    MIN(last_review) AS first_review,
    MAX(last_review) AS most_recent_review
FROM listings_clean;
```
| first_review | most_recent_review |
| ------------ | ------------------ |
| 2011-03-28   | 2019-07-08         |

</details>

<details>
<summary>##2. Pricing Analysis</summary>

**Purpose:** Statistically analyse distribution of prices, averages by room type/borough, and identify outliers.

---

### a. Minimum, maximum, mean, median prices

```sql
SELECT 
    MIN(price) AS greatest_price,
    MAX(price) AS lowest_price,
    ROUND(AVG(price),2) AS avg_price
FROM listings_clean;
```
| greatest_price | lowest_price | avg_price |
| -------------- | ------------ | --------- |
| 0              | 10000        | 152.72    |

```sql
SELECT
    PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY price) AS median_price
FROM listings_clean;
```
| median_price |
| ------------ |
| 106          |
### b. Average price by room type

```sql
SELECT room_type,
	ROUND(AVG(price), 2) AS avg_price_room_type
FROM listings_clean
	GROUP BY room_type
	ORDER BY avg_price_room_type DESC;
```
| room_type       | avg_price_room_type |
| --------------- | ------------------- |
| Entire home/apt | 211.79              |
| Private room    | 89.78               |
| Shared room     | 70.13               |

### c. Average price by borough

```sql
SELECT 
    neighbourhood_group,
    ROUND(AVG(price), 2) AS avg_price_neighbourhood_group
FROM listings_clean
GROUP BY neighbourhood_group
ORDER BY avg_price_neighbourhood_group DESC;
```
| neighbourhood_group | avg_price_neighbourhood_group |
| ------------------- | ----------------------------- |
| Manhattan           | 196.88                        |
| Brooklyn            | 124.38                        |
| Staten Island       | 114.81                        |
| Queens              | 99.52                         |
| Bronx               | 87.50                         |

### d. Identify outlier listings
Acts as a check to ensure all outliers are flagged as being `INVALID` to ensure that further processing on the flagged data would be effective. The dataset is purposefully kept somewhat dirty to demonstrate the ability to work with dirty data as well as to not be too heavy handed in the approach to dirty data by removing too aggressively. A clean dataset can be produced by removing the flagged data. 

```sql
SELECT 	id, price, numeric_flag
FROM listings_clean
	WHERE price <= 0;
```
| id       | price | numeric_flag |
| -------- | ----- | ------------ |
| 18750597 | 0     | INVALID      |
| 20333471 | 0     | INVALID      |
| 20523843 | 0     | INVALID      |
| ...      | ...   | ...          |

```sql
SELECT 	id, price, numeric_flag
FROM listings_clean
	WHERE price > 10000;
```
= 0
Therefore, this was verified with a quick check of the max price.

```sql
SELECT 	
	MAX(price) AS max_price
FROM listings_clean;
```
| max_price |
| --------- |
| 10000     |

</details>

<details>
<summary>##3. Host Analysis</summary>

**Purpose:** Identify top hosts, calculate average listings per host, and estimate potential revenue.

### a. Hosts with the most listings

```sql
SELECT host_id, host_name, COUNT(*)
FROM listings
	GROUP BY host_id, host_name
	HAVING COUNT(*) > 1
	ORDER BY COUNT DESC
	LIMIT 10;
```
| host_id   | host_name         | count |
| --------- | ----------------- | ----- |
| 219517861 | Sonder (NYC)      | 327   |
| 107434423 | Blueground        | 232   |
| 30283594  | Kara              | 121   |
| 137358866 | Kazuya            | 103   |
| 16098958  | Jeremy & Laura    | 96    |
| 12243051  | Sonder            | 96    |
| 61391963  | Corporate Housing | 91    |
| 22541573  | Ken               | 87    |
| 200380610 | Pranjal           | 65    |
| 1475015   | Mike              | 52    |

### b. Average number of listings per host

```sql
SELECT
    ROUND(AVG(listing_count), 2) AS avg_listings_per_host
FROM (
    SELECT
        host_id,
        COUNT(*) AS listing_count
    FROM listings_clean
    GROUP BY host_id
) AS host_listing_counts;
```
| avg_listings_per_host |
| --------------------- |
| 1.31                  |

### c. Top hosts by estimated total revenue (price * availability)  

```sql
SELECT
    host_name,
    SUM(price * availability_365) AS total_estimated_revenue
FROM listings_clean
GROUP BY host_name
ORDER BY total_estimated_revenue DESC
LIMIT 10;
```
| host_name    | total_estimated_revenue |
| ------------ | ----------------------- |
| Sonder (NYC) | 24563716                |
| Blueground   | 18021038                |
| Jessica      | 11045293                |
| Kara         | 10656021                |
| David        | 7834953                 |
| Red Awning   | 7686699                 |
| Kevin        | 6921026                 |
| Henry        | 6800246                 |
| Alex         | 6793232                 |
| Michael      | 6690351                 |

</details>

<details>
<summary>##4. Reviews & Popularity</summary>

**Purpose:** Evaluate reviews, average reviews per month, and correlations with price.

### a. Listings with the most reviews

```sql
SELECT 
    name, number_of_reviews 
FROM listings_clean
ORDER BY number_of_reviews DESC
LIMIT 10;
```
| name                                              | number_of_reviews |
| ------------------------------------------------- | ----------------- |
| Room near JFK Queen Bed                           | 629               |
| Great Bedroom in Manhattan                        | 607               |
| Beautiful Bedroom in Manhattan                    | 597               |
| Private Bedroom in Manhattan                      | 594               |
| Room Near JFK Twin Beds                           | 576               |
| Steps away from Laguardia airport                 | 543               |
| Manhattan Lux Loft.Like.Love.Lots.Look !          | 540               |
| Cozy Room Family Home LGA Airport NO CLEANING FEE | 510               |
| Private brownstone studio Brooklyn                | 488               |
| LG Private Room/Family Friendly                   | 480               |

### b. Average reviews per month by borough

```sql
SELECT
    neighbourhood_group,
    ROUND(AVG(reviews_per_month), 2) AS avg_reviews_per_month
FROM listings_clean
GROUP BY neighbourhood_group
ORDER BY avg_reviews_per_month DESC;
```
| neighbourhood_group | avg_reviews_per_month |
| ------------------- | --------------------- |
| Queens              | 1.94                  |
| Staten Island       | 1.87                  |
| Bronx               | 1.84                  |
| Brooklyn            | 1.28                  |
| Manhattan           | 1.27                  |

### c. Correlation between number of reviews and price

In order to identify the correlation, I had to wrap the below query within another in order to find the correlation between the two newly generated columns

```sql 
SELECT
    neighbourhood_group,
    ROUND(AVG(reviews_per_month), 2) AS avg_reviews_per_month,
    ROUND(AVG(price), 2) AS avg_price_by_borough
FROM listings_clean
GROUP BY neighbourhood_group
ORDER BY avg_reviews_per_month DESC;
```
| neighbourhood_group | avg_reviews_per_month | avg_price_by_borough |
| ------------------- | --------------------- | -------------------- |
| Queens              | 1.94                  | 99.52                |
| Staten Island       | 1.87                  | 114.81               |
| Bronx               | 1.84                  | 87.50                |
| Brooklyn            | 1.28                  | 124.38               |
| Manhattan           | 1.27                  | 196.88               |

```sql
SELECT
	CORR(avg_reviews_per_month, avg_price_by_borough) AS correlation
FROM (
	SELECT
		neighbourhood_group,
		ROUND(AVG(reviews_per_month), 2) AS avg_reviews_per_month,
		ROUND(AVG(price), 2) AS avg_price_by_borough
FROM listings_clean
GROUP BY neighbourhood_group)
borough_averages;
```
| correlation |
| ----------- |
| \-0.7644    |

Therefore, a strong negative correlation between the `avg_price_by_borough` and `avg_reviews_per_month` indicating that more expensive listings have less frequent reviews.

</details>

<details>
<summary>##5. Availability & Booking Patterns</summary>

**Purpose:** Analyse minimum nights, availability (full-year vs limited), and price vs availability.

### a. Average minimum nights by room type

```sql
SELECT
    room_type,
    ROUND(AVG(minimum_nights), 2) AS avg_min_nights
FROM listings_clean
GROUP BY room_type
ORDER BY avg_min_nights;
```
| room_type       | avg_min_nights |
| --------------- | -------------- |
| Private room    | 5.38           |
| Shared room     | 6.48           |
| Entire home/apt | 8.51           |

### b. Listings available 365 days vs limited  

```sql
SELECT
    CASE
        WHEN availability_365 = 365 THEN 'Available all year'
        ELSE 'Limited availability'
    END AS availability_type,
    COUNT(*) AS total_listings
FROM listings_clean
GROUP BY availability_type;
```
| availability_type    | total_listings |
| -------------------- | -------------- |
| Available all year   | 1295           |
| Limited availability | 47600          |

### c. Average price vs availability

```sql
SELECT
    CASE
        WHEN availability_365 = 365 THEN 'Available all year'
        ELSE 'Limited availability'
    END AS availability_type,
    COUNT(*) AS total_listings,
    ROUND(AVG(price), 2) AS avg_price_by_availability
FROM listings_clean
GROUP BY availability_type;
```
| availability_type    | total_listings | avg_price_by_availability |
| -------------------- | -------------- | ------------------------- |
| Available all year   | 1295           | 250.77                    |
| Limited availability | 47600          | 150.05                    |

</details>

<details>
<summary>##6. Geographic Analysis</summary>

**Purpose:** Examine listings per neighbourhood and identify geographic clusters.

### a. Number of listings per neighbourhood

```sql
SELECT
    neighbourhood_group,
    COUNT(id) AS number_of_listings
FROM listings_clean
GROUP BY neighbourhood_group
ORDER BY number_of_listings DESC;
```
| neighbourhood_group | number_of_listings |
| ------------------- | ------------------ |
| Manhattan           | 21661              |
| Brooklyn            | 20104              |
| Queens              | 5666               |
| Bronx               | 1091               |
| Staten Island       | 373                |

</details>

---

## 📂 Dataset
The dataset is sourced from **Inside Airbnb**: a public repository of Airbnb listings, including detailed information about listings, hosts, prices, reviews, and location.  🔗 [Dataset](http://insideairbnb.com/get-the-data.html)

### Dataset Characteristics
- 48,895 listings across NYC  
- 16 key columns including `price`, `reviews_per_month`, `availability_365`, `latitude`, `longitude`  
- Some columns contain missing or inconsistent values  

---

## 🛠 Data Preparation & Modelling
- Created PostgreSQL database and `listings` table - [Database](screenshots/3_table_created.jpg)
- Re-encoded CSV to UTF-8 to resolve import errors  - `original_dataset_utf8.csv`
- Imported CSV data using `\copy` in SQL Shell (psql) - [Import](screenshots/4_data_imported.jpg)
- Altered column types to numeric/date types where required - [Datatypes](screenshots/5_datatypes_corrected.jpg)
- Created a cleaned table for analysis - `listings_clean` 
- Flagged invalid numeric values and handled missing values  
- Ensured data consistency and integrity for analysis  

---

## 📐 Key Measures & Calculations
The EDA includes the following calculations and metrics:

- Total listings and listings per borough/room type  
- Minimum, maximum, average, and median prices
- Average price by room type and borough  
- Identification of pricing outliers  
- Hosts with the most listings and estimated total revenue  
- Average reviews per month and top-reviewed listings  
- Correlation between price and reviews  
- Average minimum nights by room type and availability patterns  

---

## 📈 Expected Outcome
The final deliverable is a **professional SQL EDA workflow** demonstrating:

- Data cleaning and transformation  
- Advanced aggregations and calculations in PostgreSQL  
- Analytical insights into listings, pricing, hosts, and reviews  
- Clear and reusable SQL scripts for future Airbnb analysis  

---

## ✨ Credits & Acknowledgements
- PostgreSQL / pgAdmin 4  
- SQL Shell (psql)  
- Inside Airbnb (data source)  

---

## 📂 Files in This Repository

| File / Folder | Description |
|---------------|-------------|
| `screenshots/` | Project screenshots |
| `README.md` | Project documentation |
| `dataset/` | Original or processed CSV files |

---

## Connect With Me

[![LinkedIn](https://img.shields.io/badge/LinkedIn-Cameron_Newbould-blue?style=flat&logo=linkedin)](https://www.linkedin.com/in/cameron-newbould-4a434a308/)

