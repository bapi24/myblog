+++
date = 2018-02-04
lastmod = 2018-02-04
draft = false
tags = ["tags", "aws", "boto3","EBS", "ec2"]
title = "Working with AWS tags"
math = true
summary = """
Tagging your services in the cloud enables you categorize infrastructure which makes it easier to manage. For example you could tag your resources based on environment and use them to implement in CI/CD.
"""

[header]
image = "headers/tag.png"
+++

[Tagging](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/Using_Tags.html) your AWS resources helps you to manage your resources.It is useful when you have large number of resources of same type-where you can quickly identify a specific resource based on a tag. For example to identify jenkins dev instance in your ec2 fleet.

You can also edit or tags of your existing infrastructure.
![Example image](/img/tags/pic1.png)


Let us work on a problem we face when dealing with large number of ec2 instances. Most of the time we tend to ignore naming EBS volumes and go default settings while launching ec2 instances. It becomes a tedious task to keep track of those volumes at enterprise level. Lets solve that using AWS tagging where we tag EBS volumes to keep track of them.

I used a python script and leveraged [Boto3 library](https://boto3.readthedocs.io/en/latest/) to filter the instances which are running and apply ec2 tags to corresponding EBS volumes.
```python
from __future__ import print_function

import boto3
import json
import logging

#setup simple logging for INFO
logger = logging.getLogger()
logger.setLevel(logging.ERROR)

#define the connection for EC2
ec2 = boto3.resource('ec2', region_name="us-east-2")

def tag_them(instance, detail):
    tempTags=[]
    v={}

    for t in instance.tags:
        #pull the name tag and add volume detail
        if t['Key'] == 'Name':
            v['Value'] = t['Value'] + " - " + str(detail)
            v['Key'] = 'Name'
            tempTags.append(v)
        #append the wanted tags to EBS
        elif t['Key'] == 'Owner':
            print("[INFO]: Owner tag " + str(t))
            tempTags.append(t)
        elif t['Key'] == 'Environment':
            print("[INFO]: Environment Tag " + str(t))
            tempTags.append(t)
        else:
            print("[INFO]: Skip Tag - " + str(t))

    print("[INFO] " + str(tempTags))
    return(tempTags)

base_instances = ec2.instances.filter(
    Filters = [
                {
                    'Name': 'instance-state-name',
                    'Values': ['running']
                },
                {
                    'Name': 'tag:Name',
                    'Values': ['Jenkins Server']
                },

            ]
)

for instance in base_instances:
    for vol in instance.volumes.all():
        tag = vol.create_tags(Tags=tag_them(instance, vol.attachments[0]['Device']))
        print("[INFO]: " + str(tag))

```

Lets go behind the scenes and check how it works through AWS console!
![pic 2](/img/tags/pic2.png)
![pic 3](/img/tags/pic3.png)
![pic 4](/img/tags/pic4.png)

You can follow these practices to get started with tagging:
https://aws.amazon.com/answers/account-management/aws-tagging-strategies/

Lets setup a [lambda function](https://aws.amazon.com/lambda/) in next post where it will run every one hour to apply tags to EBS volumes based on EC2 tags.
