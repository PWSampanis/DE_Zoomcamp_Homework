pgcli -h localhost -U root -d ny_taxi

wget https://s3.amazonaws.com/nyc-tlc/trip+data/yellow_tripdata_2021-01.csv

wget https://github.com/DataTalksClub/nyc-tlc-data/releases/download/yellow/yellow_tripdata_2021-01.csv.gz


So, to begin: 

I followed the setup video and created a VM in my GCP project. I set git bash on my local machine so I could ssh de-zoomcamp onto the machine. I copied the Data Engineering GitHub repo onto the VM and installed docker and pgcli. 

# 1 and 2 I actually did separately on my local machine because I wanted to play with docker locally separately from the GCP work I would be doing. 



# Question 1. Knowing docker tags- Which tag has the following text? - Write the image ID to the file
# Answer 1: --iidfile string
I installed docker onto my windows 11 computer and launched it via the start menu. I ran my docker commands through git bash. 

Notes: Parker@ParkersDesktop MINGW64 ~/Documents/docker_tests
$ docker build --help

Usage:  docker build [OPTIONS] PATH | URL | -

Build an image from a Dockerfile

Options:
      --add-host list           Add a custom host-to-IP mapping (host:ip)
      --build-arg list          Set build-time variables
      --cache-from strings      Images to consider as cache sources
      --disable-content-trust   Skip image verification (default true)
  -f, --file string             Name of the Dockerfile (Default is
                                'PATH/Dockerfile')
      --iidfile string          Write the image ID to the file
      --isolation string        Container isolation technology
      --label list              Set metadata for an image
      --network string          Set the networking mode for the RUN
                                instructions during build (default "default")
      --no-cache                Do not use cache when building the image
  -o, --output stringArray      Output destination (format:
                                type=local,dest=path)
      --platform string         Set platform if server is multi-platform
                                capable
      --progress string         Set type of progress output (auto, plain,
                                tty). Use plain to show container output
                                (default "auto")
      --pull                    Always attempt to pull a newer version of
                                the image
  -q, --quiet                   Suppress the build output and print image
                                ID on success
      --secret stringArray      Secret file to expose to the build (only
                                if BuildKit enabled):
                                id=mysecret,src=/local/secret
      --ssh stringArray         SSH agent socket or keys to expose to the
                                build (only if BuildKit enabled) (format:
                                default|<id>[=<socket>|<key>[,<key>]])
  -t, --tag list                Name and optionally a tag in the
                                'name:tag' format
      --target string           Set the target build stage to build.


# Question 2. Understanding docker first run. Run docker with the python:3.9 image in an interactive mode and the entrypoint of bash. Now check the python modules that are installed ( use pip list). How many python packages/modules are installed?
# Answer 2: 3

- Note:I encountered error: "the input device is not a TTY.  If you are using mintty, try prefixing the command with 'winpty'". All my commands locally need to be prefixed with winpty

Parker@ParkersDesktop MINGW64 ~/Documents/docker_tests
$ winpty docker run -it --entrypoint=bash python:3.9
- opens the docker container in bash mode so you can run pip commands
root@cf7dabd03b7c:/# pip list
Package    Version
---------- -------
pip        22.0.4
setuptools 58.1.0
wheel      0.38.4


# 3 through 6 were completed by sshing to my GCP VM which I set up per: 
https://www.youtube.com/watch?v=ae-CV2KfoN0&list=PL3MmuxUbc_hJed7dXYoJw8DoCuVHhGEQb

I loaded the data using Jupyer notebooks:
(base) parker@de-zoomcamp:~/data-engineering-zoomcamp/week_1_basics_n_setup/2_docker_sql$ 
and running jupyter notebook upload-data.ipynb

The jupyter notebook had most of what I needed to upload the data to Postgres. I needed to run two commands to get the correct files installed locally on the vm:
1) wget https://github.com/DataTalksClub/nyc-tlc-data/releases/download/green/green_tripdata_2019-01.csv.gz
2) wget https://s3.amazonaws.com/nyc-tlc/misc/taxi+_zone_lookup.csv

Small changes were required to get the Jupyter notebook to reference the files correctly and load the data into PGCLI
1. df = pd.read_csv('green_tripdata_2019-01.csv.gz', nrows=100) {adjusted from yellow_tripdata to green_tripdata}
2. df.lpep_pickup_datetime = pd.to_datetime(df.lpep_pickup_datetime) 
{yellow_taxi had tpep_pickup and tpep_dropoff, I needed to change each of these to lpep_ because the green taxi data has different column names}
3. Run the dependency code blocks, build the green taxi table:
df.head(n=0).to_sql(name='green_taxi_data', con=engine, if_exists='replace')
 and run the while true loop to append the data to the newly created green taxi table. 
