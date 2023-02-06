## Week 2 Homework

The goal of this homework is to familiarise users with workflow orchestration and observation. 


## Question 1. Load January 2020 data

Using the `etl_web_to_gcs.py` flow that loads taxi data into GCS as a guide, create a flow that loads the green taxi CSV dataset for January 2020 into GCS and run it. Look at the logs to find out how many rows the dataset has.

How many rows does that dataset have?

* 447,770
* 766,792
* 299,234
* 822,132

*Answer:* The table has 447,000 rows
Executing 'clean-b9fd7e03-0' immediately...
11:03:05 PM
   VendorID lpep_pickup_datetime  ... trip_type congestion_surcharge
0       2.0  2019-12-18 15:52:30  ...       1.0                  0.0
1       2.0  2020-01-01 00:45:58  ...       2.0                  0.0

[2 rows x 20 columns]
11:03:05 PM
clean-b9fd7e03-0
columns: VendorID                        float64
lpep_pickup_datetime     datetime64[ns]

deleted the rest of the columns to shorten


rows: 447770

11:03:05 PM
clean-b9fd7e03-0
Finished in state Completed()

## Question 2. Scheduling with Cron

Cron is a common scheduling specification for workflows. 

Using the flow in `etl_web_to_gcs.py`, create a deployment to run on the first of every month at 5am UTC. What’s the cron schedule for that?

- `0 5 1 * *` ... this works but my timezone is PST, do I need to change the timezone?
- `0 0 5 1 *`
- `5 * 1 0 *`
- `* * 5 1 0`

*Answer:* 
Per the prefect deployment scheduler, `0 5 1 * *` will run the job "At 05:00 AM on day 1 of the month". I would also to select timezone UTC instead of my browser default as I am in the Pacific time zone. 


## Question 3. Loading data to BigQuery 

Using `etl_gcs_to_bq.py` as a starting point, modify the script for extracting data from GCS and loading it into BigQuery. This new script should not fill or remove rows with missing values. (The script is really just doing the E and L parts of ETL).

The main flow should print the total number of rows processed by the script. Set the flow decorator to log the print statement.

Parametrize the entrypoint flow to accept a list of months, a year, and a taxi color. 

Make any other necessary changes to the code for it to function as required.

Create a deployment for this flow to run in a local subprocess with local flow code storage (the defaults).

Make sure you have the parquet data files for Yellow taxi data for Feb. 2019 and March 2019 loaded in GCS. Run your deployment to append this data to your BiqQuery table. How many rows did your flow code process?

- 14,851,920
- 12,282,990
- 27,235,753
- 11,338,483

*Answer:* 14,851,920
per the Pandas df row printouts: 
...
congestion_surcharge            float64
dtype: object
rows: 7019375
...
congestion_surcharge            float64
dtype: object
rows: 7832545
...


## Question 4. Github Storage Block

Using the `web_to_gcs` script from the videos as a guide, you want to store your flow code in a GitHub repository for collaboration with your team. Prefect can look in the GitHub repo to find your flow code and read it. Create a GitHub storage block from the UI or in Python code and use that in your Deployment instead of storing your flow code locally or baking your flow code into a Docker image. 

Note that you will have to push your code to GitHub, Prefect will not push it for you.

Run your deployment in a local subprocess (the default if you don’t specify an infrastructure). Use the Green taxi data for the month of November 2020.

How many rows were processed by the script?

- 88,019
- 192,297
- 88,605
- 190,225

*Answer:*
Unsure, I'm stuck at an error. 

command on my gcp vm:
prefect deployment build etl_web_to_gcs.py:etl_web_to_gcs -n "Github Storage Flow" -sb github/week2__prefect/etl_web_to_gcs_for_github.py -o /home/parker/data-engineering-zoomcamp/week_2_workflow_orchestration/etl_web_to_gcs_github_flow.yaml --apply

