---
title : "6.3. 感測器聯合校正"
weight : 30
socialshare: true
description : "我們使用空品類別資料，示範台灣微型空品感測器與官方測站進行動態校正的演算法，以做中學的方式，一步步從資料準備，特徵擷取，到機器學習、資料分析、統計與歸納，重現感測器動態校正模型演算法的原理與實作過程，讓讀者體驗如何透過疊加基本的資料分析與機器學習步驟，逐步達成進階且實用的資料應用服務。"
tags: ["Python", "空" ]
levels: ["advanced" ]
author: ["羅泉恆"]
---



[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/drive/1TNVyAI4NaCBAWgmrHPipmZkdeMJ-YZNu?usp=sharing)

{{< toc >}}

In this article, we will take the air quality data in Civil IoT Taiwan as an example to introduce how to allow two different levels of air quality sensing data to be systematically and dynamically calibrated through data science methods. At the same time, with the results of data fusion, different deployment projects can work together to create a more comprehensive air quality sensing result. We use the following two air quality sensing systems:

1. Environmental Protection Agency Air Quality Monitoring Station: In traditional air quality monitoring, extremely professional, large-scale and high-cost monitoring stations are mainly used. It cannot be deployed in every community due to high deployment and maintenance costs. Instead, local environmental agencies will select specific locations for deployment and maintenance. According to the announcement on the website of the Taiwan Environmental Protection Agency, as of July 2022, the number of official monitoring stations in Taiwan is 81.
2. Micro Air Quality Sensor: Compared with the traditional professional station, the micro air quality sensor adopts low-cost sensors, transmits data through the network, and builds a denser air quality sensor in the way of the Internet of Things network. This technology not only has low erection cost, but also provides more flexible installation conditions and expanded coverage. At the same time, this technology has the characteristics of convenient installation and maintenance, which satisfies the conditions of large-scale real-time air quality monitoring system. It can achieve the data frequency of uploading data once a minute, and also allows users to respond immediately to sudden pollution incidents, further reducing the loss.

Of course, we cannot expect low-cost micro air quality sensors to have the high accuracy of professional monitoring stations. How to improve its accuracy becomes another problem to be solved. Therefore, in the following content, we will demonstrate how to use data science approaches to adjust the sensing results of the micro air quality sensor, so that the accuracy of the sensing data reaches the level of EPA air quality monitoring station, thereby facilitating system integration and further data application.

## Package Installation and Importing

In this article, we will use the pandas, numpy, datetime, sklearn, scipy, and joblib packages, which are pre-installed on our development platform, Google Colab, and do not need to be installed manually. Thus, we can directly use the following syntax to import the relevant packages to complete the preparations in this article.

```python
import pandas as pd
import numpy as np
from datetime import datetime, timedelta
from sklearn import linear_model, svm, tree
from sklearn import metrics as sk_metrics
from sklearn.ensemble import RandomForestRegressor
from sklearn.model_selection import train_test_split, cross_validate
from sklearn.feature_selection import f_regression

import scipy.stats as stats
import joblib
```

## Initialization and Data Access

In this example, we will use the Wanhua station of the EPA air quality station (`wanhua`), and two airboxes of Academia Sinica - Micro Air Quality Sensors in Civil IoT Taiwan, which are co-located with the EPA Wanhua station (the device IDs are `08BEAC028A52` and `08BEAC028690` respectively) as examples, and evaluate the use of three training models, i.e., Linear Regression, Random Forest Regression and Support Vector Regression (SVR). We will consider the PM2.5 concentration, temperature, relative humidity, and timestamp features of the data, combined with historical data for three different lengths of time (3 days, 5 days, 8 days), and conduct a series of explorations.

For convenience, we first make the following initial program setup based on these assumptions.

