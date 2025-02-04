# Electric Vehicle Market Analysis

This project highlights the seamless integration of SQL and Power BI to analyze and visualize electric vehicle sales data. SQL was used for in-depth querying, including growth rates, penetration levels, and seasonal trends. Power BI transformed this data into interactive dashboards for actionable insights. The combination demonstrates the efficiency of integrating robust data analysis with dynamic visualization tools.

## Dataset Description

### electric_vehicle_sales_by_state.csv

| Column Name | Description | Data Type |
|----------|----------|----------|
| Date   | The date on which the data was recorded. Format: DD-MMM-YY. (Data is recorded on a monthly basis)   | Date  |
| state   | The name of the state where the sales data is recorded, indicating the geographical location within India.   | String   |
| vehicle_category   | The category of the vehicle, specifying whether it is a 2-Wheeler or a 4-Wheeler.   | String  |
| electric_vehicles_sold   | The number of electric vehicles sold in the specified state and category on the given date.   | Integer  |
| total_vehicles_sold   | TThe total number of vehicles (including both electric and non-electric) sold in the specified state and category on the given date   | Integer  |

**━━━━━━━━━━━━━━━━━━━━━━**

### electric_vehicle_sales_by_makers.csv

| Column Name | Description | Data Type |
|----------|----------|----------|
| Date   | The date on which the sales data was recorded. Format: DD-MMM-YY. (Data is recorded on a monthly basis)   | Date   |
| vehicle_category   | The category of the vehicle, specifying whether it is a 2-Wheeler or a 4-Wheeler.   | String   |
| maker   | The name of the manufacturer or brand of the electric vehicle.   | String   |
| electric_vehicles_sold   | The number of electric vehicles sold by the specified maker in the given category on the given date.   | Integer   |

**━━━━━━━━━━━━━━━━━━━━━━**

### dim_date.csv

| Column Name | Description | Data Type |
|----------|----------|----------|
| Date   | The specific date for which the data is relevant. Format: DD-MMM-YY. (Data is recorded on a monthly basis)   | Date   |
| fiscal_year   | The fiscal year to which the date belongs. This is useful for financial and business analysis.   | Integer   |
| quarter   | The fiscal quarter to which the date belongs. Fiscal quarters are typically divided as Q1, Q2, Q3, and Q4.   | String   |





# SQL Query

1) Top 3 and bottom 3 makers for the fiscal years 2023 and 2024 in terms of the number of 2-wheelers sold.

```bash
WITH RankedSales AS (
    SELECT d.fiscal_year, m.maker, 
        SUM(m.electric_vehicles_sold) AS total_electric_vehicles_sold,
        ROW_NUMBER() OVER (PARTITION BY d.fiscal_year ORDER BY SUM(m.electric_vehicles_sold) DESC) AS rank_desc,
        ROW_NUMBER() OVER (PARTITION BY d.fiscal_year ORDER BY SUM(m.electric_vehicles_sold) ASC) AS rank_asc
    FROM electric_vehicle_sales_by_makers m
    JOIN dim_date d
    On m.date = d.date
    WHERE d.fiscal_year IN (2023, 2024) AND m.vehicle_category = '2-Wheelers'
    GROUP BY d.fiscal_year, m.maker
)
SELECT fiscal_year, maker, total_electric_vehicles_sold,
    CASE WHEN rank_desc <= 3 THEN 'Top 3'
        WHEN rank_asc <= 3 THEN 'Bottom 3'
    END AS rank_type
FROM RankedSales
WHERE rank_desc <= 3 OR rank_asc <= 3
ORDER BY fiscal_year, rank_type, total_electric_vehicles_sold DESC;
```

2) Identify the top 5 states with the highest penetration rate in 2-wheeler and 4-wheeler EV sales in FY 2024.

```bash
SELECT * 
FROM ( 
SELECT state, vehicle_category,
SUM(electric_vehicles_sold)/sum(total_vehicles_sold)*100 AS penetration_rate
FROM electric_vehicle_sales_by_state AS c
JOIN dim_date AS a
on c.date=a.date 
WHERE a.fiscal_year=2024 and vehicle_category="2-wheelers"
GROUP BY state, vehicle_category
ORDER BY penetration_rate DESC LIMIT 5
) as Two 
UNION ALL 
SELECT * 
FROM ( 
SELECT state, vehicle_category,
SUM(electric_vehicles_sold)/sum(total_vehicles_sold)*100 AS penetration_rate
FROM electric_vehicle_sales_by_state AS c
JOIN dim_date AS a
on c.date=a.date 
WHERE a.fiscal_year=2024 and vehicle_category="4-wheelers"
GROUP BY state, vehicle_category
ORDER BY penetration_rate DESC limit 5
) AS four;
```

