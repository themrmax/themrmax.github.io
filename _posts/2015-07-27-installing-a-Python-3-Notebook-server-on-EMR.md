---
title: Installing a Python 3 Notebook server on EMR
layout: post
---

A coworker recently asked for my help getting a Python 3 IPython Notebook server running on an Amazon EC2 instance. It was pretty straightforward in the end, but it involved a bit of trial and error, and googling around for the exact commands, so I thought I'd collect the steps here just in someone else (or myself) needs to do this without having to run through all the steps.
I tried this on the primary node of an  EMR cluster, AMI version 3.8.0, try at your own peril on any other AMI...

1. Run this in Bash on your EC2 instance:

    `sudo yum install python34 python34-devel python34-pip`

    `sudo pip-3.4 install pyzmq jinja2 tornado jsonschema ipython`\

    `ipython notebook --no-browser --port=8890`

2. Add a tunnel in your SSH client with Source port `8890` and Destination `localhost:8890`.

3. Browse to in your local browser to
http://localhost:8890/notebooks/Untitled.ipynb?kernel_name=python3

4. Enjoy!
