# BuLogs (Bug Logs aka a log for bugs that I killed)

## S3 syncing GCS

### Issue 1
1) issue: failed to `rsync` s3 amplitude bucket with gs bucket.
2) error code: http `400` BadRequest None.
3) cause: `gsutil` new version does not allow `#` in the object names/paths.
4) how I found out: run `gsutil -D cp s3://{bucket_name}/{file_name_with_#}.json gs://{bucket_name}` provides detailed debugging session.
5) solution: rename the historic s3 objects using script, and create a lambda function to handle new incoming objects, save all in a new bucket, move all downstream pointers to this bucket. * note: for lambda function that is triggered by s3 objects, you need to grant s3 access.

### Issue 2
1) issue: failed to copy renamed object.
2) error code: http `301` moved permanently.
3) cause: bucket created in a different region than that is configured in `~/.boto` file. 
4) solution: 
    - name a new bucket, move all old objects to new bucket, chnage downstream pointers, delete old bucket;
    - fix the `~/.boto` confi when `ssh` in as `airflow` instead of root. The `s3_host` must point to the correct bucket region/zone.

## Churn Model

1) issue: getting stuck on a google requests package (although properly installed on server, it refuses to work).
2) error code: NA
3) cause: a chaotic server environment.
4) how I found out: NA
5) solution: dockerize the base image and run modeling on the saved image. Step by step see the `Build` part.

# BuiLogs (Build Blogs aka a log for things that I built)

## Airflow log bot
1. Configure airflow to send logs to gs bucket;
2. Set up Cloud Function triggered by storage bucket;
3. Write json transformation in the Cloud Function:
    - Requirements:
        1) Only send the final failed attempts with the detailed traceback;
        2) Only send the traceback pertaining `.py` scripts (not the airflow dag traceback). I wrote some helper functions with two pointers to get the part between `Traceback` and `ERROR` lines.
        3) Formatted in a reader-friendly manner.
        4) Convert timestamps to PST.
4. Send payload to Slack incoming webhook. 

## Dockerize churn model
1. Choose base image: google/cloud-sdk with Python3.5 & Python2
2. (How I found out this image is using only Python3.5): `f` formatted strings are not being recognized in this environment.
3. Dataproc API is _very_ cool.