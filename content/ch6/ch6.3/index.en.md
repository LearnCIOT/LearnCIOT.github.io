---
title : "6.3. Joint Data Calibration"
weight : 30
socialshare: true
description : "We use air quality category data of the Civil IoT Taiwan project to demonstrate the dynamic calibration algorithm for Taiwanese micro air quality sensors and official monitoring stations. In a learning-by-doing way, from data preparation, feature extraction, to machine learning, data analysis, statistics and induction, the principle and implementation process of the multi-source sensor dynamic calibration model algorithm are reproduced step by step, allowing readers to experience how to gradually realize by superimposing basic data analysis and machine learning steps to achieve advanced and practical data application services."
tags: ["Python", "Air" ]
levels: ["advanced" ]
authors: ["Quen Luo"]
---




{{< toc >}}

In this article, we're going to look at how air quality data from Civil IoT Taiwan can be used. We'll explain how we can calibrate two types of air quality sensors using data science, so they work together better. This helps us get a more complete picture of air quality. We'll focus on two kinds of air quality sensors:

1. **Environmental Protection Agency (EPA) Air Quality Monitoring Stations:** These are the traditional, professional stations that are big, expensive, and very accurate. They're not in every neighborhood because they're costly to set up and maintain. Instead, they're placed in specific spots by local environmental agencies. As of July 2022, Taiwan has 81 of these official monitoring stations.
2. **Micro Air Quality Sensors:** These are smaller, cheaper sensors that are part of the Internet of Things (IoT) network. They're easier to put up in lots of places because they're less expensive and simpler to install and look after. These sensors can quickly send data, even every minute, which is great for keeping an eye on air quality in real-time and reacting fast to sudden pollution. But they're not as accurate as the big EPA stations.

The big question is, how can we make these small, affordable sensors as accurate as the professional ones? We're going to show you how we can do this with data science techniques. By improving the accuracy of these micro sensors, we can better integrate different types of sensors and make more use of the data they collect in the future.

## Package Installation and Importing


In this article, we're going to make use of several powerful Python packages that are already pre-installed on Google Colab, our development platform. This means we won't need to go through the hassle of manually installing them. We'll be using:

- `pandas`: Great for data manipulation and analysis.
- `numpy`: Essential for numerical operations.
- `datetime`: Helpful for handling dates and times.
- `sklearn`: A go-to library for machine learning.
- `scipy`: Useful for scientific and technical computing.
- `joblib`: Handy for saving and loading Python objects.

Since these packages are ready to use on Google Colab, we can simply import them into our project. We'll do this with standard Python import statements, allowing us to quickly set up our working environment and dive into the exciting work we have planned in this article.

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

For this example, we'll focus on comparing air quality data from the Wanhua station of the EPA air quality monitoring network (`wanhua`) with data from two Micro Air Quality Sensors of Academia Sinica in the Civil IoT Taiwan network. These sensors, identified by their device IDs `08BEAC028A52` and `08BEAC028690`, are located at the same place as the EPA Wanhua station, giving us a unique opportunity to compare their readings directly.

We'll evaluate the data using three different machine learning models:

- Linear Regression: A straightforward approach for predicting a quantitative response.
- Random Forest Regression: A more complex model that uses multiple decision trees to make predictions.
- Support Vector Regression (SVR): An advanced technique that can capture complex relationships in data.

Our analysis will focus on several key features of the air quality data: PM2.5 concentration, temperature, relative humidity, and the timestamp of each reading. We'll also incorporate historical data over three different time spans - 3 days, 5 days, and 8 days - to see how the length of the historical data affects our models' performance.

To get started, let’s set up our initial programming environment with these specifics in mind. This will involve loading the necessary data and preparing our analysis tools to handle this unique set of data and models.

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

To determine the appropriate amount of data needed for system calibration and data fusion, let's visualize the process. Imagine you want to create a calibration model for a specific day, which we'll call the Nth day. The data from the day before that, the (N-1)th day, will be used to test and evaluate the accuracy of this model.

Now, if you decide to use X days' worth of data for training the model, you will need historical data from the (N-2)nd day going back to the (N-2-X)th day. This range of days will serve as the training dataset.

In our case, we're setting N to represent the current day (today), and we've decided that the maximum length for X (our training data period) is 8 days. Therefore, to proceed with our calibration and data fusion, we need to prepare a total of 10 days of historical data. This timeframe includes 8 days for training and an additional day each for testing and the current day’s data.

