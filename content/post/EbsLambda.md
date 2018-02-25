+++
date = 2018-02-14
lastmod = 2018-02-14
draft = false
tags = ["tags", "aws", "boto3","EBS","serverless", "lambda", "ec2"]
title = "Working with AWS Lambda"
math = true
summary = """
AWS Lambda lets you run code without provisioning or managing servers.Lets see how AWS Lambda works using a simple use case.
"""

[header]
image = "headers/EbsLambda.png"
+++



[AWS Lambda](https://aws.amazon.com/lambda/) automatically runs your code without requiring you to manage any servers. With Lambda, you can run code for virtually any type of application or backend service - all with zero administration. Just upload your code and Lambda takes care of everything required to run and scale your code with high availability. You can set up your code to automatically trigger from other AWS services or call it directly from any web or mobile app. You pay only for the compute time you consume - there is no charge when your code is not running.

In my [previous](https://bapi24.github.io/post/aws-tags/) post, we have worked tagging the [EBS](https://aws.amazon.com/ebs/) volumes with the help of a python script. Now lets work on deploying that function through [AWS Lambda](https://aws.amazon.com/lambda/) which gets triggered every 60 mins(customizable).

Before jumping into the code, lets see what are we trying to achieve here. We want to automate the task of appending our [EC2](https://aws.amazon.com/ec2/) tags on the corresponding EBS volumes on regular intervals of time, to help us keep the EBS volumes organized.


![Working with Lambda](/img/lambda/lambda_a.png)

### **STEPS**:
- We need to give access to Lambda function(L) to talk to EC2 instances(E) and Cloudwatch Log groups(CL).

```yaml
AWSTemplateFormatVersion: '2010-09-09'

Description: Create IAM roles and policies to tag EBS volumes based on EC2 tags

Resources:
  EbsLambdaServiceRole:
    Type: "AWS::IAM::Role"
    Properties:
      AssumeRolePolicyDocument:
        Version: "2012-10-17"
        Statement:
          -
            Effect: "Allow"
            Principal:
              Service:
                - "lambda.amazonaws.com"
            Action:
              - "sts:AssumeRole"
      Path: "/"
      ManagedPolicyArns:
        - !Ref EbsLambdaEC2Policy
        - !Ref EbsLambdaLogPolicy
      RoleName: EbsLambdaServiceRole

EbsLambdaEC2Policy:
  Type: "AWS::IAM::ManagedPolicy"
  Properties:
    Description: "Managed Policy to grant Lambda permissions to access EC2 for EBS tags"
    Path: "/"
    PolicyDocument:
      Version: "2012-10-17"
      Statement:
        -
          Sid: "AllowLambdaToAccessEC2"
          Effect: "Allow"
          Action:
            - ec2:* #ec2:Describe* #TO-DO: check for permissions
          # Condition:
          #   "StringEquals":
          #     - ec2:ResourceTag
          Resource: "*" #arn:aws:ec2:us-east-1:"acc-no":instance/*
    ManagedPolicyName: EbsLambdaEC2Policy

EbsLambdaLogPolicy:
  Type: AWS::IAM::ManagedPolicy
  Properties:
    Description: "Managed Policy to give access to Lambda to create logs in cloudwatch"
    Path: "/"
    PolicyDocument:
      Version: "2012-10-17"
      Statement:
       -
         Sid: "AllowLambdaToCreateLogs"
         Effect: Allow
         Action:
           - logs:*
         Resource: "*"
    ManagedPolicyName: EbsLambdaLogPolicy
```

- We need to trigger the lambda function(L) from the Cloudwatch events(CE) in regular intervals using [ScheduledRule](https://docs.aws.amazon.com/AmazonCloudWatch/latest/events/ScheduledEvents.html) and give Cloudwatch Events permission to [invoke Lambda](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-lambda-permission.html).

```yaml
ScheduledRule:
  Type: "AWS::Events::Rule"
  Properties:
    Description: "AWS Rule to run the function in specific intervals"
    ScheduleExpression: "rate(60 minutes)"
    State: "ENABLED"
    Targets:
      - Arn: !GetAtt lambdaEbsFunction.Arn
        Id: "TargetFunctionV1"

PermissionForEventsToInvokeLambda:
  Type: "AWS::Lambda::Permission"
  Properties:
    FunctionName: !GetAtt lambdaEbsFunction.Arn
    Action: "lambda:InvokeFunction"
    Principal: "events.amazonaws.com"
    SourceArn:
      Fn::GetAtt:
        - ScheduledRule
        - "Arn"
```

- Create a Lambda function that [tags EBS volumes of the corresponding EC2 instances](https://bapi24.github.io/post/aws-tags/) (which we worked on our previous post) through a cloudformation script

```YAML
lambdaEbsFunction:
  Type: "AWS::Lambda::Function"
  Properties:
    Code:
      ZipFile: |
        #!/usr/bin/env python
        from __future__ import print_function
        import boto3
        import json
        import logging

        #setup simple logging for INFO.
        logger = logging.getLogger()
        logger.setLevel(logging.ERROR)

        #define the connection for EC2.
        ec2 = boto3.resource('ec2', region_name="us-east-2")
        def lambda_handler(event, context):
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
                              'Name': 'tag:Name',
                              'Values': ['Jenkins Server']
                          },

                      ]
          )

          for instance in base_instances:
              for vol in instance.volumes.all():
                  tag = vol.create_tags(Tags=tag_them(instance, vol.attachments[0]['Device']))
                  print("[INFO]: " + str(tag))

    Description: "Function to add tags to EBS"
    FunctionName: !Ref lambdaFunctionName
    Handler: "index.lambda_handler"
    Role: "arn:aws:iam::123456789:role/EbsLambdaServiceRole" #TODO: account number hard-coded
    Runtime: "python3.6"
    Timeout: 5

```

You can get the complete CloudFormation scripts  [here](https://github.com/bapi24/aws_labs/tree/master/serverless/ebstags/cfn).
