# Module 1 homework

Running on local Linux machine

### Downloading datasets

```bash
wget https://github.com/DataTalksClub/nyc-tlc-data/releases/download/green/green_tripdata_2019-09.csv.gz
gunzip green_tripdata_2019-09.csv.gz
wget https://s3.amazonaws.com/nyc-tlc/misc/taxi+_zone_lookup.csv
```

### Building data ingestion container

see Dockerfile and ingest_data.py
(note: I copied verbatim the construction with while True and catching StopIteration because it's a few hours before the deadline and I don't have time to write my own code)
```bash
docker build -t taxi_ingest:v003 .  # there were previous attempts
```

### Docker for Postgres

With pgadmin and the data ingestion container

Note: I'm only running the last container interactive and others detached

```bash
docker network create pg-network
docker run -d \
  -e POSTGRES_USER="root" \
  -e POSTGRES_PASSWORD="root" \
  -e POSTGRES_DB="ny_taxi" \
  -v $(pwd)/ny_taxi_postgres_data:/var/lib/postgresql/data \
  -p 5432:5432 \
  --network=pg-network \
  --name pg-database \
  postgres:13

docker run -d \
  -e PGADMIN_DEFAULT_EMAIL="admin@admin.com" \
  -e PGADMIN_DEFAULT_PASSWORD="root" \
  -p 8080:80 \
  --network=pg-network \
  --name pgadmin-2 \
  dpage/pgadmin4

URL="https://github.com/DataTalksClub/nyc-tlc-data/releases/download/green/green_tripdata_2019-09.csv.gz"
docker run --rm -it \
  --network=pg-network \
  taxi_ingest:v003 \
    --user=root \
    --password=root \
    --host=pg-database \
    --port=5432 \
    --db=ny_taxi \
    --table_name=green_taxi_trips \
    --url=${URL}

```


## Homework

### Question 1. Knowing docker tags

Which tag has the following text? - Automatically remove the container when it exits

--rm

### Question 2. Understanding docker first run

Run docker with the python:3.9 image in an interactive mode and the entrypoint of bash. Now check the python modules that are installed ( use pip list ). What is version of the package wheel ?

```bash
docker run -it --entrypoint /bin/bash python:3.9
pip list  # 0.42.0
docker ps -a
docker rm 8812164e3a17
```

### Question 3. Count records

How many taxi trips were totally made on September 18th 2019?   - 15612

```sql
SELECT
    COUNT(1)
FROM green_taxi_trips
WHERE CAST(lpep_pickup_datetime AS DATE) = '2019-09-18' AND CAST(lpep_dropoff_datetime AS DATE) = '2019-09-18'
;
```

### Question 4. Longest trip for each day

Which was the pick up day with the longest trip distance? Use the pick up time for your calculations.

```sql
SELECT
    MAX(trip_distance) AS dist, lpep_pickup_datetime
FROM green_taxi_trips
GROUP BY lpep_pickup_datetime
ORDER BY dist DESC
LIMIT 1
;
```

341.64 on "2019-09-26 19:32:52"

### Question 5. Three biggest pick up Boroughs

Consider lpep_pickup_datetime in '2019-09-18' and ignoring Borough has Unknown. Which were the 3 pick up Boroughs that had a sum of total_amount superior to 50000?

