---
title : "3.2. Data Access under Spatial or Temporal Conditions"
weight : 20
socialshare: true
description : "We introduce how to obtain the data of a project in a specific time or time period, and the data of a project in a specific geographical area in the Civil IoT Taiwan Data Service Platform. We also demonstrate the application through a simple example."
tags: ["Python", "API", "Air"]
levels: ["intermediate"]
author: ["Sky Hung"]
---


[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/drive/1wqE5rQFKHsT22TRUx34j6srdL5-LvE60?usp=sharing)

{{< toc >}}

This article will access the data of the Civil IoT Taiwan project from the perspective of time and space, and carry out a simple implementation with the theme of air quality monitoring. It will covers the following topics:

- the usage of the datetime, math, numpy, and pandas packages
- the processing of JSON format data
- data processing using Pandas DataFrame

## Data access under temporal conditions

When using `get_data()` in pyCIOT, data can be obtained according to the start time and end time. The format is passed in the `time_range` as a dictionary (Dict), which are `start`, `end` and `num_of_data` respectively.

`start` and `end` refer to the start and end time of data collection, in ISO8601 or Datetime format. `num_of_data` will control the number of data acquisitions not to exceed this number. If the data in the range exceeds num_of_data, it will be collected at intervals, so that the time interval between data and data tends to be the same.

