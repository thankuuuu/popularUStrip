# Most Popular Check-ins Place in Instagram

## Introduction

This project scrape 'ustrip' hashtag data from Instagram social application for finding the most popular US check-ins place. Moreover, this project also use data from https://www.macrotrends.net/ for US index data, https://www.ncdc.noaa.gov/ for average temperature of each state in each month data, and https://crime-data-explorer.fr.cloud.gov/ for crime data. These 3 data are used to analysis which factors that would affect to number of check-ins in each place. You can also find the popular place to check-ins for each state in each month too. Enjoy the programing. 

## Part 1 : Scraping Data

The first process is to scrape the check-ins and date of the user from Instagram. In this project, I use selenium to connect to Instagram website and use Beautiful Soup to read the content of html. As Instagram is a infinite scroll website, I use selenium to open the browser and scroll the page down until the end to collect all the post links. Then I use for loop to open all the links and get the check-ins location and date from each posts.

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
I export the data frame to IG.csv file.

## Part 2 : Cleaning Data

As all posts didn’t contain check-ins place so the second step is clear all rows that don’t have check-ins data by delete all none values.Moreover, I would like to find the factor that would affect to number of check-ins. There are 3 variables that I think it would have some relation with number of travel which are USIndex, Temperature and Crime rate. So I use data from macrotrend website for USIndex data, National Center for Environmental information for average temperature of each state, and FEDERAL BUREAU OF INVESTIGATION for the crime data. For the temperature data, it contain separate data of each stage, so I use for loop to run all files and then concatenate all the file. Then I have to clean all data up a bit and rename some columns for more readable.

```python
import pandas as pd 
```

```python
AllIG = pd.read_csv('IG.csv')
#Clear all none location Check-ins
df = AllIG.dropna()
#Reset the index
df = IG.reset_index(drop=True)
```

```python
# Extracting Month and Year of each post and add the season
month = []
year = []
season = []
for i in range (len(df.index)):
        year.append((df.loc[i,'Date'][0:4]))
        month.append((df.loc[i,'Date'][5:7]))  
        if df.loc[i,'Date'][5:7] in ['03','04','05'] :
            season.append('Spring')
        elif df.loc[i,'Date'][5:7] in ['06','07','08'] :
            season.append('Summer')
        elif df.loc[i,'Date'][5:7] in ['09','10','11'] :
            season.append('Fall')
        elif df.loc[i,'Date'][5:7] in ['12','01','02'] :
            season.append('Winter') 
```
```python
join = {'Month':month,'Year':year,'Season':season}
df2 = pd.DataFrame(join)
Data = pd.concat([df,df2], axis=1)
Data = Data.drop(['Date'],axis =1)
```

I export the data frame to IG_clean.csv file.

## Part 3 : Finding Latitude and Longitude

After I have all location data, the next process is to find the latitude and longitude of the location. I use geo coder to find the location from the name of the check-ins. And also drop the place the cannot find the location of it. 


<b>

<p>
<center>
<font size="6">
Data Mining Project 2
</font>
</center>
</p>

<p>
<center>
<font size="5">
Instagram "ustrip" Check-ins Databease 
</font>
</center>
</p>

<p>
<center>
<font size="3">
Data Mining, Columbian College of Arts & Sciences, George Washington University
</font>
</center>
</p>

<p>
<center>
<font size="3">
Author: Voratham Tiabrat
</font>
</center>
</p>

</b>

# Part 3 : Getting Latitude and Longtitude


```python
!pip install gmplot
!pip install geopy
``` 
```python
import pandas as pd 
import numpy as np
import time
from gmplot import gmplot
from matplotlib import pyplot as plt 
from geopy.geocoders import Nominatim
```
```python
df = pd.read_csv('IG_clean.csv')
df.set_index('Location', inplace=True)
```
```python
location_address = []
latitude = []
longitude = []
# Find th Latitude and Longtitude to plot in the graph
for i in range (len(df.index)):
    try :
        geolocator = Nominatim(user_agent="my-app")
        place = geolocator.geocode(df.index[i])
        location_address.append(place.address)
        latitude.append(place.latitude)
        longitude.append(place.longitude)
    
    except :
        location_address.append(None)
        latitude.append(None) 
        longitude.append(None)
    
    #Geocoder allow user to take maximum of 1 request per second
    time.sleep(1)
```
```python
join = {'Address':location_address,'Latitude':latitude,'Longitude':longitude}
location = pd.DataFrame(join)
```
```python
#set index to match with location dataframe
df = df.reset_index()
Data = pd.concat([df,location], axis=1)
#Clear all none location Check-ins
Datadrop = Data.dropna()
```
I export the data frame to Location.csv file.

## Part 4 : Creating Database

After I got all the data, I create the Instagram 'ustrip' Check-ins Database. I check up the posts number of each year and observe that some year contain only just a small data, so I decide to use the data from 2016 to 2019.

