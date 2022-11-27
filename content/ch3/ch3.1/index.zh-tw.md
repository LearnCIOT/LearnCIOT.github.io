---
title : "3.1. 基本資料存取方法"
weight : 10
socialshare: true
description : "我們介紹如何取用民生公共物聯網開放資料平台中，有關水、空、地、災不同面向單一測站的最新一筆感測資料，如何獲取所有測站的列表，以及如何獲取所有測站當下最新的一筆感測資料。"
tags: ["Python", "API", "水", "空", "地", "災"]
levels: ["beginner"]
authors: ["洪軾凱"]
---

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/drive/1DNyxdtg4EBs1X1ulNJvsMLBeKEhXnwgP?usp=sharing)

{{< toc >}}


本章節涵蓋 pyCIOT API 使用方法，以及空氣、水源、地震、天氣、影像資料、以及災難警示資料的基本存取方法。包含單一測站的最新一筆感測資料、獲取所有測站的列表，以及如何獲取所有測站當下最新的一筆感測資料。

本章節需要讀者有基本的終端機操作能力及接觸過 Python 程式基本語法。

## pyCIOT API 是什麼

政府開放資料現已有非常多種類的資料及窗口讓我們查詢，而不同的窗口有各自不同的資料存取方式。即便這些資料逐漸使用開放授權，但在進行資料搜集時，因為下載取得資料的方式各有差異，若要將這些資料整理會變得極為麻煩。本函式庫為了解決窗口不一的困難，將所有有提供 API 的政府民生開放資料的開放搜集至此，嘗試讓開放資料獲取門檻降低，降低自動化及二手資料處理成本。****

## pyCIOT API 使用方法

### 下載函式庫

為了使用 [pyCIOT API](https://test.pypi.org/project/pyCIOT/) 服務，我們需要先將下載此服務的函式庫。pip 是一個以 Python 寫成的軟體包管理系統，用來安裝和管理 Python 軟體包。而這次使用的 pyCIOT 函式庫則是交由 Python Package Index (pypi) 管理，而我們可以在終端機上用這行指令將 pyCIOT 函式庫下載到本地，同時也會將其他必須的套件一起下載：

```powershell
!pip install pyCIOT
```

### 使用函式庫

欲使用本函式庫，僅需輸入匯入語法匯入 `pyCIOT.data` 即可：

```python
# Import pyCIOT.data
from pyCIOT.data import *
```

根據匯入函式庫的方式不同，呼叫函式的方法也不一樣。若是使用 `from ... import ...` 的語法，呼叫函式時不需加上前綴；但若是使用 `import ...` 的話，則是要在每次呼叫其函式庫下的函式時加上前綴，而使用 `import ... as ...` 則是可以根據在 `as` 後自定義前綴方便撰寫程式。

```python
# 引入函式的三種方式

import pyCIOT.data 
a = pyCIOT.data.Air().get_source()
#   ~~~~~~~~~~~~ 加虛線上的文字

import pyCIOT.data as CIoT
a = CIoT.Air().get_source()
#   ~~~~~ 可自行定義前綴

from pyCIOT.data import *
a = Air().get_source()
# 在匯入大量函式庫時盡量少用，避免函式庫名稱衝撞的問題
```

## pyCIoT API 獲取資料方式

大部分的 API 資料格式都能夠透過下列方式獲取，包含空氣、水源、地震、天氣、影像資料等：

- `.get_source()` 獲取專案代碼回傳在民生公共物聯網開放資料平台中有的專案代碼，返回格式為 array。
- `.get_station(src='')` 獲取回傳各測站資料的基本資訊及位置，返回格式為 array；可選擇性帶入 `src` 參數，指名所要查詢的專案代碼。
- `.get_data(src='', stationID='')` 獲取所有測站列表返回格式為 array，回傳各測站的基本資訊，位置及感測資料；可選擇性帶入 `src` 參數，指名所要查詢的專案代碼，或選擇性帶入 `stationID` 參數，指名所要查詢的機器代碼。

災害警示資料則適用於以下：

- `.get_alert()` 獲取警示資料返回格式為 json，包括該事件的相關示警資訊。
- `.get_notice()` 獲取通報歷史資料返回格式為 json，包括該事件的相關通報資訊。

若與 [pyCIOT Package Document](https://hackmd.io/JdFywArGS9uxcSRTyYDtBg) 內容相悖，以其為準。

## 空氣資料存取

### 獲取專案代碼 `Air().get_source()`

```python
# 回傳所有空氣相關的專案代碼
a = Air().get_source()
print(a)
```

```
['OBS:EPA', 'OBS:EPA_IoT', 'OBS:AS_IoT', 'OBS:MOST_IoT', 'OBS:NCNU_IoT']
```

專案代碼轉換列表

- `OBS:EPA`: 環保署國家空品測站
- `OBS:EPA_IoT`: 環保署智慧城鄉空品微型感測器
- `OBS:AS_IoT`: 中研院校園空品微型感測器
- `OBS:MOST_IoT`: 科技部智慧園區空品測站
- `OBS:NCNU_IoT`: 暨南大學在地空品微型感測器

### 獲取所有測站列表 `Air().get_station()`

```python
# 獲取環保署智慧城鄉空品微型感測的檢測站列表
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

### 獲取測站資料 `Air().get_data()`

```python
f = Air().get_data(src="OBS:EPA_IoT", stationID="11613429495")
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

## 水源資料存取

### 獲取專案代碼 Water().get_source()

根據引數不同會回傳不同種類的專案名稱：

- `water_level_station`: 回傳水位站的專案名稱（目前合法的代碼僅有 `WRA`, `WRA2` 和 `IA`)。
- `gate`: 回傳閘門的專案名稱（目前合法的代碼僅有 `WRA`, `WRA2` 和 `IA`)。
- `pumping_station`: 回傳抽水站的專案名稱（目前合法的代碼有 `WRA2` 和 `TPE`)。
- `sensor`: 回傳各式感測器的專案名稱（目前合法的代碼有 `WRA`, `WRA2`, `IA` 和 `CPAMI`)。
- `` (無輸入): 回傳所有水資源相關的專案名稱。

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

