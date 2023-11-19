---
title : "3.2. Data Access under Spatial or Temporal Conditions"
weight : 20
socialshare: true
description : "We introduce how to obtain the data of a project in a specific time or time period, and the data of a project in a specific geographical area in the Civil IoT Taiwan Data Service Platform. We also demonstrate the application through a simple example."
tags: ["Python", "API", "Air"]
levels: ["intermediate"]
authors: ["Sky Hung"]
---


[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/drive/1wqE5rQFKHsT22TRUx34j6srdL5-LvE60?usp=sharing)

{{< toc >}}

This article will explore the Civil IoT Taiwan project's data, focusing on how it changes over time and location. We'll provide a straightforward example with a focus on monitoring air quality. The topics covered will include:

- How to use the datetime, math, numpy, and pandas packages in our analysis.
- Steps to handle data in JSON format.
- Using the Pandas DataFrame for effective data processing.

## Data access under temporal conditions

When you use the get_data() function in pyCIOT, you can retrieve data by specifying the start and end times. You'll need to provide this information in the form of a dictionary (Dict), which includes three elements: 'start', 'end', and 'num_of_data'.

Within the dictionary used for the get_data() function in pyCIOT, 'start' and 'end' refer to the beginning and end of the data collection period. These should be provided in either ISO8601 or Datetime format. The 'num_of_data' element sets a limit on the total number of data points collected. If the available data within your specified time range exceeds this limit, the system will collect the data at regular intervals to ensure an even distribution over time.