3) Quarterly trends based on sales volume for the top 5 EV maker (4-wheelers) from 2022 to 2024? 

```bash
WITH top_5_makers AS (
SELECT b.maker AS maker,
SUM(electric_vehicles_sold) AS ev_sold
FROM electric_vehicle_sales_by_makers AS B
JOIN dim_date AS A
USING(date)
WHERE a.fiscal_year BETWEEN 2022 AND 2024 AND vehicle_category= "4-wheelers"
GROUP BY b.maker
ORDER BY ev_sold DESC LIMIT 5 
)
SELECT a.quarter, top_5_makers.maker, 
SUM(electric_vehicles_sold) AS quartely_sale
FROM dim_date AS a
JOIN electric_vehicle_sales_by_makers AS b
USING(date)
JOIN top_5_makers USING(maker)
WHERE a.fiscal_year BETWEEN 2022 AND 2024 AND vehicle_category= "4-wheelers"
GROUP BY a.quarter, top_5_makers.maker
ORDER BY a.quarter, top_5_makers.maker;
```

4) List the states with negative penetration (decline) in EV sales from 2022 to 2024?

```bash
WITH CTE2022 AS(
SELECT state,
SUM(electric_vehicles_sold)/SUM(total_vehicles_sold)*100 AS penetration_rate 
FROM electric_vehicle_sales_by_state AS c 
JOIN dim_date AS a 
on c.date=a.date
WHERE a.fiscal_year=2022 
GROUP BY state
ORDER BY state 
)
, CTE2024 AS(
SELECT state,
SUM(electric_vehicles_sold)/SUM(total_vehicles_sold)*100 AS penetration_rate
FROM electric_vehicle_sales_by_state AS c 
JOIN dim_date AS a 
on c.date=a.date
WHERE a.fiscal_year=2024 
GROUP BY state
order by state 
)
SELECT CTE2022.state, CTE2024.penetration_rate-CTE2022.penetration_rate AS difference 
FROM CTE2022 
JOIN CTE2024 
on CTE2022.state=CTE2024.state
WHERE CTE2022.penetration_rate>CTE2024.penetration_rate
GROUP BY state;
```

5) How do the EV sales and penetration rates in Delhi compare to Karnataka for 2024? 

```bash
WITH Delhi_prate AS (
SELECT state, a.fiscal_year, 
SUM(c.electric_vehicles_sold) AS total_EV_sold,
sum(c.total_vehicles_sold) AS total_vehicle_sold,
SUM(c.electric_vehicles_sold)/sum(c.total_vehicles_sold)*100 AS penetration_rate 
FROM electric_vehicle_sales_by_state AS c
JOIN dim_date AS a
on c.date=a.date
WHERE a.fiscal_year=2024 AND c.state="Delhi" 
), 
karnataka_prate AS ( SELECT state, a.fiscal_year, 
SUM(c.electric_vehicles_sold) AS total_EV_sold,
sum(c.total_vehicles_sold) AS total_vehicle_sold,
SUM(c.electric_vehicles_sold)/sum(c.total_vehicles_sold)*100 AS penetration_rate 
FROM electric_vehicle_sales_by_state AS c
JOIN dim_date AS a
on c.date=a.date
WHERE a.fiscal_year=2024 AND c.state="karnataka"
)
SELECT * 
FROM Delhi_prate
UNION
SELECT *
FROM karnataka_prate;
```

6) The compounded annual growth rate (CAGR) in 4-wheeler units for the top 5 makers from 2022 to 2024. 

