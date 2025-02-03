# Electric-Vehicle-Population-Data-Analysis

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