4. Build the zones table with the taxi zone lookup csv: df_zones.to_sql(name='zones', con=engine, if_exists='replace')




# Question 3. Count records - How many taxi trips were totally made on January 15? Tip: started and finished on 2019-01-15. Remember that lpep_pickup_datetime and lpep_dropoff_datetime columns are in the format timestamp (date and hour+min+sec) and not in date.

# Answer 3: 20530
SQL:
select count(index) from green_taxi_data  where lpep_pickup_datetime::timestamp::date = '2019-01-15' and lpep_dropoff_datetime::timestamp::date = '2019-01-15'

Notes: I ran the SQL commands for questions 3-6 on my VM using:
pgcli -h localhost -U root -d ny_taxi. I used ::timestamp::date to be able to extract the time piece of the timestamp and effectively compare dates.


# Question 4. Largest trip for each day
Which was the day with the largest trip distance? Use the pick up time for your calculations.
2019-01-18
2019-01-28
2019-01-15
2019-01-10

# Answer 4: 2019-01-15
SQL:
select lpep_pickup_datetime::timestamp::date, max(trip_distance) from green_taxi_data where lpep_pickup_datetime::timestamp::date = '2019-01-18' or
lpep_pickup_datetime::timestamp::date = '2019-01-28' or
lpep_pickup_datetime::timestamp::date = '2019-01-15' or
lpep_pickup_datetime::timestamp::date = '2019-01-10'  
group by lpep_pickup_datetime order by max(trip_distance) DESC

+----------------------+--------+
| lpep_pickup_datetime | max    |
|----------------------+--------|
| 2019-01-15           | 117.99 |
| 2019-01-18           | 80.96  |
| 2019-01-28           | 64.27  |
| 2019-01-10           | 64.2   |
| 2019-01-10           | 50.3   |
| 2019-01-18           | 47.7   |
| 2019-01-15           | 47.2   |
| 2019-01-18           | 44.72  |
Notes: It wasn't that elegant of a sql query, if I were working in BigQuery I would have used where date in ('2029-10-28',2019-01-15', etc). Postgres has a different syntax so I just went with a simple solution

# Question 5. The number of passengers. In 2019-01-01 how many trips had 2 and 3 passengers?
2: 1282 ; 3: 266
2: 1532 ; 3: 126
2: 1282 ; 3: 254
2: 1282 ; 3: 274
# Answer 5:  2:1282 ; 3:254
SQL: 
select count (index) from green_taxi_data where lpep_pickup_datetime::timestamp::date = '2019-01-01' and passenger_count = 2;
select count (index) from green_taxi_data where lpep_pickup_datetime::timestamp::date = '2019-01-01' and passenger_count = 3

Notes: I could probably have createed two sub queries and then referenced these in a single query but it would have been overly complex. Running a query for each passenger count gave me the two numbers I needed.

# Question 6. Largest tip. For the passengers picked up in the Astoria Zone which was the drop off zone that had the largest tip? We want the name of the zone, not the id.
Central Park
Jamaica
South Ozone Park
Long Island City/Queens Plaza
Note: it's not a typo, it's tip , not trip

# Answer 6: Long Island City/Queens Plaza
SQL:
1. select * from public.zones where "Zone" = 'Astoria'
- Astoria is Location ID 7
+-------+------------+---------+---------+--------------+
| index | LocationID | Borough | Zone    | service_zone |
|-------+------------+---------+---------+--------------|
| 6     | 7          | Queens  | Astoria | Boro Zone    |
+-------+------------+---------+---------+--------------+

2. Select "DOLocationID",  max(tip_amount)
from public.green_taxi_data 
where "PULocationID" = 7 
group by "DOLocationID"
order by max(tip_amount) desc LIMIT 10
- max is 146
+--------------+-------+
| DOLocationID | max   |
|--------------+-------|
| 146          | 88.0  |
| 43           | 30.0  |
| 265          | 25.0  |
| 130          | 25.0  |

3. select * from public.zones where "LocationID" = '146'
+-------+------------+---------+-------------------------------+--------------+
| index | LocationID | Borough | Zone                          | service_zone |
|-------+------------+---------+-------------------------------+--------------|
| 145   | 146        | Queens  | Long Island City/Queens Plaza | Boro Zone    |
+-------+------------+---------+-------------------------------+--------------+

Notes: I did this in 3 parts as it was late at night and I didn't want to get too deep into SQL, especially on PostGres. It has a very different syntax than BigQuery (my normal SQl DW) and PGCLI kept throwing errors at me. I would like to go back in later and familiarize myself more with PostGres to make some more efficient / elegant queries.