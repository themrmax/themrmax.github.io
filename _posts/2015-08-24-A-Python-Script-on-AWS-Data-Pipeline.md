---
title: A Python script on AWS Data Pipeline
layout: post
---
Data pipelines are a good way to deploy a simple data processing task which needs to run on a daily or weekly schedule; it will automatically provision an EMR cluster for you, run your script, and then shut down at the end. If the pipeline is more complex, it might be worth using something like [AirBnB's Airflow][2] which has recently been open-sourced and is definitely on my list of things to check out.

The other day I wrote [a quick scraper which I wanted to run every night.][1] But where to host it? Although I've always been a nerd – I was a regular on the Melbourne PC User Group BBS back in the 90s – I've never been one of those people to have their own sevrer running in the bedroom. To be honest, I've only started getting good at Linux since I've started using AWS at my current work.  

A friend at work mentioned that he's bought a three-year t2.micro instance on AWS for two-hundred-and-something dollars to use as his personal server. But is a personal server a bit of an outdated concept in the age of cloud? As a challenge I'm going to see how long I can make my AWS free tier last. (I signed up a couple weeks ago and my challenge is to see if I can last the whole year without going over the 750 hours of free computing time.)

So: how can I deploy my scraper to run every night, without burning up my 750 hours by having an instance sitting idle for the rest of the day? I decided to go with Data Pipeline (I'd have like to have gone with tried AWS Lambda but I'm waiting for them to add native Python support.) First I need a Bash script to download the file. My script just downloads all it's resources to the home directory of the EC2 instance -- probably not best practice if I was going to keep this instance going, but since it's going to be blown away after a few minutes, I get kind of lazy about creating good directory structures and the like. So here's my script -- super simple, just grab the Python script from s3 and run it.

```bash

aws s3 cp s3://iwishidknown/iwishidknown.py ./
aws s3 cp s3://iwishidknown/watchlist.txt ./
python iwishidknown.py
```

Now the Data Pipeline defininition: it's in the form of a JSON that tells AWS what setup you're going to use for your scheduled task, mine's really simple, it just tells AWS to spin up an t2.micro, run the script. (I created it in the Data Pipeline webapp in the AWS Console, and exported the json defintion using the big blue export button. However I had to add the line ``"imageId": "ami-fd9cecc7"`` since I got an error when I tried to deploy.)

```json

    {
      "objects": [
        {
          "period": "1 days",
          "name": "Every 1 day",
          "id": "DefaultSchedule",
          "type": "Schedule",
          "startAt": "FIRST_ACTIVATION_DATE_TIME"
        },
        {
          "schedule": {
            "ref": "DefaultSchedule"
          },
          "scriptUri": "s3://iwishidknown/iwishidknown.sh",
          "name": "DefaultActivity1",
          "id": "ActivityId_qyoFQ",
          "runsOn": {
            "ref": "ResourceId_KlGSk"
          },
          "type": "ShellCommandActivity"
        },
        {
          "schedule": {
            "ref": "DefaultSchedule"
          },
          "imageId": "ami-fd9cecc7",
          "instanceType": "t2.micro",
          "name": "DefaultResource1",
          "id": "ResourceId_KlGSk",
          "type": "Ec2Resource",
          "terminateAfter": "30 Minutes"
        },
        {
          "failureAndRerunMode": "CASCADE",
          "schedule": {
            "ref": "DefaultSchedule"
          },
          "resourceRole": "DataPipelineDefaultResourceRole",
          "role": "DataPipelineDefaultRole",
          "pipelineLogUri": "s3://iwishidknown-log/",
          "scheduleType": "cron",
          "name": "Default",
          "id": "Default"
        }
      ],
      "parameters": []
    }
```


Finally the deployment script, which uploads the script from Github to S3, and provisions the data pipeline. Notice that you have to do this in two steps; first create the pipeline and save the id, then set the pipeline definition. To run the deployment:
 * You'll need to have `awscli` installed; (I `brew install`'ed it, since I'm on a Macbook.)
 * Go to the "IAM" tab of the AWS console, and create a user, and give it the the "AdministratorAccess" managed policy.
 * Run `aws config` in your terminal with the access and secret keys that it gives you.

```bash

aws s3 mb s3://iwishidknown
aws s3 cp iwishidknown.sh s3://iwishidknown/
aws s3 cp iwishidknown.py s3://iwishidknown/
aws s3 cp watchlist.txt s3://iwishidknown/

response=`aws datapipeline create-pipeline --name iwishidknown --unique-id iwishidknown`
#response is of the form { "pipelineId": "df-05033941C83LSDPT0B42" }
id=${response:21:23}
aws datapipeline put-pipeline-definition --pipeline-id $id --pipeline-definition file://pipelinedef.json
aws datapipeline activate-pipeline --pipeline-id $id
```
[1]:https://github.com/themrmax/iwishidknown

[2]:http://nerds.airbnb.com/airflow/
