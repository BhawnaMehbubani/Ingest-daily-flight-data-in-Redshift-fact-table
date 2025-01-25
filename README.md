# Airlines Data Pipeline: ETL Workflow for Flight Analytics  

## Table of Contents  
1. [Overview](#overview)  
2. [Key Features](#key-features)  
3. [Architecture Workflow](#architecture-workflow)  
4. [File Structure](#file-structure)  
5. [Workflow Details](#workflow-details)  
   - [Step 1: Data Extraction](#step-1-data-extraction)  
   - [Step 2: Data Transformation](#step-2-data-transformation)  
   - [Step 3: Data Loading](#step-3-data-loading)  
   - [Step 4: Analytics](#step-4-analytics)  
6. [Setup & Usage](#setup--usage)  
   - [Prerequisites](#prerequisites)  
   - [Steps to Run](#steps-to-run)  
7. [Sample Queries](#sample-queries)  
8. [Best Practices](#best-practices)  
9. [Contributions](#contributions)  

---

## Overview  

This project implements an end-to-end **Extract, Transform, Load (ETL)** pipeline to process and analyze flight and airport data using **AWS Glue**, **Redshift**, and supporting services. The workflow extracts raw data from CSV files, performs complex transformations using Glue, and loads the processed data into Redshift for analytics-ready reporting.  


## Key Features  

- **Automated ETL Pipeline**: Extracts raw flight and airport data, transforms it to derive insights, and loads it into an analytics-friendly Redshift database.  
- **AWS Glue**: Handles the ETL processes with efficient PySpark jobs.  
- **Redshift Integration**: Provides scalable and high-performance data warehousing for querying.  
- **Data Transformation**: Includes filtering for delayed flights, enriching flight records with airport details, and schema adjustments for analytical use cases.  
- **Seamless Integration with AWS Services**: Processes data stored in S3 and ensures smooth connectivity with Redshift.  



## Architecture Workflow  

```plaintext
[airports.csv] + [flights.csv]
       |  
       v  
[S3 Bucket: airlines-dataset-gds]  
       |  
       v  
[AWS Glue Job: Data Transformation]  
       |  
       v  
    [Filter Flights] ---------> [Join Airports Data]  
                                  |  
                                  v  
                        [Transform & Map Fields]  
                                  |  
                                  v  
       [Load into Redshift: airlines.daily_flights_fact]  
                                  |  
                                  v  
                     [Redshift Queries & Analytics]  
```



## File Structure  

1. **`airports.csv`**  
   Contains airport metadata:  
   - `airport_id`: Unique ID for each airport.  
   - `city`: City where the airport is located.  
   - `state`: State abbreviation.  
   - `name`: Name of the airport.  

2. **`flights.csv`**  
   Dataset with daily flight details, including origin, destination, delays, and carrier information.  

3. **`glue_job.py`**  
   - The Glue ETL job script, written in PySpark, to process data.  
   - **Steps**:  
     - Extract airport and flight data from AWS Glue Data Catalog.  
     - Filter flights with departure delays > 60 minutes.  
     - Enrich flight records with airport details.  
     - Map fields to analytics-ready schema.  
     - Load the transformed data into Redshift.  

4. **`redshift_create_table_commands.txt`**  
   SQL commands to create Redshift tables:  
   - **`airports_dim`**: Stores airport metadata.  
   - **`daily_flights_fact`**: Stores enriched and processed flight data.  



## Workflow Details  

### Step 1: Data Extraction  
- **Source**:  
  - `airports.csv` and `flights.csv` are uploaded to an **S3 bucket** (`airlines-dataset-gds`).  
  - Data is registered in the **AWS Glue Data Catalog** for querying and transformation.  

### Step 2: Data Transformation  
- **Filtering Flights**:  
  - Filter records where `depdelay` > 60 minutes.  
- **Joining Airport Data**:  
  - Enrich flight records by joining with airport metadata on `originairportid` and `destairportid`.  
- **Field Selection**:  
  - Select relevant fields: `carrier`, `depdelay`, `arrdelay`, `dep_city`, `arr_city`, etc.  
- **Schema Mapping**:  
  - Transform field names and data types to match Redshift table schema.  

### Step 3: Data Loading  
- Load the transformed data into Redshift:  
  - **Table**: `airlines.daily_flights_fact`.  
  - **Schema**: Combines flight delays, carrier details, and enriched airport information.  

### Step 4: Analytics  
- Use **Redshift SQL** to run queries on the enriched flight dataset for insights like:  
  - Top airports with the most delays.  
  - Carriers with highest delays.  
  - City pairs with frequent delays.  



## Setup & Usage  

### Prerequisites  
- **AWS Account** with access to S3, Glue, and Redshift.  
- **AWS CLI** configured with IAM permissions for Glue and Redshift.  
- **Python 3.x** installed locally.  
- **Boto3** library installed (`pip install boto3`).  

### Steps to Run  

1. **Upload Data to S3**  
   - Place `airports.csv` and `flights.csv` in an S3 bucket: `s3://airlines-dataset-gds/`.  

2. **Create Redshift Schema and Tables**  
   - Run the commands in `redshift_create_table_commands.txt` to set up `airports_dim` and `daily_flights_fact` tables in Redshift.  

3. **Run the Glue Job**  
   - Submit the Glue job using the script in `glue_job.py`:  
     ```bash
     aws glue start-job-run --job-name airlines-etl-job
     ```  

4. **Analyze Data in Redshift**  
   - Query the processed data in `daily_flights_fact` for actionable insights.  
   - Example query:  
     ```sql
     SELECT dep_airport, COUNT(*) AS delayed_flights  
     FROM airlines.daily_flights_fact  
     WHERE dep_delay > 60  
     GROUP BY dep_airport  
     ORDER BY delayed_flights DESC;  
     ```  



## Sample Queries  

1. **Find Top 5 Airports with Departure Delays**  
   ```sql
   SELECT dep_airport, COUNT(*) AS delay_count  
   FROM airlines.daily_flights_fact  
   WHERE dep_delay > 60  
   GROUP BY dep_airport  
   ORDER BY delay_count DESC  
   LIMIT 5;  
   ```  

2. **Get Average Arrival Delay by Carrier**  
   ```sql
   SELECT carrier, AVG(arr_delay) AS avg_arr_delay  
   FROM airlines.daily_flights_fact  
   WHERE arr_delay > 0  
   GROUP BY carrier  
   ORDER BY avg_arr_delay DESC;  
   ```  



## Best Practices  

1. **AWS Glue Optimization**:  
   - Use partitioned S3 buckets for large datasets to speed up queries.  
   - Optimize Glue jobs with dynamic frame transformations.  

2. **Redshift Performance**:  
   - Enable compression for large columns in Redshift.  
   - Use proper distribution keys for better query performance.  

3. **Monitoring**:  
   - Use **AWS CloudWatch** for monitoring Glue job performance and Redshift cluster health.  



## Contributions  

Contributions, suggestions, and feature requests are welcome! Feel free to open an issue or submit a pull request.  

