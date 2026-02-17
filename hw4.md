### Question 1. dbt Lineage and Execution

Given a dbt project with the following structure:

```
models/
├── staging/
│   ├── stg_green_tripdata.sql
│   └── stg_yellow_tripdata.sql
└── intermediate/
    └── int_trips_unioned.sql (depends on stg_green_tripdata & stg_yellow_tripdata)
```

If you run `dbt run --select int_trips_unioned`, what models will be built?

>- `int_trips_unioned` only
>- We need to add a + before to run any models that it depends on and add a + after to run any models that depend on it

### Question 2. dbt Tests

You've configured a generic test like this in your `schema.yml`:

```yaml
columns:
  - name: payment_type
    data_tests:
      - accepted_values:
          arguments:
            values: [1, 2, 3, 4, 5]
            quote: false
```

Your model `fct_trips` has been running successfully for months. A new value `6` now appears in the source data.

What happens when you run `dbt test --select fct_trips`?

>- dbt will fail the test, returning a non-zero exit code

---

### Question 3. Counting Records in `fct_monthly_zone_revenue`

After running your dbt project, query the `fct_monthly_zone_revenue` model.

What is the count of records in the `fct_monthly_zone_revenue` model?

```SQL
SELECT 
COUNT(*) as total_records
FROM 
`homework4-48710.-487603.nytaxi_dbt_prod.fct_monthly_zone_revenue`;
```

>- 12,184


---

### Question 4. Best Performing Zone for Green Taxis (2020)

Using the `fct_monthly_zone_revenue` table, find the pickup zone with the **highest total revenue** (`revenue_monthly_total_amount`) for **Green** taxi trips in 2020.

Which zone had the highest revenue?

```SQL
SELECT
    pickup_zone, 
    SUM(revenue_monthly_total_amount) AS total_revenue
FROM 
    `homework4-487510.nytaxi_dbt_prod.fct_monthly_zone_revenue`
WHERE 
    service_type = 'Green' 
    AND EXTRACT(YEAR FROM revenue_month) = 2020
GROUP BY 
    pickup_zone
ORDER BY 
    total_revenue DESC
LIMIT 1;
```

>- East Harlem North

---

### Question 5. Green Taxi Trip Counts (October 2019)

```SQL
SELECT 
    SUM(total_monthly_trips) AS total_trip
FROM 
    `homework4-487510.nytaxi_dbt_prod.fct_monthly_zone_revenue`
WHERE 
    service_type = 'Green' 
    AND revenue_month = '2019-10-01';
```

Using the `fct_monthly_zone_revenue` table, what is the **total number of trips** (`total_monthly_trips`) for Green taxis in October 2019?

>- 384,624

---

### Question 6. Build a Staging Model for FHV Data

Create a staging model for the **For-Hire Vehicle (FHV)** trip data for 2019.

1. Load the [FHV trip data for 2019](https://github.com/DataTalksClub/nyc-tlc-data/releases/tag/fhv) into your data warehouse

I first modified the `ingest.py` script from last week to include the FHV trip data into my BigQuery dataset

Then, I need to let dbt know to load this data, this is done by modifying the `sources.yml` file to simply include this 

```yml
version: 2

sources:
  - name: raw_data
    description: "Raw data sources for NYC taxi rides"
    database: module-4-dbt # Project ID
    schema: nytaxi # Dataset name
    tables:
      - name: yellow_tripdata
        description: Raw yellow taxi trip data
      - name: green_tripdata
        description: Raw green taxi trip data
      - name: fhv_tripdata
        description: Raw fhv taxi trip data
```

3. Create a staging model `stg_fhv_tripdata` with these requirements:
   - Filter out records where `dispatching_base_num IS NULL`
   - Rename fields to match your project's naming conventions (e.g., `PUlocationID` → `pickup_location_id`)

Then I created the following file in the staging folder of the project 

```yml
SELECT dispatching_base_num,
    pickup_datetime,
    dropOff_datetime AS dropoff_datetime, 
    PUlocationID AS pickup_location_id, 
    DOlocationID AS dropoff_location_id,
    SR_Flag AS sr_flag,
    Affiliated_base_number AS affiliated_base_number
FROM {{ source("raw_data", "fhv_tripdata")}}
WHERE dispatching_base_num IS NOT NULL
```

I then simply run this file and then the following command

What is the count of records in `stg_fhv_tripdata`?

```SQL
SELECT 
    COUNT(*) 
FROM 
    `homework4-487510.nytaxi_dbt_prod.stg_fhv_tripdata` LIMIT 10
```
>- 43,244,693