This approach ensures that we have a comprehensive dataset, allowing our models to learn effectively from recent historical data and be tested against the most immediate past data for validation of their accuracy.

![Python output](figures/6-3-2-1.png)

To set up our analysis, we need to define three key dates in our code: the date for calibrating the model (`TODAY`), the date for the test data (`TESTDATE`), and the end date of the training data (`ENDDATE`). These dates correspond to the Nth day, the N-1th day, and the N-2th day, respectively. Here's how you can do this in Python:

```python
TODAY = datetime.today()
TESTDATE = (TODAY - timedelta(days=1)).date()
ENDDATE = (TODAY - timedelta(days=2)).date()
```

In this case, we're utilizing the data access API from Academia Sinica's Micro Air Quality Sensors. This allows you to download sensing data in a CSV file format for a specific date (input as <YYYY-MM-DD>) and device (using the device code <ID>). However, it's important to remember that this API only lets you access data from the past 30 days from your current download date. So, the date you choose must be within this 30-day window.

```
https://pm25.lass-net.org/data/history-date.php?device_id=<ID>&date=<YYY-MM-DD>&format=CSV
```

If we're looking to get the sensor data from the EPA Wanhua Station for September 21, 2022, we can do so by following these steps:

```
https://pm25.lass-net.org/data/history-date.php?device_id=EPA-Wanhua&date=2022-09-21&format=CSV
```

We've created a Python function named `getDF`. This function can retrieve the sensor data from the past 10 days for a specified device code. It then compiles this data and returns it as a single DataFrame object.

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

Next, we'll download the data from the first air quality monitoring device, known as an airbox, located at the EPA Wanhua station. This data will be saved in an object we're calling `AirBox1_DF`:

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

To proceed in the same way, we obtained the sensor data from the second airbox at the EPA Wanhua station as well as the sensor data from the EPA Wanhua station itself. These sets of data were then saved in two different objects: `AirBox2_DF` for the airbox data and `EPA_DF` for the station's data.

```python
# Second AirBox device
AirBox2_DF = getDF(AIRBOXS[1])

# EPA station
EPA_DF = getDF(EPA)
```

## Data Preprocessing

To ensure we don't use too much memory, we're streamlining our data by keeping only the essential information and removing any unnecessary details.

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

Next, we're combining the `date` and `time` information from the EPA station data into a new field called `timestamp`. This helps us have a unified way of looking at when the data was collected.

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

The airbox and EPA station data are collected at different times - the airbox data every five minutes and the EPA station data every hour. To make them match, we're adjusting the airbox data to also be on an hourly basis.

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

We're renaming the data fields from both the airbox and EPA stations to names that are easier to understand and track.

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

Since we have two airboxes that are identical and in the same place, we're treating them as a single data source. We're combining their data into one object, which we're calling `AirBoxs_DF`. After that, we're merging this with the EPA station data based on the time they were recorded. This creates a new, combined data object named `ALL_DF`.

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

We're making sure to remove any missing or incomplete data, marked as NaN (Not a Number), from our dataset.

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

Lastly, we're adding a new field called `HR` to `ALL_DF`. This field will show the hour when the data was collected, which is taken from the `timestamp` field. This hour information is important for the model we're building.

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

After we're done preparing the data, we move on to create potential calibration models. In this case, we're looking at 3 types of regression methods, 3 lengths of historical data, and 8 different combinations of features. This means we'll end up with a total of 72 (3 x 3 x 8) candidate models.

First up, we use a tool called `SlideDay`. This function helps us gather a specific amount of historical data needed for building a calibration model. It works by taking the data from `Hourly_DF` and selecting data from the `enddate` going back for the number of days specified by `day`.

```python
def SlideDay( Hourly_DF, day, enddate ):
    startdate= enddate- timedelta( days= (day-1) )
    time_mask= Hourly_DF["timestamp"].between( pd.Timestamp(startdate, tz='utc'), pd.Timestamp(enddate, tz='utc') )
return Hourly_DF[ time_mask ]
```

Then, we arrange the sensor data `Training_DF` for a particular monitoring station, referred to as `site`, covering the period from `enddate` to `day` days before. This data is organized based on the selected features (`feature` feature combination). After this, we apply one of the regression methods (`method`) to the data. Our goal here is to ensure that the predictions made by our calibration model are as close as possible to the `EPA_PM25` values in the training data.

