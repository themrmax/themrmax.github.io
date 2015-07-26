---
title: Installing a Python 3 Notebook server on EMR
layout: post
---

A coworker recently asked for my help getting a Python 3 IPython Notebook server running on an Amazon EC2 instance. It was pretty straightforward in the end, but it involved a bit of trial and error, so I thought I'd collect the steps here just in someone else (or myself) needs to do this again. I tried this on the primary node of an  EMR cluster, AMI version 3.8.0, try at your own peril on any other AMI...

Run this in Bash on your EC2 instance:
```bash
sudo yum install python34 python34-devel python34-pip
sudo pip-3.4 install pyzmq jinja2 tornado jsonschema ipython
ipython notebook --no-browser --port=8890
```
Add a tunnel in your SSH client with Source port `8890` and Destination `localhost:8890`.

Browse to in your local browser to
http://localhost:8890/notebooks/Untitled.ipynb?kernel_name=python3

Enjoy!
