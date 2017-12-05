+++
date = 2017-10-11
lastmod = 2017-12-05
draft = false
tags = ["academic", "hugo"]
title = "Setting up AWS cli on mac"
math = true
summary = """
Lets set up AWS cli on mac
"""

[header]
image = "headers/awscli.png"
+++

### **Steps**:

- Install homebrew
- Install python3 using brew
- Install aws cli using brew
- Configure aws cli
- Test with simple script

#### **Install homebrew**
[Homebrew](https://brew.sh/) is package manager for Mac OS similar to apt-get for Ubuntu. To install homebrew, Paste the following at a terminal prompt:

```bash
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```
#### **Check brew version:**
```bash
$brew --version
```
#### **Install python3**
Mac comes with python 2 installed, lets install python3 using homebrew

```bash
$brew install python3
```

#### **Test python**
To test python let us print a simple statement
```bash
$echo "print(\"hello world\")" >> hello.py
```

> make sure you run with python3

```bash
$python3 hello.py
```

#### **Install aws cli**

```bash
$brew install awscli
```

#### **Test aws cli**
```bash
$aws --version
```

#### **Configure AWS cli**
```bash
$aws configure --profile dev
```
Enter AWS access key, secret access key and default region to configure AWS cli.
