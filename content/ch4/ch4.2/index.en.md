---
title : "4.2. Time Series Data Forecast"
weight : 20
socialshare: true
description : "We use the sensing data of the Civil IoT Taiwan Data Service Platform and apply existing Python data science packages (such as scikit-learn, Kats, etc.) to compare the prediction results of different data prediction models. We use graphics to present the data and discuss the significance of the data prediction of the dataset at different time resolutions in the real field, as well as possible derived applications."
tags: ["Python", "Water", "Air" ]
levels: ["intermediate" ]
authors: ["Yu-Chi Peng"]
---


[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/drive/1lAqt91m88srcPWLSabbN9Tdbxw1y2I2n?usp=sharing)

{{< toc >}}

In the last chapter, we explored different ways to handle and understand data that changes over time, like creating visual representations and breaking down the data into smaller parts. This chapter will delve deeper into these data sets, focusing on identifying their unique features. We'll also use a range of predictive models to uncover patterns within the data and make forecasts about future trends.

## Goal
1. **Evaluating Stationarity in Time Series Data**: We aim to assess whether the time series data remains consistent over time or shows significant variations.

2. **Comparing Various Forecasting Models**: We'll compare different models used for forecasting to understand which ones work best for our specific data set.

3. **Practical Application of Time Series Data Forecasting**: Finally, we'll put our learning into practice by actually forecasting future trends based on the time series data we have.

## Setting Up the Environment

For this tutorial, we'll primarily use a set of Python packages: pandas, matplotlib, numpy, statsmodels, and warnings. Good news! If you're using Google Colab as your development platform, these packages are already installed and ready to use. 

However, we'll also need two additional packages that aren't included in Colab's default setup: **kats** and **pmdarima**. To get these up and running, we'll need to install them manually. This can be done with the following commands:
```python
!pip install --upgrade pip
!pip install kats==0.1 ax-platform==0.2.3 statsmodels==0.12.2
!pip install pmdarima
```

Once the installation of kats and pmdarima is complete, we're ready to import all the required packages into our working environment. This step is crucial for ensuring that we have access to the various functions and tools each package offers for our analysis.

```python
import warnings
import numpy as np
import pandas as pd
import pmdarima as pm
import statsmodels.api as sm
import matplotlib.pyplot as plt
import os, zipfile

from dateutil import parser as datetime_parser
from statsmodels.tsa.arima.model import ARIMA
from statsmodels.tsa.statespace.sarimax import SARIMAX
from statsmodels.tsa.stattools import adfuller, kpss
from kats.consts import TimeSeriesData, TimeSeriesIterator
from kats.detectors.outlier import OutlierDetector
from kats.models.prophet import ProphetModel, ProphetParams
from kats.models.lstm import LSTMModel, LSTMParams
from kats.models.holtwinters import HoltWintersParams, HoltWintersModel
```

## Data Access
In this study, our primary focus is on analyzing and processing time series data. We'll be using real-world datasets from the Civil IoT Taiwan Data Service Platform, specifically focusing on air quality, water level, and meteorological (weather-related) data.

For our demonstration of data access and in-depth analysis, we will particularly concentrate on the air quality data. This data, like the other types, is collected over a long period by various monitoring stations. In our dataframes (the tables where we organize this data), the time at which each observation is recorded is labeled as `timestamp`.

Since each `timestamp` is unique, it serves a dual purpose. Not only does it indicate when the data was recorded, but it also acts as the index for our dataframe, helping us organize and access the data more efficiently.

### Air Quality Data
For this project, we're focusing on long-term historical data. Instead of using the direct data access methods provided by the pyCIOT package, we'll take a different approach. We'll download the data archive titled “Academia Sinica - Micro Air Quality Sensors” from the Civil IoT Taiwan Data Service Platform's historical database. This archive will be stored in a folder named Air.

The data we download will be in a zip-compressed file format. To work with this data, our first step is to extract or decompress it. This process will produce a series of daily files, which are also compressed. We'll then need to decompress each of these daily files and save them in another folder, which we'll call CSV_Air.

```python
!mkdir Air CSV_Air
!wget -O Air/2018.zip -q "https://history.colife.org.tw/?r=/download&path=L%2Bepuuawo%2BWTgeizqi%2FkuK3noJTpmaJf5qCh5ZyS56m65ZOB5b6u5Z6L5oSf5ris5ZmoLzIwMTguemlw"
!wget -O Air/2019.zip -q "https://history.colife.org.tw/?r=/download&path=L%2Bepuuawo%2BWTgeizqi%2FkuK3noJTpmaJf5qCh5ZyS56m65ZOB5b6u5Z6L5oSf5ris5ZmoLzIwMTkuemlw"
!wget -O Air/2020.zip -q "https://history.colife.org.tw/?r=/download&path=L%2Bepuuawo%2BWTgeizqi%2FkuK3noJTpmaJf5qCh5ZyS56m65ZOB5b6u5Z6L5oSf5ris5ZmoLzIwMjAuemlw"
!wget -O Air/2021.zip -q "https://history.colife.org.tw/?r=/download&path=L%2Bepuuawo%2BWTgeizqi%2FkuK3noJTpmaJf5qCh5ZyS56m65ZOB5b6u5Z6L5oSf5ris5ZmoLzIwMjEuemlw"

folder = 'Air'
extension_zip = '.zip'
extension_csv = '.csv'

for subfolder in os.listdir(folder):
    path = f'{folder}/{subfolder}'
    if path.endswith(extension_zip):
      print(path)
      zip_ref = zipfile.ZipFile(path)
      zip_ref.extractall(folder)
      zip_ref.close()

for subfolder in os.listdir(folder):
    path = f'{folder}/{subfolder}'
    if os.path.isdir(path):
        for item in os.listdir(path):
            if item.endswith(extension_zip):
                file_name = f'{path}/{item}'
                print(file_name)
                zip_ref = zipfile.ZipFile(file_name)
                zip_ref.extractall(path)
                zip_ref.close()

        for item in os.listdir(path):
          path2 = f'{path}/{item}'
          if os.path.isdir(path2):
            for it in os.listdir(path2):
              if it.endswith(extension_zip):
                file_name = f'{path2}/{it}'
                print(file_name)
                zip_ref = zipfile.ZipFile(file_name)
                zip_ref.extractall('CSV_Air') # decide path
                zip_ref.close()
          elif item.endswith(extension_csv):
            os.rename(path2, f'CSV_Air/{item}')
```

Now that we have decompressed our data into the `CSV_Air` folder, it contains daily sensor data in CSV (Comma-Separated Values) format. Our next step is to focus on data from a specific monitoring station. Let's say we're interested in the station with the code `74DA38C7D2AC`.

Here's how we'll proceed:

1. **Reading the Data**: We'll go through each CSV file in the `CSV_Air` folder. Each file represents a day's worth of data from all the sensors.

2. **Filtering the Data**: While reading each file, we'll extract only the data that corresponds to our station of interest, `74DA38C7D2AC`.

3. **Creating a DataFrame**: We'll compile this filtered data into a DataFrame named `air`. This DataFrame will consist of all the relevant data from our chosen station, organized for easy analysis.

4. **Clean Up**: After we've successfully created the `air` DataFrame with all the necessary data, we'll delete all the downloaded files and any data generated during the decompression process. This step is important to free up storage space in the cloud environment, especially when working with large datasets.

By following these steps, we ensure that we have a focused dataset that is relevant to our analysis, while also maintaining an efficient use of our storage resources.

```python
folder = 'CSV_Air'
extension_csv = '.csv'
id = '74DA38C7D2AC'

air = pd.DataFrame()
for item in os.listdir(folder):
  file_name = f'{folder}/{item}'
  df = pd.read_csv(file_name)
  if 'pm25' in list(df.columns):
    df.rename({'pm25':'PM25'}, axis=1, inplace=True)
  filtered = df.query(f'device_id==@id')
  air = pd.concat([air, filtered], ignore_index=True)
air.dropna(subset=['timestamp'], inplace=True)

for i, row in air.iterrows():
  aware = datetime_parser.parse(str(row['timestamp']))
  naive = aware.replace(tzinfo=None)
  air.at[i, 'timestamp'] = naive
air.set_index('timestamp', inplace=True)

!rm -rf Air CSV_Air
```

The last step in preparing our data for analysis involves reorganizing it to ensure it's in the most useful format. Here's what we'll do:

1. **Rearranging the Data**: We'll adjust the data within our `air` DataFrame so that it's organized in a way that best suits our analysis needs, that is rearranging the Data in the site.

2. **Removing Unnecessary Fields**: To streamline our analysis, we'll identify and remove any fields (columns) in the data that aren't necessary for our study. This makes the data cleaner and focuses our attention only on the relevant information.

3. **Sorting by Time**: Since we're dealing with time series data, it's crucial to have our data sorted chronologically. We'll arrange the data in the `air` DataFrame in ascending order based on the `timestamp` field. This ensures that our analysis follows the natural order of time and provides accurate insights into trends and patterns.

By completing these steps, our data is now well-organized, focused, and ready for in-depth time series analysis.

```python
air.drop(columns=['device_id', 'SiteName'], inplace=True)
air.sort_values(by='timestamp', inplace=True)
air.info()
print(air.info())
```

```
<class 'pandas.core.frame.DataFrame'>
DatetimeIndex: 195305 entries, 2018-08-01 00:00:05 to 2021-12-31 23:54:46
Data columns (total 1 columns):
 \#   Column  Non-Null Count   Dtype 
---  ------  --------------   ----- 
 0   PM25    195305 non-null  object
dtypes: object(1)
memory usage: 3.0+ MB
                     PM25
timestamp                
2018-08-01 00:00:05  20.0
2018-08-01 00:30:18  17.0
2018-08-01 01:12:34  18.0
2018-08-01 01:18:36  21.0
2018-08-01 01:30:44  22.0
```