###  4.1 Preparing Data

```python
import pandas as pd 
import numpy as np
from gmplot import gmplot
from matplotlib import pyplot as plt 
from geopy.geocoders import Nominatim
import plotly
import chart_studio.plotly as py
import plotly.graph_objects as go
py.sign_in('thankuuuu', 'KEwJDOVZyb6iwoblLuQO')
```
```python
Data = pd.read_csv('Location.csv')
```
```python
for i in range (len(Data.index)) :
    if Data.loc[i,'Month'] in [1,2,3,4,5,6,7,8,9] :
        Data.loc[i,'Month'] =  '0'+str(Data.loc[i,'Month'])
        
Data['Date']  = Data.Year.astype(str).str.cat(Data.Month.astype(str), sep='/')
```
```python
Data = Data.drop(['Location'],axis =1)
Data = Data.set_index('Address')
Data = Data.dropna()
```
```python
statesheet = pd.read_csv('state-abbrevs.csv')
statelist = statesheet['state'].tolist()
statecode = statesheet['abbreviation'].tolist()
```
```python
#Extracting state from address and adding state code
for address in Data.index:
    for i in range (len(statelist)) :
        if statelist[i] in address :
            Data.loc[address,'State'] = statelist[i]
            Data.loc[address,'Code'] = statecode[i]
```
```python
#Subsetting each year
#Focusing on 2016-1019
Data_2016 = Data.loc[(Data.Year == 2016),]
Data_2017 = Data.loc[(Data.Year == 2017),]
Data_2018 = Data.loc[(Data.Year == 2018),]
Data_2019 = Data.loc[(Data.Year == 2019),]
Data = pd.concat([Data_2016,Data_2017,Data_2018,Data_2019])

#Subsetting each month
Data_01 = Data.loc[(Data.Month == '01'),]
Data_02 = Data.loc[(Data.Month == '02'),]
Data_03 = Data.loc[(Data.Month == '03'),]
Data_04 = Data.loc[(Data.Month == '04'),]
Data_05 = Data.loc[(Data.Month == '05'),]
Data_06 = Data.loc[(Data.Month == '06'),]
Data_07 = Data.loc[(Data.Month == '07'),]
Data_08 = Data.loc[(Data.Month == '08'),]
Data_09 = Data.loc[(Data.Month == '09'),]
Data_10 = Data.loc[(Data.Month == '10'),]
Data_11 = Data.loc[(Data.Month == '11'),]
Data_12 = Data.loc[(Data.Month == '12'),]

#Subsetting each season
Spring = Data.loc[(Data.Season == 'Spring'),]
Summer = Data.loc[(Data.Season == 'Summer'),]
Fall = Data.loc[(Data.Season == 'Fall'),]
Winter = Data.loc[(Data.Season == 'Winter'),]

#Total Data
Data_grouped1 = Data.groupby(['Code'])
Data_grouped1_sum = Data_grouped1.size()
Data_grouped1_sum = Data_grouped1_sum.to_frame()
Data_grouped1_sum.columns = ['TOTAL']

#Each Season Data
Data_grouped2 = Spring.groupby(['Code'])
Data_grouped2_sum = Data_grouped2.size()
Data_grouped2_sum = Data_grouped2_sum.to_frame()
Data_grouped2_sum.columns = ['SPRING']

Data_grouped3 = Summer.groupby(['Code'])
Data_grouped3_sum = Data_grouped3.size()
Data_grouped3_sum = Data_grouped3_sum.to_frame()
Data_grouped3_sum.columns = ['SUMMER']

Data_grouped3 = Fall.groupby(['Code'])
Data_grouped3_sum = Data_grouped3.size()
Data_grouped3_sum = Data_grouped3_sum.to_frame()
Data_grouped3_sum.columns = ['FALL']

Data_grouped4 = Winter.groupby(['Code'])
Data_grouped4_sum = Data_grouped4.size()
Data_grouped4_sum = Data_grouped4_sum.to_frame()
Data_grouped4_sum.columns = ['WINTER']

Data_grouped_sum1 = pd.concat([Data_grouped2_sum,Data_grouped3_sum,Data_grouped4_sum], axis=1,sort=False)

#Each Year Data
Data_grouped5 = Data_2016.groupby(['Code'])
Data_grouped5_sum = Data_grouped5.size()
Data_grouped5_sum = Data_grouped5_sum.to_frame()
Data_grouped5_sum.columns = ['2016']

Data_grouped6 = Data_2017.groupby(['Code'])
Data_grouped6_sum = Data_grouped6.size()
Data_grouped6_sum = Data_grouped6_sum.to_frame()
Data_grouped6_sum.columns = ['2017']

Data_grouped7 = Data_2018.groupby(['Code'])
Data_grouped7_sum = Data_grouped7.size()
Data_grouped7_sum = Data_grouped7_sum.to_frame()
Data_grouped7_sum.columns = ['2018']

Data_grouped8 = Data_2019.groupby(['Code'])
Data_grouped8_sum = Data_grouped8.size()
Data_grouped8_sum = Data_grouped8_sum.to_frame()
Data_grouped8_sum.columns = ['2019']

Data_grouped_sum2 = pd.concat([Data_grouped5_sum,Data_grouped6_sum,Data_grouped7_sum,Data_grouped8_sum], axis=1,sort=False)

#Each Year Month
Data_grouped9 = Data_01.groupby(['Code'])
Data_grouped9_sum = Data_grouped9.size()
Data_grouped9_sum = Data_grouped9_sum.to_frame()
Data_grouped9_sum.columns = ['JAN']

Data_grouped10 = Data_02.groupby(['Code'])
Data_grouped10_sum = Data_grouped10.size()
Data_grouped10_sum = Data_grouped10_sum.to_frame()
Data_grouped10_sum.columns = ['FEB']

Data_grouped11 = Data_03.groupby(['Code'])
Data_grouped11_sum = Data_grouped11.size()
Data_grouped11_sum = Data_grouped11_sum.to_frame()
Data_grouped11_sum.columns = ['MAR']

Data_grouped12 = Data_04.groupby(['Code'])
Data_grouped12_sum = Data_grouped12.size()
Data_grouped12_sum = Data_grouped12_sum.to_frame()
Data_grouped12_sum.columns = ['APR']

Data_grouped13 = Data_05.groupby(['Code'])
Data_grouped13_sum = Data_grouped13.size()
Data_grouped13_sum = Data_grouped13_sum.to_frame()
Data_grouped13_sum.columns = ['MAY']

Data_grouped14 = Data_06.groupby(['Code'])
Data_grouped14_sum = Data_grouped14.size()
Data_grouped14_sum = Data_grouped14_sum.to_frame()
Data_grouped14_sum.columns = ['JUN']

Data_grouped15 = Data_07.groupby(['Code'])
Data_grouped15_sum = Data_grouped15.size()
Data_grouped15_sum = Data_grouped15_sum.to_frame()
Data_grouped15_sum.columns = ['JUL']

Data_grouped16 = Data_08.groupby(['Code'])
Data_grouped16_sum = Data_grouped16.size()
Data_grouped16_sum = Data_grouped16_sum.to_frame()
Data_grouped16_sum.columns = ['AUG']

Data_grouped17 = Data_09.groupby(['Code'])
Data_grouped17_sum = Data_grouped17.size()
Data_grouped17_sum = Data_grouped17_sum.to_frame()
Data_grouped17_sum.columns = ['SEP']

Data_grouped18 = Data_10.groupby(['Code'])
Data_grouped18_sum = Data_grouped18.size()
Data_grouped18_sum = Data_grouped18_sum.to_frame()
Data_grouped18_sum.columns = ['OCT']

Data_grouped19 = Data_11.groupby(['Code'])
Data_grouped19_sum = Data_grouped19.size()
Data_grouped19_sum = Data_grouped19_sum.to_frame()
Data_grouped19_sum.columns = ['NOV']

Data_grouped20 = Data_12.groupby(['Code'])
Data_grouped20_sum = Data_grouped20.size()
Data_grouped20_sum = Data_grouped20_sum.to_frame()
Data_grouped20_sum.columns = ['DEC']


Data_grouped_sum3 = pd.concat([Data_grouped9_sum,Data_grouped10_sum,Data_grouped11_sum,Data_grouped12_sum,Data_grouped13_sum,Data_grouped14_sum,Data_grouped15_sum,Data_grouped16_sum,Data_grouped17_sum,Data_grouped18_sum,Data_grouped19_sum,Data_grouped20_sum], axis=1,sort=False)

Data_grouped_sum = pd.concat([Data_grouped1_sum,Data_grouped_sum1,Data_grouped_sum2,Data_grouped_sum3], axis=1,sort=False).fillna(0)
Data_grouped_sum = Data_grouped_sum.reset_index()
Data_grouped_sum = Data_grouped_sum.rename(columns={'index':'Code'})
```

