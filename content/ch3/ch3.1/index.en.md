---
title : "3.1. Basic Data Access Methods"
weight : 10
socialshare: true
description : "We introduce how to obtain water, air, earthquake, and disaster data in the Civil IoT Taiwan Data Service Platform, including the latest sensing data for a single site, a list of all sites, and the latest current sensing data for all sites."
tags: ["Python", "API", "Water", "Air", "Quake", "Disaster"]
levels: ["beginner"]
author: ["Sky Hung"]
---

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/drive/1DNyxdtg4EBs1X1ulNJvsMLBeKEhXnwgP?usp=sharing)

{{< toc >}}


This article describes how to use the pyCIOT package and basic access to air, water, earthquake, weather, CCTV, and disaster warning data of the Civil IoT Taiwan Data Service Platform. For different types of data, we describe how to get the latest sensing data for a single site, get a list of all sites, and get the latest current sensing data for all sites.

This article requires the reader to have basic terminal operation ability and have been exposed to the basic syntax of Python programming.

## What is pyCIOT?

The Civil IoT Taiwan project provides a wide variety of data, and different data often have different data formats and access methods. Even if the data is under an open license, organizing the data can be cumbersome due to the different ways in which it is downloaded and processed. To solve this problem, we developed the pyCIOT suite to collect all public data on people's livelihood publicly released by the government, and strive to lower the threshold for obtaining public data and reduce the cost of data processing.

## pyCIOT Basics

### Installation

We first download and install the pyCIOT suite using pip. pip is a package management system written in Python for installing and managing Python packages. The pyCIOT suite used this time is managed by the Python package index (pypi). We can use this command in the terminal to download the pyCIOT library locally, or download other required packages together:

```powershell
!pip install pyCIOT
```

### Import Package

To use this package, just enter the import syntax and import `pyCIOT.data`:

```python
# Import pyCIOT.data
from pyCIOT.data import *
```

Depending on how the package is imported, the method is called differently. If you use the `from ... import ...` syntax, you don't need to add the prefix when calling a method; but if you use `import ...` , you need to add it every time you call a method in its package. If you use `import ... as ...` , you can call a method based on the custom prefix after `as` .

```python

import pyCIOT.data 
a = pyCIOT.data.Air().get_source()

import pyCIOT.data as CIoT
a = CIoT.Air().get_source()

from pyCIOT.data import *
a = Air().get_source()
```

## Data Access

The data of the Civil IoT Taiwan project can be obtained through the following methods, including air, water, earthquake, weather, CCTV, etc.:

- `.get_source()` : Return all project codes in Civil IoT Taiwan Data Service Platform in array format according to the data type.
- `.get_station(src='')` : Return basic information of all station data in array format. The `src` parameter is optional to specify the project code to be queried.
- `.get_data(src='', stationIds=[])` : Return basic information of all stations and their latest measurement result in array format. The `src` parameter is optional to specify the project code to be queried, and the `stationIds` parameter is optional to specify the device ID to be queried.

The following applies to disaster notification data.

- `.get_alert()` : Return the alert data, along with the information of the event, in JSON format.
- `.get_notice()` : Return the notification data, along with the informaiton of the event, in JSON format.