```python
SITE= "wanhua"
EPA= "EPA-Wanhua"
AIRBOXS= ['08BEAC028A52', '08BEAC028690']
DAYS= [8, 5, 3]
METHODS= ['LinearRegression', 'RandomForestRegressor', 'SVR']
METHOD_SW= { 'LinearRegression':'LinearR', 'RandomForestRegressor':'RFR', 'SVR':'SVR' }
METHOD_FUNTION= {'LinearRegression':linear_model.LinearRegression(),
                  'RandomForestRegressor': RandomForestRegressor(n_estimators= 300, random_state= 36),
                  'SVR': svm.SVR(C=20)
                }
FIELD_SW= {'s_d0':'PM25', 'pm25':'PM25', 'PM2_5':"PM25", 'pm2_5':"PM25", 's_h0':"HUM", 's_t0':'TEM'}
FEATURES_METHOD= {'PHTR':["PM25", "HR", "TEM", "HUM"],
                   'PH':['PM25','HR'],
                   'PT':['PM25','TEM'], 
                   'PR':['PM25', 'HUM'],
                   'P':['PM25'],
                   'PHR':["PM25", "HR", "HUM"], 
                   'PTR':["PM25", "TEM", "HUM"], 
                   'PHT':["PM25", "HR", "TEM"]
                    }
```

Next we consider how much data needs to be downloaded for system calibration and data fusion. As shown in the figure below, assuming that we want to obtain the calibration model of the Nth day, the data of (N-1)'s day will be used as test data to evaluate the accuracy of the calibration model. So if we set the training data to be X days, then the historical data representing days N - 2 to N - (2+X) days will be used as training data. In our scenario, we set N to the current time (today) and the maximum possible X value is 8, so we need to prepare a total of ten days of historical data for the next operation.

![Python output](figures/6-3-2-1.png)

For later use, we first specify the date to calibrate the model (Nth day: `TODAY`), the date of the test data (N-1 th day: `TESTDATE`), and the end date of the training data (N-2 th day: `ENDDATE`) with the following code .

```python
TODAY = datetime.today()
TESTDATE = (TODAY - timedelta(days=1)).date()
ENDDATE = (TODAY - timedelta(days=2)).date()
```

In this example, we use the data access API provided by Academia Sinica - Micro Air Quality Sensors. According to the specified date <YYYY-MM-DD> and device code <ID>, the sensing data of the corresponding date and device can be downloaded in CSV file format. Note that due to the limitations of the data access API, the specified date <YYYY-MM-DD> can only be a date within the past 30 days from the date of download.

```
https://pm25.lass-net.org/data/history-date.php?device_id=<ID>&date=<YYY-MM-DD>&format=CSV
```

For example, suppose we want to download the sensing data of EPA Wanhua Station on September 21, 2022, we can download it in the following ways:

```
https://pm25.lass-net.org/data/history-date.php?device_id=EPA-Wanhua&date=2022-09-21&format=CSV
```

Using this method, we write a Python function `getDF`, which can download the sensing data of the past 10 days for the given device code, and return the collected data in the form of a single DataFrame object.

```python
def getDF(id):
  temp_list = []
  for i in range(1,11):
    date = (TODAY - timedelta(days=i)).strftime("%Y-%m-%d")

    URL = "https://pm25.lass-net.org/data/history-date.php?device_id=" + id + "&date=" + date + "&format=CSV"
    temp_DF = pd.read_csv( URL, index_col=0 )
    temp_list.append( temp_DF )

    print("ID: {id}, Date: {date}, Shape: {shape}".format(id=id, date=date, shape=temp_DF.shape))

  All_DF = pd.concat( temp_list )
  return All_DF
```

Next, we download the sensing data of the first airbox installed at the EPA Wanhua station and store it in the `AirBox1_DF` object:

```python
# First AirBox device
AirBox1_DF = getDF(AIRBOXS[0])
AirBox1_DF.head()
```

```
ID: 08BEAC028A52, Date: 2022-09-29, Shape: (208, 19)
ID: 08BEAC028A52, Date: 2022-09-28, Shape: (222, 19)
ID: 08BEAC028A52, Date: 2022-09-27, Shape: (225, 19)
ID: 08BEAC028A52, Date: 2022-09-26, Shape: (230, 19)
ID: 08BEAC028A52, Date: 2022-09-25, Shape: (231, 19)
ID: 08BEAC028A52, Date: 2022-09-24, Shape: (232, 19)
ID: 08BEAC028A52, Date: 2022-09-23, Shape: (223, 19)
ID: 08BEAC028A52, Date: 2022-09-22, Shape: (220, 19)
ID: 08BEAC028A52, Date: 2022-09-21, Shape: (222, 19)
ID: 08BEAC028A52, Date: 2022-09-20, Shape: (215, 19)
```