###  4.2 Preparing Data for Popular Place Check-ins 

```python
Latitude = Data['Latitude'].tolist()
Longitude = Data['Longitude'].tolist()
Address = Data.index.tolist()
```

#### Preparing Data for Popular Place for All check-ins from 2016 to 2019 

```python
#Sum total visit the same place
Data2 = Data.reset_index()
PopPlace = Data2.groupby(['Address'])
PopPlace_sum = PopPlace.size()
PopPlace_sum = PopPlace_sum.to_frame()
PopPlace_sum.columns = ['Total']

#Find the latitude and longitude for the popular visit place
for place in PopPlace_sum.index :
    for i in range (len(Address)) :
        if place == Address[i] :
            PopPlace_sum.loc[place,'Latitude'] = Latitude[i]
            PopPlace_sum.loc[place,'Longitude'] = Longitude[i]
```
```python
#Add State
for address in PopPlace_sum.index:
    for i in range (len(statelist)) :
        if statelist[i] in address :
            PopPlace_sum.loc[address,'State'] = statelist[i]
#Sort by State and total number of check-ins respectively
PopPlace_sum = PopPlace_sum.sort_values(by=['State','Total'], ascending=[True,False])
```

#### Preparing Data for Popular Place for All check-ins of each season

```python
#Sum total visit the same place
PopPlace2 = Data2.groupby(['Address','Season'])
PopPlace_sum2 = PopPlace2.size()
PopPlace_sum2 = PopPlace_sum2.to_frame()
#Add State
for address in PopPlace_sum2.index:
        for i in range (len(statelist)) :
            if statelist[i] in address[0] :
                PopPlace_sum2.loc[address,'State'] = statelist[i]
```
```python
#Add State
for address in PopPlace_sum2.index:
    for i in range (len(Address)) :
        if address[0] == Address[i] :
            PopPlace_sum2.loc[address,'Latitude'] = Latitude[i]
            PopPlace_sum2.loc[address,'Longitude'] = Longitude[i]
```

