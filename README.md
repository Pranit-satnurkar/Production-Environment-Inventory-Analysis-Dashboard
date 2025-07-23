
# Production Environment Inventory Analysis Dashboard

### Dashboard Link : https://app.powerbi.com/links/dTSV3HHcaV?ctid=e9f53682-9dc4-428b-ba7b-b55c3de07e9b&pbi_source=linkShare

## Problem Statement

This Power BI dashboard is designed to provide critical insights into the inventory dynamics of a production environment. The primary objective is to enable stakeholders (e.g., operations managers, supply chain analysts) to effectively monitor and manage product availability, demand, and identify potential supply shortages or excesses. By visualizing key metrics such as average daily demand, average daily availability, total supply shortage, and overall profit/loss, the dashboard aims to optimize inventory levels, minimize losses due to unmet demand or overstocking, and enhance overall operational efficiency and profitability.

## Dataset

The core dataset for this project is **`Prod Env Inventory Dataset.csv`**, which contains granular data on product availability and demand over time. This raw data underwent initial cleaning and preparation using **MySQL Workbench** before being integrated into Power BI.

## Steps Followed

1.  **Data Acquisition & Preparation (MySQL Workbench & Power BI):**
    * **MySQL Workbench Data Cleaning:** The raw inventory dataset was initially imported and processed within MySQL Workbench. SQL queries and operations were performed for:
        * Initial data cleaning and standardization.
        * Handling missing values or inconsistencies.
        * Ensuring data integrity and preparing the dataset for analytical consumption.
    * **Connecting to Power BI:** The cleaned and refined data from MySQL was then directly connected to Power BI Desktop using the "Get Data" -> "MySQL database" connector. This established a robust and efficient data pipeline from the cleaned source.
    * **Loading CSV:** The `Prod Env Inventory Dataset.csv` file was also loaded into Power BI, serving as the main source for the prepared inventory data.

2.  **Data Profiling (Power Query Editor):**
    * The `Demand/Avalability` table (or whatever your main table is named) was thoroughly profiled in Power Query Editor.
    * "Column quality," "Column distribution," and "Column profile" (under the "View" tab) were used to check data types, identify errors, empty values, and analyze column statistics for `Order Date`, `Availability`, and `Demand`.
    * "Column profiling based on entire dataset" was applied for a comprehensive data quality assessment.

3.  **Data Transformation (Power Query):**
    * **Data Type Assignment:** Ensured `Order Date (DD/MM/YYYY)` was correctly set to a `Date` data type, and `Availability` and `Demand` were set to `Whole Number` or `Decimal Number`.
    * **Handling Blanks/Errors:** Any remaining blank values or data errors after MySQL processing were addressed using Power Query's capabilities.
    * **(Specific Transformation Example - Replace if you did more):** No complex M-language transformations beyond standard cleaning and type conversions were specifically noted, as the primary logic was handled in MySQL and DAX.

4.  **Data Modeling:**
    * A straightforward data model was established, centered around the `Demand/Avalability` table.
    * (Assumption: A dedicated Date table was likely created or utilized and related to `Demand/Avalability[Order_Date_DD_MM_YYYY]` to enable robust time intelligence calculations, although not explicitly used in the provided DAX).

5.  **DAX Calculations:**
    The following key measures and a calculated column were created using DAX to drive the dashboard's analytical capabilities:

    * **`Profit/Loss` (Calculated Column):** This column calculates the difference between availability and demand for each product on a given day, indicating potential overstock (positive) or shortage (negative).
        ```dax
        Profit/Loss = 'Demand/Avalability'[availability] - 'Demand/Avalability'[demand]
        ```
        *Note: This is a calculated column that computes the difference per row.*

    * **`Total Avalability` (Measure):** Calculates the sum of all available units.
        ```dax
        Total Avalability = SUM('Demand/Avalability'[availability])
        ```

    * **`Total Demand` (Measure):** Calculates the sum of all demanded units.
        ```dax
        Total Demand = SUM('Demand/Avalability'[demand])
        ```

    * **`Total Number of Days` (Measure):** Counts the distinct number of days for which data is available.
        ```dax
        Total Number of Days = DISTINCTCOUNT('Demand/Avalability'[Order_Date_DD_MM_YYYY].[Date])
        ```
        *Note: This assumes `Order_Date_DD_MM_YYYY` is a proper date column and its date hierarchy is used.*

    * **`Avg Availabilty Per Day` (Measure):** Calculates the average availability across the total number of days.
        ```dax
        Avg Availabilty Per Day = DIVIDE([Total Avalability], [Total Number of Days])
        ```

    * **`Avg Demand Per Day` (Measure):** Calculates the average demand across the total number of days.
        ```dax
        Avg Demand Per Day = DIVIDE([Total Demand], [Total Number of Days])
        ```

    * **`Total Supply Shortage` (Measure):** Quantifies the total deficit when demand exceeds availability.
        ```dax
        Total Supply Shortage = [Total Demand] - [Total Avalability]
        ```

    * **`Total Loss` (Measure):** Calculates the total financial loss incurred due to unmet demand (where `Profit/Loss` is negative), multiplied by a hypothetical `unit_price`.
        ```dax
        Total Loss = SUMX(FILTER('Demand/Avalability', 'Demand/Avalability'[Profit/Loss] < 0), 'Demand/Avalability'[Profit/Loss] * 'Demand/Avalability'[unit_price]) * -1
        ```
        *Note: This assumes a `unit_price` column exists in your `Demand/Avalability` table.*

    * **`Total Profit` (Measure):** Calculates the total financial profit from units where availability exceeds demand (where `Profit/Loss` is positive), multiplied by a hypothetical `unit_price`.
        ```dax
        Total Profit = SUMX(FILTER('Demand/Avalability', 'Demand/Avalability'[Profit/Loss] > 0), 'Demand/Avalability'[Profit/Loss] * 'Demand/Avalability'[unit_price])
        ```
        *Note: This assumes a `unit_price` column exists in your `Demand/Avalability` table.*

    * **`Avg. Daily Loss` (Measure):** Calculates the average daily financial loss.
        ```dax
        Avg. Daily Loss = DIVIDE([Total Loss], [Total Number of Days])
        ```