![Python output](figures/6-3-2-2.png)

Using the same method, we download the sensing data of the second airbox installed at the EPA Wanhua station and the sensing data of the EPA Wanhua station, and stored them in the `AirBox2_DF` and `EPA_DF` objects, respectively.

```python
# Second AirBox device
AirBox2_DF = getDF(AIRBOXS[1])

# EPA station
EPA_DF = getDF(EPA)
```

## Data Preprocessing

Since our current data contains many unneeded fields, in order to avoid taking up too much memory space, we first simplify the data we use, leaving only what is needed.

```python
Col_need = ["timestamp", "s_d0", "s_t0", "s_h0"]
AirBox1_DF_need = AirBox1_DF[Col_need]
print(AirBox1_DF_need.head())
AirBox2_DF_need = AirBox2_DF[Col_need]
print(AirBox2_DF_need.head())

Col_need = ["time", "date", "pm2_5"]
EPA_DF_need = EPA_DF[Col_need]
print(EPA_DF_need.head())

del AirBox1_DF
del AirBox2_DF
del EPA_DF
```

```
                  timestamp  s_d0   s_t0  s_h0
index                                         
0      2022-09-30T00:03:28Z   9.0  29.75  71.0
1      2022-09-30T00:33:46Z  11.0  31.36  67.0
2      2022-09-30T00:39:51Z  10.0  31.50  67.0
3      2022-09-30T00:45:58Z  12.0  31.50  66.0
4      2022-09-30T00:52:05Z  12.0  31.86  66.0
                  timestamp  s_d0   s_t0  s_h0
index                                         
0      2022-09-30T00:00:31Z   9.0  29.36 -53.0
1      2022-09-30T00:07:17Z   9.0  29.50 -52.0
2      2022-09-30T00:23:47Z  10.0  30.25 -45.0
3      2022-09-30T00:34:24Z  10.0  31.11 -36.0
4      2022-09-30T00:40:31Z  11.0  31.25 -35.0
           time        date  pm2_5
index                             
0      00:00:00  2022-09-30    9.0
1      01:00:00  2022-09-30   10.0
2      02:00:00  2022-09-30   16.0
3      03:00:00  2022-09-30   19.0
4      04:00:00  2022-09-30   20.0
```

Next, in order to unify the data fields, we merge the original `date` and `time` fields of the EPA station data to generate a new `timestamp` field.

```python
EPA_DF_need['timestamp'] = pd.to_datetime( EPA_DF_need["date"] + "T" + EPA_DF_need["time"], utc=True )
print(EPA_DF_need.head())
```

```
           time        date  pm2_5                 timestamp
index                                                       
0      00:00:00  2022-09-30    9.0 2022-09-30 00:00:00+00:00
1      01:00:00  2022-09-30   10.0 2022-09-30 01:00:00+00:00
2      02:00:00  2022-09-30   16.0 2022-09-30 02:00:00+00:00
3      03:00:00  2022-09-30   19.0 2022-09-30 03:00:00+00:00
4      04:00:00  2022-09-30   20.0 2022-09-30 04:00:00+00:00
```

Due to the different temporal resolutions of the airbox and EPA station data, to align the data on both sides, we resample the airbox data from the original every five minutes to every hour.

```python
def getHourly(DF):
  DF = DF.set_index( pd.DatetimeIndex(DF["timestamp"]) )
  DF_Hourly = DF.resample('H').mean()
  DF_Hourly.reset_index(inplace=True)
  return DF_Hourly

AirBox1_DF_need_Hourly = getHourly( AirBox1_DF_need)
AirBox2_DF_need_Hourly = getHourly( AirBox2_DF_need)

EPA_DF_need_Hourly = getHourly( EPA_DF_need) # 可省略，原始數據已經是小時平均

del AirBox1_DF_need
del AirBox2_DF_need
del EPA_DF_need

print(AirBox1_DF_need_Hourly.head())
print(EPA_DF_need_Hourly.head())
```