```python
PopSpring = pd.DataFrame()
PopSummer = pd.DataFrame()
PopFall = pd.DataFrame()
PopWinter = pd.DataFrame()
for i in range (len(PopPlace_sum2.index)):
        temp = {'Address': PopPlace_sum2.index[i][0],'Season':PopPlace_sum2.index[i][1],'Total': PopPlace_sum2.iloc[i,0],'State': PopPlace_sum2.iloc[i,1],'Latitude':PopPlace_sum2.iloc[i,2] ,'Longitude':PopPlace_sum2.iloc[i,3]} 
        if PopPlace_sum2.index[i][1] == 'Spring' :
            PopSpring = PopSpring.append(temp, ignore_index=True)
        elif PopPlace_sum2.index[i][1] == 'Summer' :
            PopSummer = PopSummer.append(temp, ignore_index=True)
        elif PopPlace_sum2.index[i][1] == 'Fall' :
            PopFall = PopFall.append(temp, ignore_index=True)
        elif PopPlace_sum2.index[i][1] == 'Winter' :
            PopWinter = PopWinter.append(temp, ignore_index=True)           
```

```python
#Sort by State and total number of check-ins respectively
PopSpring = PopSpring.sort_values(by=['State','Total'], ascending=[True,False])
PopSummer = PopSummer.sort_values(by=['State','Total'], ascending=[True,False])
PopFall = PopFall.sort_values(by=['State','Total'], ascending=[True,False])
PopWinter = PopWinter.sort_values(by=['State','Total'], ascending=[True,False])
```

###  4.3 Preparing Crime, US Index and Temperature Data 

```python
USCrime = pd.read_csv('USCrime.csv')
USIndex = pd.read_csv('USIndex.csv')
Temperature = pd.read_csv('Temperature.csv')
```

```python
Data3 = pd.concat([Data_2016,Data_2017,Data_2018], ignore_index=True)
```
```python
Datax = Data3.groupby(['State','Date'])
Datax_sum = Datax.size()
Datax_sum = Datax_sum.to_frame()
Datax_sum.columns = ['Total']
Datax_sum = Datax_sum.unstack().fillna(0)
add = Datax_sum.sum(axis = 0, skipna = True) 
Datax_sum = Datax_sum.transpose()
Datax_sum['All'] = 0 
Datax_sum = Datax_sum.transpose()
Datax_sum.loc['All']['Total'] = [add[0],add[1],add[2],add[3],add[4],add[5],add[6],add[7],add[8],add[9],add[10],add[11],add[12],add[13],add[14],add[15],add[16],add[17],add[18],add[19],add[20],add[21],add[22],add[23],add[24],add[25],add[26],add[27],add[28],add[29],add[30],add[31],add[32],add[33],add[34],add[35]]
```
####  Preparing Data for Temperature and Check-ins Plot 

```python
Temperature = Temperature.drop(['Anomaly','Season'],axis=1)
Temperature = Temperature.set_index('State')
Temperature = Temperature[Temperature.Year != 2019]
Temperature['Date']  = Temperature.Year.astype(str).str.cat('0'+Temperature.Month.astype(str), sep='/')
Temperature = Temperature.drop(['Month','Year'],axis=1)
Temperature.rename(columns={'Value':'Temp'},inplace=True)
AvgTemp = Temperature.copy()
AvgTemp = AvgTemp.groupby('Date')['Temp'].sum().reset_index()
AvgTemp['Temp'] = AvgTemp['Temp']/49
AvgTemp['State'] = 'All'
Temperature = Temperature.reset_index()
Temperature = pd.concat([Temperature,AvgTemp], axis=0,sort = True)
Temperature = Temperature.set_index('State')
```
```python
#See how many posts for each year
Data_dategroup = Data3.groupby(['Date'])
Data_datesize = Data_dategroup.size()
Data_datesize = Data_datesize.to_frame()
Data_datesize.columns = ['Total']
Data_datesize = Data_datesize.transpose()
```

