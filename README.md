# Portfolio-Project-SQL-Airbnb
# 🏠 Airbnb NYC 2019 Exploratory Data Analysis (EDA)

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

- Demonstrate professional SQL EDA workflow for a portfolio project  
- Explore the distribution and characteristics of Airbnb listings across NYC  
- Identify pricing patterns and outliers  
- Evaluate host activity and total revenue potential  
- Analyse reviews, popularity, and booking patterns  
- Investigate availability trends and seasonal effects  
- Examine geographic distribution and potential clusters of listings  
- **Generate a cleaned table `listings_clean` for further analysis with Tableau**
---

## 📂 Dataset
The dataset is sourced from **Inside Airbnb**: a public repository of Airbnb listings, including detailed information about listings, hosts, prices, reviews, and location.  🔗 [Dataset](http://insideairbnb.com/get-the-data.html)

### Dataset Characteristics
- 48,895 listings across NYC  
- 16 key columns including `price`, `reviews_per_month`, `availability_365`, `latitude`, `longitude`  
- Some columns contain missing or inconsistent values  

---

## 🛠 Data Preparation & Modelling
- Created PostgreSQL database and `listings` table  
- Imported CSV data using `\copy` in SQL Shell (psql)  
- Re-encoded CSV to UTF-8 to resolve import errors  
- Altered column types to numeric/date types where required  
- Created a cleaned table `listings_clean` for analysis  
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
- Geographic clusters based on latitude and longitude  

---

# 📊 EDA Sections

<details>
<summary>1. General Overview</summary>

**Purpose:** Provide high-level statistics and counts for listings, boroughs, room types, and review dates.

---

### a. Count total listings

```sql
SELECT COUNT(*) AS total_listings 
FROM listings_clean;

| total_listings |
| -------------- |
| 48895          |

### b. Count listings per borough 

```sql
SELECT neighbourhood_group, COUNT(*) AS total_listings
FROM listings_clean
GROUP BY neighbourhood_group
ORDER BY total_listings DESC;

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

| first_review | most_recent_review |
| ------------ | ------------------ |
| 2011-03-28   | 2019-07-08         |
</details>

<details>
<summary>2. Pricing Analysis</summary>

**Purpose:** Analyse distribution of prices, averages by room type/borough, and identify outliers.

**SQL queries and results:**  
*(Leave blank to fill in your SQL and results)*

</details>

<details>
<summary>3. Host Analysis</summary>

**Purpose:** Identify top hosts, calculate average listings per host, and estimate potential revenue.

**SQL queries and results:**  
*(Leave blank to fill in your SQL and results)*

</details>

<details>
<summary>4. Reviews & Popularity</summary>

**Purpose:** Evaluate reviews, average reviews per month, and correlations with price.

**SQL queries and results:**  
*(Leave blank to fill in your SQL and results)*

</details>

<details>
<summary>5. Availability & Booking Patterns</summary>

**Purpose:** Analyse minimum nights, availability (full-year vs seasonal), and price vs availability.

**SQL queries and results:**  
*(Leave blank to fill in your SQL and results)*

</details>

<details>
<summary>6. Geographic Analysis</summary>

**Purpose:** Examine listings per neighbourhood and identify geographic clusters.

**SQL queries and results:**  
*(Leave blank to fill in your SQL and results)*

</details>

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
| `SQL Scripts/` | Scripts for table creation, cleaning, and EDA |
| `README.md` | Project documentation |
| `Dataset/` | Original or processed CSV files |

---

## Connect With Me

[![LinkedIn](https://img.shields.io/badge/LinkedIn-Cameron_Newbould-blue?style=flat&logo=linkedin)](https://www.linkedin.com/in/cameron-newbould-4a434a308/)