```
                  timestamp       s_d0       s_t0       s_h0
0 2022-09-21 00:00:00+00:00   9.555556  27.024444  62.111111
1 2022-09-21 01:00:00+00:00  10.166667  30.036667  59.500000
2 2022-09-21 02:00:00+00:00  11.100000  29.919000  61.100000
3 2022-09-21 03:00:00+00:00   8.400000  29.980000  61.900000
4 2022-09-21 04:00:00+00:00   5.250000  31.308750  56.000000
                  timestamp  pm2_5
0 2022-09-21 00:00:00+00:00    6.0
1 2022-09-21 01:00:00+00:00   14.0
2 2022-09-21 02:00:00+00:00    NaN
3 2022-09-21 03:00:00+00:00   11.0
4 2022-09-21 04:00:00+00:00   10.0
```

We also replace the data fields from both sources with easily identifiable field names for easy follow-up and identification.

```python
Col_rename = {"s_d0":"PM25", "s_h0":"HUM", "s_t0":"TEM"}
AirBox1_DF_need_Hourly.rename(columns=Col_rename, inplace=True)
AirBox2_DF_need_Hourly.rename(columns=Col_rename, inplace=True)

Col_rename = {"pm2_5":"EPA_PM25"}
EPA_DF_need_Hourly.rename(columns=Col_rename, inplace=True)

print(AirBox1_DF_need_Hourly.head())
print(EPA_DF_need_Hourly.head())
```

```
                  timestamp       PM25        TEM        HUM
0 2022-09-21 00:00:00+00:00   9.555556  27.024444  62.111111
1 2022-09-21 01:00:00+00:00  10.166667  30.036667  59.500000
2 2022-09-21 02:00:00+00:00  11.100000  29.919000  61.100000
3 2022-09-21 03:00:00+00:00   8.400000  29.980000  61.900000
4 2022-09-21 04:00:00+00:00   5.250000  31.308750  56.000000
                  timestamp  EPA_PM25
0 2022-09-21 00:00:00+00:00       6.0
1 2022-09-21 01:00:00+00:00      14.0
2 2022-09-21 02:00:00+00:00       NaN
3 2022-09-21 03:00:00+00:00      11.0
4 2022-09-21 04:00:00+00:00      10.0
```

Since the two airboxes have the same hardware and the same location, we treat them as the same data source and combine the data of the two airboxes to generate the `AirBoxs_DF` object. We then intersect the new data object with the EPA air quality station data according to the time field, producing a merged `ALL_DF` object.

```python
AirBoxs_DF = pd.concat([AirBox1_DF_need_Hourly, AirBox2_DF_need_Hourly]).reset_index(drop=True)

All_DF = pd.merge( AirBoxs_DF, EPA_DF_need_Hourly, on=["timestamp"], how="inner" )

print(All_DF.head(10))
```

```
                  timestamp       PM25        TEM        HUM  EPA_PM25
0 2022-09-21 00:00:00+00:00   9.555556  27.024444  62.111111       6.0
1 2022-09-21 00:00:00+00:00   8.100000  27.083000 -65.300000       6.0
2 2022-09-21 01:00:00+00:00  10.166667  30.036667  59.500000      14.0
3 2022-09-21 01:00:00+00:00   8.100000  30.194000 -57.100000      14.0
4 2022-09-21 02:00:00+00:00  11.100000  29.919000  61.100000       NaN
5 2022-09-21 02:00:00+00:00   9.000000  30.418889 -59.222222       NaN
6 2022-09-21 03:00:00+00:00   8.400000  29.980000  61.900000      11.0
7 2022-09-21 03:00:00+00:00   7.100000  29.408000 -64.400000      11.0
8 2022-09-21 04:00:00+00:00   5.250000  31.308750  56.000000      10.0
9 2022-09-21 04:00:00+00:00   4.777778  29.882222 -64.000000      10.0
```