#### Preparing Data for US Index and Check-ins Plot 

```python
USIndex.rename(columns={'date':'Date'},inplace=True)
USIndex = USIndex[USIndex.Year != 2019]
for i in range (len(USIndex.index)) :
    if USIndex.loc[i,'Month'] in [1,2,3,4,5,6,7,8,9] :
        USIndex.loc[i,'Month'] =  '0'+str(USIndex.loc[i,'Month'])
        
USIndex['Date']  = USIndex.Year.astype(str).str.cat(USIndex.Month.astype(str), sep='/')
```
```python
USIndex = USIndex.drop(['Month','Year','Season'],axis=1)
USIndex.rename(columns={' value':'USIndex'},inplace=True)
USIndex = USIndex.set_index('Date')
USIndex = USIndex.transpose()
```

#### Preparing Data for Crime Per Population and Check-ins Plot 

```python
USCrime['Crime/Population'] = USCrime['Total']/USCrime['population']
USCrime = USCrime.set_index('State')
for state in USCrime.index :
    for i in range (len(statecode)):
        if statecode[i] == state:
            USCrime.loc[state,'State'] = statelist[i]

USCrime = USCrime.fillna('All')
USCrime = USCrime.set_index('State')
```
```python
USCrime2 = USCrime.loc[:,['year','Crime/Population']]
USCrime2 = USCrime2.sort_index()
```

```python
Data_grouped_sum3 = Data_grouped_sum2.copy()
for state in Data_grouped_sum3.index :
    for i in range (len(statecode)):
        if statecode[i] == state:
            Data_grouped_sum3 = Data_grouped_sum3.rename(index={state:statelist[i]})   
Data_grouped_sum3 = Data_grouped_sum3.loc[:,['2016','2017','2018']]
Data_grouped_sum3 = Data_grouped_sum3.fillna(0)
add2 = Data_grouped_sum3.sum(axis = 0) 
Data_grouped_sum3.loc['All'] = [add2[0], add2[1], add2[2]]
```

## Instagram Check-ins Database

