+++
date = 2017-11-06
lastmod = 2017-11-06
draft = false
tags = ["python", "data analytics"]
title = "Webscraping using Python"
math = true
summary = """
In this tutorial let us see how to extract data from web using python
"""


+++

Let us see how to extract data from websites using python

<title> <b = 3> What is Webscraping? </b> </title>
Web scraping is used to extract data and present it in a more readable format. In this tutorial we shall focus on xml data, but web scraping can be used in a wide variety of situations.

### Getting Started
Let us extract some data using python language and beautifulsoup4 library

>```bash
note: the commands in this section can be run directly in terminal
```

> to check version of python installed on your machine:
```bash
python --version
```

>install beautifulsoup4 library using pip
```bash
pip install beautifulsoup4
```

### Python Script
Here is the Python script that hits the CTA(Chicago transit authority) bus API at [CTA](http://ctabustracker.com/bustime/map/getStopPredictions.jsp?&stop=14791&route=22)
We want to extract info from the xml data to check the bus arrival

If there is a bus arriving in less than 2 mins, we print "Almost here!!" or if its less than 15 min we print "Coming Soon!!"  or else return "Not anytime soon"

```python
#import necessary libraries
from bs4 import BeautifulSoup
import requests
import re

resp = requests.get('http://ctabustracker.com/bustime/map/getStopPredictions.jsp?&stop=14791&route=22')
#read xml data
xml = resp.text
#create BeautifulSoup object for xml
soup = BeautifulSoup(xml, 'lxml')

#filter cta arrivals by pt tag
next_arrival = soup.pt

#check if the bus is appproaching
if next_arrival.text == 'APPROACHING':
    print("Almost here!!")
    next_arrival_time = 0
else:
    #extract number from the tag
    next_arrival_time = int(re.search(r'\d+', next_arrival.text).group())

    #check if the arrival is less than 15 min
    if next_arrival_time <= 15:  
        print("Coming Soon!!")
    else:
         #return false for arrivals greater than 15 min
        print("Not anytime soon!!")
```
