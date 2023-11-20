---
title : "3.1. Basic Data Access Methods"
weight : 10
socialshare: true
description : "We introduce how to obtain water, air, earthquake, and disaster data in the Civil IoT Taiwan Data Service Platform, including the latest sensing data for a single site, a list of all sites, and the latest current sensing data for all sites."
tags: ["Python", "API", "Water", "Air", "Quake", "Disaster"]
levels: ["beginner"]
authors: ["Sky Hung"]
---

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/drive/1DNyxdtg4EBs1X1ulNJvsMLBeKEhXnwgP?usp=sharing)

{{< toc >}}


This guide provides step-by-step instructions on how to use the pyCIOT package to access various data types such as air, water, earthquake, weather, CCTV, and disaster warnings from the Civil IoT Taiwan Data Service Platform. It covers methods to obtain the most recent sensor data from an individual location, a comprehensive list of all available locations, and the latest sensor data from every location.

Please note that this article is designed for individuals who have basic knowledge of operating a terminal and are familiar with the fundamental syntax of Python programming.

## What is pyCIOT?

The Civil IoT Taiwan project offers a diverse range of data, each type having unique formats and access protocols. Despite being openly licensed, managing and processing this data can be challenging due to the varied methods required for downloading and handling it. To address these challenges, the pyCIOT suite was developed. This suite central

## pyCIOT Basics

### Installation

Firstly, we download and install the pyCIOT suite using pip. Pip is a package management system designed for Python, facilitating the installation and management of Python packages. The pyCIOT suite for this tutorial is hosted on the Python Package Index (PyPI). You can execute the following command in the terminal to install the pyCIOT library on your local machine, or to install other necessary packages alongside it:

```powershell
# Install the pyCIOT package using pip.
!pip install pyCIOT
```

### Import Package

To start using the package, simply input the import syntax and include `pyCIOT.data` in your Python script:

```python
# Import all functions and classes from the data module of pyCIOT.
from pyCIOT.data import *
```

The way you call a method from the package varies depending on how you import it. When using the `from ... import ...` syntax, there's no need to include a prefix while calling a method. However, if you use the `import ...` approach, you must prefix the method name each time you use it. Alternatively, using `import ... as ...` allows you to call a method with a custom prefix specified after `as`.

```python

# Import the whole module and use it with the module name as a prefix.
import pyCIOT.data 
a = pyCIOT.data.Air().get_source()

# Import the module with an alias to shorten the module name.
import pyCIOT.data as CIoT
a = CIoT.Air().get_source()

# Import all functions/classes from the module directly. Beware of name collisions in large projects.
from pyCIOT.data import *
a = Air().get_source()
```

## Data Access

You can access data from the Civil IoT Taiwan project, including air, water, earthquake, weather, CCTV, and more, using the following methods:

- `.get_source()`: This function retrieves all project codes available on the Civil IoT Taiwan Data Service Platform, presented in an array format based on the type of data.
- `.get_station(src='')`: This method returns basic details of all stations in an array format. The `src` parameter is optional and can be used to specify a particular project code.
- `.get_data(src='', stationID='')`: This function provides basic information about all stations along with their most recent measurement results, also in an array format. Here, `src` is an optional parameter for specifying the project code, and `stationID` is also optional for specifying the device ID.

For accessing disaster notification data, the following methods are used:

- `.get_alert()`: This retrieves alert data including details about the event, formatted in JSON.
- `.get_notice()`: This method returns notification data along with event information, also in JSON format.

