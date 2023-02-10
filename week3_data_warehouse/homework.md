## Homework
For this week, I pulled the data from GCS to BigQuery using dbt. I also used dbt to create two persisted tables to analyze the data (one standard and one with partitioning and clustering). The DBT project repo can be found here: 
https://github.com/PWSampanis/zoomcamp_dbt_2023_parker

### Question 1: 
**What is count for fhv vehicles data for year 2019**  


## Question 1:
What is the count for fhv vehicle records for year 2019?
- 65,623,481
- 43,244,696
- 22,978,333
- 13,942,414
Answer: 43,244,696
- I would normally run a count (*) from table but the table's details tab shows the number of rows


## Question 2:
Write a query to count the distinct number of affiliated_base_number for the entire dataset on both the tables.</br> 
What is the estimated amount of data that will be read when this query is executed on the External Table and the Table?

- 25.2 MB for the External Table and 100.87MB for the BQ Table
- 225.82 MB for the External Table and 47.60MB for the BQ Table
- 0 MB for the External Table and 0MB for the BQ Table
- 0 MB for the External Table and 317.94MB for the BQ Table 

0 MB for external table, 317.94 MB for BQ table (SELECT count(distinct (Affiliated_base_number)) FROM `zoomcamp-2023.dbt_psampanis.dbt_taxis_persisted`)



## Question 3:
How many records have both a blank (null) PUlocationID and DOlocationID in the entire dataset?
- 717,748
- 1,215,687
- 5
- 20,332
Answer: 717748. SELECT count(*) FROM `zoomcamp-2023.dbt_psampanis.dbt_taxis_persisted` 
where  PUlocationID is null and DOlocationID is null

## Question 4:
What is the best strategy to optimize the table if query always filter by pickup_datetime and order by affiliated_base_number?
- Cluster on pickup_datetime Cluster on affiliated_base_number
- Partition by pickup_datetime Cluster on affiliated_base_number
- Partition by pickup_datetime Partition by affiliated_base_number
- Partition by affiliated_base_number Cluster on pickup_datetime

Answer: partition by pickup_datetime, cluster on affilitaed_base_number

## Question 5:
Implement the optimized solution you chose for question 4. Write a query to retrieve the distinct affiliated_base_number between pickup_datetime 03/01/2019 and 03/31/2019 (inclusive).</br> 
Use the BQ table you created earlier in your from clause and note the estimated bytes. Now change the table in the from clause to the partitioned table you created for question 4 and note the estimated bytes processed. What are these values? Choose the answer which most closely matches.
- 12.82 MB for non-partitioned table and 647.87 MB for the partitioned table
- 647.87 MB for non-partitioned table and 23.06 MB for the partitioned table
- 582.63 MB for non-partitioned table and 0 MB for the partitioned table
- 646.25 MB for non-partitioned table and 646.25 MB for the partitioned table

Answer: 647.87 MB to run against non partitioned, 23.06 mb for partitioned+clustered. SELECT count(distinct(Affiliated_base_number)) FROM `zoomcamp-2023.dbt_psampanis.dbt_taxis_persisted` where date(pickup_datetime) between "2019-03-01" and "2019-03-31"

partitioning and clustering managed in dbt:
{{ config(
   tags=["- Partitioned by pickup_datetime Cluster on affiliated_base_number"],
   partition_by={
       "field": "pickup_datetime",
       "data_type": "timestamp"
   },
   cluster_by=["affiliated_base_number"]
) }}
select * from {{ ref("dbt_taxis_persisted") }}

## Question 6: 
Where is the data stored in the External Table you created?

- Big Query
- GCP Bucket
- Container Registry
- Big Table
Answer: The data is stored in Cloud storage. When we query the external table in BigQuery, BigQuery is sending an api call to Storage to pull over the data and temporarily persist it.

## Question 7:
It is best practice in Big Query to always cluster your data:
- True
- False

Answer: False, clustering can cause some issues. I've had work pipelines break due to clustering (I can't recall if the issue was with BigQuery Data Transfers or Scheduled Queries). In addition, clustering doesn't really provide benefit for data under 1 GB in size and you cannot accurately estimate query costs on a clustered table. 


## (Not required) Question 8:
A better format to store these files may be parquet. Create a data pipeline to download the gzip files and convert them into parquet. Upload the files to your GCP Bucket and create an External and BQ Table. 


Note: Column types for all files used in an External Table must have the same datatype. While an External Table may be created and shown in the side panel in Big Query, this will need to be validated by running a count query on the External Table to check if any errors occur. 
 
## Submitting the solutions

* Form for submitting: https://forms.gle/rLdvQW2igsAT73HTA
* You can submit your homework multiple times. In this case, only the last submission will be used. 

Deadline: 13 February (Monday), 22:00 CET


## Solution

We will publish the solution here