Taking air quality data as an example, the acquired data can go back one day at most, so when the `end` variable is set greater than one day, no data will be acquired. In addition, since the update frequency of each sensor in the Civil IoT Taiwan project is different, the number of data sets per day for different sensors will also be different. For details, please refer to: [https://ci.taiwan.gov.tw/dsp/dataset_air.aspx](https://ci.taiwan.gov.tw/dsp/dataset_air.aspx)

```python
from datetime import datetime, timedelta

end_date = datetime.now() # current datetime
isodate_end = end_date.isoformat().split(".")[0]+"Z" # convert to ISO8601 format
start_date = datetime.now() + timedelta(days = -1) # yesterday
isodate_start = start_date.isoformat().split(".")[0]+"Z" # convert to ISO8601 format

time = {
    "start": isodate_start, 
    "end": isodate_end, 
    "num_of_data": 15
}

data = Air().get_data("OBS:EPA_IoT", stationIds=["11613429495"], time_range=time) 
data
```

The data will be stored in `data` in List format, and stored separately according to different properties. The data of temperature, relative humidity and PM2.5 will be stored in the '`values`' list under the corresponding name respectively, and the time of each data record will be marked and displayed in ISO8601.

```
[{'name': '智慧城鄉空品微型感測器-11613429495',
  'description': '智慧城鄉空品微型感測器-11613429495',
  'properties': {'city': '新竹市',
   'areaType': '一般社區',
   'isMobile': 'false',
   'township': '香山區',
   'authority': '行政院環境保護署',
   'isDisplay': 'true',
   'isOutdoor': 'true',
   'stationID': '11613429495',
   'locationId': 'HC0154',
   'Description': 'AQ1001',
   'areaDescription': '新竹市香山區'},
  'data': [{'name': 'Temperature',
    'description': '溫度',
    'values': [{'timestamp': '2022-08-27T12:53:10.000Z', 'value': 30.6},
     {'timestamp': '2022-08-27T12:52:09.000Z', 'value': 30.6},
     {'timestamp': '2022-08-27T12:51:09.000Z', 'value': 30.6},
     {'timestamp': '2022-08-27T12:50:09.000Z', 'value': 30.6},
     {'timestamp': '2022-08-27T12:49:09.000Z', 'value': 30.7},
     {'timestamp': '2022-08-27T12:48:10.000Z', 'value': 30.7},
     {'timestamp': '2022-08-27T12:47:10.000Z', 'value': 30.7},
     {'timestamp': '2022-08-27T12:46:10.000Z', 'value': 30.7},
     {'timestamp': '2022-08-27T12:45:10.000Z', 'value': 30.7},
     {'timestamp': '2022-08-27T12:44:10.000Z', 'value': 30.7},
     {'timestamp': '2022-08-27T12:43:09.000Z', 'value': 30.7},
     {'timestamp': '2022-08-27T12:42:10.000Z', 'value': 30.7},
     {'timestamp': '2022-08-27T12:41:09.000Z', 'value': 30.7},
     {'timestamp': '2022-08-27T12:40:10.000Z', 'value': 30.7},
     {'timestamp': '2022-08-27T12:39:10.000Z', 'value': 30.7}]},
   {'name': 'Relative humidity',
    'description': '相對溼度',
    'values': [{'timestamp': '2022-08-27T12:54:10.000Z', 'value': 100},
     {'timestamp': '2022-08-27T12:53:10.000Z', 'value': 100},
     {'timestamp': '2022-08-27T12:52:09.000Z', 'value': 100},
     {'timestamp': '2022-08-27T12:51:09.000Z', 'value': 100},
     {'timestamp': '2022-08-27T12:50:09.000Z', 'value': 100},
     {'timestamp': '2022-08-27T12:49:09.000Z', 'value': 100},
     {'timestamp': '2022-08-27T12:48:10.000Z', 'value': 100},
     {'timestamp': '2022-08-27T12:47:10.000Z', 'value': 100},
     {'timestamp': '2022-08-27T12:46:10.000Z', 'value': 100},
     {'timestamp': '2022-08-27T12:45:10.000Z', 'value': 100},
     {'timestamp': '2022-08-27T12:44:10.000Z', 'value': 100},
     {'timestamp': '2022-08-27T12:43:09.000Z', 'value': 100},
     {'timestamp': '2022-08-27T12:42:10.000Z', 'value': 100},
     {'timestamp': '2022-08-27T12:41:09.000Z', 'value': 100},
     {'timestamp': '2022-08-27T12:40:10.000Z', 'value': 100}]},
   {'name': 'PM2.5',
    'description': '細懸浮微粒 PM2.5',
    'values': [{'timestamp': '2022-08-27T12:53:10.000Z', 'value': 11.9},
     {'timestamp': '2022-08-27T12:52:09.000Z', 'value': 12.15},
     {'timestamp': '2022-08-27T12:51:09.000Z', 'value': 12.2},
     {'timestamp': '2022-08-27T12:50:09.000Z', 'value': 12.22},
     {'timestamp': '2022-08-27T12:49:09.000Z', 'value': 12.54},
     {'timestamp': '2022-08-27T12:48:10.000Z', 'value': 12.54},
     {'timestamp': '2022-08-27T12:47:10.000Z', 'value': 12.31},
     {'timestamp': '2022-08-27T12:46:10.000Z', 'value': 12.19},
     {'timestamp': '2022-08-27T12:45:10.000Z', 'value': 12.26},
     {'timestamp': '2022-08-27T12:44:10.000Z', 'value': 12.17},
     {'timestamp': '2022-08-27T12:43:09.000Z', 'value': 12.04},
     {'timestamp': '2022-08-27T12:42:10.000Z', 'value': 11.7},
     {'timestamp': '2022-08-27T12:41:09.000Z', 'value': 11.67},
     {'timestamp': '2022-08-27T12:40:10.000Z', 'value': 11.56},
     {'timestamp': '2022-08-27T12:39:10.000Z', 'value': 11.56}]}],
  'location': {'latitude': 24.81796, 'longitude': 120.92664, 'address': None}}]
```

## Data access under spatial conditions

In pyCIOT, there are also methods to get specific data based on the region. If we take the latitude and longitude of a specific location as the center, we can use the radius distance to form a search range (circle) to get the station ID and sensing value of the specific space.

The data format of a specific area is also passed in as a dictionary (Dict), where the latitude, longitude and radius are "`latitude`", "`longitude`" and "`distance`" respectively. The filtering functions of a specific area and a specific time can be used at the same time. When searching for a specific area, the stations to be observed can also be put into the "`stationIds`", and the stations outside the area can be removed by the way.

```python
loc = {
    "latitude": 24.990550, # 緯度
    "longitude": 121.507532, # 經度
    "distance": 3.0 # 半徑(km)
}
c = Air().get_data(src="OBS:EPA_IoT", location = loc)
c[0]
```

```
{
'name': '智慧城鄉空品微型感測器-10382640142',
'description': '智慧城鄉空品微型感測器-10382640142',
'properties': {
    'city': '新北市',
    'areaType': '交通',
    'isMobile': 'false',
    'township': '中和區',
    'authority': '行政院環境保護署',
    'isDisplay': 'true',
    'isOutdoor': 'true',
    'stationID': '10382640142',
    'locationId': 'TW040203A0506917',
    'Description': '廣域SAQ-210',
    'areaDescription': '中和區'},
    'data': [
        {
        'name': 'Temperature',
        'description': '溫度',
        'values': [{'timestamp': '2022-08-27T08:07:03.000Z', 'value': 35.84}]
        },{
        'name': 'Relative humidity',
        'description': '相對溼度',
        'values': [{'timestamp': '2022-08-27T08:07:03.000Z', 'value': 59.5}]
        },{
        'name': 'PM2.5',
        'description': '細懸浮微粒 PM2.5',
        'values': [{'timestamp': '2022-08-27T08:07:03.000Z', 'value': 11.09}]
        }
    ],
    'location': {
        'latitude': 24.998769,
        'longitude': 121.512717,
        'address': None
    }
}
```

---

The above are the basic methods commonly used to obtain pyCIOT station data, using time and space as the filtering criteria, and applicable to all data including `location` and `timestamp` types. To demonstrate, we give some simple examples and implement them using these pyCIOT packages.

## Case study: Is the air quality here worse than there?

- Project codes: OBS:EPA_IoT (low-cost air quality stations by EPA）
- Target location: Nanshijiao MRT Station Exit 1 (GPS coordinates: 24.990550, 121.507532)
- Region of interest: Zhonghe District, New Taipei City

### Import data

First, we need to get all the information about the target location and the region of interest. We can use the method of "data access under spatial conditions" to set the latitude and longitude at Exit 1 of Nanshijiao MRT Station, and set the distance to three kilometers. Then, we simply use `Air().get_data()` to obtain the data:

```python
loc = {
    "latitude": 24.990550, 
    "longitude": 121.507532, 
    "distance": 3.0 # (km)
}
EPA_IoT_zhonghe_data_raw = Air().get_data(src="OBS:EPA_IoT", location = loc)
print("len:", len(EPA_IoT_zhonghe_data_raw))
EPA_IoT_zhonghe_data_raw[0]
```

```
len: 70
{'name': '智慧城鄉空品微型感測器-10382640142',
 'description': '智慧城鄉空品微型感測器-10382640142',
 'properties': {'city': '新北市',
  'areaType': '交通',
  'isMobile': 'false',
  'township': '中和區',
  'authority': '行政院環境保護署',
  'isDisplay': 'true',
  'isOutdoor': 'true',
  'stationID': '10382640142',
  'locationId': 'TW040203A0506917',
  'Description': '廣域SAQ-210',
  'areaDescription': '中和區'},
 'data': [{'name': 'Relative humidity',
   'description': '相對溼度',
   'values': [{'timestamp': '2022-09-11T09:58:21.000Z', 'value': 94.84}]},
  {'name': 'PM2.5',
   'description': '細懸浮微粒 PM2.5',
   'values': [{'timestamp': '2022-09-11T09:58:21.000Z', 'value': 3.81}]},
  {'name': 'Temperature',
   'description': '溫度',
   'values': [{'timestamp': '2022-09-11T09:58:21.000Z', 'value': 25.72}]}],
 'location': {'latitude': 24.998769, 'longitude': 121.512717, 'address': None}}
```

### Remove invalid data

Of the in-scope sites, not every station is still running smoothly. In order to remove potentially problematic sites, we observe the data characteristics of invalid stations and find that all three data (temperature, humidity, PM2.5 concentration) are 0. So we just need to pick and delete this data before we can move on to the next step.

```python
# Data cleaning
EPA_IoT_zhonghe_data = []
for datajson in EPA_IoT_zhonghe_data_raw:
	# 確認資料存在
  if "data" not in datajson:
    continue;
	# 將格式轉換為 Temperature, Relative_Humidity 和 PM2_5
  for rawdata_array in datajson['data']:
    if(rawdata_array['name'] == 'Temperature'):
      datajson['Temperature'] = rawdata_array['values'][0]['value']
    if(rawdata_array['name'] == 'Relative humidity'):
      datajson['Relative_Humidity'] = rawdata_array['values'][0]['value']
    if(rawdata_array['name'] == 'PM2.5'):
      datajson['PM2_5'] = rawdata_array['values'][0]['value']
  datajson.pop('data')
	# 確認所有資料皆為有效，同時去除無資料之檢測站
  if "Relative_Humidity" not in datajson.keys():
    continue
  if "PM2_5" not in datajson.keys():
    continue
  if "Temperature" not in datajson.keys():
    continue
  if(datajson['Relative_Humidity'] == 0 and datajson['PM2_5'] == 0 and datajson['Temperature'] == 0):
    continue
  EPA_IoT_zhonghe_data.append(datajson)

print("len:", len(EPA_IoT_zhonghe_data))
EPA_IoT_zhonghe_data[0]
```

```
len: 70
{'name': '智慧城鄉空品微型感測器-10382640142',
 'description': '智慧城鄉空品微型感測器-10382640142',
 'properties': {'city': '新北市',
  'areaType': '交通',
  'isMobile': 'false',
  'township': '中和區',
  'authority': '行政院環境保護署',
  'isDisplay': 'true',
  'isOutdoor': 'true',
  'stationID': '10382640142',
  'locationId': 'TW040203A0506917',
  'Description': '廣域SAQ-210',
  'areaDescription': '中和區'},
 'location': {'latitude': 24.998769, 'longitude': 121.512717, 'address': None},
 'PM2_5': 2.61,
 'Relative_Humidity': 94.27,
 'Temperature': 26.24}
```

### Calculate distance

Assuming that there is no error in the data of each station, the data of the station closest to the target position is the data to be compared. To find the nearest station, we need to calculate the distance between each station and the target location.

We can use the point-to-point distance formula to calculate and sort to find the closest station to the target location. But here we use the standard [Haversine formula](https://en.wikipedia.org/wiki/Haversine_formula) to calculate the spherical distance between two points on Earth. The following is the implementation in the WGS84 coordinate system:

```python
import math
def LLs2Dist(lat1, lon1, lat2, lon2):
    R = 6371
    dLat = (lat2 - lat1) * math.pi / 180.0
    dLon = (lon2 - lon1) * math.pi / 180.0
    a = math.sin(dLat / 2) * math.sin(dLat / 2) + math.cos(lat1 * math.pi / 180.0) * math.cos(lat2 * math.pi / 180.0) * math.sin(dLon / 2) * math.sin(dLon / 2)
    c = 2 * math.atan2(math.sqrt(a), math.sqrt(1 - a))
    dist = R * c
    return dist

for data in EPA_IoT_zhonghe_data:
  data['distance'] = LLs2Dist(data['location']['latitude'], data['location']['longitude'], 24.990550, 121.507532)# (24.990550, 121.507532)
EPA_IoT_zhonghe_data[0]
```

```
{'name': '智慧城鄉空品微型感測器-10382640142',
 'description': '智慧城鄉空品微型感測器-10382640142',
 'properties': {'city': '新北市',
  'areaType': '交通',
  'isMobile': 'false',
  'township': '中和區',
  'authority': '行政院環境保護署',
  'isDisplay': 'true',
  'isOutdoor': 'true',
  'stationID': '10382640142',
  'locationId': 'TW040203A0506917',
  'Description': '廣域SAQ-210',
  'areaDescription': '中和區'},
 'location': {'latitude': 24.998769, 'longitude': 121.512717, 'address': None},
 'PM2_5': 2.61,
 'Relative_Humidity': 94.27,
 'Temperature': 26.24,
 'distance': 1.052754763080127}
```

### Pandas package

Pandas is a Python package of commonly used data manipulation and analysis. DataFrame format is used to store two-dimensional or multi-column data format, which is very suitable for data analysis. We convert the processed data into a DataFrame, pick out the required fields, and sort them according to the previously calculated distance from small to large.

```python
import pandas as pd

df = pd.json_normalize(EPA_IoT_zhonghe_data) 
#Results contain the required data
df

EPA_IoT_zhonghe_data_raw = df[['distance', 'PM2_5', 'Temperature', 'Relative_Humidity', 'properties.stationID', 'location.latitude', 'location.longitude', 'properties.areaType']]
EPA_IoT_zhonghe_data_raw = EPA_IoT_zhonghe_data_raw.sort_values(by=['distance', 'PM2_5'], ascending=True)
EPA_IoT_zhonghe_data_raw
```

![Python Output](figures/3-2-1.png)

### Results

To know if the air quality at the target location is better than the area of interest, we can roughly use the distribution of air quality across all stations. You can use tools such as the numpy package, a common data science processing library in Python, or directly calculate the mean and standard deviation to get the answer.

```python
import numpy as np
zhonghe_target = EPA_IoT_zhonghe_data_raw.iloc[0,1]
zhonghe_ave = np.mean(EPA_IoT_zhonghe_data_raw.iloc[:,1].values)
zhonghe_std = np.std(EPA_IoT_zhonghe_data_raw.iloc[:,1].values)
result = (zhonghe_target-zhonghe_ave)/zhonghe_std

print('Mean:', zhonghe_ave, 'std:', zhonghe_std)
print('PM2.5 of the neareat station:', zhonghe_target)
print('The target is ', result, 'standard deviations from the mean.\n')

if(result>0):
    print('Result: The air quality at the target location is worse.')
else:
    print('Result: The air quality at the target location is better.')
```

```
Mean: 6.71 std: 3.18
PM2.5 of the neareat station:7.38
The target is 0.21 standard deviations from the mean.

Result: The air quality at the target location is worse.
```

## References

- Python pyCIOT package ([https://pypi.org/project/pyCIOT/](https://pypi.org/project/pyCIOT/))
- pandas - Python Data Analysis Library ([https://pandas.pydata.org/](https://pandas.pydata.org/))
- 10 minutes to pandas — pandas documentation ([https://pandas.pydata.org/pandas-docs/stable/user_guide/10min.html](https://pandas.pydata.org/pandas-docs/stable/user_guide/10min.html))
- NumPy ([https://numpy.org/](https://numpy.org/))
- NumPy quickstart ([https://numpy.org/doc/stable/user/quickstart.html](https://numpy.org/doc/stable/user/quickstart.html))
- Haversine formula - Wikipedia ([https://en.wikipedia.org/wiki/Haversine_formula](https://en.wikipedia.org/wiki/Haversine_formula))
