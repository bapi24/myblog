+++
date = 2017-12-04
lastmod = 2017-12-04
draft = false
tags = ["cfn", "aws", "CloudFormation", "ec2"]
title = "Deploy EC2 using Sample CloudFormation stack"
math = true
summary = """
Lets launch an ec2 instance using cloudformation and AWS cli
"""

[header]
image = "headers/AWS_lego_cropped.png"
+++



Let us launch a test ec2 instance using AWS  [CloudFormation](https://aws.amazon.com/cloudformation/). CloudFormation is an orchestration tool to provision infrastructure in AWS and makes your deployment much faster and easier. Although you can use config management tools like Ansible, puppet and Chef to deploy AWS services, there are some advantages using orchestration tools like CloudFormation and Terraform. [This](https://blog.gruntwork.io/why-we-use-terraform-and-not-chef-puppet-ansible-saltstack-or-cloudformation-7989dad2865c) article explains in detail the differences between the two.

To get started with CloudFormation, you can refer to these sample CloudFormation stacks  [Sample CloudFormation stacks](http://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/cfn-sample-templates.html).
If you prefer working with YAML to JSON, you can make use of [cfn-flip](https://github.com/awslabs/aws-cfn-template-flip) to convert JSON to YAML.

Below is the CloudFormation template to launch a test EC2 instance.
```yaml

AWSTemplateFormatVersion: '2010-09-09'

Description: 'This template shows how to associate an Elastic IP address with an Amazon EC2 instance
  **WARNING** This template creates an Amazon EC2 instance and an Elastic IP Address.
  You will be billed for the AWS resources used if you create a stack from this template.'

Parameters:
  InstanceType:
    Description: WebServer EC2 instance type
    Type: String
    Default: t2.micro
    AllowedValues:
      - t1.micro
      - t2.nano
      - t2.micro
      - t2.small
      - t2.medium
    ConstraintDescription: must be a valid EC2 instance type.
  KeyName:
    Description: Name of an existing EC2 KeyPair to enable SSH access to the instances
    Type: AWS::EC2::KeyPair::KeyName
    ConstraintDescription: must be the name of an existing EC2 KeyPair.
    Default: baws.admin
  SSHLocation:
    Description: The IP address range that can be used to SSH to the EC2 instances
    Type: String
    MinLength: '9'
    MaxLength: '18'
    Default: 0.0.0.0/0
    AllowedPattern: (\d{1,3})\.(\d{1,3})\.(\d{1,3})\.(\d{1,3})/(\d{1,2})
    ConstraintDescription: must be a valid IP CIDR range of the form x.x.x.x/x.

Mappings:
  AWSInstanceType2Arch:
    t1.micro:
      Arch: PV64
    t2.nano:
      Arch: HVM64
    t2.micro:
      Arch: HVM64
    t2.small:
      Arch: HVM64
    t2.medium:
      Arch: HVM64

  AWSInstanceType2NATArch:
    t1.micro:
      Arch: NATPV64
    t2.nano:
      Arch: NATHVM64
    t2.micro:
      Arch: NATHVM64
    t2.small:
      Arch: NATHVM64
    t2.medium:
      Arch: NATHVM64

  AWSRegionArch2AMI:
    us-east-1:
      PV64: ami-2a69aa47
      HVM64: ami-6869aa05
      HVMG2: ami-1f12e965
    us-west-2:
      PV64: ami-7f77b31f
      HVM64: ami-7172b611
      HVMG2: ami-5c9b6124
    us-west-1:
      PV64: ami-a2490dc2
      HVM64: ami-31490d51
      HVMG2: ami-7291a112
    us-east-2:
      PV64: NOT_SUPPORTED
      HVM64: ami-f6035893
      HVMG2: NOT_SUPPORTED

Resources:
  EC2Instance:
    Type: AWS::EC2::Instance
    Properties:
      UserData: !Base64
        Fn::Join:
          - ''
          - - IPAddress=
            - !Ref 'IPAddress'
      InstanceType: !Ref 'InstanceType'
      SecurityGroups:
        - !Ref 'InstanceSecurityGroup'
      KeyName: !Ref 'KeyName'
      ImageId: !FindInMap [AWSRegionArch2AMI, !Ref 'AWS::Region', !FindInMap [AWSInstanceType2Arch,
          !Ref 'InstanceType', Arch]]
  InstanceSecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: Enable SSH access
      SecurityGroupIngress:
        - IpProtocol: tcp
          FromPort: '22'
          ToPort: '22'
          CidrIp: !Ref 'SSHLocation'
  IPAddress:
    Type: AWS::EC2::EIP
  IPAssoc:
    Type: AWS::EC2::EIPAssociation
    Properties:
      InstanceId: !Ref 'EC2Instance'
      EIP: !Ref 'IPAddress'

Outputs:
  InstanceId:
    Description: InstanceId of the newly created EC2 instance
    Value: !Ref 'EC2Instance'
  InstanceIPAddress:
    Description: IP address of the newly created EC2 instance
    Value: !Ref 'IPAddress'
```

Once you have AWS cli set up(if not refer [this](https://bapi24.github.io/post/setup-aws-cli-on-mac/)) you can create a CFN stack from above template using AWS cli:

```bash
aws cloudformation create-stack --stack-name test-instance \
--template-body file://ec2-elastic-ip.yaml
```