專案代碼轉換列表：

- `WRA`: 水利署
- `WRA2`: 水利署（與縣市政府合建）
- `IA`: 農田水利署
- `CPAMI`: 營建署
- `TPE`: 臺北市

### 獲取所有測站列表 `Water().get_station()`

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

### 獲取測站資料 `Water().get_data()`

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

## 地震資料存取

### 獲取專案代碼 `Quake().get_source()`

```python
q = Quake().get_source()
q
```

```
['EARTHQUAKE:CWB+NCREE']
```

專案代碼轉換列表：

- `EARTHQUAKE:CWB+NCREE`: 中央氣象局與國震中心地震監測站（因資料儲存方式不同，CWB 及 NCREE 的測站資料無法分別查詢，因此在每個事件中，會回傳來源中所有測站的監測資料）

### 獲取地震監測站列表 `Quake().get_station()`

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

### 獲取地震資料 `Quake().get_data()`

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

### 獲取單一地震資料`Quake().get_data()`

```python
q = Quake().get_data(src="EARTHQUAKE:CWB+NCREE", eventID="2022083")
q
# 獲得資料之格式和上述相同
```

## 天氣資料存取

### 獲取專案代碼 `Weather().get_source()`

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

根據引數不同會回傳不同種類的專案名稱：

- `GENERAL`: 回傳氣象站的專案代碼
- `RAINFALL`: 回傳雨量站/雨量感測器的專案代碼
- `IMAGE`: 回傳雷達回波圖的專案代碼
- `` (無輸入): 回傳所有氣象相關的專案代碼。

專案代碼轉換列表：

- `GENERAL:CWB`: 中央氣象局局屬氣象站
- `GENERAL:CWB_IoT`: 中央氣象局自動氣象站
- `RAINFALL:CWB`:中央氣象局雨量站
- `RAINFALL:WRA`: 水利署雨量感測器
- `RAINFALL:WRA2`: 水利署（與縣市政府合建）雨量感測器
- `RAINFALL:IA`: 農田水利署雨量感測器
- `IMAGE:CWB`: 中央氣象局雷達整合回波圖

### 獲取所有測站列表 `Weather().get_station()`

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

### 獲得測站資料 `Weather().get_data()`

```python
# 南投縣 雨量站-U2HA40-臺大內茅埔 的雨量資料
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

## 影像資料存取

### 獲取專案代碼 `CCTV().get_source()`

```python
cctv = CCTV().get_source()
cctv
```

```
['IMAGE:EPA', 'IMAGE:WRA', 'IMAGE:COA']
```

專案代碼轉換列表：

- IMAGE:EPA: 環保署空品監測即時影像器
- IMAGE:WRA: 水利署水利防災用影像器
- IMAGE:COA: 行政院農委會土石流觀測站影像

### 獲取影像資料 `CCTV().get_data()`

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

## 災難警示資料存取

民生公共物聯網資料平台上提供的災情示警有 58 個項目以上，而災情通報共計有 41 個項目以上；不同災情示警個由不同單位負責，而災情通報則為中央災害應變中心負責。

因專案代碼轉換列表過長，可參考 [pyCIOT Package Document](https://hackmd.io/@cclljj/pyCIOT_doc)。

### 取得災情示警 `Disaster().get_alert()`

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

### 獲取災情通報歷史資料 `Disaster().get_notice()`

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

## 參考資料

- Python pyCIOT pypi package ([https://pypi.org/project/pyCIOT/](https://pypi.org/project/pyCIOT/))
- Python pyCIOT Document ([https://hackmd.io/@cclljj/pyCIOT_doc](https://hackmd.io/@cclljj/pyCIOT_doc))