```bash
WITH Top5makers AS (
SELECT maker
FROM electric_vehicle_sales_by_makers AS b
JOIN dim_date AS a
ON b.date = a.date 
WHERE b.vehicle_category="4-wheelers" AND a.fiscal_year BETWEEN 2022 AND 2024
GROUP BY maker 
 ORDER BY SUM(electric_vehicles_sold) DESC
 LIMIT 5
 ),
 yearly_sales AS( 
 SELECT maker, 
 SUM(CASE WHEN fiscal_year=2022 THEN b.electric_vehicles_sold END) as 2022_sales,
 SUM(CASE WHEN fiscal_year=2024 THEN b.electric_vehicles_sold end) as 2024_sales
 FROM dim_date AS a
 JOIN electric_vehicle_sales_by_makers as b
 USING(date)
GROUP BY maker
)
SELECT *,
ROUND(POWER(2024_sales/2022_sales , 1.0/2)-1,2) AS CAGR
FROM top5makers AS tm
JOIN yearly_sales AS ys
USING (maker)
ORDER BY CAGR DESC;
```

7) Top 10 states that had the highest compounded annual growth rate (CAGR) from 2022 to 2024 in total vehicles sold. 

```bash
WITH Top10states AS (
SELECT state
FROM electric_vehicle_sales_by_state AS c
JOIN dim_date AS a
ON c.date = a.date 
WHERE a.fiscal_year BETWEEN 2022 AND 2024
GROUP BY state 
ORDER BY SUM(total_vehicles_sold) DESC
LIMIT 10
 ),
 yearly_sales AS( 
 SELECT state, 
 SUM(CASE WHEN fiscal_year=2022 THEN c.total_vehicles_sold END) as 2022_sales,
 SUM(CASE WHEN fiscal_year=2024 THEN c.total_vehicles_sold end) as 2024_sales
 FROM dim_date AS a
 JOIN electric_vehicle_sales_by_state as c
 USING(date)
 GROUP BY state
 )
SELECT *,
ROUND(POWER(2024_sales/2022_sales , 1.0/2)-1,2) AS CAGR
FROM top10states AS ts
JOIN yearly_sales AS ys
USING (state)
ORDER BY CAGR DESC;
```

8) Peak and low season months for EV sales based on the data from 2022 to 2024? 

```bash
WITH peak_season AS (
SELECT "Peak month" AS season,
 DATE_FORMAT(date,'%M') AS peak_months,
SUM(electric_vehicles_sold) AS ev_sales 
FROM electric_vehicle_sales_by_state AS c
JOIN dim_date AS a
USING(date)
WHERE a.fiscal_year BETWEEN 2022 and 2024 
GROUP BY peak_months
ORDER BY ev_sales DESC LIMIT 1 
),
Low_season AS ( 
SELECT "Low month" AS season,
DATE_FORMAT(date,'%M') AS low_months,
SUM(electric_vehicles_sold) AS ev_sales 
FROM electric_vehicle_sales_by_state AS c
JOIN dim_date AS a
USING(date)
WHERE a.fiscal_year BETWEEN 2022 and 2024 
GROUP BY low_months
ORDER BY ev_sales LIMIT 1 
)
SELECT *  FROM Peak_season 
UNION 
SELECT *  FROM low_season ;
```

9) Projected number of EV sales (including 2-wheelers and 4-wheelers) for the top 10 states by penetration rate in 2030, based on the compounded annual growth rate (CAGR) from previous years?

```bash
WITH Top10states AS (
SELECT state,
SUM(c.total_vehicles_sold) AS total_sales,
SUM(electric_vehicles_sold) / SUM(total_vehicles_sold) AS penetration_rate
FROM electric_vehicle_sales_by_state AS c
JOIN dim_date AS a
ON c.date = a.date 
WHERE a.fiscal_year= 2024
GROUP BY state 
ORDER BY penetration_rate DESC
LIMIT 10
 ),
CAGR AS ( 
SELECT c.state,
round(power((SUM(CASE WHEN a.fiscal_year = 2024 THEN c.total_vehicles_sold END) /
		SUM(CASE WHEN a.fiscal_year = 2022 THEN c.total_vehicles_sold END)),(1.0/2)) - 1 ,2)AS CAGR
FROM electric_vehicle_sales_by_state AS c
JOIN dim_date AS a
USING(date)
GROUP BY c.state
)
SELECT ts.state,
ROUND(ts.total_sales * POWER(1 + CAGR.CAGR, 2030 - 2024),2) AS projected_sales_2030
FROM top10states AS ts
LEFT JOIN CAGR 
USING(state)
ORDER BY projected_sales_2030 DESC
```


