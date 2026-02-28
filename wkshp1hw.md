### Question 1: What is the start date and end date of the dataset?

```SQL 
SELECT 
    MIN(trip_pickup_date_time),
    MAX(trip_pickup_date_time)
FROM _data_engineering_zoomcamp_api
```
>- 2009-06-01 to 2009-07-01

### Question 2: What proportion of trips are paid with credit card?

```SQL
SELECT 
    ROUND(SUM(CASE WHEN Payment_Type = 'Credit' THEN 1 ELSE 0 END) / COUNT(*) * 100.0, 2)
FROM taxi_pipeline_dataset._data_engineering_zoomcamp_api
```

>- 26.66%


### Question 3: What is the total amount of money generated in tips?

```SQL
SELECT SUM(Total_Amt) as total_amount
FROM taxi_pipeline_dataset._data_engineering_zoomcamp_api
```
>- $10,063.41