Please be aware that the pyCIOT package is continuously being updated. In cases where there is a discrepancy between this guide and the [pyCIOT Package Document](https://hackmd.io/JdFywArGS9uxcSRTyYDtBg), the information in the package document should be considered as the most accurate and up-to-date.

## Air Quality Data

### Get all project codes: `Air().get_source()`

```python
# Use the get_source function of the Air class to retrieve all air-related project codes.
a = Air().get_source()
# Display the retrieved project codes.
print(a)
```

```
['OBS:EPA', 'OBS:EPA_IoT', 'OBS:AS_IoT', 'OBS:MOST_IoT', 'OBS:NCNU_IoT']
```

Here are the valid project codes for accessing air quality data:

- `OBS:EPA`: This code refers to national-level monitoring stations operated by the Environmental Protection Agency (EPA).
- `OBS:EPA_IoT`: This is used for low-cost air quality stations also managed by the EPA.
- `OBS:AS_IoT`: These are micro air quality stations run by Academia Sinica.
- `OBS:MOST_IoT`: This code pertains to low-cost air quality

### Get all stations: `Air().get_station()`

```python
# Use the get_station function from the Air class, specifying the data source as EPA's Smart Urban Air Quality Micro-sensing, to retrieve the station list.
b = Air().get_station(src="OBS:EPA_IoT")
# Retrieve and view the first five station items from the list
b[0:5]
```

```
[
    {
    'name': '智慧城鄉空品微型感測器-10287974676',
    'description': '智慧城鄉空品微型感測器-10287974676',
    'properties': {
        'city': '新北市',
        'areaType': '社區',
        'isMobile': 'false',
        'township': '鶯歌區',
        'authority': '行政院環境保護署',
        'isDisplay': 'true',
        'isOutdoor': 'true',
        'stationID': '10287974676',
        'locationId': 'TW040203A0507221',
        'Description': '廣域SAQ-210',
        'areaDescription': '鶯歌區' },
    'location': {
        'latitude': 24.9507,
        'longitude': 121.3408416,
        'address': None }
    },
    ...
]
```

### Get data of a station: `Air().get_data()`

```python
# Use the get_data function from the Air class, specifying the data source and station ID, to retrieve air quality data for the specified station.
f = Air().get_data(src="OBS:EPA_IoT", stationID="11613429495")
# View the retrieved air quality data.
f
```

```
[
{'name': '智慧城鄉空品微型感測器-11613429495',
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
    'values': [{'timestamp': '2022-08-28T06:22:08.000Z', 'value': 30.6}]},
   {'name': 'Relative humidity',
    'description': '相對溼度',
    'values': [{'timestamp': '2022-08-28T06:23:08.000Z', 'value': 100}]},
   {'name': 'PM2.5',
    'description': '細懸浮微粒 PM2.5',
    'values': [{'timestamp': '2022-08-28T06:22:08.000Z', 'value': 9.8}]}],
  'location': {'latitude': 24.81796, 'longitude': 120.92664, 'address': None}}
]
```

```python
# Print the description of the air quality dataset
print(f[0]['description'])
for f_data in f[0]['data']:
  # If data described as "temperature" is found, print the corresponding value and timestamp.
  if f_data['description'] == '溫度':
    print(f_data['description'], ': ', f_data['values'][0]['value'], ' (', f_data['values'][0]['timestamp'], ')', sep='')
```

```
智慧城鄉空品微型感測器-11613429495
溫度: 30.6 (2022-08-28T06:22:08.000Z)
```

## Water Resource Data

### Get all project codes: `Water().get_source()`

The function will return project names based on the specified input parameter:

- `water_level_station`: Retrieves the names of projects related to water level monitoring. The current valid codes are `WRA`, `WRA2`, and `IA`.
- `gate`: Returns the names of projects associated with water gates. Valid codes for this category include `WRA`, `WRA2`, and `IA`.
- `pumping_station`: This parameter provides names of projects related to pumping stations, with valid codes being `WRA2` and `TPE`.
- `sensor`: Lists the names of water sensor-related projects. The relevant codes here are `WRA`, `WRA2`, `IA`, and `CPAMI`.
- ``(none): When no parameter is specified, it returns the names of all water-related projects.

```python
# Use the get_source function of the Water class to retrieve all water-related project codes.
wa = Water().get_source()
# View the retrieved project codes.
wa
```

```
['WATER_LEVEL:WRA_RIVER',
 'WATER_LEVEL:WRA_GROUNDWATER',
 'WATER_LEVEL:WRA2_DRAINAGE',
 'WATER_LEVEL:IA_POND',
 'WATER_LEVEL:IA_IRRIGATION',
 'GATE:WRA',
 'GATE:WRA2',
 'GATE:IA',
 'PUMPING:WRA2',
 'PUMPING:TPE',
 'FLOODING:WRA',
 'FLOODING:WRA2']
```

Here are the valid project codes for accessing water resource data:

- `WRA`: Represents the Water Resource Agency.
- `WRA2`: Refers to the Water Resource Agency, in collaboration with county and city governments.
- `IA`: Stands for the Irrigation Agency.
- `CPAMI`: Denotes the Construction and Planning Agency.
- `TPE`: Indicates projects specific to Taipei City.

### Get all stations: `Water().get_station()`

```python
# Use the get_station function from the Water class, specifying the data source "WATER_LEVEL:WRA_RIVER", to retrieve the water level station list.
wa = Water().get_station(src="WATER_LEVEL:WRA_RIVER")
# View the first item from the station list.
wa[0]
```

```
{
  'name': '01790145-cd7e-4498-9240-f0fcd9061df2',
  'description': '現場觀測',
  'properties': {'authority': '水利署水文技術組',
   'stationID': '01790145-cd7e-4498-9240-f0fcd9061df2',
   'stationCode': '2200H007',
   'stationName': '延平',
   'authority_type': '水利署'},
  'location': {'latitude': 22.8983536,
   'longitude': 121.0845795,
   'address': None}
}
```

### Get data of a station: `Water().get_data()`

```python
# Use the get_data function from the Water class, specifying the data source and station ID, to retrieve water level data for the specified station.
wa = Water().get_data(src="WATER_LEVEL:WRA_RIVER", stationID="01790145-cd7e-4498-9240-f0fcd9061df2")
# View the retrieved water level data.
wa
```

```
[{'name': '01790145-cd7e-4498-9240-f0fcd9061df2',
  'description': '現場觀測',
  'properties': {'authority': '水利署水文技術組',
   'stationID': '01790145-cd7e-4498-9240-f0fcd9061df2',
   'stationCode': '2200H007',
   'stationName': '延平',
   'authority_type': '水利署'},
  'data': [{'name': '水位',
    'description': ' Datastream_id=016e5ea0-7c7f-41a2-af41-eabacdbb613f, Datastream_FullName=延平.水位, Datastream_Description=現場觀測, Datastream_Category_type=河川水位站, Datastream_Category=水文',
    'values': [{'timestamp': '2022-08-28T06:00:00.000Z', 'value': 157.41}]}],
  'location': {'latitude': 22.8983536,
   'longitude': 121.0845795,
   'address': None}}]
```

## Earthquake Data

### Get all project codes: `Quake().get_source()`

```python
# Use the get_source function of the Quake class to retrieve all quake-related project codes.
q = Quake().get_source()
# View the retrieved project codes.
q
```

```
['EARTHQUAKE:CWB+NCREE']
```

The valid project code for accessing earthquake data is:

- `EARTHQUAKE:CWB+NCREE`: This code is used for seismic monitoring stations jointly operated by the Central Weather Bureau and the National Center for Research on Earthquake Engineering.

### Get all stations: `Quake().get_station()`

```python
# Use the get_station function from the Quake class, specifying the data source "EARTHQUAKE:CWB+NCREE", to retrieve the earthquake station list.
q = Quake().get_station(src="EARTHQUAKE:CWB+NCREE")
# View the first two items from the station list.
q[0:2]
```

```
[{'name': '地震監測站-Jiqi-EGC',
  'description': '地震監測站-Jiqi-EGC',
  'properties': {'authority': '中央氣象局',
   'stationID': 'EGC',
   'deviceType': 'FBA',
   'stationName': 'Jiqi'},
  'location': {'latitude': 23.708, 'longitude': 121.548, 'address': None}},
 {'name': '地震監測站-Xilin-ESL',
  'description': '地震監測站-Xilin-ESL',
  'properties': {'authority': '中央氣象局',
   'stationID': 'ESL',
   'deviceType': 'FBA',
   'stationName': 'Xilin'},
  'location': {'latitude': 23.812, 'longitude': 121.442, 'address': None}}]
```

### Get data of a station: `Quake().get_data()`

```python
# Use the get_data function from the Quake class, specifying the data source "EARTHQUAKE:CWB+NCREE", to retrieve earthquake data.
q = Quake().get_data(src="EARTHQUAKE:CWB+NCREE")
# View the last record from the data.
q[-1]
```

```
{'name': '第2022083號地震',
 'description': '第2022083號地震',
 'properties': {'depth': 43.6, 'authority': '中央氣象局', 'magnitude': 5.4},
 'data': [
       {
       'name': '地震監測站-Jiqi-EGC',
       'description': '地震監測站-Jiqi-EGC',
       'timestamp': '2022-07-28T08:16:10.000Z',
       'value': 'http://140.110.19.16/STA_Earthquake_v2/v1.0/Observations(18815)'
       },{
       'name': '地震監測站-Taichung City-TCU',
       'description': '地震監測站-Taichung City-TCU',
       'timestamp': '2022-07-28T08:16:10.000Z',
       'value': 'http://140.110.19.16/STA_Earthquake_v2/v1.0/Observations(18816)'
       },
   ...
   ]
 'location': {'latitude': 22.98, 'longitude': 121.37, 'address': None}}
```

### Get data of an earthquake event`Quake().get_data()`

```python
# Use the get_data function from the Quake class, specifying the data source and event ID, to retrieve earthquake data for the specified event.
q = Quake().get_data(src="EARTHQUAKE:CWB+NCREE", eventID="2022083")
# View the retrieved earthquake data.
q
```

## Weather Data

### Get all project codes: `Weather().get_source()`

```python
# Use the get_source function of the Weather class to retrieve all weather-related project codes.
w = Weather().get_source()
# View the retrieved project codes.
w
```

```
['GENERAL:CWB',
 'GENERAL:CWB_IoT',
 'RAINFALL:CWB',
 'RAINFALL:WRA',
 'RAINFALL:WRA2',
 'RAINFALL:IA',
 'IMAGE:CWB']
```

The function returns project names based on the given input parameter:

- `GENERAL`: Retrieves names of projects related to weather stations.
- `RAINFALL`: Lists names of projects associated with rainfall monitoring stations.
- `IMAGE`: Provides names of projects involving radar integrated echo maps.
- (none): When no parameter is specified, it returns the names of all weather-related projects.

Valid project codes for weather data include:

- `GENERAL:CWB`: For standard weather stations operated by the Central Weather Bureau.
- `GENERAL:CWB_IoT`: Refers to automatic weather monitoring stations by the Central Weather Bureau.
- `RAINFALL:CWB`: Codes for rainfall monitoring stations managed by the Central Weather Bureau.
- `RAINFALL:WRA`: Represents rainfall monitoring stations run by the Water Resource Agency.
- `RAINFALL:WRA2`: For rainfall monitoring stations by the Water Resource Agency, in collaboration with county and city governments.
- `RAINFALL:IA`: Indicates rainfall monitoring stations managed by the Irrigation Agency.
- `IMAGE:CWB`: Used for radar integrated echo maps created by the Central Weather Bureau.

### Get all stations: `Weather().get_station()`

```python
# Use the get_station function from the Weather class, specifying the data source "RAINFALL:CWB", to retrieve the rainfall station list.
w = Weather().get_station(src="RAINFALL:CWB")
# View the retrieved station list.
w
```

```
[{'name': '雨量站-C1R120-上德文',
  'description': '雨量站-C1R120-上德文',
  'properties': {'city': '屏東縣',
   'township': '三地門鄉',
   'authority': '中央氣象局',
   'stationID': 'C1R120',
   'stationName': '上德文',
   'stationType': '局屬無人測站'},
  'location': {'latitude': 22.765, 'longitude': 120.6964, 'address': None}},
  ...
]
```

### Get data of a station: `Weather().get_data()`

```python
# Use the get_data function from the Weather class, specifying the data source and station ID, to retrieve rainfall data for the specified station.
w = Weather().get_data(src="RAINFALL:CWB", stationID="U2HA40")
# View the retrieved rainfall data.
w
```

```
[{'name': '雨量站-U2HA40-臺大內茅埔',
  'description': '雨量站-U2HA40-臺大內茅埔',
  'properties': {'city': '南投縣',
   'township': '信義鄉',
   'authority': '中央氣象局',
   'stationID': 'U2HA40',
   'stationName': '臺大內茅埔',
   'stationType': '中央氣象局'},
  'data': [{'name': 'HOUR_12',
    'description': '12小時累積雨量',
    'values': [{'timestamp': '2022-08-28T05:30:00.000Z', 'value': 0.0}]},
   {'name': 'MIN_10',
    'description': '10分鐘累積雨量',
    'values': [{'timestamp': '2022-08-28T05:30:00.000Z', 'value': 0.0}]},
   {'name': 'RAIN',
    'description': '60分鐘累積雨量',
    'values': [{'timestamp': '2022-08-28T05:30:00.000Z', 'value': 0.0}]},
   {'name': 'HOUR_6',
    'description': '6小時累積雨量',
    'values': [{'timestamp': '2022-08-28T05:30:00.000Z', 'value': 0.0}]},
   {'name': 'HOUR_3',
    'description': '3小時累積雨量',
    'values': [{'timestamp': '2022-08-28T05:30:00.000Z', 'value': 0.0}]},
   {'name': 'HOUR_24',
    'description': '24小時累積雨量',
    'values': [{'timestamp': '2022-08-28T05:30:00.000Z', 'value': 0.0}]},
   {'name': 'NOW',
    'description': '本日累積雨量',
    'values': [{'timestamp': '2022-08-28T05:30:00.000Z', 'value': 0.0}]},
   {'name': 'ELEV',
    'description': '高度',
    'values': [{'timestamp': '2022-08-28T05:30:00.000Z', 'value': 507.0}]}],
  'location': {'latitude': 23.6915, 'longitude': 120.843, 'address': None}}]
```

## CCTV Data

### Get all project codes: `CCTV().get_source()`

```python
# Use the get_source function of the CCTV class to retrieve all CCTV-related project codes.
cctv = CCTV().get_source()
# View the retrieved project codes.
cctv
```

```
['IMAGE:EPA', 'IMAGE:WRA', 'IMAGE:COA']
```

The valid project codes for accessing CCTV data are:

- `IMAGE:EPA`: This code is used for real-time images from air quality monitoring stations managed by the Environmental Protection Agency (EPA).
- `IMAGE:WRA`: Refers to images related to water conservancy and disaster prevention provided by the Water Resource Agency.
- `IMAGE:COA`: Represents real-time images from landslide observation stations operated by the Council of Agriculture.

### Get data of a station: `CCTV().get_data()`

```python
# Use the get_data function from the CCTV class, specifying the data source "IMAGE:EPA", to retrieve CCTV image data from the EPA.
cEPA = CCTV().get_data("IMAGE:EPA")
# View the third record from the data.
cEPA[2]
```

```
{
    'name': '環保署空品監測即時影像器-萬里',
    'description': '環保署-萬里-空品監測即時影像器',
    'properties': {
        'city': '新北市',
        'basin': '北部空品區',
        'authority': '環保署',
        'stationName': '萬里',
        'CCDIdentifier': '3'},
    'data': [
        {
        'name': '即時影像',
        'description': '環保署-萬里-空品監測即時影像器',
        'values': [
            {
            'timestamp': '2022-06-13T09:00:00.000Z',
            'value': 'https://airtw.epa.gov.tw/AirSitePic/20220613/003-202206131700.jpg'
            }
        ]
        }
    ],
    'location': {'latitude': 25.179667, 'longitude': 121.689881, 'address': None}
}
```

```python
# Use the get_data function from the CCTV class, specifying the data source "IMAGE:COA", to retrieve CCTV image data from the COA.
cCOA = CCTV().get_data("IMAGE:COA")
# View the first record from the data.
cCOA[0]
```

```
{'name': '行政院農委會土石流觀測站影像-大粗坑下游攝影機',
 'description': '行政院農委會-大粗坑下游攝影機-土石流觀測站影像',
 'properties': {'city': '新北市',
  'township': '瑞芳鎮',
  'StationId': '7',
  'authority': '行政院農委會',
  'stationName': '大粗坑下游攝影機',
  'CCDIdentifier': '2'},
 'data': [{'name': '即時影像',
   'description': '行政院農委會-大粗坑下游攝影機-土石流觀測站影像',
   'values': [{'timestamp': '2099-12-31T00:00:00.000Z',
     'value': 'http://dfm.swcb.gov.tw/debrisFinal/ShowCCDImg-LG.asp?StationID=7&CCDId=2'}]}],
 'location': {'latitude': 25.090878,
  'longitude': 121.837815,
  'address': '新北市瑞芳鎮弓橋里大粗坑'}}
```

## Disaster Alert and Notification Data

The Civil IoT Taiwan Data Service Platform offers an extensive array of disaster-related information, featuring over 58 different disaster alerts and more than 41 distinct disaster notifications. Various units handle specific types of disaster alerts, while the central disaster response center is tasked with issuing disaster notifications.

For a comprehensive list of project codes related to these services, you can refer to the [pyCIOT Package Document](https://hackmd.io/@cclljj/pyCIOT_doc).

### Get disaster alerts: `Disaster().get_alert()`

```python
# Use the get_alert function from the Disaster class, specifying the parameter as "5", to retrieve disaster alert information.
d = Disaster().get_alert("5")
# View the retrieved disaster alert information.
d
```

```
{'id': 'https://alerts.ncdr.nat.gov.tw/Json.aspx',
 'title': 'NCDR_CAP-即時防災資訊(Json)',
 'updated': '2021-10-12T08:32:00+08:00',
 'author': {'name': 'NCDR'},
 'link': {'@rel': 'self',
  '@href': 'https://alerts.ncdr.nat.gov.tw/JSONAtomFeed.ashx?AlertType=5'},
 'entry': [
       {
        'id': 'CWB-Weather_typhoon-warning_202110102030001',
        'title': '颱風',
        'updated': '2021-10-10T20:30:05+08:00',
        'author': {'name': '中央氣象局'},
        'link': {'@rel': 'alternate',
        '@href': 'https://b-alertsline.cdn.hinet.net/Capstorage/CWB/2021/Typhoon_warnings/fifows_typhoon-warning_202110102030.cap'},
        'summary': {
            '@type': 'html',
            '#text': '1SEA18KOMPASU圓規2021-10-10T12:00:00+00:0018.20,126.702330992150輕度颱風TROPICAL STORM2021-10-11T12:00:00+00:0019.20,121.902835980220輕度颱風 圓規（國際命名 KOMPASU）10日20時的中心位置在北緯 18.2 度，東經 126.7 度，即在鵝鑾鼻的東南東方約 730 公里之海面上。中心氣壓 992 百帕，近中心最大風速每秒 23 公尺（約每小時 83 公里），相當於 9 級風，瞬間最大陣風每秒 30 公尺（約每小時 108 公里），相當於 11 級風，七級風暴風半徑 150 公里，十級風暴風半徑 – 公里。以每小時21公里速度，向西進行，預測11日20時的中心位置在北緯 19.2 度，東經 121.9 度，即在鵝鑾鼻的南南東方約 320 公里之海面上。根據最新資料顯示，第18號颱風中心目前在鵝鑾鼻東南東方海面，向西移動，其暴風圈正逐漸向巴士海峽接近，對巴士海峽將構成威脅。巴士海峽航行及作業船隻應嚴加戒備。第18號颱風外圍環流影響，易有短延時強降雨，今(10日)晚至明(11)日基隆北海岸、宜蘭地區及新北山區有局部大雨發生的機率，請注意。＊第18號颱風及其外圍環流影響，今(10日)晚至明(11)日巴士海峽及臺灣附近各海面風浪逐漸增大，基隆北海岸、東半部（含蘭嶼、綠島）、西南部、恆春半島沿海易有長浪發生，前往海邊活動請特別注意安全。＊第18號颱風外圍環流影響，今(10日)晚至明(11)日臺南以北、東半部(含蘭嶼、綠島)、恆春半島、澎湖、金門、馬祖沿海及空曠地區將有9至12級強陣風，內陸地區及其他沿海空曠地區亦有較強陣風，請注意。＊第18號颱風外圍環流沉降影響，明(11)日南投、彰化至臺南及金門地區高溫炎熱，局部地區有36度以上高溫發生的機率，請注意。＊本警報單之颱風半徑為平均半徑，第18號颱風之7級風暴風半徑西南象限較小約60公里，其他象限約180公里，平均半徑約為150公里。'},
       'category': {'@term': '颱風'}
       },{
       ...
       }, ...
   ]
}
```

### Get historical data of disaster notifications: `Disaster().get_notice()`

```python
# Use the get_notice function from the Disaster class, specifying the parameter as "ERA2_F1", to retrieve the transportation disaster report (road and bridge section).
d = Disaster().get_notice("ERA2_F1") # 交通災情通報表（道路、橋梁部分）
# View the retrieved transportation disaster report information.
d
```

```
"maincmt":{
      "prj_no":"專案代號",
      "org_name":"填報機關",
      "rpt_approval":"核定人",
      "rpt_phone":"聯絡電話",
      "rpt_mobile_phone":"行動電話",
      "rpt_no":"通報別",
      "rpt_user":"通報人",
      "rpt_time":"通報時間"
   },
   "main":{
      "prj_no":"2014224301",
      "org_name":"交通部公路總局",
      ...
   },
   "detailcmt":{
      "trfstatus":"狀態",
      ...
   },
   ...
}
```

## References

- Python pyCIOT pypi package ([https://pypi.org/project/pyCIOT/](https://pypi.org/project/pyCIOT/))
- Python pyCIOT Document ([https://hackmd.io/@cclljj/pyCIOT_doc](https://hackmd.io/@cclljj/pyCIOT_doc))
