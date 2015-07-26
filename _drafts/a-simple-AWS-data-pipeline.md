---
title: A simple AWS Data Pipeline
layout: post
---
Data pipelines are a good way to deploy a simple data processing task which needs to run on a daily or weekly schedule; it will automatically provision an EMR cluster for you, run your script, and then shut down at the end. If the pipeline is more complex, it might be worth using something like [AirBnB's Airflow][1] which has recently been open-sourced and is definitely on my list of things to check out.

First launch an EC2 instance with the `DataPipelineDefaultResourceRole` IAM role. If you don't have these roles yet, you'll have to first create a data pipelines using the data pipeline web app.

I created my instance using EMR web app,  since this is what I'm most used to, launching a single-node cluster, and selecting the *Custom* option under the **IAM Roles** section. This is probably not the best way (since the smallest is an `m1.xlarge`, when a `t1.micro` would be enough for this) but I haven't bothered to learn the normal way of starting EC2 instances yet.

Now SSH into the EC2 instance, and you can manage your data pipelines using the `aws` CLI. For example you can list your actives pipelines with `aws datapipeline list-pipelines`:


    [hadoop@ip-10-241-36-6 ~]$ aws datapipeline list-pipelines
    {
        "pipelineIdList": [
            {
                "id": "df-05707631BML72LK1COZH",
                "name": "pipeline1"
            },
            {
                "id": "df-03353883P28DFWSUC799",
                "name": "pipeline2"
            },
            {
                "id": "df-011789115PMSDFP8HB3R",
                "name": "pipeline3"
            }
        ]
    }

[1]:http://nerds.airbnb.com/airflow/