### Water Level Data
Similar to how we handled the air quality data, we will now focus on obtaining and preparing long-term historical data for groundwater levels. Here's the process we'll follow, paralleling our approach with the air quality data:

1. **Data Download**: Instead of using the `pyCIOT` suite's direct data access methods, we will download the data archive titled “Water Resources Agency - Groundwater Level Station” from the historical database of the Civil IoT Taiwan Data Service Platform. This archive will be stored in a folder named `Water`.

2. **Decompressing the Archive**: The data from the Civil IoT Taiwan Data Service Platform is provided in a zip compressed file format. Our first task will be to decompress this main archive file. This action will reveal several smaller compressed files, each representing data for a single day.

3. **Decompressing Daily Files**: We'll then proceed to decompress each of these daily compressed files. The decompressed files, which will be in CSV format, will be stored in a new folder called `CSV_Water`.

By following these steps, we'll have all the necessary groundwater level data organized and ready for analysis, just like we did with the air quality data. The data in `CSV_Water` will then be primed for further processing and analysis as per our project's requirements.

```python
!mkdir Water CSV_Water
!wget -O Water/2018.zip "https://history.colife.org.tw/?r=/download&path=L%2BawtOizh%2Ba6kC%2FmsLTliKnnvbJf5rKz5bed5rC05L2N56uZLzIwMTguemlw"
!wget -O Water/2019.zip "https://history.colife.org.tw/?r=/download&path=L%2BawtOizh%2Ba6kC%2FmsLTliKnnvbJf5rKz5bed5rC05L2N56uZLzIwMTkuemlw"
!wget -O Water/2020.zip "https://history.colife.org.tw/?r=/download&path=L%2BawtOizh%2Ba6kC%2FmsLTliKnnvbJf5rKz5bed5rC05L2N56uZLzIwMjAuemlw"
!wget -O Water/2021.zip "https://history.colife.org.tw/?r=/download&path=L%2BawtOizh%2Ba6kC%2FmsLTliKnnvbJf5rKz5bed5rC05L2N56uZLzIwMjEuemlw"

folder = 'Water'
extension_zip = '.zip'
extension_csv = '.csv'

for subfolder in os.listdir(folder):
    path = f'{folder}/{subfolder}'
    if path.endswith(extension_zip):
      print(path)
      zip_ref = zipfile.ZipFile(path)
      zip_ref.extractall(folder)
      zip_ref.close()
for subfolder in os.listdir(folder):
    path = f'{folder}/{subfolder}'
    if os.path.isdir(path):
        for item in os.listdir(path):
            if item.endswith(extension_zip):
                file_name = f'{path}/{item}'
                print(file_name)
                zip_ref = zipfile.ZipFile(file_name)
                zip_ref.extractall(path)
                zip_ref.close()

        for item in os.listdir(path):
          path2 = f'{path}/{item}'
          if os.path.isdir(path2):
            for it in os.listdir(path2):
              if it.endswith(extension_zip) and not it.endswith('QC.zip'):
                file_name = f'{path2}/{it}'
                print(file_name)
                zip_ref = zipfile.ZipFile(file_name)
                zip_ref.extractall('CSV_Water') # decide path
                zip_ref.close()
          elif item.endswith(extension_csv):
            os.rename(path2, f'CSV_Water/{item}')
```
With the groundwater level data now available in the `CSV_Water` folder in CSV format, our next steps will mirror what we did with the air quality data:

1. **Reading the Data**: We'll sequentially open each CSV file in the `CSV_Water` folder. Each file contains a day's worth of data from various monitoring stations.

2. **Filtering by Station**: As we read through each file, we'll specifically extract data from the station with the code `338c9c1c-57d8-41d7-9af2-731fb86e632c`. This targeted approach ensures we only gather data relevant to our study.

3. **Creating the `water` DataFrame**: All the data from the specified station will be compiled into a DataFrame named `water`. This DataFrame will serve as the central repository for all the groundwater level data we need for our analysis.

4. **Cleanup**: After successfully gathering and organizing the data in the `water` DataFrame, we'll delete all the original downloaded files and any intermediate data created during the decompression process. This step is essential for managing storage space efficiently, especially when working in a cloud environment.

By completing these steps, we ensure a streamlined and focused dataset for our groundwater level analysis, maintaining an efficient and organized workflow.

```python
folder = 'CSV_Water'
extension_csv = '.csv'
id = '338c9c1c-57d8-41d7-9af2-731fb86e632c'

water = pd.DataFrame()
for item in os.listdir(folder):
  file_name = f'{folder}/{item}'
  df = pd.read_csv(file_name)
  if 'pm25' in list(df.columns):
    df.rename({'pm25':'PM25'}, axis=1, inplace=True)
  filtered = df.query(f'station_id==@id')
  water = pd.concat([water, filtered], ignore_index=True)
water.dropna(subset=['timestamp'], inplace=True)

for i, row in water.iterrows():
  aware = datetime_parser.parse(str(row['timestamp']))
  naive = aware.replace(tzinfo=None)
  water.at[i, 'timestamp'] = naive
water.set_index('timestamp', inplace=True)

!rm -rf Water CSV_Water
```
The last phase in preparing our groundwater level data for analysis involves a few key steps to ensure the data is both efficient and easy to work with:

Rearranging the Data in the site: We'll organize the data within the water DataFrame to ensure it's in the most effective format for our analysis. This could include ordering the columns in a specific sequence or grouping similar data types together for better clarity.

Removing Unneeded Fields: To streamline our analysis, it's important to eliminate any fields (columns) in the data that aren't necessary for our study. This helps reduce clutter and focus our attention on the most relevant information.

Sorting by Time: Given that we're dealing with time series data, sorting the data chronologically is essential. We'll arrange the data in the water DataFrame in ascending order based on the timestamp field. This step is crucial for maintaining a logical sequence of data, which is particularly important for identifying trends and patterns over time.

```python
water.drop(columns=['station_id', 'ciOrgname', 'ciCategory', 'Organize_Name', 'CategoryInfos_Name', 'PQ_name', 'PQ_fullname', 'PQ_description', 'PQ_unit', 'PQ_id'], inplace=True)
water.sort_values(by='timestamp', inplace=True)
water.info()
print(water.head())
```

```
<class 'pandas.core.frame.DataFrame'>
DatetimeIndex: 213466 entries, 2018-01-01 00:20:00 to 2021-12-07 11:00:00
Data columns (total 1 columns):
 \#   Column  Non-Null Count   Dtype  
---  ------  --------------   -----  
 0   value   213465 non-null  float64
dtypes: float64(1)
memory usage: 3.3 MB
                         value
timestamp                     
2018-01-01 00:20:00  49.130000
2018-01-01 00:25:00  49.139999
2018-01-01 00:30:00  49.130001
2018-01-01 00:35:00  49.130001
2018-01-01 00:40:00  49.130001
```

### Meteorological Data

For the meteorological data, we'll follow a similar process to what we did with the air quality and groundwater level data:

1. **Downloading the Data Archive**: We'll download the "Central Weather Bureau - Automatic Weather Station" data archive from the Civil IoT Taiwan Data Service Platform. This archive will be saved in a folder named `Weather`.

2. **Decompressing the Main Archive**: The downloaded data will be in a zip-compressed file format. Our first task is to decompress this main archive. This will reveal several smaller compressed files, each representing a day's worth of meteorological data.

3. **Decompressing Daily Files**: We then decompress each of these daily compressed files. The resulting decompressed files, in CSV format, will be stored in a new folder called `CSV_Weather`.

By following these steps, we'll have all the necessary meteorological data organized and ready for analysis. This data in `CSV_Weather` will be primed for further processing and analysis as required for our project.

```python
!mkdir Weather CSV_Weather
!wget -O Weather/2019.zip "https://history.colife.org.tw/?r=/download&path=L%2Bawo%2BixoS%2FkuK3lpK7msKPosaHlsYBf6Ieq5YuV5rCj6LGh56uZLzIwMTkuemlw"
!wget -O Weather/2020.zip "https://history.colife.org.tw/?r=/download&path=L%2Bawo%2BixoS%2FkuK3lpK7msKPosaHlsYBf6Ieq5YuV5rCj6LGh56uZLzIwMjAuemlw"
!wget -O Weather/2021.zip "https://history.colife.org.tw/?r=/download&path=L%2Bawo%2BixoS%2FkuK3lpK7msKPosaHlsYBf6Ieq5YuV5rCj6LGh56uZLzIwMjEuemlw"

folder = 'Weather'
extension_zip = '.zip'
extension_csv = '.csv'

for subfolder in os.listdir(folder):
    path = f'{folder}/{subfolder}'
    if path.endswith(extension_zip):
      print(path)
      zip_ref = zipfile.ZipFile(path)
      zip_ref.extractall(folder)
      zip_ref.close()

for subfolder in os.listdir(folder):
    path = f'{folder}/{subfolder}'
    if os.path.isdir(path):
        for item in os.listdir(path):
            if item.endswith(extension_zip):
                file_name = f'{path}/{item}'
                print(file_name)
                zip_ref = zipfile.ZipFile(file_name)
                zip_ref.extractall(path)
                zip_ref.close()

        for item in os.listdir(path):
          path2 = f'{path}/{item}'
          if os.path.isdir(path2):
            for it in os.listdir(path2):
              if it.endswith(extension_zip):
                file_name = f'{path2}/{it}'
                print(file_name)
                zip_ref = zipfile.ZipFile(file_name)
                zip_ref.extractall('CSV_Weather') # decide path
                zip_ref.close()
          elif item.endswith(extension_csv):
            os.rename(path2, f'CSV_Weather/{item}')
```

With the meteorological data now available in the `CSV_Weather` folder, we'll process it similarly to the air quality and groundwater level data:

1. **Reading the Data**: Each CSV file in the `CSV_Weather` folder contains a day's worth of data from various weather stations. We'll go through these files one by one.

