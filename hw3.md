```SQL
-- Created the Parquet Table
CREATE OR REPLACE EXTERNAL TABLE `homework2-486113.homework2.external_data`
OPTIONS (
  format='PARQUET',
  uris = ['gs://dezoomcamp_hw3_2026_sriram/yellow_tripdata_2024-*.parquet']
)

-- Create a non partitioned table from external parquet table
CREATE OR REPLACE TABLE `homework2-486113.homework2.non_partitioned` AS
SELECT * FROM `homework2-486113.homework2.external_data`;
```

## Question 1. Counting records

What is count of records for the 2024 Yellow Taxi Data?

```SQL
SELECT COUNT(*) AS count
FROM `homework2-486113.homework2.external_data`
```

<img width="319" height="94" alt="image" src="https://github.com/user-attachments/assets/43210c9e-a415-4e4b-8d53-263119dffe3c" />

>- 20,332,093

## Question 2. Data read estimation

Write a query to count the distinct number of PULocationIDs for the entire dataset on both the tables.

```SQL
SELECT COUNT(DISTINCT PULocationID) AS distinct_pu_location_ids
FROM `homework2-486113.homework2.external_data`

SELECT COUNT(DISTINCT PULocationID) AS distinct_pu_location_ids
FROM `homework2-486113.homework2.non_partitioned`
```
<img width="603" height="184" alt="image" src="https://github.com/user-attachments/assets/f6e409ee-c456-4988-84e6-f097e15138c8" />

<img width="778" height="93" alt="image" src="https://github.com/user-attachments/assets/60bf5cb5-54c5-445b-b022-54f5f2a945ac" />

What is the **estimated amount** of data that will be read when this query is executed on the External Table and the Table?

>- 0 MB for the External Table and 155.12 MB for the Materialized Table

## Question 3. Understanding columnar storage

Write a query to retrieve the PULocationID from the table (not the external table) in BigQuery. Now write a query to retrieve the PULocationID and DOLocationID on the same table.

Why are the estimated number of Bytes different?

>- BigQuery is a columnar database, and it only scans the specific columns requested in the query. Querying two columns (PULocationID, DOLocationID) requires 
reading more data than querying one column (PULocationID), leading to a higher estimated number of bytes processed.

## Question 4. Counting zero fare trips

How many records have a fare_amount of 0?

```SQL
SELECT COUNT(*) AS zero_trip_fares
FROM `homework2-486113.homework2.non_partitioned`
WHERE fare_amount = 0;
```
<img width="306" height="109" alt="image" src="https://github.com/user-attachments/assets/4e4018ab-b268-4a47-8291-f4d4bf3f9e33" />

>- 8,333

## Question 5. Partitioning and clustering

What is the best strategy to make an optimized table in Big Query if your query will always filter based on tpep_dropoff_datetime and order the results by VendorID (Create a new table with this strategy)

>- Partition by tpep_dropoff_datetime and Cluster on VendorID

## Question 6. Partition benefits

Write a query to retrieve the distinct VendorIDs between tpep_dropoff_datetime
2024-03-01 and 2024-03-15 (inclusive)

```SQL
SELECT DISTINCT(VendorID)
FROM `homework2-486113.homework2.non_partitioned`
WHERE DATE(tpep_dropoff_datetime) BETWEEN '2024-03-01' AND '2024-03-15';
```

<img width="858" height="151" alt="image" src="https://github.com/user-attachments/assets/0d1305ce-7dfb-43e6-8e92-c4c5aa1f59cf" /> 

Use the materialized table you created earlier in your from clause and note the estimated bytes. Now change the table in the from clause to the partitioned table you created for question 5 and note the estimated bytes processed. What are these values? 

```SQL

--create partitioned table
CREATE OR REPLACE TABLE `homework2-486113.homework2.partitioned`
PARTITION BY DATE(tpep_dropoff_datetime) AS
SELECT * FROM `homework2-486113.homework2.external_data`;

--query for comparison
SELECT DISTINCT(VendorID)
FROM `homework2-486113.homework2.partitioned`
WHERE DATE(tpep_dropoff_datetime) BETWEEN 2024-03-01 AND 2024-03-15;
```
<img width="889" height="163" alt="image" src="https://github.com/user-attachments/assets/389dd2f7-f66c-44d5-8d29-3c9d5261831b" />

Choose the answer which most closely matches.
 
>- 310.24 MB for non-partitioned table and 26.84 MB for the partitioned table

## Question 7. External table storage

Where is the data stored in the External Table you created?

>- GCP Bucket

## Question 8. Clustering best practices

It is best practice in Big Query to always cluster your data:

>- False
>- Clustering has an additional cost, it's advisable to perform only when required