```python
def menu():
    print()
    print('1. Geomap of Total Check-ins by each state')
    print('2. Total Check-ins by each state from JAN2016 to OCT2019')
    print('3. Top Popular Check-ins')
    print('4. Geomap of Top Popular Check-ins')
    print('5. Check-ins vs US Index')
    print('6. Check-ins vs Temperature')
    print('7. Check-ins vs Crime Rate')
    print('8. State name sheet')
    print('q. Quit the program')
    print()
    print('Please type "8" for each state name for input')
    print()

def processRequest(selection):
    if selection == '1':
        Geomapplot(selection)
    elif selection == '2':
        NumberCheckins(selection)    
    elif selection == '3':
        PopCheckins(selection)
    elif selection == '4':
        GeoPopCheckins(selection)
    elif selection == '5':
        CheckinsVSUSIndex(selection)
    elif selection == '6':
        CheckinsVSTemp(selection)
    elif selection == '7':
        CheckinsVSCrime(selection)  
    elif selection == '8':
        StateNameSheet(selection) 
    else:
        return 'q' 

#using plotly to draw a choropleth map of a US
def Geomapplot(selection):
    print()
    print('Input can be year(2016-2019), season(Spring,Summer,Fall,Winter) and month(JAN-DEC)')
    print('Type "Total" for all data from 2016-2019')
    print()
    search = input('Enter input : ').upper().replace(" ","")
    
    try :
        fig = go.Figure(data=go.Choropleth(
            locations=Data_grouped_sum['Code'], 
            z = Data_grouped_sum[search].astype(float),
            locationmode = 'USA-states', 
            colorscale = 'Greens',
            colorbar_title ='Number of Check-ins',
        ))

        fig.update_layout(
            title_text = search + ' Number of Instagram Check-ins',
            geo_scope='usa',
        )
        print('Data from 01/01/2016 to 10/29/2019')
        fig.show()
                       
    except :
        print()
        print('Error : Typing Wrong Input')
                       

def NumberCheckins(selection):
    print()
    print("Input State name starting with Capital Letter (input 'All' to see all state)")
    print('For State name guideline, please selection "8" on the main menu')
    print()
    state = input('Enter State : ')
    
    try :
        Plot = Datax_sum.loc[state].plot(legend=True, figsize=(16,10),title = state + ' Number of Check-ins from JAN2016 to OCT2019')
        Plot.set_ylabel('Number of Check-ins')
        Plot.set_xlabel('Year (JAN2016 to OCT2019)')
        plt.show()
    
    except :
        print()
        print('Error : Typing Wrong State Name or no Data for this state')
        print('Please selection "8" on the main menu to see all state name')
        
def PopCheckins(selection):
    print()
    print("Input Season (input 'All' to see all season)")
    print()
    season = input('Enter Season (Spring,Summer,Fall,Winter) : ').upper().replace(" ","")
    print()
    print("Input State name starting with Capital Letter (input 'All' to see all state)")
    print('For State name guideline, please selection "8" on the main menu')
    print()
    state = input('Enter State : ')
    print()
    print('Input Number of Place to show')
    print()
    number = input('Enter number : ')
    seasonlist = ['SUMMER','WINTER','FALL','SPRING']
    
    if season == 'ALL':
        PopPlaceState = PopPlace_sum.sort_values(by=['Total'], ascending=False)
        
        try :
            if state != 'All':
                PopPlaceState = PopPlace_sum.loc[(PopPlace_sum.State == state),]
            PopPlaceState = PopPlaceState.loc[:,['State','Total']]
            print('Top ' + number + ' Poplular Place Check-ins from 01/01/2016 to 10/29/2019 of ' + state)
            print(PopPlaceState.head(int(number)))

        except :
            print()
            print('Error : Typing Wrong State Name or no Data for this state')
            print('Please selection "8" on the main menu to see all state name')
    
    elif season in seasonlist:
        if season == 'SPRING':
            dataframe = PopSpring 
        elif season == 'SUMMER':
            dataframe = PopSummer 
        elif season == 'FALL':
            dataframe = PopFall 
        elif season == 'WINTER':
            dataframe = PopWinter    
    
        try :
            if state != 'All':
                dataframe = dataframe.loc[(dataframe.State == state),]   
                
            PopPlaceState = dataframe.sort_values(by=['Total'], ascending=False)
            PopPlaceState = PopPlaceState.loc[:,['Address','State','Total']]
            PopPlaceState = PopPlaceState.set_index('Address')
            print('Top '+ number + ' Poplular Place Check-ins on '+ season + ' of ' + state)
            print(PopPlaceState.head(int(number)))

        except :
            print()
            print('Error : Typing Wrong State Name or no Data for this state in this season')
            print('Please selection "8" on the main menu to see all state name')
          
    else :
        print()
        print('Error : Typing Wrong Season')

def GeoPopCheckins(selection):
    print()
    print("Input Season (input 'All' to see all season)")
    print()
    season = input('Enter Season (Spring,Summer,Fall,Winter) : ').upper().replace(" ","")
    print()
    print("Input State name starting with Capital Letter (input 'All' to see all state)")
    print('For State name guideline, please selection "8" on the main menu')
    print()
    state = input('Enter State : ')
    print()
    print('Input Number of Place to show')
    print()
    number = input('Enter number : ')
    seasonlist = ['SUMMER','WINTER','FALL','SPRING']
    
    if season == 'ALL':
        Pop = PopPlace_sum.sort_values(by=['Total'], ascending=False)
        if state != 'All':
               Pop =  Pop.loc[(Pop.State == state),]
        Pop = Pop.head(int(number))
        
        try :
            scl = [0,"rgb(150,0,90)"],[0.125,"rgb(0, 0, 200)"],[0.25,"rgb(0, 25, 255)"],\
            [0.375,"rgb(0, 152, 255)"],[0.5,"rgb(44, 255, 150)"],[0.625,"rgb(151, 255, 0)"],\
            [0.75,"rgb(255, 234, 0)"],[0.875,"rgb(255, 111, 0)"],[1,"rgb(255, 0, 0)"]

            fig = go.Figure(data=go.Scattergeo(
                    locationmode = 'USA-states',
                    lat = Pop['Latitude'],
                    lon = Pop['Longitude'],
                    text = Pop.index + ', ' + Pop['State'] + ', ' + 'Check-ins: ' +Pop['Total'].astype(str),
                    mode = 'markers',
                    marker = dict(
                        size = 5,
                        opacity = 0.8,
                        reversescale = True,
                        autocolorscale = False,
                        line = dict(
                            width=1,
                            color='rgba(102, 102, 102)'
                        ),
                        colorscale = scl,
                        color = Pop['Total'],
                        cmax = Pop['Total'].max(),
                        colorbar_title="Number of Check-ins"
                    )))

            fig.update_layout(
                    title = 'Top ' + number + ' of '+ state + ' Most Check-ins Place',
                    geo = dict(
                        scope='usa',
                        projection_type='albers usa',
                        showland = True,
                        landcolor = "rgb(250, 250, 250)",
                        subunitcolor = "rgb(217, 217, 217)",
                        countrycolor = "rgb(217, 217, 217)",
                        countrywidth = 0.5,
                        subunitwidth = 0.5
                    ),
                )
            fig.show()
        except :
            print()
            print('Error : Typing Wrong State Name or no Data for this state')
            print('Please selection "8" on the main menu to see all state name')
       
    elif season in seasonlist:
        if season == 'SPRING':
            dataframe = PopSpring 
        elif season == 'SUMMER':
            dataframe = PopSummer 
        elif season == 'FALL':
            dataframe = PopFall 
        elif season == 'WINTER':
            dataframe = PopWinter 
        
        if state != 'All':
            dataframe =  dataframe.loc[(dataframe.State == state),]
        
        try :
            Pop = dataframe
            Pop = Pop.set_index('Address')
            Pop = Pop.sort_values(by=['Total'], ascending=False)
            Pop = Pop.head(int(number))
            
            scl = [0,"rgb(150,0,90)"],[0.125,"rgb(0, 0, 200)"],[0.25,"rgb(0, 25, 255)"],\
            [0.375,"rgb(0, 152, 255)"],[0.5,"rgb(44, 255, 150)"],[0.625,"rgb(151, 255, 0)"],\
            [0.75,"rgb(255, 234, 0)"],[0.875,"rgb(255, 111, 0)"],[1,"rgb(255, 0, 0)"]

            fig = go.Figure(data=go.Scattergeo(
                    locationmode = 'USA-states',
                    lat = Pop['Latitude'],
                    lon = Pop['Longitude'],
                    text = Pop.index + ', ' + Pop['State'] + ', ' + 'Check-ins: ' +Pop['Total'].astype(str),
                    mode = 'markers',
                    marker = dict(
                        size = 5,
                        opacity = 0.8,
                        reversescale = True,
                        autocolorscale = False,
                        line = dict(
                            width=1,
                            color='rgba(102, 102, 102)'
                        ),
                        colorscale = scl,
                        color = Pop['Total'],
                        cmax = Pop['Total'].max(),
                        colorbar_title="Number of Check-ins"
                    )))

            fig.update_layout(
                    title = 'Top ' + number + ' of '+state + ' Most Check-ins Place in '+ season,
                    geo = dict(
                        scope='usa',
                        projection_type='albers usa',
                        showland = True,
                        landcolor = "rgb(250, 250, 250)",
                        subunitcolor = "rgb(217, 217, 217)",
                        countrycolor = "rgb(217, 217, 217)",
                        countrywidth = 0.5,
                        subunitwidth = 0.5
                    ),
                )
            fig.show()
       
            
        except :
            print()
            print('Error : Typing Wrong State Name or no Data for this state in this season')
            print('Please selection "8" on the main menu to see all state name')
            
    else :
        print()
        print('Error : Typing Wrong Season')
        
def CheckinsVSUSIndex(selection):
    #combine two graph with different y-axis
    fig, ax1 = plt.subplots()
    ax2 = fig.add_axes()
    ax2 = ax1.twinx()

    #First graph plot
    ax1.set_ylabel('Number of US Trip Check-ins)', color='red')
    ax1.tick_params(axis='y', labelcolor='red')
    lns1 = ax1.plot(Data_datesize.loc['Total'], color='red', lw=3, alpha= 0.6, label='Number of US Trip Check-ins', marker='o')

    #Second graph plot
    ax2.set_ylabel('US Index', color='blue')
    ax2.tick_params(axis='y', labelcolor='blue')
    lns2 = ax2.plot(USIndex.loc['USIndex'], color='blue', lw=3, alpha= 0.6, label='US Index', marker='o')

    plt.title('Number of US Trip Check-ins vs US Index JAN2016 to DEC2018') 
    # Label 2 line in the graph
    leg = lns1 + lns2
    labs = [l.get_label() for l in leg]
    ax1.legend(leg, labs, loc=1)
    plt.grid()
    plt.xticks([])
    plt.show()

def CheckinsVSTemp(selection):
    print()
    print("Input State name starting with Capital Letter (input 'All' to see for all state)")
    print('For State name guideline, please selection "8" on the main menu')
    print()
    state = input('Enter input : ')
    try :
        Checkins = Datax_sum.loc[state]
        Checkins = Checkins.loc['Total']
        Checkins = Checkins.to_frame()
        
        Temp = Temperature[Temperature.index == state]
        Temp = Temp.loc[:,('Date','Temp')]
        Temp = Temp.set_index('Date')

        #combine two graph with different y-axis
        fig, ax1 = plt.subplots()
        ax2 = fig.add_axes()
        ax2 = ax1.twinx()

        #First graph plot
        ax1.set_ylabel('Number of US Trip Check-ins)', color='red')
        ax1.tick_params(axis='y', labelcolor='red')
        lns1 = ax1.plot(Checkins, color='red', lw=3, alpha= 0.6, label='Number of US Trip Check-ins', marker='o')

        #Second graph plot
        ax2.set_ylabel('Temperature (℉)', color='blue')
        ax2.tick_params(axis='y', labelcolor='blue')
        lns2 = ax2.plot(Temp, color='blue', lw=3, alpha= 0.6, label='Temperature', marker='o')

        plt.title(state + ' :Number of US Trip Check-ins vs Temperature (℉) JAN2016 to DEC2018') 
        # Label 2 line in the graph
        leg = lns1 + lns2
        labs = [l.get_label() for l in leg]
        ax1.legend(leg, labs, loc=1)
        plt.grid()
        plt.xticks([])
        plt.show() 
    except :
        print()
        print('Error : Typing Wrong State Name or no Data for this state')
        print('Please selection "8" on the main menu to see all state name') 

def CheckinsVSCrime(selection):
    print()
    print("Input State name starting with Capital Letter (input 'All' to see for all state)")
    print('For State name guideline, please selection "8" on the main menu')
    print()
    state = input('Enter input : ')
    try :
        Crime = USCrime2[USCrime2.index == state]
        Crime = Crime.loc[:,('year','Crime/Population')]
        Crime = Crime.set_index('year')
        Crime = Crime.sort_index()

        Checkins = Data_grouped_sum3.loc[state].to_frame()
        Checkins.index.name = 'year'
        Checkins.index = Checkins.index.astype(str).astype(int)

        #combine two graph with different y-axis
        fig, ax1 = plt.subplots()
        ax2 = fig.add_axes()
        ax2 = ax1.twinx()

        #First graph plot
        ax1.set_ylabel('Number of US Trip Check-ins)', color='red')
        ax1.tick_params(axis='y', labelcolor='red')
        lns1 = ax1.plot(Checkins, color='red', lw=3, alpha= 0.6, label='Number of US Trip Check-ins', marker='o')

        #Second graph plot
        ax2.set_ylabel('CrimePerPopulation', color='blue')
        ax2.tick_params(axis='y', labelcolor='blue')
        lns2 = ax2.plot(Crime, color='blue', lw=3, alpha= 0.6, label='CrimePerPopulation', marker='o')

        plt.title(state + ' :Number of US Trip Check-ins vs CrimePerPopulation 2016 to 2018') 
        # Label 2 line in the graph
        leg = lns1 + lns2
        labs = [l.get_label() for l in leg]
        ax1.legend(leg, labs, loc=1)
        plt.grid()
        plt.xticks([])
        plt.show()
     
    except :
        print()
        print('Error : Typing Wrong State Name or no Data for this state')
        print('Please selection "8" on the main menu to see all state name')    

def StateNameSheet(selection) :
    statenamesheet = statesheet.copy()
    statenamesheet = statenamesheet.set_index('state')
    print(statenamesheet)

def main():
    print('Welcome to US Popular Place to Check-ins Database')     
    selection = ''
    while selection != 'q':
        menu()
        selection = input('Make a selection: ')
        response = processRequest(selection)
        if response == 'q':
            break
                                 
main()
```

