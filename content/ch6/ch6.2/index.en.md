---
title : "6.2. Anomaly Detection"
weight : 20
socialshare : true
description : "We use air quality data to demonstrate the anomaly detection framework commonly used in Taiwan's micro air quality sensing data. We learn by doing, step by step, from data preparation and feature extraction to data analysis, statistics, and induction. The readers will experience how to gradually achieve advanced and practical data application services by superimposing basic data analysis methods."
tags: ["Python", "Air" ]
levels: ["advanced" ]
authors: ["Quen Luo"]
---

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/drive/1cmvIQZKf70_qtejzE0h4SBTv-o7-uMjg?usp=sharing)

{{< toc >}}

## Anomaly Detection Framework

In various countries and cities, we've seen the successful implementation of expansive networks for monitoring air quality at a micro level. One key issue with these tiny sensors is making sure they provide accurate data and can spot any unusual readings as they happen. Back in 2018, a team from the Network Research Laboratory at Academia Sinica's Institute of Information Science in Taiwan developed a special system for this purpose, known as the [Anomaly Detection Framework](https://ieeexplore.ieee.org/document/8081731) (ADF).

The ADF is composed of four main parts:

1. **Time-Sliced Anomaly Detection (TSAD):** This part is all about catching odd data from the sensors, either related to time or location, right when it occurs. It then sends this information to other parts of the system for deeper analysis.
2. **Real-time Emission Detection (RED):** Using the findings from TSAD, RED focuses on identifying possible pollution events happening in a specific area, doing this in real-time.
3. **Sensor Reliability Evaluation (Device Ranking, DR):** DR takes the data flagged by TSAD to assess how trustworthy each small sensor device is.
4. **Abnormal Use of Machine Detection (Malfunction Detection, MD):** Similar to DR, MD also uses TSAD's data but with a different goal. It looks for sensors that might not be used properly. For example, it can find sensors placed indoors or near constant sources of pollution, which could affect their readings.

Each of these modules plays a crucial role in ensuring the micro sensors used in air quality monitoring systems work effectively and reliably.

![ADF Framework](figures/6-2-1-1.png)

### Types of abnormal events

In the ADF framework, the TSAD module assesses unusual occurrences in time or space whenever the micro air quality sensor gets new data. Let's use the micro air quality sensor as an example to explain this:

- Time-related abnormal events: We start with the assumption that air spreads out evenly and slowly. Therefore, the readings from the same micro air quality sensor should change only slightly over a short period. If there's a rapid and significant change in these readings in a short time, it could indicate something abnormal happening related to time.

- Space-related abnormal events: We generally expect that outdoor air disperses uniformly over a geographical area. This means that the readings from a micro air quality sensor should be roughly similar to those from sensors nearby. If the readings from one specific sensor are drastically different from those of its neighbors at the same time, it might suggest an unusual event happening in the location of that sensor.

### Possible causes of abnormal events

There are several reasons why the abnormal events we've talked about might happen. Some of the most common ones include:

- Putting the sensor in the wrong place: Sometimes, the sensor is set up in a specific area, which means it can't accurately measure the overall environment. This could be because it's placed next to a temple, inside a barbecue shop, or in any indoor area that doesn't have good air flow.
- Issues with the sensor itself or how it's set up: For instance, the sensor might be installed facing the wrong way, which affects how it takes in air. Or, the fan in the sensor could be dirty, which would make it work poorly.
- Sudden nearby pollution: This could be something like someone smoking close to the sensor, a nearby fire, or other pollutants being released right next to it.

## Case Study

In this article, we're going to explore air quality data from the Civil IoT Taiwan project. Specifically, we'll look at data from small air quality sensors placed around campuses in Kaohsiung City. Our focus will be on how to use the ADF detection framework to identify sensors that might be inside buildings or near sources of pollution. Additionally, we'll discuss how to assess and rank these sensors based on how reliable their data is.

### Package Installation and Importing

In this article, we're going to work with several tools that are already set up for us on Google Colab, our development platform. These include pandas, numpy, plotly, and geopy packages. Since they are pre-installed, we don't need to worry about installing them ourselves. To get started, we'll simply import these packages using the syntax provided below. This will set everything up for the tasks we'll be tackling in this article.

```python
import pandas as pd
import numpy as np

import plotly.express as px
from geopy.distance import geodesic
```

### Initialization and Data Access

For our analysis, we're going to use small air quality sensors located in Kaohsiung, which are part of the Civil IoT Taiwan Project. We'll focus on specific areas and timeframes:

- For the area, we'll look at places between latitudes 22.631231 and 22.584989, and longitudes 120.263422 and 120.346764.
- The time period we're interested in is from October 15, 2022, to October 28, 2022.

You can find the raw data from these air quality sensors on the [Civil IoT Taiwan Data Service Platform](https://ci.taiwan.gov.tw/dsp/) at this link. This data is typically used for academic purposes, but to make things easier, we've already downloaded it and put it into a file named [allLoc.csv](https://LearnCIOT.github.io/data/allLoc.csv), which you can use for the examples in this article.

Here's what we do with this data:

1. We start by opening the data file to see what's inside.
```python
DF = pd.read_csv("https://LearnCIOT.github.io/data/allLoc.csv")
DF.head()
```

![Python output](figures/6-2-2-1.png)

2. Next, we look for the GPS coordinates of each sensor in the file. Since these sensors don't move, we calculate the average longitude and latitude for each one. This gives us a fixed geographical location for each sensor.
```python
dfId = DF[["device_id","lon","lat"]].groupby("device_id").mean().reset_index()
print(dfId.head())
```

```
      device_id      lon     lat
0  74DA38F207DE  120.340  22.603
1  74DA38F20A10  120.311  22.631
2  74DA38F20B20  120.304  22.626
3  74DA38F20B80  120.350  22.599
4  74DA38F20BB6  120.324  22.600
```

3. Finally, we create a map showing where each sensor is located. This helps us see how the sensors are spread out geographically in the dataset.
```python
fig_map = px.scatter_mapbox(dfId, lat="lat", lon="lon",
                  color_continuous_scale=px.colors.cyclical.IceFire, zoom=9,
                  mapbox_style="carto-positron")
fig_map.show()
```


![Python output](figures/6-2-2-2.png)

### Find nearby sensors

The micro air quality sensors on the campus are permanently installed, so their GPS locations remain the same. To make data analysis faster later on, we first create a list of "neighbors" for each sensor. In our approach, two small sensors are considered neighbors if they are 3 kilometers or less apart from each other.

We begin by creating a function called `countDis` that measures the actual distance in kilometers between any two given GPS coordinates.

```python
def countDis(deviceA, deviceB):
    return geodesic((deviceA["lat"], deviceB['lon']), (deviceB["lat"], deviceB['lon'])).km
```

Next, we change the format of the sensor data from a DataFrame, which is a specific type of data structure, to a Dictionary, another type of data structure. This allows us to calculate the distance between each pair of sensors. If the distance between any two sensors is less than 3 kilometers, we add them to each other’s list of neighboring sensors, which we call `dicNeighbor`.

```python
# set the maximum distance of two neighbors
DISTENCE = 3

## convert Dataframe -> Dictionary
# {iD: {'lon': 0.0, 'lat': 0.0}, ...}
dictId = dfId.set_index("device_id").to_dict("index")

## obtain the list of sensor device_id
listId = dfId["device_id"].to_list()

## initialize dicNeighbor
# {iD: []}
dicNeighbor = {iD: [] for iD in listId}

## use countDis to calculate distance of every two sensors
# The two sensors are deem to be neighbors of each other if their distance is less than 3km
# Time complexity: N!
for x in range(len(listId)):
    for y in range(x+1, len(listId)):
        if ( countDis( dictId[listId[x]], dictId[listId[y]]) < DISTENCE ):
            dicNeighbor[listId[x]].append( listId[y] )
            dicNeighbor[listId[y]].append( listId[x] )
```

### Time slicing every five minutes

In the original data, each sensor's readings aren't aligned in time. To tackle this in the ADF framework, we break down the sensor data into segments based on regular time intervals. This process creates a 'time slice' that gives us a comprehensive view of the data collected over each time period. We start by combining the `date` and `time` fields from the original data into a single field, using Python's `datetime` data type, and call this new field datetime. After creating `datetime`, we remove fields that are no longer needed, such as `date`, `time`, `RH`, `temperature`, `lat`, and `lon`.

```python
# combine the 'date' and 'time' columns to a new column 'datetime'
DF["datetime"] = DF["date"] + " " + DF["time"]

# remove some non-necessary columns
DF.drop(columns=["date","time", "RH","temperature","lat","lon"], inplace=True)

# convert the new 'datetime' column to datetime data type
DF['datetime'] = pd.to_datetime(DF.datetime)
```

Considering the data from campus micro air quality sensors is updated roughly every 5 minutes, we set our time slice interval, named `FREQ`, to 5 minutes. This means we calculate the average of the sensor readings every 5 minutes. To ensure the data's accuracy, we've also conducted extra checks and eliminated any readings where the PM2.5 levels were incorrectly showing as negative values.

```python
FREQ = '5min'
dfMean = DF.groupby(['device_id', pd.Grouper(key = 'datetime', freq = FREQ)]).agg('mean')

# remove invalid records (i.e., PM2.5 < 0)
dfMean = dfMean[ dfMean['PM2.5'] >= 0 ]
print(dfMean.head())
```

```
                                  PM2.5
device_id    datetime                  
74DA38F207DE 2022-10-15 00:00:00   38.0
             2022-10-15 00:05:00   37.0
             2022-10-15 00:10:00   37.0
             2022-10-15 00:20:00   36.0
             2022-10-15 00:25:00   36.0
```

### Average the sensing values of nearby sensors every time slice

To calculate the average sensing value of the neighboring sensors of a specific sensor on a particular time slice, we write the `cal_mean` function, which can return the number and average sensing values of the neighboring sensors according to the input device number `iD` and time stamp `dt`.

```python
def cal_mean(iD, dt):
  neighborPM25 = dfMean[ 
              (dfMean.index.get_level_values('datetime') == dt) 
              & (dfMean.index.get_level_values(0).isin(dicNeighbor[iD])) 
              ]["PM2.5"]

  avg = neighborPM25.mean()
  neighbor_num = neighborPM25.count()
  return avg, neighbor_num
```

Then, for each timestamp of each sensor in dfMean, we calculate the number of neighboring sensors and the average sensing value and store them in two new fields, `avg` and `neighbor_num`, respectively. Note that we use the syntax of `zip` and `apply` it to bring the values of DataFrame into the function for operation:

1. We use the `zip` syntax for packaging the two parameter values returned by `apply_func`.
2. We use the syntax of `apply` to match the rules of DataFrame and receive the return value of `apply_func`.

```python
def apply_func(x):
  return cal_mean(x.name[0], x.name[1])

dfMean['avg'], dfMean['neighbor_num'] = zip(*dfMean.apply(apply_func, axis=1))
print(dfMean.head())
```

```
                                  PM2.5        avg  neighbor_num
device_id    datetime                                           
74DA38F207DE 2022-10-15 00:00:00   38.0  13.400000            10
             2022-10-15 00:05:00   37.0  19.888889             9
             2022-10-15 00:10:00   37.0  16.500000            12
             2022-10-15 00:20:00   36.0  16.750000             8
             2022-10-15 00:25:00   36.0  17.000000            11
```

### Abnormal event judgements

We've been looking into what counts as an "abnormal event" in our data. Basically, this means when a sensor's reading is way off what we'd normally expect. What's considered normal can depend on different things – like readings from nearby sensors, past readings from the same sensor, or even other kinds of information.

But deciding what's "way off" from normal isn't so straightforward, as it really depends on how big or small the sensor readings usually are.

To tackle this, we've broken down the average values (noted as `dfMean['avg']`) into nine different ranges. For each range, we've worked out the typical variation (or standard deviation) for PM2.5 sensor readings. This helps us set specific limits to decide if a reading is abnormal.

For instance, if a sensor usually reads 10 (in micrograms per cubic meter), we'd expect normal readings from nearby sensors to be within 6.6 above or below this (so between `10-6.6` and `10+6.6`). If it's outside this range, we'd consider it abnormal.

| Raw value (ug/m3) | threshold |
| --- | --- |
| 0-11 | 6.6 |
| 12-23 | 6.6 |
| 24-35 | 9.35 |
| 36-41 | 13.5 |
| 42-47 | 17.0 |
| 48-58 | 23.0 |
| 59-64 | 27.5 |
| 65-70 | 33.5 |
| 71+ | 91.5 |

We've written a function called `THRESHOLD` that uses these limits to check readings. It adds this info to our data in a new column, `PM_thr`.

```python

def THRESHOLD(value):
    if value<12:
        return 6.6
    elif value<24:
        return 6.6
    elif value<36:
        return 9.35
    elif value<42:
        return 13.5
    elif value<48:
        return 17.0
    elif value<54:
        return 23.0
    elif value<59:
        return 27.5
    elif value<65:
        return 33.5
    elif value<71:
        return 40.5
    else:
        return 91.5

dfMean['PM_thr'] = dfMean['PM2.5'].apply(THRESHOLD)
```

Since our latest data is from October 28, 2022, we're using October 29, 2022, as our reference date. We've added another column, `day`, to keep track of how each reading's date compares to this reference date. Below, you can see the current setup of our data table, `dfMEAN`.

```python
TARGET_DATE = "2022-10-29"
dfMean = dfMean.assign(days = lambda x: ( (pd.to_datetime(TARGET_DATE + " 23:59:59") - x.index.get_level_values('datetime')).days ) )
print(dfMean.head())
```

```
                                  PM2.5        avg  neighbor_num  PM_thr  days
device_id    datetime                                                         
74DA38F207DE 2022-10-15 00:00:00   38.0  13.400000            10    13.5    14
             2022-10-15 00:05:00   37.0  19.888889             9    13.5    14
             2022-10-15 00:10:00   37.0  16.500000            12    13.5    14
             2022-10-15 00:20:00   36.0  16.750000             8    13.5    14
             2022-10-15 00:25:00   36.0  17.000000            11    13.5    14
```

### Implementation of the Malfunction Detection module

In this example, we set up a system to figure out if a sensor is either indoors or near a pollution source. We do this by comparing the PM2.5 (a type of pollution measurement) readings from a specific sensor with other sensors located within a 3-kilometer radius.

- If a sensor's PM2.5 reading is lower than the average of nearby sensors minus a certain acceptable threshold (called `PM_thr`), we label it as `indoor`.
- If it's higher than the average plus the threshold, we label it as `emission`, indicating it might be near a pollution source.
- 
However, to make sure our conclusions are accurate, we only consider areas where there are more than two other sensors nearby.

```python
MINIMUM_NEIGHBORS = 2

dfMean["indoor"] = ((dfMean['avg'] - dfMean['PM2.5']) > dfMean['PM_thr']) & (dfMean['neighbor_num'] >= MINIMUM_NEIGHBORS)
dfMean["emission"] = ((dfMean['PM2.5'] - dfMean['avg']) > dfMean['PM_thr']) & (dfMean['neighbor_num'] >= MINIMUM_NEIGHBORS)

dfMean
```

![Python output](figures/6-2-2-3.png)

We've noticed that the labels `indoor` and `emission` can change based on daily air quality. So, to get a more reliable result, we look at data over different time periods: 1 day, 7 days, and 14 days. For each sensor, we calculate how often it was labeled `indoor` or `emission` during these periods.

To improve accuracy and avoid errors caused by unusual environmental conditions, we adjust our calculations. If a sensor is labeled `indoor` or `emission` less than one-third of the time in any period, we disregard that label.

```python
# initialize
dictIndoor = {iD: [] for iD in listId}
dictEmission = {iD: [] for iD in listId}

for iD in listId:
    dfId = dfMean.loc[iD]
    for day in [1, 7, 14]:
        indoor = (dfId[ dfId['days'] <= day]['indoor'].sum() / len(dfId[ dfId['days'] <= day])).round(3)
        dictIndoor[iD].append( 
            indoor if indoor > 0.333 else 0
        )
        
        emission = (dfId[ dfId['days'] <= day]['emission'].sum() / len(dfId[ dfId['days'] <= day])).round(3)
        dictEmission[iD].append(
            emission if emission > 0.333 else 0
        )
```

We keep track of our findings in two records: `dictIndoor` for indoor sensors and `dictEmission` for sensors near pollution sources. By examining these records, we can see how the time period affects our labeling.

```python
dictIndoor
```

```
{'74DA38F207DE': [0, 0, 0],
 '74DA38F20A10': [0, 0, 0],
 '74DA38F20B20': [0.995, 0.86, 0.816],
 '74DA38F20B80': [0.995, 0.872, 0.867],
 '74DA38F20BB6': [0, 0, 0],
 '74DA38F20C16': [0.989, 0.535, 0],
 '74DA38F20D7C': [0, 0, 0],
 '74DA38F20D8A': [0, 0, 0],
 '74DA38F20DCE': [0, 0, 0],
 '74DA38F20DD0': [0.984, 0.871, 0.865],
 '74DA38F20DD8': [0, 0.368, 0.369],
 '74DA38F20DDC': [0, 0, 0],
 '74DA38F20DE0': [0, 0, 0],
 '74DA38F20DE2': [0, 0, 0],
 '74DA38F20E0E': [0, 0, 0],
 '74DA38F20E42': [0.99, 0.866, 0.87],
 '74DA38F20E44': [0, 0, 0],
 '74DA38F20F0C': [0.979, 0.872, 0.876],
 '74DA38F20F2C': [0, 0, 0],
 '74DA38F210FE': [0.99, 0.847, 0.864]}
```

```python
dictEmission
```

```
{'74DA38F207DE': [0.92, 0.737, 0.735],
 '74DA38F20A10': [0, 0, 0],
 '74DA38F20B20': [0, 0, 0],
 '74DA38F20B80': [0, 0, 0],
 '74DA38F20BB6': [0.492, 0.339, 0],
 '74DA38F20C16': [0, 0, 0],
 '74DA38F20D7C': [0.553, 0.342, 0.337],
 '74DA38F20D8A': [0.672, 0.457, 0.388],
 '74DA38F20DCE': [0.786, 0.556, 0.516],
 '74DA38F20DD0': [0, 0, 0],
 '74DA38F20DD8': [0, 0, 0],
 '74DA38F20DDC': [0.345, 0, 0],
 '74DA38F20DE0': [0.601, 0.503, 0.492],
 '74DA38F20DE2': [0, 0, 0],
 '74DA38F20E0E': [0.938, 0.75, 0.75],
 '74DA38F20E42': [0, 0, 0],
 '74DA38F20E44': [0.938, 0.69, 0.6],
 '74DA38F20F0C': [0, 0, 0],
 '74DA38F20F2C': [0.744, 0.575, 0.544],
 '74DA38F210FE': [0, 0, 0]}
```

Since different time lengths give us different insights, we use a weighting method. We assign different weights to the results from 1-day, 7-day, and 14-day periods, using `A` for 1 day, `B` for 7 days, and `1-A-B` for 14 days.

We also factor in that a sensor operates about 8 hours a day, 40 hours over 7 days, and 80 hours over 14 days. A sensor is definitively labeled as `indoor` or `emission` if its weighted score meets or exceeds a certain threshold (`MD_thresh`). This threshold is calculated based on the operation hours and the weights assigned to different time periods, i.e., 

`MD_thresh` = (8.0/24.0)*A+(40.0/168.0)B+(80.0/336.0)(1-A-B)

For instance, if we set `A` as 0.2 and `B` as 0.3, we can then determine which sensors are consistently `indoor` or `emission` based on these weighted calculations.

```python
A=0.2
B=0.3
MD_thresh=(8.0/24.0)*A+(40.0/168.0)*B+(80.0/336.0)*(1-A-B)

listIndoorDevice = []
listEmissionDevice = []

for iD in listId:
    rate1 = A*dictIndoor[iD][0] + B*dictIndoor[iD][1] + (1-A-B)*dictIndoor[iD][2]
    if rate1 > MD_thresh:
        listIndoorDevice.append( (iD, rate1) )

    rate2 = A*dictEmission[iD][0] + B*dictEmission[iD][1] + (1-A-B)*dictEmission[iD][2]
    if rate2 > MD_thresh:
        listEmissionDevice.append( (iD, rate2) )
```

Finally, we review the results in `listIndoorDevice` and `listEmissionDevice` to see our weighted judgment outcomes.

```python
listIndoorDevice
```

```
[('74DA38F20B20', 0.865),
 ('74DA38F20B80', 0.8941),
 ('74DA38F20C16', 0.3583),
 ('74DA38F20DD0', 0.8906),
 ('74DA38F20DD8', 0.2949),
 ('74DA38F20E42', 0.8928),
 ('74DA38F20F0C', 0.8954),
 ('74DA38F210FE', 0.8841)]
```

```python
listEmissionDevice
```

```
[('74DA38F207DE', 0.7726),
 ('74DA38F20D7C', 0.38170000000000004),
 ('74DA38F20D8A', 0.4655),
 ('74DA38F20DCE', 0.5820000000000001),
 ('74DA38F20DE0', 0.5171),
 ('74DA38F20E0E', 0.7876),
 ('74DA38F20E44', 0.6945999999999999),
 ('74DA38F20F2C', 0.5933)]
```

### Implementation of the Real-time Emission Detection module

To detect air pollution as it happens, we use a method where we focus on the readings from small sensors. If the most recent measurement from a sensor is at least 20% higher than the one before it, we consider this a sign that there might be a significant increase in air pollution nearby. This approach is particularly effective for tracking changes in PM2.5 levels, a common air pollutant. However, we only apply this method when the sensor's readings are above 20. We do this because when the readings are very low, even a 20% increase might not indicate actual pollution – it could just be a normal fluctuation or a minor error in the sensor's measurement. This way, we avoid mistakenly identifying a situation as pollution when it isn't.

```python
dfMean['red'] = False
for iD in listId:
    dfId = dfMean.loc[iD]
    p_index = ''
    p_row = []
    for index, row in dfId.iterrows():
      red = False
      if p_index:
        diff = row['PM2.5'] - p_row['PM2.5']
        if p_row['PM2.5']>20 and diff>p_row['PM2.5']/5:
          red = True

      dfMean.loc[pd.IndexSlice[iD, index.strftime('%Y-%m-%d %H:%M:%S')], pd.IndexSlice['red']] = red
      p_index = index
      p_row = row

dfMean
```

![Python output](figures/6-2-2-4.png)

### Implementation of the Device Ranking module

To better understand the effectiveness of microsensors, we evaluate their reliability through a process we call Device Ranking (DR). This involves two key ideas:

1. If a sensor frequently reports data that seems unusual, either in terms of time or location, this could signal potential issues with the sensor's hardware or its environment. Such cases warrant further investigation.
2. Conversely, if a sensor rarely shows abnormal data, it suggests that its readings are in good agreement with those of nearby sensors, indicating high reliability.

To assess this, we look at each sensor's daily data and count how many times it records anomalies in time (marked as `red=True`) or space (marked as `indoor=True` or `emission=True`). We then compare these counts to the total data entries for that day. This comparison forms the basis for evaluating the sensor's reliability.

```python
device_rank = pd.DataFrame()
for iD in listId:
    dfId = dfMean.loc[iD]
    abnormal = {}
    num = {}
    for index, row in dfId.iterrows():
      d = index.strftime('%Y-%m-%d')
      if d not in abnormal:
        abnormal[d] = 0
      if d not in num:
        num[d] = 0
      num[d] = num[d] + 1
			if row['indoor'] or row['emission'] or row['red']:
        abnormal[d] = abnormal[d] + 1
    for d in num:
      device_rank = device_rank.append(pd.DataFrame({'device_id': [iD], 'date': [d], 'rank': [1 - abnormal[d]/num[d]]}))
device_rank.set_index(['device_id','date'])
```

![Python output](figures/6-2-2-5.png)

## References

- Ling-Jyh Chen, Yao-Hua Ho, Hsin-Hung Hsieh, Shih-Ting Huang, Hu-Cheng Lee, and Sachit Mahajan. ADF: an Anomaly Detection Framework for Large-scale PM2.5 Sensing Systems. IEEE Internet of Things Journal, volume 5, issue 2, pp. 559-570, April, 2018. ([https://dx.doi.org/10.1109/JIOT.2017.2766085](https://dx.doi.org/10.1109/JIOT.2017.2766085))