7.  **Dashboard Design & Visualization (2-Page Report):**
    The report is designed with two dedicated pages, primarily utilizing **Card visuals** to highlight key inventory metrics.

    * **Page 1: Overview - Demand & Availability**
        * **Card Visual:** Displaying `Avg. Demand Per Day`, providing a quick glance at the average daily consumption rate.
        * **Card Visual:** Displaying `Average Availability Per Day`, showing the average daily stock levels.
        * **Card Visual:** Displaying `Total Supply Shortage`, immediately highlighting any overall deficits in supply.
        * (Assumption: This page likely includes slicers for filtering by `Order Date`, `Product ID`, or other relevant dimensions for focused analysis.)
        * (Assumption: Other visuals like line charts for daily trends of demand/availability might be present to complement the cards.)

    * **Page 2: Financial Impact - Profit & Loss**
        * **Card Visual:** Displaying `Total Profit`, showing the overall financial gain from efficient inventory.
        * **Card Visual:** Displaying `Total Loss`, quantifying the financial impact of unmet demand or overstock.
        * **Card Visual:** Displaying `Avg. Daily Loss`, indicating the average daily financial drain.
        * (Assumption: This page might also include visuals like bar charts breaking down profit/loss by `Product ID` or trend lines for `Total Profit` and `Total Loss` over time.)

8.  **Publishing to Power BI Service:** The finalized interactive report was published to the Power BI Service, ensuring easy accessibility and sharing with relevant stakeholders.

## Dashboard Snapshots

Here are visual representations of the key pages within the Production Environment Inventory Analysis Dashboard.
*(Please capture high-quality screenshots of your actual Power BI report pages and place them in an `images/` folder in your GitHub repository. Then, update the paths below to your correct file names.)*

**Page 1 - Overview: Demand & Availability:**
<img width="1258" height="730" alt="Image" src="https://github.com/user-attachments/assets/7ead0ebb-7a09-4a9d-81c9-1873921077c6" />

**Page 2 - Financial Impact: Profit & Loss:**
<img width="1262" height="727" alt="Image" src="https://github.com/user-attachments/assets/c405a791-0686-4299-881c-73aab0667f79" />

## Key Insights

This dashboard enables critical insights for optimizing production inventory management:

* **Supply-Demand Balance:** Quickly assess the relationship between `Average Demand Per Day` and `Average Availability Per Day` to identify periods of surplus or shortage.
* **Quantifying Shortages:** The `Total Supply Shortage` metric provides a clear understanding of unmet demand, allowing for proactive adjustments.
* **Financial Performance:** Directly observe the `Total Profit` and `Total Loss` associated with inventory management, highlighting the financial implications of operational decisions.
* **Daily Loss Monitoring:** `Avg. Daily Loss` helps in consistently tracking and minimizing daily financial drains related to inventory inefficiencies.
* **(Add Your Specific, Quantified Insights Here, e.g., "Product ID 19 consistently shows the highest supply shortage," or "Average daily loss increased by 5% in Q3 due to unexpected demand spikes.")**

## Technologies Used

* **Microsoft Power BI Desktop:** Used for comprehensive data modeling, advanced DAX calculations, and the creation of interactive and visually engaging dashboards.
* **MySQL Workbench:** Leveraged for initial data cleaning, transformation, and efficient querying of the raw inventory data before loading into Power BI.
* **Power Query (M Language):** Employed for advanced data extraction, cleaning, and transformation processes within Power BI, ensuring data quality and readiness for analysis.
* **DAX (Data Analysis Expressions):** Utilized for crafting complex measures and calculated columns, deriving key business logic, performance metrics, and profitability insights.
* **CSV Files:** Served as the initial source for the detailed production inventory data.
