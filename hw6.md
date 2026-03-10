## Question 1: Install Spark and PySpark

- Install Spark
- Run PySpark
- Create a local spark session
- Execute spark.version.

I ran the test_pyspark.py file given in the input and this is what I get 

What's the output?

<img width="238" height="43" alt="image" src="https://github.com/user-attachments/assets/6008fc2c-f5ae-425b-bcbe-47ed041f79e7" />

>- 4.11

## Question 2: Yellow November 2025

Read the November 2025 Yellow into a Spark Dataframe.

Repartition the Dataframe to 4 partitions and save it to parquet.

```Python
df = spark.read \
    .option("header","true") \
    .parquet("yellow_tripdata_2025-11.parquet")

df = df.repartition(4)

df.write.parquet('yellowtrip/november')
```
What is the average size of the Parquet (ending with .parquet extension) Files that were created (in MB)? Select the answer which most closely matches.

<img width="1030" height="184" alt="image" src="https://github.com/user-attachments/assets/51b1293e-be68-4a07-8c27-1c4c0115df74" />

>- 25MB

## Question 3: Count records

How many taxi trips were there on the 15th of November?

Consider only trips that started on the 15th of November.

```Python
from pyspark.sql import functions as F

df.withColumn('pickup_date', F.to_date(df.tpep_pickup_datetime)) \
  .filter(F.col('pickup_date') == '2025-11-15') \
  .select(F.count('pickup_date')) \
  .show()
```

<img width="361" height="142" alt="image" src="https://github.com/user-attachments/assets/b41de49d-83d5-4193-93c2-13fb9a42fb96" />


>- 162,604


## Question 4: Longest trip

What is the length of the longest trip in the dataset in hours?

```Python
from pyspark.sql import functions as F

df.withColumn('duration_hours', 
    (F.unix_timestamp('tpep_dropoff_datetime') - F.unix_timestamp('tpep_pickup_datetime')) / 3600) \
  .select('duration_hours', 'trip_distance') \
  .orderBy(F.col('duration_hours').desc()) \
  .limit(1) \
  .show()
```

<img width="415" height="154" alt="image" src="https://github.com/user-attachments/assets/2c460cfa-0a8a-4e26-9813-28be363f186e" />

>- 90.6


## Question 5: User Interface

Spark's User Interface which shows the application's dashboard runs on which local port?

 <img width="1893" height="909" alt="image" src="https://github.com/user-attachments/assets/6d2c2fe2-115d-4236-9cc3-ee634c3fa7b7" />

>- 4040

## Question 6: Least frequent pickup location zone

Load the zone lookup data into a temp view in Spark:

```bash
wget https://d37ci6vzurychx.cloudfront.net/misc/taxi_zone_lookup.csv
```

Using the zone lookup data and the Yellow November 2025 data, what is the name of the LEAST frequent pickup location Zone?

```Python
df_join = df.join(df_zones, df.PULocationID == df_zones.LocationID)

df_join.groupBy('Zone') \
  .agg(F.count('Zone').alias('n_trips')) \
  .orderBy(F.col('n_trips').asc()) \
  .show()
```

<img width="370" height="175" alt="image" src="https://github.com/user-attachments/assets/b416ef0e-610c-4e9f-a917-5a7bb7b2b6d2" />

>- Governor's Island/Ellis Island/Liberty Island
>- Arden Heights

If multiple answers are correct, select any