This are the function in database :

1. Geomap plot for total check-ins of each state in each season, month or year. 
2. The plot of total number of check-ins of each state from JAN 2016 to OCT 2019.
3. Top check-ins place for each state and number of check-ins. 
4. Map plot top Top check-ins place for each state and number of check-ins.
5. The number of checkins with US Index. 
6. The number of checkins with temperature of each state.
7. The number of checkins with crime rate 

## Conclusion :

![output_45_3](https://user-images.githubusercontent.com/53027899/68407831-3d8fc600-0152-11ea-9c64-62512b619eeb.png)

![output_45_7](https://user-images.githubusercontent.com/53027899/68407832-3d8fc600-0152-11ea-93c8-8f3d9e4e8bdd.png)

![output_45_9](https://user-images.githubusercontent.com/53027899/68407833-3e285c80-0152-11ea-96ce-9c0f026871e2.png)

![output_45_11](https://user-images.githubusercontent.com/53027899/68407834-3e285c80-0152-11ea-8ec7-d21b3f641ff9.png)

From the database, you can find the popular place or state to visit in each season. Moreover, it also show that there are mostly likely to have 2 peak check-ins periods for each year, which are March to April and August to September. From the US Index graph, it suggest that US Index are not seem to affect to number of check-ins. I think this is because even the US Index is high, but if it is in the high season, they would still get out to travel in US anyway. From the temperature graph, it number of check-ins is in the same trend as temperature. This must because international tourist would likely to have some cold experience or see some snow as their country  didn’t have such a thing, so new experince is what people want.Finally, the crime graph show that the crime rate is continuously drop since 2016 and the number of chech-ins also rise up too. Thererfore, the safety of place would also affect number of tourist too.