We first exclude the presence of null values (NaN) in the data.

```python
All_DF.dropna(how="any", inplace=True)
All_DF.reset_index(inplace=True, drop=True)
print(All_DF.head(10))
```

```
                  timestamp       PM25        TEM        HUM  EPA_PM25
0 2022-09-21 00:00:00+00:00   9.555556  27.024444  62.111111       6.0
1 2022-09-21 00:00:00+00:00   8.100000  27.083000 -65.300000       6.0
2 2022-09-21 01:00:00+00:00  10.166667  30.036667  59.500000      14.0
3 2022-09-21 01:00:00+00:00   8.100000  30.194000 -57.100000      14.0
4 2022-09-21 03:00:00+00:00   8.400000  29.980000  61.900000      11.0
5 2022-09-21 03:00:00+00:00   7.100000  29.408000 -64.400000      11.0
6 2022-09-21 04:00:00+00:00   5.250000  31.308750  56.000000      10.0
7 2022-09-21 04:00:00+00:00   4.777778  29.882222 -64.000000      10.0
8 2022-09-21 05:00:00+00:00   5.000000  30.033333  60.333333       8.0
9 2022-09-21 05:00:00+00:00   4.500000  28.777500 -66.875000       8.0
```

Finally, since the daily hour value information will be used when building the model, we add an `HR` field to `All_DF` that contains the content of the hour value in the `timestamp`.

```python
def return_HR(row):
    row['HR'] = int(row[ "timestamp" ].hour)
    return row

All_DF = All_DF.apply(return_HR , axis=1)
print(All_DF.head(10))
```

```
                  timestamp       PM25        TEM        HUM  EPA_PM25  HR
0 2022-09-21 00:00:00+00:00   9.555556  27.024444  62.111111       6.0   0
1 2022-09-21 00:00:00+00:00   8.100000  27.083000 -65.300000       6.0   0
2 2022-09-21 01:00:00+00:00  10.166667  30.036667  59.500000      14.0   1
3 2022-09-21 01:00:00+00:00   8.100000  30.194000 -57.100000      14.0   1
4 2022-09-21 03:00:00+00:00   8.400000  29.980000  61.900000      11.0   3
5 2022-09-21 03:00:00+00:00   7.100000  29.408000 -64.400000      11.0   3
6 2022-09-21 04:00:00+00:00   5.250000  31.308750  56.000000      10.0   4
7 2022-09-21 04:00:00+00:00   4.777778  29.882222 -64.000000      10.0   4
8 2022-09-21 05:00:00+00:00   5.000000  30.033333  60.333333       8.0   5
9 2022-09-21 05:00:00+00:00   4.500000  28.777500 -66.875000       8.0   5
```

## Calibration Model Establishment and Validation

After completing the data preparation, we next start building candidate calibration models. Since we discussed a total of 3 regression methods, 3 historical data lengths, and 8 feature combinations in this case, we will generate a total of 3 x 3 x 8 = 72 candidate models.

First, we designed the `SlideDay` function to retrieve a specific length of historical data when building a calibration model. The function returns the input data `Hourly_DF` from `enddate` to the total `day` day data according to the input historical data length `day`.

```python
def SlideDay( Hourly_DF, day, enddate ):
    startdate= enddate- timedelta( days= (day-1) )
    time_mask= Hourly_DF["timestamp"].between( pd.Timestamp(startdate, tz='utc'), pd.Timestamp(enddate, tz='utc') )
return Hourly_DF[ time_mask ]
```

Next, we organize the sensing data `Training_DF` of the `site` station from `enddate` to `day` days before, and organize it into training data according to the `feature` feature combination. Then we apply the `method` regression method, so that the predicted value produced by the generated calibration model can approximate the data value of the `EPA_PM25` field in the training data. 

We calculate both the Mean Absolute Error (MAE) and the Mean Squared Error (MSE) of the corrected model itself. Among them, the mean absolute error (MAE) is the sum of the absolute values of the difference between the target value and the predicted value, which may reflect the actual situation of the predicted value error. The smaller the value, the better the performance. The mean squared error (MSE) is the mean of the square of the difference between the predicted value and the actual observed value. The smaller the MSE value, the better the accuracy of the prediction model in describing the experimental data.