Error logs: 
Found flow 'etl-web-to-gcs'
Traceback (most recent call last):
  File "/home/parker/anaconda3/envs/zoomcamp/lib/python3.9/site-packages/prefect/client/orion.py", line 1211, in read_block_document_by_name
    response = await self._client.get(
  File "/home/parker/anaconda3/envs/zoomcamp/lib/python3.9/site-packages/httpx/_client.py", line 1757, in get
    return await self.request(
  File "/home/parker/anaconda3/envs/zoomcamp/lib/python3.9/site-packages/httpx/_client.py", line 1533, in request
    return await self.send(request, auth=auth, follow_redirects=follow_redirects)
  File "/home/parker/anaconda3/envs/zoomcamp/lib/python3.9/site-packages/prefect/client/base.py", line 251, in send
    response.raise_for_status()
  File "/home/parker/anaconda3/envs/zoomcamp/lib/python3.9/site-packages/httpx/_models.py", line 749, in raise_for_status
    raise HTTPStatusError(message, request=request, response=self)
httpx.HTTPStatusError: Client error '404 Not Found' for url 'http://127.0.0.1:4200/api/block_types/slug/github/block_documents/name/week2__prefect?include_secrets=true'
For more information check: https://httpstatuses.com/404

The above exception was the direct cause of the following exception:

Traceback (most recent call last):
  File "/home/parker/anaconda3/envs/zoomcamp/lib/python3.9/site-packages/prefect/blocks/core.py", line 687, in load
    block_document = await client.read_block_document_by_name(
  File "/home/parker/anaconda3/envs/zoomcamp/lib/python3.9/site-packages/prefect/client/orion.py", line 1217, in read_block_document_by_name
    raise prefect.exceptions.ObjectNotFound(http_exc=e) from e
prefect.exceptions.ObjectNotFound

The above exception was the direct cause of the following exception:

Traceback (most recent call last):
  File "/home/parker/anaconda3/envs/zoomcamp/lib/python3.9/site-packages/prefect/cli/_utilities.py", line 41, in wrapper
    return fn(*args, **kwargs)
  File "/home/parker/anaconda3/envs/zoomcamp/lib/python3.9/site-packages/prefect/utilities/asyncutils.py", line 230, in coroutine_wrapper
    return run_async_in_new_loop(async_fn, *args, **kwargs)
  File "/home/parker/anaconda3/envs/zoomcamp/lib/python3.9/site-packages/prefect/utilities/asyncutils.py", line 181, in run_async_in_new_loop
    return anyio.run(partial(__fn, *args, **kwargs))
  File "/home/parker/anaconda3/envs/zoomcamp/lib/python3.9/site-packages/anyio/_core/_eventloop.py", line 70, in run
    return asynclib.run(func, *args, **backend_options)
  File "/home/parker/anaconda3/envs/zoomcamp/lib/python3.9/site-packages/anyio/_backends/_asyncio.py", line 292, in run
    return native_run(wrapper(), debug=debug)
  File "/home/parker/anaconda3/envs/zoomcamp/lib/python3.9/asyncio/runners.py", line 44, in run
    return loop.run_until_complete(main)
  File "/home/parker/anaconda3/envs/zoomcamp/lib/python3.9/asyncio/base_events.py", line 647, in run_until_complete
    return future.result()
  File "/home/parker/anaconda3/envs/zoomcamp/lib/python3.9/site-packages/anyio/_backends/_asyncio.py", line 287, in wrapper
    return await func(*args)
  File "/home/parker/anaconda3/envs/zoomcamp/lib/python3.9/site-packages/prefect/cli/deployment.py", line 872, in build
    storage = await Block.load(storage_block)
  File "/home/parker/anaconda3/envs/zoomcamp/lib/python3.9/site-packages/prefect/client/utilities.py", line 47, in with_injected_client
    return await fn(*args, **kwargs)
  File "/home/parker/anaconda3/envs/zoomcamp/lib/python3.9/site-packages/prefect/blocks/core.py", line 691, in load
    raise ValueError(
ValueError: Unable to find block document named week2__prefect for block type github
An exception occurred.


## Question 5. Email or Slack notifications

Q5. It’s often helpful to be notified when something with your dataflow doesn’t work as planned. Choose one of the options below for creating email or slack notifications.

The hosted Prefect Cloud lets you avoid running your own server and has Automations that allow you to get notifications when certain events occur or don’t occur. 

Create a free forever Prefect Cloud account at app.prefect.cloud and connect your workspace to it following the steps in the UI when you sign up. 

Set up an Automation that will send yourself an email when a flow run completes. Run the deployment used in Q4 for the Green taxi data for April 2019. Check your email to see the notification.

Alternatively, use a Prefect Cloud Automation or a self-hosted Orion server Notification to get notifications in a Slack workspace via an incoming webhook. 

Join my temporary Slack workspace with [this link](https://join.slack.com/t/temp-notify/shared_invite/zt-1odklt4wh-hH~b89HN8MjMrPGEaOlxIw). 400 people can use this link and it expires in 90 days. 

In the Prefect Cloud UI create an [Automation](https://docs.prefect.io/ui/automations) or in the Prefect Orion UI create a [Notification](https://docs.prefect.io/ui/notifications/) to send a Slack message when a flow run enters a Completed state. Here is the Webhook URL to use: https://hooks.slack.com/services/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

Test the functionality.

Alternatively, you can grab the webhook URL from your own Slack workspace and Slack App that you create. 


How many rows were processed by the script?

- `125,268`
- `377,922`
- `728,390`
- `514,392`


## Question 6. Secrets

Prefect Secret blocks provide secure, encrypted storage in the database and obfuscation in the UI. Create a secret block in the UI that stores a fake 10-digit password to connect to a third-party service. Once you’ve created your block in the UI, how many characters are shown as asterisks (*) on the next page of the UI?

- 5
- 6
- 8
- 10


## Submitting the solutions

* Form for submitting: https://forms.gle/PY8mBEGXJ1RvmTM97
* You can submit your homework multiple times. In this case, only the last submission will be used. 

Deadline: 6 February (Monday), 22:00 CET


## Solution

We will publish the solution here