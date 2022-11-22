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

Multiple large-scale micro air quality monitoring systems have been successfully deployed in different countries and cities. However, one of these micro sensors' main challenges is ensuring data quality and detecting possible anomalies in real-time. In 2018, the research team of the Network Research Laboratory of the Institute of Information Science, Academia Sinica, Taiwan, proposed an anomaly detection framework that can be used in real environments called the [Anomaly Detection Framework](https://ieeexplore.ieee.org/document/8081731) (ADF).

This anomaly detection framework consists of four modules:

1. Time-Sliced Anomaly Detection (TSAD): It can detect abnormal sensor data in space or time in real-time and output the results to other modules for further analysis.
2. Real-time Emission Detection (RED): It can detect potential regional pollution events in real-time through the detection results of TSAD.
3. Sensor reliability evaluation module (Device Ranking, DR): It can accumulate the detection results of TSAD and evaluate the reliability of each miniature sensor device
4. Abnormal use of machine detection module (Malfunction Detection, MD): It can accumulate the detection results of TSAD, and through data analysis, it can identify micro-sensors that may be used abnormally, such as machines installed indoors, placed in continuous Machines next to sources of sexual pollution, etc.

![ADF Framework](figures/6-2-1-1.png)

### Types of abnormal events

In the ADF framework, the TSAD module will judge the abnormal event of time or space every time the micro air quality sensor receives new sensing data. We will use the micro air quality sensor as an example to illustrate :

- Time-type abnormal events: We assume that air diffusion is uniform and slow, so the value change of the same micro air quality sensor in a short period should be extremely gentle. There are drastic changes in a short period, which means that abnormal events may occur in the time dimension.
- Spatial abnormal events: We can assume that the outdoor air will spread evenly in geographical space, so the sensing value of the micro-air sensor should be similar to the surrounding sensors. Suppose there is a considerable difference between the sensing value of one particular sensor and its neighboring at the same time. In that case, an abnormal event may occur in the space where the sensor is located.

### Possible causes of abnormal events

There are many possible causes for the abnormal events described above. Common ones are:

- Abnormal installation environment: The sensor is installed in a specific environment, so it cannot show the overall environmental phenomenon, such as installed next to a temple, in a barbecue shop, or in other indoor places without ventilation.
- Machine failure or installation error: For example, when the sensor is installed, the direction of the air intake is wrong, or the fan of the sensor is fouled so that the operation is not smooth.
- A temporary source of contamination occurs: For example, someone smoking, a fire, or emitting pollutants right next to the sensor.

## Case Study

In this article, we will take the Civil IoT Taiwan project air quality data as an example. We will use some campus micro air quality sensors installed in Kaohsiung City for analysis and introduce how to use the ADF detection framework to find potential sensors that may be installed indoors or located near pollution sources. We will also rank the sensors depending on their trustworthiness.

### Package Installation and Importing

In this article, we will use the pandas, numpy, plotly, and geopy packages, which are pre-installed on our development platform, Google Colab, and do not need to be installed manually. We can use the following syntax to import the relevant packages to complete the preparations in this article.

```python
import pandas as pd
import numpy as np

import plotly.express as px
from geopy.distance import geodesic
```

### Initialization and Data Access

In this case, we will use some of the micro air quality sensors deployed in Kaohsiung under the Civil IoT Taiwan Project for analysis. We consider the following spatial and temporal ranges of the data:

- Spatial range: latitude in the range of `22.631231 - 22.584989`, and longitude in the range of `120.263422 - 120.346764`
- Temporal range: 2022.10.15 - 2022.10.28