Note that since this package is still under revision, if it is inconsistent with the content of the [pyCIOT Package Document](https://hackmd.io/JdFywArGS9uxcSRTyYDtBg), it shall prevail.

## Air Quality Data

### Get all project codes: `Air().get_source()`

```python
a = Air().get_source()
print(a)
```

```
['OBS:EPA', 'OBS:EPA_IoT', 'OBS:AS_IoT', 'OBS:MOST_IoT', 'OBS:NCNU_IoT']
```

The followings are valid project codes for air quality data:

- `OBS:EPA`: national level monitoring stations by EPA
- `OBS:EPA_IoT`: low-cost air quality stations by EPA
- `OBS:AS_IoT`: micro air quality stations by Academia Sinica
- `OBS:MOST_IoT`: low-cost air quality stations by MOST
- `OBS:NCNU_IoT`: low-cost air quality stations by National Chi Nan University

### Get all stations: `Air().get_station()`

```python
b = Air().get_station(src="OBS:EPA_IoT")
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
f = Air().get_data(src="OBS:EPA_IoT", stationIds=["11613429495"])
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
print(f[0]['description'])
for f_data in f[0]['data']:
  if f_data['description'] == '溫度':
    print(f_data['description'], ': ', f_data['values'][0]['value'], ' (', f_data['values'][0]['timestamp'], ')', sep='')
```

```
智慧城鄉空品微型感測器-11613429495
溫度: 30.6 (2022-08-28T06:22:08.000Z)
```

## Water Resource Data

### Get all project codes: `Water().get_source()`

Return the project names based on the input parameter:

- `water_level_station`: Return the names of water level related projects (currently valid codes are `WRA`, `WRA2` and `IA`)
- `gate`: Return the names of water gate related projects (currently valid codes are `WRA`, `WRA2` and `IA`)
- `pumping_station`: Return the names of pumping station related projects (currently valid codes are `WRA2` and `TPE`)
- `sensor`: Return the names of water sensor related projects (currently valid codes are `WRA`, `WRA2`, `IA` and `CPAMI`)
- `` (none): Return the names of all water related projects

```python
wa = Water().get_source()
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

The followings are valid project codes for water resource data:

- `WRA`: Water Resource Agency
- `WRA2`: Water Resource Agency（co-constructed with county and city governments）
- `IA`: Irrigation Agency
- `CPAMI`: Construction and Planning Agency
- `TPE`: Taipei City

### Get all stations: `Water().get_station()`

```python
wa = Water().get_station(src="WATER_LEVEL:WRA_RIVER")
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
wa = Water().get_data(src="WATER_LEVEL:WRA_RIVER", stationID="01790145-cd7e-4498-9240-f0fcd9061df2")
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
q = Quake().get_source()
q
```

```
['EARTHQUAKE:CWB+NCREE']
```

The followings are valid project codes for earthquake data:

- `EARTHQUAKE:CWB+NCREE`: seismic monitoring stations by Central Weather Bureau and National Center for Research on Earthquake Engineering

### Get all stations: `Quake().get_station()`

```python
q = Quake().get_station(src="EARTHQUAKE:CWB+NCREE")
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
q = Quake().get_data(src="EARTHQUAKE:CWB+NCREE")
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
q = Quake().get_data(src="EARTHQUAKE:CWB+NCREE", eventID="2022083")
q
```

## Weather Data

### Get all project codes: `Weather().get_source()`

```python
w = Weather().get_source()
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

Return the project names based on the input parameter:

- `GENERAL`: Return the names of weather station related projects
- `RAINFALL`: Return the names of rainfall monitoring station related projects
- `IMAGE`: Return the names of radar integrated echo map related projects
- (none): Return the names of all weather related projects

The followings are valid project codes for weather data:

- `GENERAL:CWB`: standard weather stations by Central Weather Bureau
- `GENERAL:CWB_IoT`: automatic weather monitoring stations by Central Weather Bureau
- `RAINFALL:CWB`: rainfall monitoring stations by Central Weather Bureau
- `RAINFALL:WRA`: rainfall monitoring stations by Water Resource Agency
- `RAINFALL:WRA2`: rainfall monitoring stations by Water Resource Agency（co-constructed with county and city governments）
- `RAINFALL:IA`: rainfall monitoring stations by Irrigation Agency
- `IMAGE:CWB`: radar integrated echo map by Central Weather Bureau

### Get all stations: `Weather().get_station()`

```python
w = Weather().get_station(src="RAINFALL:CWB")
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
w = Weather().get_data(src="RAINFALL:CWB", stationID="U2HA40")
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
cctv = CCTV().get_source()
cctv
```

```
['IMAGE:EPA', 'IMAGE:WRA', 'IMAGE:COA']
```

The followings are valid project codes for CCTV data:

- IMAGE:EPA: realtime images by EPA air quality monitoring stations
- IMAGE:WRA: images for water conservancy and disaster prevention by Water Resource Agency
- IMAGE:COA: realtime images of landslide observation stations by Council of Agriculture

### Get data of a station: `CCTV().get_data()`

```python
cEPA = CCTV().get_data("IMAGE:EPA")
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
cCOA = CCTV().get_data("IMAGE:COA")
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

The Civil IoT Taiwan Data Service Platform provides more than 58 disaster alerts, with a total of more than 41 disaster notifications. Different units are responsible for different disaster alerts, and the central disaster response center is responsible for disaster notification.

The complete list of project codes is available in the [pyCIOT Package Document](https://hackmd.io/@cclljj/pyCIOT_doc)。

### Get disaster alerts: `Disaster().get_alert()`

```python
d = Disaster().get_alert("5")
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
d = Disaster().get_notice("ERA2_F1") # 交通災情通報表（道路、橋梁部分）
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