To evaluate the accuracy of our model, we use two methods: Mean Absolute Error (MAE) and Mean Squared Error (MSE). MAE measures how far off our predictions are on average, by calculating the average of the absolute differences between predicted and actual values. A lower MAE means better accuracy. MSE, on the other hand, calculates the average of the squares of these differences. Again, a lower MSE indicates better model performance.

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

Beyond just looking at the MAE and MSE for the training data, we also assess how well our model performs on data it hasn't been trained on. So, for the `lm` model, we generate predictions based on the required feature data (`feature`) and test data, and then compare these predictions to the `EPA_PM25` values in the test data to calculate both MAE and MSE. This helps us gauge the effectiveness of different calibration models.

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

To wrap up, we bring together the `SlideDay`, `BuildModel`, and `TestModel` processes to complete all 72 calibration models. We calculate the MAE and MSE for both the training and testing data for each model and compile all these results into an `AllResult_DF` object for further analysis.

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

When we talk about the "best calibrated model of the day" in data analysis, it's important to understand that we can only identify this "best model" after the day has ended. This is because we need to collect a full day's worth of data (24 hours) before we can accurately measure the performance of different models. We use two methods, Mean Absolute Error (MAE) and Mean Squared Error (MSE), to evaluate these models. Only after comparing these can we determine which model was the best for that day.

However, in the real world, we often need a calibrated model during the day, not after it has ended. So, what we typically do is assume that the best model from yesterday will also work well for today. We use this model to correct and process today's data. For instance, if we're using MSE to judge our best model, we can identify yesterday's top performer using a specific method.

```python
FIELD= "test_MSE"
BEST= AllResult_DF[ AllResult_DF[FIELD]== AllResult_DF[FIELD].min() ]
BEST
```

![Python output](figures/6-3-5-1.png)

To make this model work for today, we take the best parameters from yesterday's model (things like how much historical data it looks at, the method of analysis it uses, and the types of data it considers) and apply them to today's data. This helps us create a new model for today.

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

For example, let's say that the new model we create for today has a `Train_MSE` of 3.915. This is higher than yesterday's 2.179, which means it's not as accurate. However, since we can't know today's best model until the day is over, we use yesterday's model as the next best thing.

Finally, we save this model in a special file format (.joblib) and share it for others to use. You can find more details on how to do this in the reference materials provided.

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

Since May 2020, the method we discuss in this article has been put to use in the "Academia Sinica - Micro Air Quality Sensors" as part of the Civil IoT Taiwan initiative. We've shared the daily calibration models developed through this method on the [Dynamic Calibration Model](https://pm25.lass-net.org/DCF/) website.

Here's what we did: We picked 31 air quality monitoring stations run by the Environmental Protection Agency (EPA) and set up two airboxes at each. To find the most effective daily calibration model for each location, we experimented with a variety of factors. This included using three different lengths of historical data, combining eight types of data features, and applying seven different regression techniques. Altogether, this meant testing 168 different combinations (3 historical data lengths x 8 data features x 7 regression methods).

Once we determined the best calibration model for each station, we used it to calibrate the data from the nearest airbox. This means the air quality data from each airbox is adjusted using the most suitable model based on its location. After implementing this system, we observed a significant improvement: the discrepancy between the readings from our micro air quality sensors and those from the EPA's monitoring stations was greatly reduced. The success of this approach not only improved our air quality data accuracy but also set a strong example of how data from different systems can be effectively integrated for air quality monitoring. The figure below illustrates the reduced data discrepancy after implementing this method.

![Open In Colab](figures/6-3-6-1.png)

## References

- Dynamic Calibration Model Status Report ([https://pm25.lass-net.org/DCF/](https://pm25.lass-net.org/DCF/))
- scikit-learn: machine learning in Python ([https://scikit-learn.org/stable/](https://scikit-learn.org/stable/))
- Joblib: running Python functions as pipeline jobs ([https://joblib.readthedocs.io/](https://joblib.readthedocs.io/))
- Jason Brownlee, [Save and Load Machine Learning Models in Python with scikit-learn](https://machinelearningmastery.com/save-load-machine-learning-models-python-scikit-learn/), Machine Learning Mastery ([https://machinelearningmastery.com/save-load-machine-learning-models-python-scikit-learn/](https://machinelearningmastery.com/save-load-machine-learning-models-python-scikit-learn/))
