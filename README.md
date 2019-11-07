# Most Popular Check-ins Place in Instagram

## Introduction

This project scrape 'ustrip' hashtag data from Instagram social application for finding the most popular US check-ins place. Moreover, this project also use data from https://www.macrotrends.net/ for US index data, https://www.ncdc.noaa.gov/ for average temperature of each state in each month data, and https://crime-data-explorer.fr.cloud.gov/ for crime data. These 3 data are used to analysis which factors that would affect to number of check-ins in each place. You can also find the popular place to check-ins for each state in each month too. Enjoy the programing. 

## Part 1 : Scraping Data

```python
from selenium import webdriver
from bs4 import BeautifulSoup
import re
import time
import pandas as pd
import numpy as np
```
```python
# Create the driver to open instagram browser
driver = webdriver.Chrome()
driver.get('https://www.instagram.com/explore/tags/ustrip')
driver.maximize_window()
height = driver.execute_script("return document.body.scrollHeight")
links = []

for i in range(20000):
    # Instagram use infinite scroll webpage, therefore to scrape the data, have to scroll down for updating the new content
    driver.execute_script("window.scrollTo(0, document.body.scrollHeight);")
    source = driver.page_source
    data = BeautifulSoup(source, 'lxml')
    body = data.find('body')
    span = body.find('span')
    for link in span.findAll('a'):
        # As in the html, the link of each posts would contain /p before each unique value
        if re.match("/p", link.get('href')):
            links.append('https://www.instagram.com'+link.get('href'))
    update_height = driver.execute_script("return document.body.scrollHeight")
    #Sometimes,the webpage failed to load the new content, then I scroll the page up to refresh it  
    if update_height == height:
        driver.execute_script("window.scrollBy(0,-1000)") 
    height = update_height
    #Set each scrolling time for waiting the website to load the information
    time.sleep(1)
    
### Caution : This process take a lot of time (nearly a day)
```
```python
# As I collect all the links from each loop, it will contain a lot of duplicate link while scroll down in each loop.
len(links)
```
```python
# Delete the duplicate links
links2 = list(dict.fromkeys(links))
len(links2)
```
```python
check_ins = []
date = []
driver2 = webdriver.Chrome()

# Collect the chech-ins location and date
for url in links2 :
    try :
        #Find the specific div of Check-ins
        driver2.get(url)
        page = driver2.page_source
        soup = BeautifulSoup(page, 'lxml')
        body = soup.find("body")
        span = body.find("span")
        section = span.find("section")
        main = section.find("main")
        div = main.find("div")
        div2 = div.find("div")
        article = div2.find("article")
        header = article.find("header")
        div3 = header.find("div",attrs={"class": "JF9hh"})
        check_ins.append(div3.text)
    
        #Find the specific div of Check-ins Date page = driver.page_source
        div4 = article.find("div",attrs={"class": "eo2As"})
        div5 = div4.find("div",attrs={"class": "k_Q0X NnvRN"})
        time = div.find("time")
        date.append(time.get("datetime"))
    
    except :
        pass
```

```python
# Create data frame
join = {'Location':check_ins,'Date':date}
df = pd.DataFrame(join)
# Set index to Location
df.set_index('Location', inplace=True)
```

The first process is to scrape the check-ins and date of the user from Instagram. In this project, I use selenium to connect to Instagram website and use Beautiful Soup to read the content of html. As Instagram is a infinite scroll website, I use selenium to open the browser and scroll the page down until the end to collect all the post links. Then I use for loop to open all the links and get the check-ins location and date from each posts.

## Part 2 : Cleaning Data

As all posts didn’t contain check-ins place so the second step is clear all rows that don’t have check-ins data by delete all none values.Moreover, I would like to find the factor that would affect to number of check-ins. There are 3 variables that I think it would have some relation with number of travel which are USIndex, Temperature and Crime rate. So I use data from macrotrend website for USIndex data, National Center for Environmental information for average temperature of each state, and FEDERAL BUREAU OF INVESTIGATION for the crime data. For the temperature data, it contain separate data of each stage, so I use for loop to run all files and then concatenate all the file. Then I have to clean all data up a bit and rename some columns for more readable.

## Part 3 : Finding Latitude and Longitude

After I have all location data, the next process is to find the latitude and longitude of the location. I use geo coder to find the location from the name of the check-ins. And also drop the place the cannot find the location of it. 

## Part 4 : Creating Database

After I got all the data, I create the Instagram 'ustrip' Check-ins Database. I check up the posts number of each year and observe that some year contain only just a small data, so I decide to use the data from 2016 to 2019.

This are the function in database :

1. Geomap plot for total check-ins of each state in each season, month or year. 
2. The plot of total number of check-ins of each state from JAN 2016 to OCT 2019.
3. Top check-ins place for each state and number of check-ins. 
4. Map plot top Top check-ins place for each state and number of check-ins.
5. The number of checkins with US Index. 
6. The number of checkins with temperature of each state.
7. The number of checkins with crime rate 

## Conclusion :

From the database, you can find the popular place or state to visit in each season. Moreover, it also show that there are mostly likely to have 2 peak check-ins periods for each year, which are March to April and August to September. From the US Index graph, it suggest that US Index are not seem to affect to number of check-ins. I think this is because even the US Index is high, but if it is in the high season, they would still get out to travel in US anyway. From the temperature graph, it number of check-ins is in the same trend as temperature. This must because international tourist would likely to have some cold experience or see some snow as their country  didn’t have such a thing, so new experince is what people want.Finally, the crime graph show that the crime rate is continuously drop since 2016 and the number of chech-ins also rise up too. Thererfore, the safety of place would also affect number of tourist too.