```python
def BuildModel( site, enddate, feature, day, method, Training_DF ):

    X_train = Training_DF[ FEATURES_METHOD[ feature ] ]
    Y_train = Training_DF[ "EPA_PM25" ]
    model_result = {}
    model_result["site"], model_result["day"], model_result["feature"], model_result["method"] = site, day, feature, method
    model_result["datapoints"], model_result["modelname"] = X_train.shape[0], (site + "_" + str(day) + "_" + METHOD_SW[method] + "_" + feature)
    model_result["date"] = enddate.strftime( "%Y-%m-%d" )
    # add timestamp field
    Now_Time = datetime.utcnow().strftime( "%Y-%m-%d %H:%M:%S" )
    model_result['create_timestamp_utc'] = Now_Time

    ### training model ###
    print( "[BuildR]-\"{method}\" with {day}/{feature}".format(method=method, day=day, feature=feature) )
    
    # fit
    lm = METHOD_FUNTION[ method ]
    lm.fit( X_train, Y_train )

    # get score
    Y_pred = lm.predict( X_train )
    model_result['Train_MSE'] = MSE = sk_metrics.mean_squared_error( Y_train, Y_pred )
    model_result['Train_MAE'] = sk_metrics.mean_absolute_error( Y_train, Y_pred )

    return model_result, lm
```

In addition to evaluating the MAE and MSE of the training data when building the calibrated model, we also consider the prediction performance of the calibrated model on non-training data. Therefore, for the model `lm`, according to the feature data `feature` that it needs to bring in, as well as the test data, we generate the predicted value, and compare it with the `EPA_PM25` field in the test data to calculate the MAE and MSE respectively. , as a reference for subsequent evaluation of the applicability of different calibration models.

```python
def TestModel( site, feature, modelname, Testing_DF, lm ):

    X_test = Testing_DF[ FEATURES_METHOD[ feature ] ]
    Y_test = Testing_DF[ "EPA_PM25" ]

    # add timestamp field
    Now_Time = datetime.utcnow().strftime( "%Y-%m-%d %H:%M:%S" )

    ### testing model ###
    # predict
    Y_pred = lm.predict( X_test )

    # get score
    test_result = {}
    test_result["test_MSE"] = round( sk_metrics.mean_squared_error( Y_test, Y_pred ), 3)
    test_result["test_MAE"] = round( sk_metrics.mean_absolute_error( Y_test, Y_pred ), 3)

    return test_result
```

Finally, we integrate the just completed `SlideDay`, `BuildModel` and `TestModel`, and complete a total of 72 calibration models. For each calibrated model, we compute MAE and MSE for training and testing data separately, and store all results in an `AllResult_DF` object.

```python
AllResult_list = []

for day in DAYS:
  for method in METHODS:
    for feature in FEATURES_METHOD:
      Training_DF = SlideDay(All_DF, day, ENDDATE)[ FEATURES_METHOD[feature] + ["EPA_PM25"] ]
      result, lm = BuildModel( SITE, TESTDATE, feature, day, method, Training_DF )
      test_result = TestModel(SITE, feature, result["modelname"], SlideDay(All_DF, 1, TESTDATE), lm)
      R_DF = pd.DataFrame.from_dict( [{ **result, **test_result }] )
      AllResult_list.append( R_DF )

AllResult_DF = pd.concat(AllResult_list)
AllResult_DF.head()
```