2. **Filtering by Station**: As we read each file, our goal is to extract data specifically from the station with the code `C0U750`. This selective process ensures we're only gathering the data relevant to our analysis needs.

3. **Creating the `weather` DataFrame**: We'll compile all the data from station `C0U750` into a DataFrame named `weather`. This DataFrame becomes the central collection of all the meteorological data we need for our study.

4. **Cleanup**: Once we've gathered and organized the necessary data in the `weather` DataFrame, we'll proceed to delete all the original downloaded files and any data generated during the decompression process. This step is crucial for effective cloud storage management, especially when dealing with large datasets.

Following these steps, we ensure a focused and streamlined dataset for our meteorological analysis, maintaining an organized and efficient data processing workflow.

```python
folder = 'CSV_Weather'
extension_csv = '.csv'
id = 'C0U750'

weather = pd.DataFrame()
for item in os.listdir(folder):
  file_name = f'{folder}/{item}'
  df = pd.read_csv(file_name)
  if 'pm25' in list(df.columns):
    df.rename({'pm25':'PM25'}, axis=1, inplace=True)
  filtered = df.query(f'station_id==@id')
  weather = pd.concat([weather, filtered], ignore_index=True)
weather.rename({'obsTime':'timestamp'}, axis=1, inplace=True)
weather.dropna(subset=['timestamp'], inplace=True)

for i, row in weather.iterrows():
  aware = datetime_parser.parse(str(row['timestamp']))
  naive = aware.replace(tzinfo=None)
  weather.at[i, 'timestamp'] = naive
weather.set_index('timestamp', inplace=True)

!rm -rf Weather CSV_Weather
```

The last step in preparing our meteorological data for analysis is to ensure that it is well-organized and free of any unnecessary information. Here's how we'll do it:

1. **Rearranging the Data in the site**: We'll adjust the data within the `weather` DataFrame to best suit our analysis needs, that is rearranging the Data in the site.

2. **Removing Unneeded Fields**: To make our analysis as efficient as possible, we'll identify and remove any fields (columns) in the data that aren't necessary for our study. This step helps to focus our analysis on the most relevant data points.

3. **Sorting by Time**: Given the nature of time series data, it's crucial to have the data sorted in chronological order. We'll organize the data in the `weather` DataFrame based on the `timestamp` field, arranging it in ascending order. This chronological arrangement is key for accurately analyzing trends and patterns over time.

With these steps completed, our meteorological data is now in an optimal format for analysis. It's streamlined, focused, and arranged in a logical sequence, setting the stage for effective and meaningful time series analysis.

```python
weather.drop(columns=['station_id'], inplace=True)
weather.sort_values(by='timestamp', inplace=True)
weather.info()
print(weather.head())
```

```
<class 'pandas.core.frame.DataFrame'>
DatetimeIndex: 27093 entries, 2019-01-01 00:00:00 to 2021-12-31 23:00:00
Data columns (total 15 columns):
 \#   Column  Non-Null Count  Dtype  
---  ------  --------------  -----  
 0   ELEV    27093 non-null  float64
 1   WDIR    27089 non-null  float64
 2   WDSD    27089 non-null  float64
 3   TEMP    27093 non-null  float64
 4   HUMD    27089 non-null  float64
 5   PRES    27093 non-null  float64
 6   SUN     13714 non-null  float64
 7   H_24R   27089 non-null  float64
 8   H_FX    27089 non-null  float64
 9   H_XD    27089 non-null  object 
 10  H_FXT   23364 non-null  object 
 11  D_TX    27074 non-null  object 
 12  D_TXT   7574 non-null   object 
 13  D_TN    27074 non-null  object 
 14  D_TNT   17 non-null     object 
dtypes: float64(9), object(6)
memory usage: 3.3+ MB
                      ELEV  WDIR  WDSD  TEMP  HUMD   PRES   SUN  H_24R  H_FX  \
timestamp                                                                      
2019-01-01 00:00:00  398.0  35.0   5.8  13.4  0.99  981.1 -99.0   18.5 -99.0   
2019-01-01 01:00:00  398.0  31.0   5.7  14.1  0.99  981.0 -99.0    0.5  10.8   
2019-01-01 02:00:00  398.0  35.0   5.3  13.9  0.99  980.7 -99.0    1.0 -99.0   
2019-01-01 03:00:00  398.0  32.0   5.7  13.8  0.99  980.2 -99.0    1.5 -99.0   
2019-01-01 04:00:00  398.0  37.0   6.9  13.8  0.99  980.0 -99.0    2.0  12.0   

                     H_XD H_FXT  D_TX D_TXT  D_TN D_TNT  
timestamp                                                
2019-01-01 00:00:00 -99.0 -99.0  14.5   NaN  13.4   NaN  
2019-01-01 01:00:00  35.0   NaN  14.1   NaN  13.5   NaN  
2019-01-01 02:00:00 -99.0 -99.0  14.1   NaN  13.5   NaN  
2019-01-01 03:00:00 -99.0 -99.0  14.1   NaN  13.5   NaN  
2019-01-01 04:00:00  39.0   NaN  14.1   NaN  13.5   NaN
```

We have successfully illustrated how to access and prepare three different types of data from the Civil IoT Taiwan Data Service Platform: air quality data (air), water level data (water), and meteorological data (weather). Each dataset has been meticulously processed, filtered, and organized for our analysis. Our next steps - Basic Time Series Data Processing: By following along and applying these techniques to different datasets, you'll gain a comprehensive understanding of how to process and analyze time series data effectively. Whether for academic, professional, or personal projects, these skills are invaluable in extracting meaningful insights from temporal data.

## Data Preprocessing

Resampling is a crucial technique in time series analysis, especially when working with datasets like air quality measurements, which can have numerous readings over short intervals. By resampling, we can aggregate these readings into more manageable time frames like hourly, daily, or monthly averages. This process not only simplifies the data but also helps in identifying broader trends and patterns.

```python
air_hour = air.resample('H').mean()
air_day = air.resample('D').mean()
air_month = air.resample('M').mean()
```

To clean up the `air_hour` data, we first eliminate any unusual values, known as outliers, using the method we explained in Section 4.1. After that, we address any gaps in the data by applying the Forward Fill technique, which replaces missing values with the nearest preceding non-missing value.

```python
air_ts = TimeSeriesData(air_hour.reset_index(), time_col_name='timestamp')

# remove the outliers
outlierDetection = OutlierDetector(air_ts, 'additive')
outlierDetection.detector()
outliers_removed = outlierDetection.remover(interpolate=False)

air_hour_df = outliers_removed.to_dataframe()
air_hour_df.rename(columns={'time': 'timestamp', 'y_0': 'PM25'}, inplace=True)
air_hour_df.set_index('timestamp', inplace=True)
air_hour = air_hour_df
air_hour = air_hour.resample('H').mean()

# fill in the missing data with the Forward fill method
air_hour.ffill(inplace=True)
```

## Stationary Evaluation

Before we start predicting the data, it's important to first verify its  [stationarity](https://www.itl.nist.gov/div898/handbook/pmc/section4/pmc442.htm) , which means checking if its statistical properties like mean and variance stay constant over time. To do this, we choose a specific time period (like June 10 to June 17, 2020) and then gather all the data from this interval into a variable we call `data`.

```python
data = air_hour.loc['2020-06-10':'2020-06-17']
```
Next, we calculate the mean (mean) and the variation (var) of these data points. After calculating these, we visually represent them by plotting them on a graph.

```python
nmp = data.PM25.to_numpy()
size = np.size(nmp)

nmp_mean = np.zeros(size)
nmp_var = np.zeros(size)
for i in range(size):
  nmp_mean[i] = nmp[:i+1].mean()
  nmp_var[i] = nmp[:i+1].var()

y1 = nmp_mean[:]
y2 = nmp_var[:]
y3 = nmp
x = np.arange(size)
plt.plot(x, y1, label='mean')
plt.plot(x, y2, label='var')
plt.legend()
plt.show()
```

![Python output](figures/4-2-2-1.png)
From the graph, we notice that the average (mean) of the data remains relatively consistent, but the spread (variance) shows significant variation. This indicates that the data are not very stable over time, a condition we refer to as poor stationarity. In contrast, stationary data would show little to no time-related changes in both mean and variance.

Simply put, if the data's distribution shows a trend or pattern changing over time, it lacks stationarity. But if the data's distribution remains constant over time, with steady mean and variance, it is considered stationary. Understanding whether data is stationary is crucial for selecting the right model to predict future values.

There are two common methods to test for stationarity:

1. **Augmented Dickey Fuller (ADF) Test**: This uses a [unit root test](https://en.wikipedia.org/wiki/Unit_root_test) approach. Here, if the *p-value* is less than 0.05, the data is considered stationary.
2. **Kwiatkowski-Phillips-Schmidt-Shin (KPSS) Test**: This test works oppositely to the ADF test. In this case, if the *p-value* is less than 0.05, it suggests the data is not stationary.

```python
# ADF Test
result = adfuller(data.PM25.values, autolag='AIC')
print(f'ADF Statistic: {result[0]}')
print(f'p-value: {result[1]}')
for key, value in result[4].items():
    print('Critial Values:')
    print(f'   {key}, {value}')

# KPSS Test
result = kpss(data.PM25.values, regression='c')
print('\nKPSS Statistic: %f' % result[0])
print('p-value: %f' % result[1])
for key, value in result[3].items():
    print('Critial Values:')
    print(f'   {key}, {value}')
```

```
ADF Statistic: -2.7026194088541704
p-value: 0.07358609270498144
Critial Values:
   1%, -3.4654311561944873
Critial Values:
   5%, -2.8769570530458792
Critial Values:
   10%, -2.574988319755886

KPSS Statistic: 0.620177
p-value: 0.020802
Critial Values:
   10%, 0.347
Critial Values:
   5%, 0.463
Critial Values:
   2.5%, 0.574
Critial Values:
   1%, 0.739
```
In our example with the sample data, the Augmented Dickey Fuller (ADF) test yields a *p-value* of 0.073. Since this value is greater than 0.05, it indicates that our data is not stationary. To address this, we proceed by differentiating the data. This means we subtract the value at each previous point (i-1) from the current point (i) in the data sequence. We then use these differentiated values for a retest.

In terms of data handling, especially when working with a dataframe format, we can easily differentiate the data using the `data.diff()` function. The resulting differentiated data is then stored in a new variable named `data_diff`.

```python
data_diff = data.diff()
data_diff
```

```
                         PM25
timestamp	
2020-06-10 00:00:00	NaN
2020-06-10 01:00:00	-14.700000
2020-06-10 02:00:00	-8.100000
2020-06-10 03:00:00	0.200000
2020-06-10 04:00:00	-1.900000
...	...
2020-06-17 19:00:00	0.750000
2020-06-17 20:00:00	4.875000
2020-06-17 21:00:00	-3.375000
2020-06-17 22:00:00	1.930556
2020-06-17 23:00:00	3.944444
```

When we differentiate the data using the method described, the first data point turns out to be *NaN* (Not a Number). This happens because there's no preceding data point for the first entry to be subtracted from. To address this, we need to remove or discard this first data point, which lacks a valid value.

```python
data_diff = data_diff[1:]
data_diff
```

```
                         PM25
timestamp	
2020-06-10 01:00:00	-14.700000
2020-06-10 02:00:00	-8.100000
2020-06-10 03:00:00	0.200000
2020-06-10 04:00:00	-1.900000
2020-06-10 05:00:00	-1.300000
...	...
2020-06-17 19:00:00	0.750000
2020-06-17 20:00:00	4.875000
2020-06-17 21:00:00	-3.375000
2020-06-17 22:00:00	1.930556
2020-06-17 23:00:00	3.944444
```

After discarding the first data point, we then plot the remaining differentiated data. This plot helps us observe how the mean (average) and variance (spread) of the data behave over time following the differentiation process. 

```python
nmp = data_diff.PM25.to_numpy()
size = np.size(nmp)

nmp_mean = np.zeros(size)
nmp_var = np.zeros(size)
for i in range(size):
  nmp_mean[i] = nmp[:i+1].mean()
  nmp_var[i] = nmp[:i+1].var()

y1 = nmp_mean[:]
y2 = nmp_var[:]
y3 = nmp
x = np.arange(size)
plt.plot(x, y1, label='mean')
plt.plot(x, y2, label='var')
plt.legend()
plt.show()
```

![Python output](figures/4-2-2-2.png)

Based on the results from our plot of the differentiated data, we observe that the change in the mean remains minimal, while the variance shows a reduced level of change. This suggests an improvement towards stationarity. To confirm this, we'll repeat the earlier steps for evaluating stationarity:

1. We'll re-calculate the mean and variance of this newly differentiated data.
2. We'll create a new plot to visually assess these recalculated statistical properties.
3. We'll perform the stationarity tests again (like the Augmented Dickey Fuller (ADF) and Kwiatkowski-Phillips-Schmidt-Shin (KPSS) tests) on this differentiated data to check if it now meets the criteria for stationarity.

```python
# PM25
# ADF Test
result = adfuller(data_diff.PM25.values, autolag='AIC')
print(f'ADF Statistic: {result[0]}')
print(f'p-value: {result[1]}')
for key, value in result[4].items():
    print('Critial Values:')
    print(f'   {key}, {value}')

# KPSS Test
result = kpss(data_diff.PM25.values, regression='c')
print('\nKPSS Statistic: %f' % result[0])
print('p-value: %f' % result[1])
for key, value in result[3].items():
    print('Critial Values:')
    print(f'   {key}, {value}')
```

```
ADF Statistic: -13.350457196046884
p-value: 5.682260865619701e-25
Critial Values:
   1%, -3.4654311561944873
Critial Values:
   5%, -2.8769570530458792
Critial Values:
   10%, -2.574988319755886

KPSS Statistic: 0.114105
p-value: 0.100000
Critial Values:
   10%, 0.347
Critial Values:
   5%, 0.463
Critial Values:
   2.5%, 0.574
Critial Values:
   1%, 0.739
```

Following the differentiation and re-testing, the Augmented Dickey Fuller (ADF) test gives a *p-value* of 5.68e-25. This extremely small value indicates that the data, after being differentiated, is now stationary. This result is significant as it confirms that the data is suitable for accurate analysis and prediction. We will use this stationary data in the prediction models we develop in the subsequent steps.

## Data Forecast
After preparing the data, we move on to experimenting with different models for predicting time series data. The models we'll explore include:

1. **[ARIMA](https://otexts.com/fpp2/arima.html)**: This stands for AutoRegressive Integrated Moving Average. It's a popular statistical model used for forecasting time series data, especially when the data is stationary.

2. **[SARIMAX](https://www.statsmodels.org/stable/examples/notebooks/generated/statespace_sarimax_stata.html)**: This model extends ARIMA by incorporating Seasonal trends and potentially external or exogenous factors into the forecasting.

3. **[auto_arima](https://alkaline-ml.com/pmdarima/modules/generated/pmdarima.arima.auto_arima.html)**: This is an automated version of ARIMA that simplifies the process of model selection by automatically finding the best parameters.

4. **[Prophet](https://facebook.github.io/prophet/)**: Developed by Facebook, this model is designed for forecasting with daily observations that display patterns on different time scales.

5. **[LSTM](https://en.wikipedia.org/wiki/Long_short-term_memory)**: Long Short-Term Memory networks are a type of deep learning model suited for making predictions based on time series data.

6. **[Holt-Winter](https://otexts.com/fpp2/holt-winters.html)**: This model, also known as the Triple Exponential Smoothing method, is useful for forecasting data with seasonal patterns.

Each of these models has its strengths and is suitable for different types of time series data. The choice of model depends on the specific characteristics of the data being analyzed.

### ARIMA

The ARIMA model, which is an advanced form of the ARMA model, includes an additional component for handling non-stationary data. To understand ARIMA, let's first break down the ARMA model into its two core parts:

1. **Autoregressive model (AR)**: This part uses a parameter 'p' to make predictions based on a linear combination of the previous 'p' historical values. It's like saying that the future value is a function of the past 'p' values.

2. **Moving Average model (MA)**: This component, characterized by a parameter 'q', uses a linear combination of the previous 'q' forecast errors (made by the AR model) to predict the current value. It's akin to adjusting predictions based on how much we've been off in the recent past.

The ARIMA model then adds an extra parameter 'd', which stands for the degree of differencing needed to make the data stationary. If the data isn't stationary (i.e., its statistical properties change over time), we use differencing to stabilize it. The 'd' parameter specifies how many times the data needs to be differenced to achieve stationarity.

In practice, using the ARIMA model involves these steps:

1. Plot the data to visually inspect it and select the appropriate portion for analysis.
2. Determine the values of the parameters 'p', 'd', and 'q' for the ARIMA model. This involves identifying how many lags of historical data (p), how many times the data needs to be differenced (d), and how many lags of forecast errors (q) should be included in the model.
3. Apply the ARIMA model to the selected data to forecast future values.

For our exercise, we'll be using air quality data. The first step is to plot this data to identify the specific segment we want to analyze and model.

```python
air_hour.loc['2020-06-01':'2020-06-30']['PM25'].plot(figsize=(12, 8))
```

![Python output](figures/4-2-3-1.png)

For our exercise with the air quality data, we first choose a specific segment of the data that we want to focus on. Then, we divide this chosen data into two distinct parts for the purposes of our analysis:

1. **Train Data**: This portion of the data is used to train our model. During training, the model learns and identifies the most suitable parameters for making accurate predictions based on this data.

2. **Test Data**: This set is used to evaluate how well our model performs. It tests the model's ability to accurately predict data values it hasn't seen before, which helps us gauge its effectiveness and reliability.

In the example we're working on, we decide that the length of the test data will be the last 48 hours of our data set. This is achieved by setting `train_len=-48`. It means we take all the data available to us except for the last 48 hours, which we reserve for testing. The rest of the data, which is everything up to the last 48 hours, is used as training data. This approach allows us to train the model on a significant portion of the data while still keeping a meaningful and recent segment for testing its predictive accuracy.

```python
data_arima = air_hour.loc['2020-06-17':'2020-06-21']
train_len = -48
train = data_arima.iloc[:train_len]
test = data_arima.iloc[train_len:]
```
We first assess the stationarity of our chosen air quality data segment. The number of times we need to differentiate this data to achieve stationarity determines the 'd' parameter value for the ARIMA model.

```python
# Run Dicky-Fuller test
result = adfuller(train)

# Print test statistic
print('The test stastics:', result[0])

# Print p-value
print("The p-value:",result[1])
```

```
The test stastics: -3.1129543556288826
The p-value: 0.025609243615341074
```
Since the *p-value* from our stationarity test is less than 0.05, it means our data is already stationary. Thus, we don't need to differentiate the data, setting the 'd' parameter in the ARIMA model to 0. Next, we'll focus on determining the best values for the 'p' (autoregressive) and 'q' (moving average) parameters. Here's how we do it:

1. **Combining 'p' and 'q' Values**: We explore different combinations of 'p' and 'q'. Initially, we can limit both parameters to a range between 0 and 2. This creates nine possible combinations (0,0; 0,1; 0,2; 1,0; 1,1; 1,2; 2,0; 2,1; 2,2).

2. **Evaluating Model Quality**: To judge which combination of 'p' and 'q' works best, we use criteria like the Akaike Information Criterion ([AIC](https://en.wikipedia.org/wiki/Akaike_information_criterion)) or the Bayesian Information Criterion ([BIC](https://en.wikipedia.org/wiki/Bayesian_information_criterion)). These criteria help assess the model's fit to the training data.

3. **Selecting Parameters Based on AIC/BIC**: Generally, the lower the AIC or BIC value, the better the model's performance. We'll test each of the nine combinations and observe their AIC and BIC values. The combination of 'p' and 'q' yielding the smallest AIC or BIC value will be chosen as our model's parameters.

This methodical approach allows us to systematically identify the most suitable 'p' and 'q' values for our ARIMA model, ensuring an optimal balance between model complexity and fit.

```python
warnings.filterwarnings('ignore')
order_aic_bic =[]

# Loop over p values from 0-2
for p in range(3):
    # Loop over q values from 0-2
    for q in range(3):
      
        try:
            # create and fit ARMA(p,q) model
            model = sm.tsa.statespace.SARIMAX(train['PM25'], order=(p, 0, q))
            results = model.fit()
            
            # Print order and results
            order_aic_bic.append((p, q, results.aic, results.bic))            
        except:
            print(p, q, None, None)
            
# Make DataFrame of model order and AIC/BIC scores
order_df = pd.DataFrame(order_aic_bic, columns=['p', 'q', 'aic','bic'])

# lets sort them by AIC and BIC

# Sort by AIC
print("Sorted by AIC ")
# print("\n")
print(order_df.sort_values('aic').reset_index(drop=True))

# Sort by BIC
print("Sorted by BIC ")
# print("\n")
print(order_df.sort_values('bic').reset_index(drop=True))
```

```
Sorted by AIC 
   p  q         aic         bic
0  1  0  349.493661  354.046993
1  1  1  351.245734  358.075732
2  2  0  351.299268  358.129267
3  1  2  352.357930  361.464594
4  2  1  353.015921  362.122586
5  2  2  353.063243  364.446574
6  0  2  402.213407  409.043405
7  0  1  427.433962  431.987294
8  0  0  493.148188  495.424854
Sorted by BIC 
   p  q         aic         bic
0  1  0  349.493661  354.046993
1  1  1  351.245734  358.075732
2  2  0  351.299268  358.129267
3  1  2  352.357930  361.464594
4  2  1  353.015921  362.122586
5  2  2  353.063243  364.446574
6  0  2  402.213407  409.043405
7  0  1  427.433962  431.987294
8  0  0  493.148188  495.424854
```

Having determined that the combination (p, q) = (1, 0) yields the smallest values for both the Akaike Information Criterion (AIC) and the Bayesian Information Criterion (BIC), we conclude that this is the best configuration for our model. This means that the autoregressive component (p) is significant, but the moving average component (q) is not needed for this particular data set.

Therefore, we set up our ARIMA model with the parameters p, d, and q as 1, 0, and 0, respectively. The 'p' value of 1 indicates we are using one lag value in the autoregressive part of the model. The 'd' value of 0 shows that no differencing is needed as the data is already stationary. Finally, the 'q' value of 0 indicates that we are not using the moving average component in this model.

```python
# Instantiate model object
model = ARIMA(train, order=(1,0,0))

# Fit model
results = model.fit()
print(results.summary())
results.plot_diagnostics(figsize=(10, 10))
```

```
                               SARIMAX Results
==============================================================================
Dep. Variable:                   PM25   No. Observations:                   72
Model:                 ARIMA(1, 0, 0)   Log Likelihood                -168.853
Date:                Fri, 26 Aug 2022   AIC                            343.706
Time:                        05:01:13   BIC                            350.536
Sample:                    06-17-2020   HQIC                           346.425
                         - 06-19-2020
Covariance Type:                  opg
==============================================================================
                 coef    std err          z      P>|z|      [0.025      0.975]
------------------------------------------------------------------------------
const          6.3774      1.959      3.255      0.001       2.537      10.218
ar.L1          0.7792      0.047     16.584      0.000       0.687       0.871
sigma2         6.2934      0.746      8.438      0.000       4.832       7.755
===================================================================================
Ljung-Box (L1) (Q):                   0.08   Jarque-Bera (JB):                76.49
Prob(Q):                              0.77   Prob(JB):                         0.00
Heteroskedasticity (H):               2.30   Skew:                             1.44
Prob(H) (two-sided):                  0.05   Kurtosis:                         7.15
===================================================================================

```

![Python output](figures/4-2-3-2.png)

Upon using the test data for predictions with the ARIMA model, the resulting graph shows that the predicted curve is too smooth and differs significantly from the actual values. While ARIMA captures the data's general trend, it falls short in accurately predicting the regular fluctuations, indicating a need for more sophisticated modeling to close this gap.

```python
# Add a 'forecast' column to the 'data_arima' DataFrame with ARIMA model predictions.
data_arima['forecast'] = results.predict(start=24*5-48, end=24*5)

# Plot the 'PM25' values along with the forecasted values.
data_arima[['PM25', 'forecast']].plot(figsize=(12, 8))
```

![Python output](figures/4-2-3-3.png)

### SARIMAX

```python
# Extract a time range from the 'air_hour' DataFrame: June 17th to June 21st, 2020.
data_sarimax = air_hour.loc['2020-06-17':'2020-06-21']

# Define the length of the training data (excluding the last 48 hours).
train_len = -48

# Split the 'data_sarimax' DataFrame into training and testing sets.
train = data_sarimax.iloc[:train_len]
test = data_sarimax.iloc[train_len:]
```

The SARIMAX model, an extension of the ARIMA model, includes seven parameters: `p`, `d`, `q`, `P`, `D`, `Q`, and `s`. These parameters are divided into two groups:

1. **`order=(p, d, q)`**: These are the same as in the ARIMA model. 
   - `p`: Number of lag observations in the model (autoregressive part).
   - `d`: Degree of differencing required to make the series stationary.
   - `q`: Size of the moving average window.

2. **`seasonal_order=(P, D, Q, s)`**: These relate to the model's seasonal components.
   - `P`: Number of seasonal autoregressive terms.
   - `D`: Degree of seasonal differencing.
   - `Q`: Number of seasonal moving average terms.
   - `s`: Length of the seasonal cycle.

The SARIMAX model thus not only captures the non-seasonal aspects of the time series (through `p`, `d`, `q`) but also accounts for seasonality and external factors (through `P`, `D`, `Q`, `s`), making it more versatile for complex datasets with seasonal patterns.

| 參數 | 說明 |
| --- | --- |
| p | AR 模型參數 |
| d | 達到平穩性所需要的差分次數 |
| q | MA 模型參數 |
| P | 週期性的 AR 模型參數 |
| D | 週期上達到平穩性所需要的差分次數 |
| Q | 週期性的 MA 模型參數 |
| s | 週期長度 |

Given our observation of a 24-hour periodic change in the data, we set `s=24` in the SARIMAX model. This aligns the model's seasonal component with the data's daily pattern, allowing us to build the model using this setting to better predict the observed trends.

```python
# Instantiate model object
model = SARIMAX(train, order=(1,0,0), seasonal_order=(0, 1, 0, 24))

# Fit model
results = model.fit()
print(results.summary())
results.plot_diagnostics(figsize=(10, 10))
```

```
                                     SARIMAX Results
==========================================================================================
Dep. Variable:                               PM25   No. Observations:                   72
Model:             SARIMAX(1, 0, 0)x(0, 1, 0, 24)   Log Likelihood                -121.463
Date:                            Fri, 26 Aug 2022   AIC                            246.926
Time:                                    05:01:26   BIC                            250.669
Sample:                                06-17-2020   HQIC                           248.341
                                     - 06-19-2020
Covariance Type:                              opg
==============================================================================
                 coef    std err          z      P>|z|      [0.025      0.975]
------------------------------------------------------------------------------
ar.L1          0.6683      0.069      9.698      0.000       0.533       0.803
sigma2         9.1224      1.426      6.399      0.000       6.328      11.917
===================================================================================
Ljung-Box (L1) (Q):                   0.11   Jarque-Bera (JB):                 5.70
Prob(Q):                              0.75   Prob(JB):                         0.06
Heteroskedasticity (H):               2.03   Skew:                             0.42
Prob(H) (two-sided):                  0.17   Kurtosis:                         4.46
===================================================================================

```

![Python output](figures/4-2-3-4.png)

Next, we make predictions using the test data and visualize the prediction results. Although the SARIMA model's prediction results still have room to improve, they are already much better than the ARIMA model.

```python
# Add a 'forecast' column to the 'data_sarimax' DataFrame with SARIMA model predictions.
data_sarimax['forecast'] = results.predict(start=24*5-48, end=24*5)

# Plot the 'PM25' values along with the forecasted values.
data_sarimax[['PM25', 'forecast']].plot(figsize=(12, 8))
```

![Python output](figures/4-2-3-5.png)

### auto_arima

We use the [pmdarima](https://pypi.org/project/pmdarima/) Python package, which is similar to the `auto.arima` model in R. The package can automatically find the most suitable ARIMA model parameters, increasing users' convenience when using ARIMA models. The `pmdarima.ARIMA` object in the `pmdarima` package currently contains three models: ARMA, ARIMA, and SARIMAX. When using the `pmdarima.auto_arima` method, as long as the parameters `p`, `q,` `P`, and `Q` ranges are provided, the most suitable parameter combination is found within the specified range.

Next, we will implement how to use `pmdarima.auto_arima`, and first divide the data set into training data and test data:

```python
# Extract a time range from the 'air_hour' DataFrame: June 17th to June 21st, 2020.
data_autoarima = air_hour.loc['2020-06-17':'2020-06-21']

# Define the length of the training data (excluding the last 48 hours).
train_len = -48

# Split the 'data_autoarima' DataFrame into training and testing sets.
train = data_autoarima.iloc[:train_len]
test = data_autoarima.iloc[train_len:]
```

For the four parameters `p`, `q`, `P`, `Q`, we use `start` and `max` to specify the corresponding ranges. We also set the periodic parameter `seasonal` to True and the periodic variable `m` to 24 hours. Then we can directly get the best model parameter combination and model fitting results.

```python
# Fit an AutoARIMA model to the training data.
results = pm.auto_arima(train, start_p=0, d=0, start_q=0, max_p=5, max_d=5, max_q=5, start_P=0, D=1, start_Q=0, max_P=5, max_D=5, max_Q=5, m=24, seasonal=True, error_action='warn', trace=True, suppress_warnings=True, stepwise=True, random_state=20, n_fits=20)

# Print a summary of the model's results.
print(results.summary())
```

```
Performing stepwise search to minimize aic
 ARIMA(0,0,0)(0,1,0)[24] intercept   : AIC=268.023, Time=0.04 sec
 ARIMA(1,0,0)(1,1,0)[24] intercept   : AIC=247.639, Time=0.85 sec
 ARIMA(0,0,1)(0,1,1)[24] intercept   : AIC=250.711, Time=0.79 sec
 ARIMA(0,0,0)(0,1,0)[24]             : AIC=271.305, Time=0.04 sec
 ARIMA(1,0,0)(0,1,0)[24] intercept   : AIC=247.106, Time=0.09 sec
 ARIMA(1,0,0)(0,1,1)[24] intercept   : AIC=247.668, Time=0.45 sec
 ARIMA(1,0,0)(1,1,1)[24] intercept   : AIC=inf, Time=2.63 sec
 ARIMA(2,0,0)(0,1,0)[24] intercept   : AIC=249.013, Time=0.15 sec
 ARIMA(1,0,1)(0,1,0)[24] intercept   : AIC=248.924, Time=0.21 sec
 ARIMA(0,0,1)(0,1,0)[24] intercept   : AIC=250.901, Time=0.11 sec
 ARIMA(2,0,1)(0,1,0)[24] intercept   : AIC=250.579, Time=0.30 sec
 ARIMA(1,0,0)(0,1,0)[24]             : AIC=246.926, Time=0.06 sec
 ARIMA(1,0,0)(1,1,0)[24]             : AIC=247.866, Time=0.28 sec
 ARIMA(1,0,0)(0,1,1)[24]             : AIC=247.933, Time=0.31 sec
 ARIMA(1,0,0)(1,1,1)[24]             : AIC=inf, Time=2.35 sec
 ARIMA(2,0,0)(0,1,0)[24]             : AIC=248.910, Time=0.08 sec
 ARIMA(1,0,1)(0,1,0)[24]             : AIC=248.893, Time=0.09 sec
 ARIMA(0,0,1)(0,1,0)[24]             : AIC=252.779, Time=0.08 sec
 ARIMA(2,0,1)(0,1,0)[24]             : AIC=250.561, Time=0.17 sec

Best model:  ARIMA(1,0,0)(0,1,0)[24]
Total fit time: 9.122 seconds
                                     SARIMAX Results
==========================================================================================
Dep. Variable:                                  y   No. Observations:                   72
Model:             SARIMAX(1, 0, 0)x(0, 1, 0, 24)   Log Likelihood                -121.463
Date:                            Fri, 26 Aug 2022   AIC                            246.926
Time:                                    05:01:37   BIC                            250.669
Sample:                                06-17-2020   HQIC                           248.341
                                     - 06-19-2020
Covariance Type:                              opg
==============================================================================
                 coef    std err          z      P>|z|      [0.025      0.975]
------------------------------------------------------------------------------
ar.L1          0.6683      0.069      9.698      0.000       0.533       0.803
sigma2         9.1224      1.426      6.399      0.000       6.328      11.917
===================================================================================
Ljung-Box (L1) (Q):                   0.11   Jarque-Bera (JB):                 5.70
Prob(Q):                              0.75   Prob(JB):                         0.06
Heteroskedasticity (H):               2.03   Skew:                             0.42
Prob(H) (two-sided):                  0.17   Kurtosis:                         4.46
===================================================================================
```

Finally, we use the best model found for data prediction, and plot the prediction results and test data on the same graph in the form of an overlay. Since the best model found this time is the SARIMAX model just introduced, the results of both predictors are roughly the same.

```python
# Generate forecasts for the next 10 periods using the fitted AutoARIMA model.
results.predict(n_periods=10)
```

```
2020-06-20 00:00:00    10.371336
2020-06-20 01:00:00    13.142043
2020-06-20 02:00:00    13.505843
2020-06-20 03:00:00     9.506395
2020-06-20 04:00:00     7.450378
2020-06-20 05:00:00     7.782850
2020-06-20 06:00:00     7.633757
2020-06-20 07:00:00     5.200781
2020-06-20 08:00:00     3.634188
2020-06-20 09:00:00     3.946824
Freq: H, dtype: float64
```

```python
# Generate forecasts for the next 48 periods using the fitted AutoARIMA model.
data_autoarima['forecast'] = pd.DataFrame(results.predict(n_periods=48), index=test.index)

# Plot the 'PM25' values along with the forecasted values.
data_autoarima[['PM25', 'forecast']].plot(figsize=(12, 8))

```

![Python output](figures/4-2-3-6.png)

### Prophet

Next, we use the [Prophet](https://facebook.github.io/prophet/) model provided in the kats suite for data prediction. This model is proposed by Facebook's data science team, and it is good at predicting periodic time series data and can tolerate missing data, data shift, and outliers.

We first divide the dataset into training and prediction data and observe the changes in the training data by drawing.

```python
# Extract a time range from the 'air_hour' DataFrame: June 17th to June 21st, 2020.
data_prophet = air_hour.loc['2020-06-17':'2020-06-21']

# Define the length of the training data (excluding the last 48 hours).
train_len = -48

# Split the 'data_prophet' DataFrame into training and testing sets.
train = data_prophet.iloc[:train_len]
test = data_prophet.iloc[train_len:]

# Create a TimeSeriesData object for the training data with a 'timestamp' column.
trainData = TimeSeriesData(train.reset_index(), time_col_name='timestamp')

# Plot the 'PM25' values from the training data.
trainData.plot(cols=["PM25"])
```

![Python output](figures/4-2-3-7.png)

We then use `ProphetParams` to set the Prophet model's parameters and the training data and parameters to initialize the `ProphetModel`. Then we use the `fit` method to build the model and use the `predict` method to predict the data, and then we can get the final prediction result.

```python
# Specify Prophet model parameters with seasonality mode as "multiplicative."
params = ProphetParams(seasonality_mode="multiplicative")

# Create a Prophet model instance with the training data and specified parameters.
m = ProphetModel(trainData, params)

# Fit the Prophet model to the training data.
m.fit()

# Generate forecasts for the next 48 hours with an hourly frequency.
fcst = m.predict(steps=48, freq="H")

# Add the forecasted values to the 'data_prophet' DataFrame.
data_prophet['forecast'] = fcst[['time', 'fcst']].set_index('time')

# Display the forecasts.
fcst
```

```
                 time	    fcst	fcst_lower fcst_upper
0	2020-06-20 00:00:00	14.705192	12.268361	17.042476
1	2020-06-20 01:00:00	15.089580	12.625568	17.573396
2	2020-06-20 02:00:00	14.921077	12.459802	17.411335
3	2020-06-20 03:00:00	13.846131	11.444988	16.200284
4	2020-06-20 04:00:00	12.278140	9.863531	14.858334
5	2020-06-20 05:00:00	10.934739	8.372450	13.501025
6	2020-06-20 06:00:00	10.126712	7.654647	12.658054
7	2020-06-20 07:00:00	9.535067	7.034313	11.762639
8	2020-06-20 08:00:00	8.661877	6.255147	11.132732
9	2020-06-20 09:00:00	7.424133	5.055052	9.770750
10	2020-06-20 10:00:00	6.229786	3.640543	8.625856
11	2020-06-20 11:00:00	5.464764	3.039011	7.939283
12	2020-06-20 12:00:00	4.998005	2.692023	7.550191
13	2020-06-20 13:00:00	4.334771	1.961382	6.506875
14	2020-06-20 14:00:00	3.349172	1.059836	5.768178
15	2020-06-20 15:00:00	2.819902	0.399350	5.226658
16	2020-06-20 16:00:00	4.060070	1.556264	6.322976
17	2020-06-20 17:00:00	7.792830	5.331987	10.237182
18	2020-06-20 18:00:00	13.257767	10.873149	15.542380
19	2020-06-20 19:00:00	18.466805	15.895210	20.874602
20	2020-06-20 20:00:00	21.535994	19.150397	23.960260
21	2020-06-20 21:00:00	22.005943	19.509141	24.691836
22	2020-06-20 22:00:00	21.014449	18.610361	23.661906
23	2020-06-20 23:00:00	20.191905	17.600568	22.868388
24	2020-06-21 00:00:00	20.286952	17.734177	22.905280
25	2020-06-21 01:00:00	20.728067	18.235829	23.235212
26	2020-06-21 02:00:00	20.411124	17.755181	22.777073
27	2020-06-21 03:00:00	18.863739	16.261775	21.315573
28	2020-06-21 04:00:00	16.661351	13.905466	19.374726
29	2020-06-21 05:00:00	14.781150	12.401465	17.499478
30	2020-06-21 06:00:00	13.637436	11.206142	16.239831
31	2020-06-21 07:00:00	12.793609	9.940829	15.319559
32	2020-06-21 08:00:00	11.580455	9.059603	14.261605
33	2020-06-21 09:00:00	9.891025	7.230943	12.471543
34	2020-06-21 10:00:00	8.271552	5.840853	10.677227
35	2020-06-21 11:00:00	7.231671	4.829449	9.733231
36	2020-06-21 12:00:00	6.592515	4.108251	9.107216
37	2020-06-21 13:00:00	5.699548	3.288052	8.019402
38	2020-06-21 14:00:00	4.389985	1.848621	6.825121
39	2020-06-21 15:00:00	3.685033	1.196467	6.150064
40	2020-06-21 16:00:00	5.289956	2.907623	8.012851
41	2020-06-21 17:00:00	10.124029	7.397842	12.676256
42	2020-06-21 18:00:00	17.174959	14.670539	19.856592
43	2020-06-21 19:00:00	23.856724	21.102924	26.712359
44	2020-06-21 20:00:00	27.746195	24.636118	30.673178
45	2020-06-21 21:00:00	28.276321	25.175013	31.543197
46	2020-06-21 22:00:00	26.932054	23.690073	29.882014
47	2020-06-21 23:00:00	25.811943	22.960132	28.912079
```

```python
# Display the dataframe.
data_prophet
```

```
          timestamp		  PM25	forecast
2020-06-17 00:00:00	6.300000	NaN
2020-06-17 01:00:00	11.444444	NaN
2020-06-17 02:00:00	6.777778	NaN
2020-06-17 03:00:00	4.875000	NaN
2020-06-17 04:00:00	5.444444	NaN
...	...	...
2020-06-21 19:00:00	18.777778	23.856724
2020-06-21 20:00:00	21.400000	27.746195
2020-06-21 21:00:00	11.222222	28.276321
2020-06-21 22:00:00	9.800000	26.932054
2020-06-21 23:00:00	8.100000	25.811943
```

We use the built-in drawing method of `ProphetModel` to draw the training data (black curve) and prediction results (blue curve).

```python
# Plot the result of Prophet model.
m.plot()
```

![Python output](figures/4-2-3-8.png)

To evaluate the correctness of the prediction results, we also use another drawing method to draw the training data (black curve), test data (black curve), and prediction results (blue curve) at the same time. The figure shows that the blue and black curves are roughly consistent in the changing trend and value range, and overall the data prediction results are satisfactory.

```python
# Create a figure and axis for the plot with a specified figsize.
fig, ax = plt.subplots(figsize=(12, 7))

# Plot the training data with a black color and label 'train'.
train.plot(ax=ax, label='train', color='black')

# Plot the testing data with a black color.
test.plot(ax=ax, color='black')

# Plot the Prophet model forecasts with a blue color.
fcst.plot(x='time', y='fcst', ax=ax, color='blue')

# Fill the area between the lower and upper forecast bounds with a slight transparency.
ax.fill_between(test.index, fcst['fcst_lower'], fcst['fcst_upper'], alpha=0.1)

# Remove the legend from the plot.
ax.get_legend().remove()
```

![Python output](figures/4-2-3-9.png)

### LSTM

Next, we introduce the Long Short-Term Memory (LSTM) model for data prediction. The LSTM model is a predictive model suitable for continuous data because it will generate different long-term and short-term memories for data at different times and use it to predict the final result. Currently, the LSTM model is provided in the kats package, so we can directly use the syntax similar to using the Prophet model.

We first divide the dataset into training and prediction data and observe the changes in the training data by drawing.

```python
# Extract a time range from the 'air_hour' DataFrame: June 17th to June 21st, 2020.
data_lstm = air_hour.loc['2020-06-17':'2020-06-21']

# Define the length of the training data (excluding the last 48 hours).
train_len = -48

# Split the 'data_lstm' DataFrame into training and testing sets.
train = data_lstm.iloc[:train_len]
test = data_lstm.iloc[train_len:]

# Create a TimeSeriesData object for the training data with a 'timestamp' column.
trainData = TimeSeriesData(train.reset_index(), time_col_name='timestamp')

# Plot the 'PM25' values from the training data.
trainData.plot(cols=["PM25"])
```

![Python output](figures/4-2-3-10.png)

Then we select the parameters of the LSTM model in order, namely the number of training times (`num_epochs`), the time length of data read in at one time (`time_window`), and the number of neural network layers related to long-term and short-term memory (`hidden_size`). Then you can directly perform model training and data prediction.

```python
# Specify LSTM model parameters, including the number of hidden layers, time window, and number of epochs.
params = LSTMParams(
    hidden_size=10,  # Number of hidden layers
    time_window=24,  # Time window for model input
    num_epochs=30  # Number of training epochs
)

# Create an LSTM model instance with the training data and specified parameters.
m = LSTMModel(trainData, params)

# Fit the LSTM model to the training data.
m.fit()

# Generate forecasts for the next 48 hours with an hourly frequency.
fcst = m.predict(steps=48, freq="H")

# Add the forecasted values to the 'data_lstm' DataFrame.
data_lstm['forecast'] = fcst[['time', 'fcst']].set_index('time')

# Display the forecasts.
fcst
```

```
                 time	    fcst	fcst_lower	fcst_upper
0	2020-06-20 00:00:00	11.905971	11.310672	12.501269
1	2020-06-20 01:00:00	10.804338	10.264121	11.344554
2	2020-06-20 02:00:00	9.740741	9.253704	10.227778
3	2020-06-20 03:00:00	8.696406	8.261586	9.131226
4	2020-06-20 04:00:00	7.656923	7.274077	8.039769
5	2020-06-20 05:00:00	6.608442	6.278019	6.938864
6	2020-06-20 06:00:00	5.543790	5.266600	5.820979
7	2020-06-20 07:00:00	4.469023	4.245572	4.692474
8	2020-06-20 08:00:00	3.408312	3.237897	3.578728
9	2020-06-20 09:00:00	2.411980	2.291381	2.532578
10	2020-06-20 10:00:00	1.564808	1.486567	1.643048
11	2020-06-20 11:00:00	0.982147	0.933040	1.031255
12	2020-06-20 12:00:00	0.792612	0.752981	0.832242
13	2020-06-20 13:00:00	1.105420	1.050149	1.160691
14	2020-06-20 14:00:00	1.979013	1.880062	2.077964
15	2020-06-20 15:00:00	3.408440	3.238018	3.578862
16	2020-06-20 16:00:00	5.337892	5.070997	5.604786
17	2020-06-20 17:00:00	7.659332	7.276365	8.042299
18	2020-06-20 18:00:00	10.104147	9.598940	10.609355
19	2020-06-20 19:00:00	12.047168	11.444809	12.649526
20	2020-06-20 20:00:00	12.880240	12.236228	13.524252
21	2020-06-20 21:00:00	12.748750	12.111312	13.386187
22	2020-06-20 22:00:00	12.128366	11.521947	12.734784
23	2020-06-20 23:00:00	11.311866	10.746273	11.877459
24	2020-06-21 00:00:00	10.419082	9.898128	10.940036
25	2020-06-21 01:00:00	9.494399	9.019679	9.969119
26	2020-06-21 02:00:00	8.551890	8.124296	8.979485
27	2020-06-21 03:00:00	7.592260	7.212647	7.971873
28	2020-06-21 04:00:00	6.613075	6.282421	6.943729
29	2020-06-21 05:00:00	5.614669	5.333936	5.895402
30	2020-06-21 06:00:00	4.605963	4.375664	4.836261
31	2020-06-21 07:00:00	3.611552	3.430974	3.792129
32	2020-06-21 08:00:00	2.679572	2.545593	2.813550
33	2020-06-21 09:00:00	1.887442	1.793070	1.981814
34	2020-06-21 10:00:00	1.340268	1.273255	1.407282
35	2020-06-21 11:00:00	1.156494	1.098669	1.214318
36	2020-06-21 12:00:00	1.440240	1.368228	1.512252
37	2020-06-21 13:00:00	2.251431	2.138859	2.364002
38	2020-06-21 14:00:00	3.592712	3.413076	3.772347
39	2020-06-21 15:00:00	5.415959	5.145161	5.686757
40	2020-06-21 16:00:00	7.613187	7.232528	7.993847
41	2020-06-21 17:00:00	9.918564	9.422636	10.414493
42	2020-06-21 18:00:00	11.755348	11.167580	12.343115
43	2020-06-21 19:00:00	12.576593	11.947764	13.205423
44	2020-06-21 20:00:00	12.489052	11.864599	13.113504
45	2020-06-21 21:00:00	11.915885	11.320090	12.511679
46	2020-06-21 22:00:00	11.133274	10.576610	11.689938
47	2020-06-21 23:00:00	10.264495	9.751270	10.777719
```

We also use the built-in drawing method of `LSTMModel` to draw the training data (black curve) and prediction results (blue curve).

```python
# Plot the result of LSTMModel model.
m.plot()
```

![Python output](figures/4-2-3-11.png)

To evaluate the correctness of the prediction results, we also use another drawing method to draw the training data (black curve), test data (black curve), and prediction results (blue curve) at the same time. The figure shows that the blue and black curves are roughly consistent in the changing trend and value range, but overall, the data prediction result (blue curve) is slightly lower than the test data (black curve).

```python
# Create a figure and axis for the plot with a specified figsize.
fig, ax = plt.subplots(figsize=(12, 7))

# Plot the training data with a black color and label 'train'.
train.plot(ax=ax, label='train', color='black')

# Plot the testing data with a black color.
test.plot(ax=ax, color='black')

# Plot the LSTM model forecasts with a blue color.
fcst.plot(x='time', y='fcst', ax=ax, color='blue')

# Fill the area between the lower and upper forecast bounds with a slight transparency.
ax.fill_between(test.index, fcst['fcst_lower'], fcst['fcst_upper'], alpha=0.1)

# Remove the legend from the plot.
ax.get_legend().remove()
```

![Python output](figures/4-2-3-12.png)

### Holt-Winter

We also use the Holt-Winter model provided by the kats package, a method that uses moving averages to assign weights to historical data for data forecasting. We first divide the dataset into training and prediction data and observe the changes in the training data by drawing.

```python
# Extract a time range from the 'air_hour' DataFrame: June 17th to June 21st, 2020.
data_hw = air_hour.loc['2020-06-17':'2020-06-21']

# Define the length of the training data (excluding the last 48 hours).
train_len = -48

# Split the 'data_hw' DataFrame into training and testing sets.
train = data_hw.iloc[:train_len]
test = data_hw.iloc[train_len:]

# Create a TimeSeriesData object for the training data with a 'timestamp' column.
trainData = TimeSeriesData(train.reset_index(), time_col_name='timestamp')

# Plot the 'PM25' values from the training data.
trainData.plot(cols=["PM25"])
```

![Python output](figures/4-2-3-13.png)

Then we need to set the parameters of the Holt-Winter model, which are to select whether to use addition or multiplication to decompose the time series data (the following example uses multiplication, `mul`), and the length of the period (the following example uses 24 hours). Then we can perform model training and data prediction.

```python
# Ignore warnings for cleaner output.
warnings.simplefilter(action='ignore')

# Specify parameters for the Holt-Winters model.
params = HoltWintersParams(
    trend="multiplicative",  # Multiplicative trend component.
    seasonal="multiplicative",  # Multiplicative seasonal component.
    seasonal_periods=24,  # Seasonal period of 24 hours.
)

# Create an instance of the Holt-Winters model with the specified parameters.
m = HoltWintersModel(
    data=trainData,  # Training data provided as a TimeSeriesData object.
    params=params  # Use the defined parameters.
)

# Fit the Holt-Winters model to the training data.
m.fit()

# Forecast future values for 48 time steps with a frequency of 'H' (hourly).
fcst = m.predict(steps=48, freq='H')

# Add the forecasted values to the 'data_hw' DataFrame with a 'timestamp' index.
data_hw['forecast'] = fcst[['time', 'fcst']].set_index('time')

# Display the forecast.
fcst
```

```
                   time	    fcst
72	2020-06-20 00:00:00	14.140232
73	2020-06-20 01:00:00	14.571588
74	2020-06-20 02:00:00	12.797056
75	2020-06-20 03:00:00	10.061594
76	2020-06-20 04:00:00	9.927476
77	2020-06-20 05:00:00	8.732691
78	2020-06-20 06:00:00	10.257460
79	2020-06-20 07:00:00	8.169070
80	2020-06-20 08:00:00	6.005400
81	2020-06-20 09:00:00	5.038056
82	2020-06-20 10:00:00	6.391835
83	2020-06-20 11:00:00	5.435677
84	2020-06-20 12:00:00	3.536135
85	2020-06-20 13:00:00	2.725477
86	2020-06-20 14:00:00	2.588198
87	2020-06-20 15:00:00	2.967987
88	2020-06-20 16:00:00	3.329448
89	2020-06-20 17:00:00	4.409821
90	2020-06-20 18:00:00	10.295263
91	2020-06-20 19:00:00	10.587033
92	2020-06-20 20:00:00	14.061718
93	2020-06-20 21:00:00	18.597275
94	2020-06-20 22:00:00	12.040684
95	2020-06-20 23:00:00	12.124081
96	2020-06-21 00:00:00	13.522973
97	2020-06-21 01:00:00	13.935499
98	2020-06-21 02:00:00	12.238431
99	2020-06-21 03:00:00	9.622379
100	2020-06-21 04:00:00	9.494116
101	2020-06-21 05:00:00	8.351486
102	2020-06-21 06:00:00	9.809694
103	2020-06-21 07:00:00	7.812468
104	2020-06-21 08:00:00	5.743248
105	2020-06-21 09:00:00	4.818132
106	2020-06-21 10:00:00	6.112815
107	2020-06-21 11:00:00	5.198396
108	2020-06-21 12:00:00	3.381773
109	2020-06-21 13:00:00	2.606503
110	2020-06-21 14:00:00	2.475216
111	2020-06-21 15:00:00	2.838426
112	2020-06-21 16:00:00	3.184109
113	2020-06-21 17:00:00	4.217320
114	2020-06-21 18:00:00	9.845847
115	2020-06-21 19:00:00	10.124881
116	2020-06-21 20:00:00	13.447887
117	2020-06-21 21:00:00	17.785455
118	2020-06-21 22:00:00	11.515076
119	2020-06-21 23:00:00	11.594832
```

We also use the built-in drawing method of `HoltWintersModel` to draw the training data (black curve) and prediction results (blue curve).

```python
# Plot the results of the Holt-Winters model.
m.plot()
```

![Python output](figures/4-2-3-14.png)

To evaluate the correctness of the prediction results, we also use another drawing method to draw the training data (black curve), test data (black curve), and prediction results (blue curve) at the same time. The figure shows that the blue and black curves are roughly consistent in the changing trend and value range. Still, overall, the data prediction result (blue curve) responds slightly slower to the rising slope than the test data (black curve).

```python
# Create a figure and axis for the plot with a specified figsize.
fig, ax = plt.subplots(figsize=(12, 7))

# Plot the training data with a label and black color.
train.plot(ax=ax, label='train', color='black')

# Plot the test data in black.
test.plot(ax=ax, color='black')

# Plot the forecasted data with 'time' as the x-axis and 'fcst' as the y-axis in blue.
fcst.plot(x='time', y='fcst', ax=ax, color='blue')

# Remove the legend from the plot.
ax.get_legend().remove()
```

![Python output](figures/4-2-3-15.png)

### Comparison

Finally, to facilitate observation and comparison, we will draw the prediction results of the six models introduced in the figure below simultaneously (Note: You must first run all the codes of the above prediction models to see the results of these six prediction models). We can observe and compare the prediction accuracy of the six models under different time intervals and curve change characteristics, which is convenient for users to decide on the final model selection and possible future applications.

```python
# Create a figure with a 3x2 grid of subplots and a specified figsize.
fig, axes = plt.subplots(nrows=3, ncols=2, figsize=(12, 8))

# Plot the 'PM25' and 'forecast' data for ARIMA in the top-left subplot with the title 'ARIMA'.
data_arima[['PM25', 'forecast']].plot(ax=axes[0, 0], title='ARIMA')

# Plot the 'PM25' and 'forecast' data for SARIMAX in the middle-left subplot with the title 'SARIMAX'.
data_sarimax[['PM25', 'forecast']].plot(ax=axes[1, 0], title='SARIMAX')

# Plot the 'PM25' and 'forecast' data for auto_arima in the bottom-left subplot with the title 'auto_arima'.
data_autoarima[['PM25', 'forecast']].plot(ax=axes[2, 0], title='auto_arima')

# Plot the 'PM25' and 'forecast' data for Prophet in the top-right subplot with the title 'Prophet'.
data_prophet[['PM25', 'forecast']].plot(ax=axes[0, 1], title='Prophet')

# Plot the 'PM25' and 'forecast' data for LSTM in the middle-right subplot with the title 'LSTM'.
data_lstm[['PM25', 'forecast']].plot(ax=axes[1, 1], title='LSTM')

# Plot the 'PM25' and 'forecast' data for Holt-Winter in the bottom-right subplot with the title 'Holt-Winter'.
data_hw[['PM25', 'forecast']].plot(ax=axes[2, 1], title='Holt-Winter')

# Adjust the layout of subplots for better spacing.
fig.tight_layout(pad=1, w_pad=2, h_pad=5)
```

![Python output](figures/4-2-3-16.png)

## References

- Civil IoT Taiwan: Historical Data ([https://history.colife.org.tw/](https://history.colife.org.tw/#/))
- Rob J Hyndman and George Athanasopoulos, Forecasting: Principles and Practice, 3rd edition ([https://otexts.com/fpp3/](https://otexts.com/fpp3/))
- Stationarity, NIST Engineering Statistics Handbook ([https://www.itl.nist.gov/div898/handbook/pmc/section4/pmc442.htm](https://www.itl.nist.gov/div898/handbook/pmc/section4/pmc442.htm))
- Unit root test - Wikipedia ([https://en.wikipedia.org/wiki/Unit_root_test](https://en.wikipedia.org/wiki/Unit_root_test))
- Akaike information criterion (AIC) - Wikipedia ([https://en.wikipedia.org/wiki/Akaike_information_criterion](https://en.wikipedia.org/wiki/Akaike_information_criterion))
- Bayesian information criterion (BIC) - Wikipedia ([https://en.wikipedia.org/wiki/Bayesian_information_criterion](https://en.wikipedia.org/wiki/Bayesian_information_criterion))
- ARIMA models ([https://otexts.com/fpp2/arima.html](https://otexts.com/fpp2/arima.html))
- SARIMAX: Introduction ([https://www.statsmodels.org/stable/examples/notebooks/generated/statespace_sarimax_stata.html](https://www.statsmodels.org/stable/examples/notebooks/generated/statespace_sarimax_stata.html))
- Prophet: Forecasting at scale ([https://facebook.github.io/prophet/](https://facebook.github.io/prophet/))
- Long short-term memory (LSTM) – Wikipedia ([https://en.wikipedia.org/wiki/Long_short-term_memory](https://en.wikipedia.org/wiki/Long_short-term_memory))
- Time Series Forecasting with ARIMA Models In Python [Part 1] | by Youssef Hosni | May, 2022 | Towards AI ([https://pub.towardsai.net/time-series-forecasting-with-arima-models-in-python-part-1-c2940a7dbc48?gi=264dc7630363](https://pub.towardsai.net/time-series-forecasting-with-arima-models-in-python-part-1-c2940a7dbc48?gi=264dc7630363))
- Time Series Forecasting with ARIMA Models In Python [Part 2] | by Youssef Hosni | May, 2022 | Towards AI ([https://pub.towardsai.net/time-series-forecasting-with-arima-models-in-python-part-2-91a30d10efb0](https://pub.towardsai.net/time-series-forecasting-with-arima-models-in-python-part-2-91a30d10efb0))
- Kats: a Generalizable Framework to Analyze Time Series Data in Python | by Khuyen Tran | Towards Data Science ([https://towardsdatascience.com/kats-a-generalizable-framework-to-analyze-time-series-data-in-python-3c8d21efe057](https://towardsdatascience.com/kats-a-generalizable-framework-to-analyze-time-series-data-in-python-3c8d21efe057))
- Kats - Time Series Forecasting By Facebook | by Himanshu Sharma | MLearning.ai | Medium ([https://medium.com/mlearning-ai/kats-time-series-forecasting-by-facebook-a2741794d814](https://medium.com/mlearning-ai/kats-time-series-forecasting-by-facebook-a2741794d814))
