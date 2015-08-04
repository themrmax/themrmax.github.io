---
title: Deploying a scheduled task on AWS data pipeline
layout: post
---
Sorry if this is a bit morbid but my partner mentioned that it's sad when old aquanintances die and you don't hear about it until years later. I thought that there must be some free service that would provide notifcations, but all I could find was a ridiculously overpriced service. [find the link?]

So I wrote a quick scraper to parse the death notices in The Age, check against a watchlist of names and send an email notification if a match is found (I decided to try out SES -- AWS's "Simple Email Service" to send the emails.)

But where to host it? Although I've always been a nerd -- I was on the melbourne PC User group's BBS in the 90s -- I've never been one of those people to have their own serer running in the bedroom. To be honest, I've only started getting good at Linux, since I've started using AWS instances at my current work.  A friend at work mentioned that he's bought a three-year t2.micro instance on AWS to use as his personal server. But is a personal server a bit of a dated concept in the age of cloud? As an experiment I'm going to see how long I can make my AWS free tier last. (I signed up the a couple weeks ago and the challenge is to see if I can last the whole year without going over 750 hours of computing time.)

So: how can I deploy my scraper to run every night, without burning up my 750 hours by having an instance sitting idle for the rest of the day? Data pipeline of course! First I need a Bash script to download the file (I could upload it to s3, but to make things a little simpler I'm going to download it straight form Github). My script just unzips the package in the home directory of the EC2 instance -- probably not best practice if I was going to keep this instance going, but since it's going to be blown away after a few minutes, I get kind of lazy about creating good directory structures and the like. So here's my script -- super simple.

    wget https://github.com/themrmax/iwishidknown/archive/master.zip
    mkdir /iwishidknown
    unzip master.zip ./
    python iwishidknown.py

Now the data pipeline defininition: it's in the form of a JSON that tells AWS what setup you're going to use for your scheduled task, mine's really simple, it just tells AWS to spin up an t2.micro, run the script.

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
          "scriptUri": "s3://scripts/iwishidknown.sh",
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



Finally the deployment script, which uploads the script from Github to S3, and provisions the data pipeline. To run the deployment, you'll need to have `awscli` installed; I `brew install`'ed it, since I'm on my Macbook at home.

    wget https://raw.githubusercontent.com/themrmax/iwishidknown/master/iwishidknown.sh
    aws s3 mb s3://iwishidknown-log
    aws s3 mb s3://iwishidknown
    aws s3 cp iwishidknown.sh s3://iwishidknown/iwishidknown.sh
    aws datapipeline create-pipeline --name iwishidknown --unique-id iwishidknown --region ap-southeast-2b