For example, if we look at air quality data, the system allows us to retrieve data from up to one day in the past. Therefore, if you set the 'end' time to more than a day ago, you won't get any data. Also, it's important to note that in the Civil IoT Taiwan project, different sensors update at different frequencies. This means the amount of data collected each day varies from sensor to sensor. For more detailed information, please refer to: [https://ci.taiwan.gov.tw/dsp/dataset_air.aspx](https://ci.taiwan.gov.tw/dsp/dataset_air.aspx)

```python
# Import datetime and timedelta classes from the datetime module.
from datetime import datetime, timedelta

# Get the current time.
end_date = datetime.now()
# Convert time to ISO8601 format, remove milliseconds and add "Z".
isodate_end = end_date.isoformat().split(".")[0]+"Z"
# Get the time of the previous day.
start_date = datetime.now() + timedelta(days = -1)
# Convert time to ISO8601 format, remove milliseconds and add "Z".
isodate_start = start_date.isoformat().split(".")[0]+"Z"

# Define time range and the number of data to be retrieved.
time = {
    "start": isodate_start, 
    "end": isodate_end, 
    "num_of_data": 15
}

# Retrieve 15 pieces of data within a day from a specific sensor.
data = Air().get_data("OBS:EPA_IoT", stationIds=["11613429495"], time_range=time) 
data
```

The data is organized and stored in a List format, with different types of data kept separately based on their characteristics. For instance, temperature, relative humidity, and PM2.5 (a measure of air pollution) data are each stored in a 'values' list under their respective names. Additionally, the time for each piece of data is recorded and displayed in the ISO8601 format.

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

In pyCIOT, you can also find specific data based on a geographical area. By using the latitude and longitude of a certain location as a central point, you can set a radius distance to create a circular search area. This allows you to obtain the station ID and sensor readings from that specific area.

To access data from a specific area in pyCIOT, you also use a dictionary (Dict) format. In this dictionary, you specify the 'latitude', 'longitude', and the search 'radius' using the keys 'latitude', 'longitude', and 'distance', respectively. You have the flexibility to use both the area-specific and time-specific filtering functions simultaneously. When searching within a specific area, you can also list the station IDs you want to observe under 'stationIds'. This way, any stations outside your specified area can be excluded from the search results.

```python
# Define a location information containing latitude, longitude, and search radius.
loc = {
    "latitude": 24.990550, # 緯度
    "longitude": 121.507532, # 經度
    "distance": 3.0 # 半徑(km)
}
# Fetch air information related to the defined location from the specified data source
c = Air().get_data(src="OBS:EPA_IoT", location = loc)
# Get the first piece of data from the search results.
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

The methods we've discussed are the basic ways to access station data in pyCIOT. These methods rely on time and location as the main filters and can be applied to all types of data, including those with location and timestamp information. To illustrate how this works, we'll provide some simple examples and show how to implement them using the pyCIOT packages.

## Case study: Is the air quality here worse than there?

- Project Reference: OBS:EPA_IoT (This refers to low-cost air quality stations operated by the Environmental Protection Agency).
- Specific Location for Study: Nanshijiao MRT Station Exit 1, with GPS coordinates at 24.990550, 121.507532.
- Area of Focus: Zhonghe District in New Taipei City.

### Import data

To start, we need to collect all relevant information about our chosen location and the surrounding area. We'll apply the 'data access under spatial conditions' approach. This involves setting the latitude and longitude coordinates to match Exit 1 of the Nanshijiao MRT Station and defining our search area to be within a three-kilometer radius. After setting these parameters, we can easily retrieve the data by using the Air().get_data() function.

```python
# Define a location information containing latitude, longitude, and search radius.
loc = {
    "latitude": 24.990550, 
    "longitude": 121.507532, 
    "distance": 3.0 # (km)
}
# Fetch air data related to the defined location from the "OBS:EPA_IoT" data source.
EPA_IoT_zhonghe_data_raw = Air().get_data(src="OBS:EPA_IoT", location = loc)
# Print the total number of obtained monitoring stations.
print("len:", len(EPA_IoT_zhonghe_data_raw))
# Get and print the first piece of data from the search results.
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

Among the stations we're considering, some may not be functioning properly. To identify and eliminate these problematic stations, we look at the data they provide. If a station's data shows zeros for all three measurements - temperature, humidity, and PM2.5 concentration - we consider this data invalid. By selecting and removing these zero-value data sets, we can ensure our analysis only includes reliable information before proceeding to the next step.

```python
# Data cleaning.
EPA_IoT_zhonghe_data = []
for datajson in EPA_IoT_zhonghe_data_raw:
  # Check if the key "data" exists.
  if "data" not in datajson:
    continue;
  # Convert raw data format to 'Temperature', 'Relative_Humidity', and 'PM2_5'.
  for rawdata_array in datajson['data']:
    if(rawdata_array['name'] == 'Temperature'):
      datajson['Temperature'] = rawdata_array['values'][0]['value']
    if(rawdata_array['name'] == 'Relative humidity'):
      datajson['Relative_Humidity'] = rawdata_array['values'][0]['value']
    if(rawdata_array['name'] == 'PM2.5'):
      datajson['PM2_5'] = rawdata_array['values'][0]['value']
  # Remove the key 'data' as we have extracted the information we need.
  datajson.pop('data')
  # Ensure all required keys are in the data and remove stations with all values being 0.
  if "Relative_Humidity" not in datajson.keys():
    continue
  if "PM2_5" not in datajson.keys():
    continue
  if "Temperature" not in datajson.keys():
    continue
  if(datajson['Relative_Humidity'] == 0 and datajson['PM2_5'] == 0 and datajson['Temperature'] == 0):
    continue
  EPA_IoT_zhonghe_data.append(datajson)

# Print the number of data entries after cleaning.
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

Next, we assume that the data from all stations are accurate. To compare, we need the data from the station closest to our target location. To find this nearest station, we calculate the distance between each station and the target.

We'll use the [Haversine formula](https://en.wikipedia.org/wiki/Haversine_formula), a standard method for determining the spherical distance between two points on Earth, to make these calculations. This method is particularly effective when working with the WGS84 coordinate system, which is commonly used for global positioning. Below is how we implement this calculation:

```python
import math
# Define a function to compute the distance between two points given their latitudes and longitudes.
def LLs2Dist(lat1, lon1, lat2, lon2):
    R = 6371
    # Radius of the Earth in kilometers.
    dLat = (lat2 - lat1) * math.pi / 180.0
    dLon = (lon2 - lon1) * math.pi / 180.0
    a = math.sin(dLat / 2) * math.sin(dLat / 2) + math.cos(lat1 * math.pi / 180.0) * math.cos(lat2 * math.pi / 180.0) * math.sin(dLon / 2) * math.sin(dLon / 2)
    c = 2 * math.atan2(math.sqrt(a), math.sqrt(1 - a))
    dist = R * c
    return dist

# Calculate the distance of each data point to the Nanshijiao Station and store the result in the 'distance' column.
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

Pandas is a popular Python package for data manipulation and analysis. It's known for its DataFrame feature, which is ideal for handling two-dimensional or multi-column data — perfect for data analysis tasks. We'll take our processed data and convert it into a DataFrame format. This allows us to easily select the necessary fields and sort them based on the distance we calculated earlier, from the shortest to the longest.

```python
import pandas as pd

# Use pandas' json_normalize method to convert the JSON data into DataFrame.
df = pd.json_normalize(EPA_IoT_zhonghe_data) 
# This is the full data after conversion.
df

# Select the columns we need from the full data.
EPA_IoT_zhonghe_data_raw = df[['distance', 'PM2_5', 'Temperature', 'Relative_Humidity', 'properties.stationID', 'location.latitude', 'location.longitude', 'properties.areaType']]
# Sort the data based on the 'distance' and 'PM2_5' columns.
EPA_IoT_zhonghe_data_raw = EPA_IoT_zhonghe_data_raw.sort_values(by=['distance', 'PM2_5'], ascending=True)
EPA_IoT_zhonghe_data_raw
```

![Python Output](figures/3-2-1.png)

### Results

To determine whether the air quality at our target location is better or worse than in the surrounding area, we can look at the air quality data from all the nearby stations. For this analysis, tools like the numpy package, which is widely used in Python for data science, can be very helpful. By calculating the average (mean) and the variation (standard deviation) of air quality data from these stations, we can get a clearer picture of the air quality at our specific location compared to the overall area.

```python
import numpy as np
# Get the PM2.5 value of the nearest monitoring station from EPA_IoT_zhonghe_data_raw.
zhonghe_target = EPA_IoT_zhonghe_data_raw.iloc[0,1]
# Calculate the average and standard deviation of PM2.5.
zhonghe_ave = np.mean(EPA_IoT_zhonghe_data_raw.iloc[:,1].values)
zhonghe_std = np.std(EPA_IoT_zhonghe_data_raw.iloc[:,1].values)
# Calculate the difference between the target PM2.5 value and the average, expressed in terms of standard deviations.
result = (zhonghe_target-zhonghe_ave)/zhonghe_std

# Print out the computed results.
print('Mean:', zhonghe_ave, 'std:', zhonghe_std)
print('PM2.5 of the neareat station:', zhonghe_target)
print('The target is ', result, 'standard deviations from the mean.\n')

# Print out the air quality assessment based on the result.
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