Note that you can go to the [Civil IoT Taiwan Data Service Platform](https://ci.taiwan.gov.tw/dsp/) to download the raw data of micro air quality sensors on campus. However, for simplicity, we pre-download the data and make it available as [allLoc.csv](https://LearnCIOT.github.io/data/allLoc.csv) for the examples described in this article.

We first load the data file and preview the contents of the data:

```python
DF = pd.read_csv("https://LearnCIOT.github.io/data/allLoc.csv")
DF.head()
```

![Python output](figures/6-2-2-1.png)

Then we fetch the GPS geolocation coordinates of each sensor in the data file. Since the GPS coordinates of these sensors will not change, we average each sensor's longitude and latitude data in the data file as the geographic coordinates of the sensor.

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

We plot the sensor locations on a map to get an overview of the geographic distribution of sensors in the data file.

```python
fig_map = px.scatter_mapbox(dfId, lat="lat", lon="lon",
                  color_continuous_scale=px.colors.cyclical.IceFire, zoom=9,
                  mapbox_style="carto-positron")
fig_map.show()
```


![Python output](figures/6-2-2-2.png)

### Find nearby sensors

Since the campus micro air quality sensors are fixedly installed, their GPS location coordinates will not change. To save the computing time for subsequent data analysis, we first calculate each sensor's "neighbor" list in a unified manner. In our case, we define that two tiny sensors become neighbors if their mutual distance is less than or equal to 3 kilometers.

We first write a function `countDis` to calculate the physical kilometer distance between the two input GPS geographic coordinates.

```python
def countDis(deviceA, deviceB):
    return geodesic((deviceA["lat"], deviceB['lon']), (deviceB["lat"], deviceB['lon'])).km
```

Then we convert the sensor list of raw data from DataFrame data type to Dictionary data type and calculate the distance between any two sensors. As long as the distance between the two is less than 3km, we store each other in the neighbor sensor list `dicNeighbor`.

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

Since each sensor in the original data is not synchronized in time, it is proposed in the ADF framework to divide the sensing data at intervals of each unit of time to obtain the time slice of the overall sensing result. We first merge the `date` and `time` fields in the original data and store them in the datetime data type of the Python language to form a new `datetime` field, and then delete unnecessary fields such as `date`, `time`, `RH`, `temperature`, `lat`, and `lon`.

```python
# combine the 'date' and 'time' columns to a new column 'datetime'
DF["datetime"] = DF["date"] + " " + DF["time"]

# remove some non-necessary columns
DF.drop(columns=["date","time", "RH","temperature","lat","lon"], inplace=True)

# convert the new 'datetime' column to datetime data type
DF['datetime'] = pd.to_datetime(DF.datetime)
```

Since the data frequency of the campus micro air quality sensors is about 5 minutes, we set the unit time `FREQ` of the time slice to 5 minutes and calculate the average value of the sensing values returned by each sensor every 5 minutes. To ensure the correctness of the data, we have also done additional verification and deleted the data with negative PM2.5 sensing values.

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

Through observation, we found that the so-called "abnormal event" refers to the sensing value of a sensor being too far from the reasonable value in our mind. This reasonable value may be the sensing value of the adjacent sensor (spatial type anomaly), the previous sensing value of the same sensor (time type anomaly), or a reasonable estimate based on other information sources. At the same time, we also found that the so-called "too far from the reasonable value" is a very vague term, and its specific value varies significantly with the size of the sensing value.

Therefore, we first divide the distribution of `dfMean['avg']` values into nine intervals and calculate the standard deviation of the PM2.5 sensing value in each interval. Then, we use these values as the threshold values for sensing data judgment, as shown in the table below. For example, when the original value is 10 (ug/m3), if the average value of the surrounding sensors is higher than `10+6.6` or lower than `10-6.6`, we will determine that this sensing value is an abnormal event.

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

According to this table, we write the following function `THRESHOLD` to return the corresponding threshold value according to the input sensing value, and store this threshold value in the new column `PM_thr` of `dfMEAN`.

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

Since the last record time of the original data is 2022-10-28 23:45, we set the judgment time as 2022-10-29. Then, we created a new column, `day`, to represent the difference between each record in the original data and the judgment time of the data. We preview the current state of the `dfMEAN` data table below.

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

In the following example, we implement the Malfunction Detection module in ADF. The core concept is that if we compare the PM2.5 value of a certain micro-sensor with other sensors within 3 kilometers around, A machine is considered an indoor machine (labeled `indoor`) if its sensed value is below the average value (`avg`) of neighboring sensors minus the acceptable threshold (`PM_thr`). A machine is considered to be installed next to a pollution source (labeled `emission`) if its sensed value is higher than the average value (`avg`) of neighboring sensors plus an acceptable threshold (`PM_thr`). To avoid misjudgment due to insufficient neighboring sensors, we only take cases with more than two other sensors in the neighboring area.

```python
MINIMUM_NEIGHBORS = 2

dfMean["indoor"] = ((dfMean['avg'] - dfMean['PM2.5']) > dfMean['PM_thr']) & (dfMean['neighbor_num'] >= MINIMUM_NEIGHBORS)
dfMean["emission"] = ((dfMean['PM2.5'] - dfMean['avg']) > dfMean['PM_thr']) & (dfMean['neighbor_num'] >= MINIMUM_NEIGHBORS)

dfMean
```

![Python output](figures/6-2-2-3.png)

We can observe from the results that `indoor` and `emission` judgment results are likely to be different due to different daily air quality conditions. Thus, to obtain a more convincing judgment result, we consider the three time lengths of the past one day, seven days, and 14 days to calculate the ratios of each sensor's judgment results (`indoor` or `emission`). At the same time, to avoid the misjudgment caused by the external environment from affecting the final judgment result, we forcibly modified the calculation result, and we changed the value with a ratio of less than 1/3 to 0.

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

We then print out the contents of `dictIndoor` and `dictEmission`, respectively, and observe the judged results.

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

From the above results, it can be found that with the length of the past reference time, there will be some differences in the indoor and emission judgment results. Since different lengths of time represent different reference meanings, we adopt a weighting method, assigning weight to 1-day reference time `A`, weighting to 7-day reference time `B`, and weighting to 14-day reference time `1-A-B`.

At the same time, we consider working about 8 hours out of 24 hours in a day, 40 hours out of 168 hours in seven days, and 80 hours out of 336 hours in 14 days. Therefore, if a sensor is judged as `indoor` or `emission` type, its weighted proportion should be greater than or equal to `MD_thresh`, and

`MD_thresh` = (8.0/24.0)*A+(40.0/168.0)B+(80.0/336.0)(1-A-B)

In our example, we assume `A=0.2` and `B=0.3`, and we can obtain the weighted `indoor` and `emission` sensor list through the following codes.

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

We then print out the contents of `listIndoorDevice` and `listEmissionDevice`, respectively, and observe the results of the weighted judgment.

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

For real-time emission detection (RED), we assume that if the latest sensing value of a tiny sensor is 1/5 more than the previous sensing value, the surrounding environment has undergone drastic changes, which is very likely due to the emission of air pollution around. Since the change of 1/5 is still very slight in the actual concentration change of PM2.5 when the sensing value is small, we exclude the situation that the sensing value is less than 20 to avoid the error caused by the sensor, leading to misjudgment of real-time pollution detection.

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

Finally, we summarize the judgment results of the RED and MD modules and conduct a reliability evaluation (Device Ranking, DR) for the microsensor. The main concepts are:

- If it is often judged that the sensing data of the sensor is abnormal in time or space, it indicates that there may be potential problems in the hardware or environment of the sensor, which needs to be further clarified.
- If the sensing data of the sensor is seldom judged to be abnormal, it means that the sensing value of the sensor is highly consistent with the values of the surrounding sensors, so the reliability is high.

We calculate the total number of time (`red=True`) or space (`indoor=True` or `emission=True`) anomalies for each sensor per day based on all the sensor data per day. Then we calculate its proportion of all data items in a day as sensor information Basis for reliability assessment.

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
