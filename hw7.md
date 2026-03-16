## Question 1. Redpanda version

Run `rpk version` inside the Redpanda container:

```bash
docker exec -it hw7-redpanda-1 rpk version
```

What version of Redpanda are you running?

<img width="1242" height="198" alt="image" src="https://github.com/user-attachments/assets/6b73a269-993f-4e83-b6df-e0938fc7f0d0" />

> v25.3.9

## Question 2. Sending data to Redpanda

Create a topic called `green-trips`:

```bash
docker exec -it hw7-redpanda-1 rpk topic create green-trips
```

```Python 
import dataclasses
import json
import sys
import time
from pathlib import Path

sys.path.insert(0, str(Path(__file__).parent.parent))

import pandas as pd
from kafka import KafkaProducer
from models import Ride, ride_from_row

# Download Green trip data
url = "/workspaces/data-engineering-zoomcamp/hw7/green_tripdata_2025-10.parquet"
columns = ['lpep_pickup_datetime','lpep_dropoff_datetime', 'PULocationID', 'DOLocationID', 'passenger_count', 
            'trip_distance', 'tip_amount','total_amount']

df = pd.read_parquet(url, columns=columns)

def ride_serializer(ride):
    ride_dict = dataclasses.asdict(ride)
    json_str = json.dumps(ride_dict)
    return json_str.encode('utf-8')

server = 'localhost:9092'

producer = KafkaProducer(
    bootstrap_servers=[server],
    value_serializer=ride_serializer
)
t0 = time.time()

topic_name = 'rides'

for _, row in df.iterrows():
    ride = ride_from_row(row)
    producer.send(topic_name, value=ride)

producer.flush()

t1 = time.time()
print(f'took {(t1 - t0):.2f} seconds')
```

How long did it take to send the data?

<img width="1087" height="60" alt="image" src="https://github.com/user-attachments/assets/1f4135c4-f3d5-48b3-bf7c-c01f6bb86200" />

>- 10 seconds

## Question 3. Consumer - trip distance

Write a Kafka consumer that reads all messages from the `green-trips` topic
(set `auto_offset_reset='earliest'`).

Count how many trips have a `trip_distance` greater than 5.0 kilometers.

How many trips have `trip_distance` > 5?

```Python 
import sys
from pathlib import Path
sys.path.insert(0, str(Path(__file__).parent.parent))
from kafka import KafkaConsumer
from models import ride_deserializer

server = 'localhost:9092'
topic_name = 'rides'

consumer = KafkaConsumer(
    topic_name,
    bootstrap_servers=[server],
    auto_offset_reset='earliest',
    group_id='rides-console-2',
    value_deserializer=ride_deserializer, 
    consumer_timeout_ms=300000  # 5 minutes
)

print(f"Listening to {topic_name}...")


long_trip_count = 0

for message in consumer:
    ride = message.value

    if ride.trip_distance > 5:
        long_trip_count += 1

consumer.close()

print(f"The number of rides longer than 5 km are: {long_trip_count}")

```

<img width="1113" height="76" alt="image" src="https://github.com/user-attachments/assets/a01e4c5c-6e11-444e-a72e-bae20c4eeb47" />

>- 8506


## Part 2: PyFlink (Questions 4-6)

For the PyFlink questions, you'll adapt the workshop code to work with
the green taxi data. The key differences from the workshop:

- Topic name: `green-trips` (instead of `rides`)
- Datetime columns use `lpep_` prefix (instead of `tpep_`)
- You'll need to handle timestamps as strings (not epoch milliseconds)

You can convert string timestamps to Flink timestamps in your source DDL:

```sql
lpep_pickup_datetime VARCHAR,
event_timestamp AS TO_TIMESTAMP(lpep_pickup_datetime, 'yyyy-MM-dd HH:mm:ss'),
WATERMARK FOR event_timestamp AS event_timestamp - INTERVAL '5' SECOND
```

Before running the Flink jobs, create the necessary PostgreSQL tables
for your results.

Important notes for the Flink jobs:

- Place your job files in `workshop/src/job/` - this directory is
  mounted into the Flink containers at `/opt/src/job/`
- Submit jobs with:
  `docker exec -it workshop-jobmanager-1 flink run -py /opt/src/job/your_job.py`
- The `green-trips` topic has 1 partition, so set parallelism to 1
  in your Flink jobs (`env.set_parallelism(1)`). With higher parallelism,
  idle consumer subtasks prevent the watermark from advancing.
- Flink streaming jobs run continuously. Let the job run for a minute
  or two until results appear in PostgreSQL, then query the results.
  You can cancel the job from the Flink UI at http://localhost:8081
- If you sent data to the topic multiple times, delete and recreate
  the topic to avoid duplicates:
  `docker exec -it workshop-redpanda-1 rpk topic delete green-trips`


## Question 4. Tumbling window - pickup location

Create a Flink job that reads from `green-trips` and uses a 5-minute
tumbling window to count trips per `PULocationID`.

Write the results to a PostgreSQL table with columns:
`window_start`, `PULocationID`, `num_trips`.

After the job processes all data, query the results:

```sql
SELECT PULocationID, num_trips
FROM <your_table>
ORDER BY num_trips DESC
LIMIT 3;
```

Which `PULocationID` had the most trips in a single 5-minute window?

- 42
- 74
- 75
- 166


## Question 5. Session window - longest streak

Create another Flink job that uses a session window with a 5-minute gap
on `PULocationID`, using `lpep_pickup_datetime` as the event time
with a 5-second watermark tolerance.

A session window groups events that arrive within 5 minutes of each other.
When there's a gap of more than 5 minutes, the window closes.

Write the results to a PostgreSQL table and find the `PULocationID`
with the longest session (most trips in a single session).

How many trips were in the longest session?

- 12
- 31
- 51
- 81


## Question 6. Tumbling window - largest tip

Create a Flink job that uses a 1-hour tumbling window to compute the
total `tip_amount` per hour (across all locations).

Which hour had the highest total tip amount?

- 2025-10-01 18:00:00
- 2025-10-16 18:00:00
- 2025-10-22 08:00:00
- 2025-10-30 16:00:00

