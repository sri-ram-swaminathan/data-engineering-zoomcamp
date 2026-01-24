## Question 1. Understanding Docker images

Run docker with the `python:3.13` image. Use an entrypoint `bash` to interact with the container.

What's the version of `pip` in the image?

>- Steps: (1) docker run -it --entrypoint=bash python:3.13 (2) pip -V
>- Output: 25.3

## Question 2. Understanding Docker networking and docker-compose

Given the following `docker-compose.yaml`, what is the `hostname` and `port` that pgadmin should use to connect to the postgres database?

```yaml
services:
  db:
    container_name: postgres
    image: postgres:17-alpine
    environment:
      POSTGRES_USER: 'postgres'
      POSTGRES_PASSWORD: 'postgres'
      POSTGRES_DB: 'ny_taxi'
    ports:
      - '5433:5432'
    volumes:
      - vol-pgdata:/var/lib/postgresql/data

  pgadmin:
    container_name: pgadmin
    image: dpage/pgadmin4:latest
    environment:
      PGADMIN_DEFAULT_EMAIL: "pgadmin@pgadmin.com"
      PGADMIN_DEFAULT_PASSWORD: "pgadmin"
    ports:
      - "8080:80"
    volumes:
      - vol-pgadmin_data:/var/lib/pgadmin

volumes:
  vol-pgdata:
    name: vol-pgdata
  vol-pgadmin_data:
    name: vol-pgadmin_data
```
>- postgres:5432

## Question 3. Counting short trips

For the trips in November 2025 (lpep_pickup_datetime between '2025-11-01' and '2025-12-01', exclusive of the upper bound), how many trips had a `trip_distance` of less than or equal to 1 mile?

```SQL
SELECT COUNT(*)
FROM 'green_tripdata_2025-11.parquet'
WHERE lpep_pickup_datetime >= TIMESTAMP '2025-11-01'
  AND lpep_pickup_datetime <  TIMESTAMP '2025-12-01'
  AND trip_distance <= 1
```
>- 8,007

## Question 4. Longest trip for each day

Which was the pick up day with the longest trip distance? Only consider trips with `trip_distance` less than 100 miles (to exclude data errors).

Use the pick up time for your calculations.

```SQL
SELECT DATE(lpep_pickup_datetime), MAX(trip_distance)
FROM 'green_tripdata_2025-11.parquet'
WHERE trip_distance < 100 
GROUP BY DATE(lpep_pickup_datetime)	
ORDER BY MAX(trip_distance) DESC
LIMIT 1
```
>- 2025-11-14

## Question 5. Biggest pickup zone

Which was the pickup zone with the largest `total_amount` (sum of all trips) on November 18th, 2025?

```SQL
SELECT z.Zone, SUM(t.total_amount) AS total_amount
FROM 'green_tripdata_2025-11.parquet.parquet' AS t
LEFT JOIN 'taxi_zone_lookup.csv' AS z
  ON t.PULocationID = z.LocationID
WHERE DATE(t.lpep_pickup_datetime) = DATE '2025-11-18'
GROUP BY z.Zone
ORDER BY total_amount DESC
LIMIT 1;
```

>- East Harlem North

## Question 6. Largest tip

For the passengers picked up in the zone named "East Harlem North" in November 2025, which was the drop off zone that had the largest tip?

Note: it's `tip` , not `trip`. We need the name of the zone, not the ID.

```SQL
SELECT dz.Zone AS dropoff_zone, MAX(t.tip_amount) AS max_tip
FROM 'green_tripdata_2025-11.parquet.parquet' AS t
LEFT JOIN 'taxi_zone_lookup.csv' AS dz
  ON t.DOLocationID = dz.LocationID
LEFT JOIN 'taxi_zone_lookup.csv' AS pz
  ON t.PULocationID = pz.LocationID
WHERE pz.Zone = 'East Harlem North'
  AND DATE(t.lpep_pickup_datetime) >= DATE '2025-11-01'
  AND DATE(t.lpep_pickup_datetime) < DATE '2025-12-01'
GROUP BY dz.Zone
ORDER BY max_tip DESC
LIMIT 1;
```
>- Yorkville West

## Question 7. Terraform Workflow

Which of the following sequences, respectively, describes the workflow for:
1. Downloading the provider plugins and setting up backend,
2. Generating proposed changes and auto-executing the plan
3. Remove all resources managed by terraform`

>- terraform init, terraform apply -auto-approve, terraform destroy
