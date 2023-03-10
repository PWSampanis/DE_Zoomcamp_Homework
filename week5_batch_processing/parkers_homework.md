## Week 5 Homework 

In this homework we'll put what we learned about Spark in practice.

For this homework we will be using the FHVHV 2021-06 data found here. [FHVHV Data](https://github.com/DataTalksClub/nyc-tlc-data/releases/download/fhvhv/fhvhv_tripdata_2021-06.csv.gz )


### Question 1: 

**Install Spark and PySpark** 

- Install Spark
- Run PySpark
- Create a local spark session
- Execute spark.version.

What's the output?
- 3.3.2
- 2.1.4
- 1.2.3
- 5.4

Answer: 3.3.2 (run spark.version in a notebook file after importing pyspark)

### Question 2: 

**HVFHW June 2021**

Read it with Spark using the same schema as we did in the lessons.
We will use this dataset for all the remaining questions.
Repartition it to 12 partitions and save it to parquet.
What is the average size of the Parquet (ending with .parquet extension) Files that were created (in MB)? Select the answer which most closely matches.


- 2MB
- 24MB
- 100MB
- 250MB

Answer: 24MB (Actual filesize is ~22.5MB)

### Question 3: 

**Count records**  

How many taxi trips were there on June 15?
Consider only trips that started on June 15.

- 308,164
- 12,856
- 452,470
- 50,982

Answer: 452470
- spark.sql("""
SELECT count(dispatching_base_num) FROM tripdata
where date(pickup_datetime) = '2021-06-15'
""").show()


### Question 4: 

**Longest trip for each day**  

Now calculate the duration for each trip.
How long was the longest trip in Hours?

- 66.87 Hours
- 243.44 Hours
- 7.68 Hours
- 3.32 Hours

Answer: 66.87
- spark.sql("""
select pickup_datetime, dropoff_datetime, 
(unix_timestamp(dropoff_datetime)-unix_timestamp(pickup_datetime))/(3600) as Duration_In_Hours 
from tripdata
order by Duration_In_Hours  desc
""").show()

### Question 5: 

**User Interface**

 Spark’s User Interface which shows application's dashboard runs on which local port?

- 80
- 443
- 4040
- 8080

Answer: Port 4040 is used for the spark interface


### Question 6: 

**Most frequent pickup location zone**

Load the zone lookup data into a temp view in Spark
[Zone Data](https://github.com/DataTalksClub/nyc-tlc-data/releases/download/misc/taxi_zone_lookup.csv)

Using the zone lookup data and the fhvhv June 2021 data, what is the name of the most frequent pickup location zone?

- East Chelsea
- Astoria
- Union Sq
- Crown Heights North
answer: Crown Heights North
- spark.sql("""
SELECT z.Zone, count(t.dispatching_base_num)  as ride_count 
FROM tripdata t
left outer join zone_lookup z 
on t.PULocationID = z.LocationID
group by z.Zone
order by ride_count desc
""").show()

+--------------------+----------+
|                Zone|ride_count|
+--------------------+----------+
| Crown Heights North|    231279|
|        East Village|    221244|
|         JFK Airport|    188867|




## Submitting the solutions

* Form for submitting: https://forms.gle/EcSvDs6vp64gcGuD8
* You can submit your homework multiple times. In this case, only the last submission will be used. 

Deadline: 06 March (Monday), 22:00 CET