```
[BuildR]-"LinearRegression" with 8/PHTR
[BuildR]-"LinearRegression" with 8/PH
[BuildR]-"LinearRegression" with 8/PT
[BuildR]-"LinearRegression" with 8/PR
[BuildR]-"LinearRegression" with 8/P
[BuildR]-"LinearRegression" with 8/PHR
[BuildR]-"LinearRegression" with 8/PTR
[BuildR]-"LinearRegression" with 8/PHT
[BuildR]-"RandomForestRegressor" with 8/PHTR
[BuildR]-"RandomForestRegressor" with 8/PH
[BuildR]-"RandomForestRegressor" with 8/PT
[BuildR]-"RandomForestRegressor" with 8/PR
[BuildR]-"RandomForestRegressor" with 8/P
[BuildR]-"RandomForestRegressor" with 8/PHR
[BuildR]-"RandomForestRegressor" with 8/PTR
[BuildR]-"RandomForestRegressor" with 8/PHT
[BuildR]-"SVR" with 8/PHTR
[BuildR]-"SVR" with 8/PH
[BuildR]-"SVR" with 8/PT
[BuildR]-"SVR" with 8/PR
[BuildR]-"SVR" with 8/P
[BuildR]-"SVR" with 8/PHR
[BuildR]-"SVR" with 8/PTR
[BuildR]-"SVR" with 8/PHT
[BuildR]-"LinearRegression" with 5/PHTR
[BuildR]-"LinearRegression" with 5/PH
[BuildR]-"LinearRegression" with 5/PT
[BuildR]-"LinearRegression" with 5/PR
[BuildR]-"LinearRegression" with 5/P
[BuildR]-"LinearRegression" with 5/PHR
[BuildR]-"LinearRegression" with 5/PTR
[BuildR]-"LinearRegression" with 5/PHT
[BuildR]-"RandomForestRegressor" with 5/PHTR
[BuildR]-"RandomForestRegressor" with 5/PH
[BuildR]-"RandomForestRegressor" with 5/PT
[BuildR]-"RandomForestRegressor" with 5/PR
[BuildR]-"RandomForestRegressor" with 5/P
[BuildR]-"RandomForestRegressor" with 5/PHR
[BuildR]-"RandomForestRegressor" with 5/PTR
[BuildR]-"RandomForestRegressor" with 5/PHT
[BuildR]-"SVR" with 5/PHTR
[BuildR]-"SVR" with 5/PH
[BuildR]-"SVR" with 5/PT
[BuildR]-"SVR" with 5/PR
[BuildR]-"SVR" with 5/P
[BuildR]-"SVR" with 5/PHR
[BuildR]-"SVR" with 5/PTR
[BuildR]-"SVR" with 5/PHT
[BuildR]-"LinearRegression" with 3/PHTR
[BuildR]-"LinearRegression" with 3/PH
[BuildR]-"LinearRegression" with 3/PT
[BuildR]-"LinearRegression" with 3/PR
[BuildR]-"LinearRegression" with 3/P
[BuildR]-"LinearRegression" with 3/PHR
[BuildR]-"LinearRegression" with 3/PTR
[BuildR]-"LinearRegression" with 3/PHT
[BuildR]-"RandomForestRegressor" with 3/PHTR
[BuildR]-"RandomForestRegressor" with 3/PH
[BuildR]-"RandomForestRegressor" with 3/PT
[BuildR]-"RandomForestRegressor" with 3/PR
[BuildR]-"RandomForestRegressor" with 3/P
[BuildR]-"RandomForestRegressor" with 3/PHR
[BuildR]-"RandomForestRegressor" with 3/PTR
[BuildR]-"RandomForestRegressor" with 3/PHT
[BuildR]-"SVR" with 3/PHTR
[BuildR]-"SVR" with 3/PH
[BuildR]-"SVR" with 3/PT
[BuildR]-"SVR" with 3/PR
[BuildR]-"SVR" with 3/P
[BuildR]-"SVR" with 3/PHR
[BuildR]-"SVR" with 3/PTR
[BuildR]-"SVR" with 3/PHT
```

![Python output](figures/6-3-4-1.png)

## Best Calibration Model of the Day

When discussing the "best calibrated model of the day", it must be admitted that the so-called "best model" is only available after the end of the day. That is to say, the MAE and MSE of different candidate models can only be obtained after collecting complete 24-hour data, and the best model can be obtained after comparative analysis. Therefore, it will not be possible to systematically generate a truly optimized calibration model before the end of today's 24 hours.

However, based on practical needs, we often need to have a calibration model that can be applied in the data production process. So, in practice, we usually assume that "yesterday's best model will also perform well today" and use that as a way to correct that day's data. For example, assuming we use MSE as the criterion for deciding the best model, we can get information about the best model using the following syntax:

```python
FIELD= "test_MSE"
BEST= AllResult_DF[ AllResult_DF[FIELD]== AllResult_DF[FIELD].min() ]
BEST
```

![Python output](figures/6-3-5-1.png)

Next, to adapt to today's situation, we use yesterday's best calibrated model parameters (historical data length, regression method, feature data combination), recalculate training and test data from today's date range, and regenerate today's calibrated model .

```python
BEST_DC= BEST.to_dict(orient="index")[0]

Training_DF= SlideDay(All_DF, BEST_DC["day"], TESTDATE)[ FEATURES_METHOD[BEST_DC["feature"]]+ ["EPA_PM25"] ]
result, lm= BuildModel( SITE, TODAY, BEST_DC["feature"], BEST_DC["day"], BEST_DC["method"], Training_DF )

result
```

```
[BuildR]-"SVR" with 3/PHT
{'site': 'wanhua',
 'day': 3,
 'feature': 'PHT',
 'method': 'SVR',
 'datapoints': 80,
 'modelname': 'wanhua_3_SVR_PHT',
 'date': '2022-09-30',
 'create_timestamp_utc': '2022-09-30 11:19:48',
 'Train_MSE': 3.91517342356589,
 'Train_MAE': 1.42125724796098}
```

From this example, we can see that the newly produced calibration model has a `Train_MSE` of 3.915, which is much higher than the original 2.179. However, since we have no way to know the real best model of the day before the end of the day, we can only apply yesterday's best model if there is no other better choice. Finally, we export the .joblib file with the following syntax and release the model for sharing (please refer to the reference materials for details).

```python
model_dumpname= result["modelname"]+ ".joblib"

MODEL_OUTPUT_PATH= ""
try:
    joblib.dump( lm, MODEL_OUTPUT_PATH+ model_dumpname )
    print( "[BuildR]-dump {}".format( MODEL_OUTPUT_PATH+model_dumpname ) )
except Exceptionas e:
    print( "ERROR! [dump model] {}".format( result["modelname"] ) )
    error_msg(e)
```

```
[BuildR]-dump wanhua_3_SVR_PHT.joblib
```

## Calibration Results

The joint calibration method described in this article has been officially applied to "Academia Sinica - Micro Air Quality Sensors" in Civil IoT Taiwan from 2020/5, and the daily output calibration model has been published on the [Dynamic Calibration Model](https://pm25.lass-net.org/DCF/) website. In the implementation, a total of 31 EPA air quality monitoring stations were selected to install two airboxes, and we considered 3 historical data lengths, 8 data feature combinations, and 7 regression methods (i.e. a total of 3 x 8 x 7 = 168 combinations) to generate the best daily calibration model for each of these 31 sites. Then, for the sensing data of each airbox, the best calibration model of the station closest to its geographical location is used as the application model of its data calibration, and its sensing data is generated and published. From the observation of the actual operation after the mechanism is launched, the data difference between the micro air quality sensor device and the EPA air quality monitoring station can be effectively reduced (as shown in the figure below). This achievement also established a good cooperation model for cross-system data integration of air quality monitoring.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/eeafd803-98e3-426f-8fbe-dcea8d5d90e6/Untitled.png)

## References

- Dynamic Calibration Model Status Report ([https://pm25.lass-net.org/DCF/](https://pm25.lass-net.org/DCF/))
- scikit-learn: machine learning in Python ([https://scikit-learn.org/stable/](https://scikit-learn.org/stable/))
- Joblib: running Python functions as pipeline jobs ([https://joblib.readthedocs.io/](https://joblib.readthedocs.io/))
- Jason Brownlee, [Save and Load Machine Learning Models in Python with scikit-learn](https://machinelearningmastery.com/save-load-machine-learning-models-python-scikit-learn/), Machine Learning Mastery ([https://machinelearningmastery.com/save-load-machine-learning-models-python-scikit-learn/](https://machinelearningmastery.com/save-load-machine-learning-models-python-scikit-learn/))
