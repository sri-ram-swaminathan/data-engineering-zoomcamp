## Module 2 Homework


1) Within the execution for `Yellow` Taxi data for the year `2020` and month `12`: what is the uncompressed file size (i.e. the output file `yellow_tripdata_2020-12.csv` of the `extract` task)?

>- 128.3 MiB

You can find this by tracing the executions for that specific file and checking the logs for the required output. This is found in: Executions > Output > extract > outputFiles: yellow_tripdata_2020-12.csv. I did this at first but forgot that the backfill was modified to pruge all files, so I could see the output but not access it after execution. Remember to disable the purge option. 

2) What is the rendered value of the variable `file` when the inputs `taxi` is set to `green`, `year` is set to `2020`, and `month` is set to `04` during execution?

>- `green_tripdata_2020-04.csv`

3) How many rows are there for the `Yellow` Taxi data for all CSV files in the year 2020?

>- 24,648,499

```SQL
SELECT  COUNT(*) 
FROM yellow_tripdata 
WHERE EXTRACT(YEAR FROM tpep_pickup_datetime) = 2020;
```

4) How many rows are there for the `Green` Taxi data for all CSV files in the year 2020?

>- 1,734,051

```SQL
SELECT  COUNT(*) 
FROM green_tripdata 
WHERE EXTRACT(YEAR FROM lpep_pickup_datetime) = 2020;
```

5) How many rows are there for the `Yellow` Taxi data for the March 2021 CSV file?

>- 1,925,152

```SQL
SELECT  COUNT(*) 
FROM yellow_tripdata 
WHERE EXTRACT(YEAR FROM tpep_pickup_datetime) = 2020 AND EXTRACT(MONTH FROM tpep_pickup_time) = 03;
```

6) How would you configure the timezone to New York in a Schedule trigger?

>- Add a `timezone` property set to `America/New_York` in the `Schedule` trigger configuration

Note: the options provided match the results we get from checking the details of the corresponding files in the bucket, the SQL queries return answers slightly smaller. 
