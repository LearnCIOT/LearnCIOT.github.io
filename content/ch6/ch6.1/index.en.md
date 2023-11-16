---
title : "6.1. Machine Learning Preliminaries"
weight : 10
socialshare : true
description : "We use air quality and water level data, combined with weather observations, using machine learning for data classification and data grouping. We demonstrate the standard process of machine learning and introduce how to further predict data through data classification and how to further explore data through data grouping."
tags: ["Python", "Water", "Air" ]
levels: ["intermediate" ]
authors: ["Jen-Wei Huang","Hung-Ying Chen"]
---

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/drive/14xfpf7Nv_1uadSeJh3vC61HL7r4z6KT8?usp=sharing)

{{< toc >}}

## Preamble

In the earlier sections, we've explored the extensive open data available through Civil IoT Taiwan. We've also looked at how this data can be analyzed and processed, focusing on different aspects of time and space. In this section, we're going to delve deeper into how machine learning can be applied to this data. We'll introduce two fundamental machine learning concepts: classification and clustering.

### Classification

In machine learning, a common task is called a classification problem. Think of it like sorting things into different categories. Imagine we have a bunch of data points, let's call this group `X`, and we've sorted each one into specific categories, which we'll call `Y`. The challenge in classification is to create a smart tool, known as a classifier, that can look at new, unsorted data (`X'`) and correctly predict which category (`Y'`) they belong to, based on what it has learned from our sorted data.

So, the main goal here is to build a really good classifier. To do this, we start by creating a model. This model is trained using our categorized data, which teaches it to recognize patterns and make good predictions. We want our model to be good at understanding how our data is spread out and then use it to guess the categories of new, unseen data.

This whole process is what's called supervised learning in the field of machine learning. Some popular types of classifiers are *[Nearest Neighbors](https://en.wikipedia.org/wiki/K-nearest_neighbors_algorithm)*, *[SVM Classifier](https://en.wikipedia.org/wiki/Support-vector_machine)*, *[Decision Tree](https://en.wikipedia.org/wiki/Decision_tree)*, and *[Random Forest](https://en.wikipedia.org/wiki/Random_forest)*. We won't go into the details of how each model works in future articles but will use them as tools. For those interested in learning more about these models, plenty of resources are out there for a deeper dive.

### Clustering

Clustering issues are a lot like classification issues, with a key difference. In classification, we use data with known labels to understand data with unknown labels. Clustering, on the other hand, is like starting from scratch. It involves organizing data into different groups without any prior labels.

To put it in a more mathematical way, imagine we have a bunch of data, `X`, with no labels at all. The challenge of clustering is to split this `X` data into `k` groups using a specific method. The goal is to make sure that data within each group are very similar to each other, but quite different from data in other groups.

The methods we use for clustering depend heavily on the nature of the data. These methods look at how similar or different the data points are from one another. They try to bring similar data points together, while keeping the different ones separate. Some well-known clustering techniques include [K-Means](https://en.wikipedia.org/wiki/K-means_clustering), [DBSCAN](https://en.wikipedia.org/wiki/DBSCAN), [Hierarchical Clustering](https://en.wikipedia.org/wiki/Hierarchical_clustering), and [BIRCH](https://en.wikipedia.org/wiki/BIRCH). In future articles, we won't go into the details of each method. Instead, we'll use them as tools. If you're curious about these methods, you can find more information in other resources to explore them further.

## Package Installation and Importing

In this article, we're working with several software packages that are already set up and ready to use on Google Colab, our development platform. These include `andas`, `numpy`, `matplotlib`, `json`, `os`, `glob`, `math`, `seaborn`, `tqdm`, `datetime`, `geopy`, `scipy`, and `warnings`. You won't need to install these separately.

However, there are a few additional tools we'll need which aren't included in Google Colab by default. These are the `pyCIOT`, `fastdtw`, and `sklearn` packages, along with the `TaipeiSansTCBeta-Regular` font. We'll need to install these ourselves.

```python
!pip3 install fastdtw --quiet
!pip3 install scikit-learn --quiet
!pip3 install pyCIOT --quiet
!wget -q -O TaipeiSansTCBeta-Regular.ttf https://drive.google.com/uc?id=1eGAsTN1HBpJAkeVM57_C7ccp7hbgSz3_&export=download
```

Once the installation is finished, we can use the commands below to import the necessary packages, setting the stage for the tasks discussed in this article.

```python
from pyCIOT.data import *
import json, os, glob, math
import numpy as np
import pandas as pd
import matplotlib as mpl
import matplotlib.pyplot as plt
from matplotlib.font_manager import fontManager
from tqdm import tqdm_notebook as tqdm
from datetime import datetime, timedelta

import seaborn as sns
sns.set(font_scale=0.8)
fontManager.addfont('TaipeiSansTCBeta-Regular.ttf')
mpl.rc('font', family='Taipei Sans TC Beta')

import warnings
warnings.simplefilter(action='ignore')

import geopy.distance
from scipy.spatial.distance import euclidean
from fastdtw import fastdtw

import sklearn
from sklearn.metrics import accuracy_score, confusion_matrix
from sklearn.neural_network import MLPClassifier
from sklearn.neighbors import KNeighborsClassifier
from sklearn.svm import SVC
from sklearn.gaussian_process.kernels import RBF
from sklearn.tree import DecisionTreeClassifier
from sklearn.ensemble import RandomForestClassifier, AdaBoostClassifier
from sklearn.naive_bayes import GaussianNB
from sklearn.discriminant_analysis import QuadraticDiscriminantAnalysis
from sklearn.cluster import KMeans
```

Now, let's take a look at how we can organize and group data using the water level and air quality information from Civil IoT Taiwan.

## Case 1: Location Type Classification of Air Quality Sensors

In this example, we're exploring how to use the data from the Environmental Protection Agency's tiny air quality sensors (known as '`OBS:EPA_IoT`') within the Civil IoT Taiwan framework to show how to categorize data.

First, we utilize the `Air().get_data()` function from the pyCIOT library to fetch the most recent readings from all the EPA's micro air quality sensors. Keep in mind, this process might take a bit of time because there's a lot of data to gather.

```python
Source = 'OBS:EPA_IoT'
Data = Air().get_data(src=Source)
Data = [datapoint for datapoint in Data if len(datapoint['data']) == 3 and 'areaType' in datapoint['properties'].keys()]
print(json.dumps(Data[0], indent=4, ensure_ascii=False))
```

```
{
    "name": "智慧城鄉空品微型感測器-10287974676",
    "description": "智慧城鄉空品微型感測器-10287974676",
    "properties": {
        "city": "新北市",
        "areaType": "社區",
        "isMobile": "false",
        "township": "鶯歌區",
        "authority": "行政院環境保護署",
        "isDisplay": "true",
        "isOutdoor": "true",
        "stationID": "10287974676",
        "locationId": "TW040203A0507221",
        "Description": "廣域SAQ-210",
        "areaDescription": "鶯歌區"
    },
    "data": [
        {
            "name": "Relative humidity",
            "description": "相對溼度",
            "timestamp": "2022-08-05T06:51:29.000Z",
            "value": 70.77
        },
        {
            "name": "Temperature",
            "description": "溫度",
            "timestamp": "2022-08-05T06:51:29.000Z",
            "value": 33.78
        },
        {
            "name": "PM2.5",
            "description": "細懸浮微粒 PM2.5",
            "timestamp": "2022-08-05T06:51:29.000Z",
            "value": 9.09
        }
    ],
    "location": {
        "latitude": 24.9507,
        "longitude": 121.3408416,
        "address": null
    }
}
```

In the data collected from each sensor, you'll find details like temperature, relative humidity, and PM2.5 levels (a measure of air pollution). Additionally, each sensor's basic information is included, such as the city, town, sensor ID, location ID, and the type of area it's in, etc. For our purpose, we're going to focus on the "location type" as a label for our data and create a classifier using the sensor data (temperature, relative humidity, and PM2.5 levels). First, let's take a look at what the current "AreaType" data looks like.

```python
Label = list(dict.fromkeys([datapoint['properties']['areaType'] for datapoint in Data if datapoint['properties']['areaType']]))
count = dict.fromkeys(Label, 0)
for datapoint in Data:
    count[datapoint['properties']['areaType']] += 1
print("Before data cleaning, There are {} records.".format(len(Data)))
print(json.dumps(count, indent=4, ensure_ascii=False))
```

```
Before data cleaning, There are 8620 records.
{
    "社區": 223,
    "交通": 190,
    "一般社區": 2021,
    "工業": 12,
    "測站比對": 66,
    "工業區": 3333,
    "交通區": 683,
    "鄰近工業區社區": 948,
    "輔助區": 165,
    "特殊區(民眾陳情熱區)": 143,
    "特殊區(敏感族群聚集區)": 200,
    "特殊區(測站比對)": 32,
    "輔助區(無測站區)": 4,
    "工業感測": 196,
    "特殊感測": 4,
    "輔助感測": 295,
    "交通感測": 102,
    "機動感測": 2,
    "社區感測": 1
}
```

### Data Cleansing

We started with 8,620 records from 19 different types of places. To make things clearer, we combined similar place types and narrowed our focus to four main categories: general communities, transportation areas, industrial areas, and communities near industrial zones. We organized the data using a specific code:

```python
for datapoint in Data:
    if datapoint['properties']['areaType'] == '社區':
        datapoint['properties']['areaType'] = '一般社區'
    elif datapoint['properties']['areaType'] == '社區感測':
        datapoint['properties']['areaType'] = '一般社區'
    elif datapoint['properties']['areaType'] == '交通':
        datapoint['properties']['areaType'] = '交通區'
    elif datapoint['properties']['areaType'] == '交通感測':
        datapoint['properties']['areaType'] = '交通區'
    elif datapoint['properties']['areaType'] == '工業':
        datapoint['properties']['areaType'] = '工業區'
    elif datapoint['properties']['areaType'] == '工業感測':
        datapoint['properties']['areaType'] = '工業區'
    if not datapoint['properties']['areaType'] in ['一般社區', '交通區', '工業區', '鄰近工業區社區']:
        datapoint['properties']['areaType'] = None
Data = [datapoint for datapoint in Data if datapoint['properties']['areaType'] != None]
Label = ['一般社區', '交通區', '工業區', '鄰近工業區社區']
count = dict.fromkeys(Label, 0)
for datapoint in Data:
    count[datapoint['properties']['areaType']] += 1
print("After data cleaning, There are {} records.".format(len(Data)))
print(json.dumps(count, indent=4, ensure_ascii=False))
```

```
After data cleaning, There are 7709 records.
{
    "一般社區": 2245,
    "交通區": 975,
    "工業區": 3541,
    "鄰近工業區社區": 948
}
```

After cleaning up the data, we were left with 7,709 records across these four main areas. We looked at each record's temperature, relative humidity, and PM2.5 levels. In our three-dimensional map, we used different colors to show the data for each type of area.

```python
DataX, DataY = [], []
for datapoint in Data:
    TmpX = [None]*3
    TmpY = None
    for rawdata_array in datapoint['data']:
        if(rawdata_array['name'] == 'Temperature'):
            TmpX[0] = rawdata_array['values'][0].get('value')
        if(rawdata_array['name'] == 'Relative humidity'):
            TmpX[1] = rawdata_array['values'][0].get('value')
        if(rawdata_array['name'] == 'PM2.5'):
            TmpX[2] = rawdata_array['values'][0].get('value')
    TmpY = Label.index(datapoint['properties']['areaType'])
    DataX.append(TmpX)
    DataY.append(TmpY)

DataX_Numpy = np.array(DataX)
DataY_Numpy = np.array(DataY)
plt.rc('legend',fontsize="xx-small")
fig = plt.figure(figsize=(8, 6), dpi=150)
ax = fig.add_subplot(projection='3d')
for i in range(len(Label)):
    ax.scatter(DataX_Numpy[DataY_Numpy==i][:,0],DataX_Numpy[DataY_Numpy==i][:,1],DataX_Numpy[DataY_Numpy==i][:,2], s=0.1, label=Label[i])
ax.legend()
ax.set_xlabel('Temperature(℃)')
ax.set_ylabel('Relative Humidity(%)')
ax.set_zlabel('PM2.5(μg/$m^{3}$)')
plt.show()
```

![Python output](figures/6-1-3-1.png)

The map revealed that most data points were clustered in certain areas, but some were spread out, far from the main group. These isolated data points are known as outliers. When we're classifying or grouping data, outliers can skew our model or algorithm, making it less effective. Therefore, we removed these outliers to improve our analysis.

### Outlier removal

We use a specific method to identify and remove unusual data points, or outliers, from our dataset. This method is based on standard statistical techniques and can be adjusted based on the unique requirements of different projects. In our case, we consider any sensor data from the Civil IoT Taiwan Data Platform an outlier if it's more than two standard deviations away from the average. By removing these outliers, we then create a new three-dimensional map to better understand the data's pattern.

```python
def Outlier_Filter(arr, k):
    Boolean_Arr =  np.ones(arr.shape[0], dtype=bool)
    for i in range(arr.shape[1]):
        Boolean_Arr = Boolean_Arr & (abs(arr[:,i] - np.mean(arr[:,i])) <  k*np.std(arr[:,i]))
    return Boolean_Arr

OutlierFilter = Outlier_Filter(DataX_Numpy, 2)
DataX_Numpy = DataX_Numpy[OutlierFilter]
DataY_Numpy = DataY_Numpy[OutlierFilter]
print("After removing Outliers, there are {} records left.".format(DataX_Numpy.shape[0]))
plt.rc('legend',fontsize="xx-small")
fig = plt.figure(figsize=(8, 6), dpi=150)
ax = fig.add_subplot(projection='3d')
for i in range(len(Label)):
    ax.scatter(DataX_Numpy[DataY_Numpy==i][:,0],DataX_Numpy[DataY_Numpy==i][:,1],DataX_Numpy[DataY_Numpy==i][:,2], s=0.1, label=Label[i])
ax.legend()
ax.set_xlabel('Temperature(℃)')
ax.set_ylabel('Relative Humidity(%)')
ax.set_zlabel('PM2.5(μg/$m^{3}$)')
plt.show()
```

```
After removing Outliers, there are 7161 records left.
```

![Python output](figures/6-1-3-2.png)

After this cleanup process, we found that we removed a total of 548 outliers (7,709 originally minus 7,161 remaining). The data that's left shows a more consistent pattern in the three-dimensional space, with fewer extreme values on the edges. To make it easier to see these patterns, we create graphs showing the relationships between data points in two dimensions at a time.

```python
plt.rc('legend',fontsize="large")
fig, axes = plt.subplots(1,3,figsize=(24, 6))
for i in range(DataX_Numpy.shape[1]):
    for j in range(len(Label)):
        axes[i].scatter(DataX_Numpy[DataY_Numpy==j][:,i%3],DataX_Numpy[DataY_Numpy==j][:,(i+1)%3], s=1, label=Label[j])
        axes[i].legend(loc=2)
        Axis_label = ['Temperature(℃)', 'Relative Humidity(%)', 'PM2.5(μg/$m^{3}$)']
        axes[i].set_xlabel(Axis_label[i%3])
        axes[i].set_ylabel(Axis_label[(i+1)%3])
plt.tight_layout()
```

![Python output](figures/6-1-3-3.png)


These graphs reveal interesting relationships between data points of different colors, which represent different types of areas in Civil IoT. Although it's hard to explain these relationships in simple terms, we plan to use a classification model to develop a specific classifier to make sense of them.

### Train data and test data

Before we start training our classifier model, we need to do one important thing: divide our existing data into two parts – training data and test data. Basically, we'll use the training data to teach the classifier how to do its job. The test data, on the other hand, is there to check how well our classifier can handle new, unseen data. To split our dataset into training and test data, we follow this example program, which divides them in a 4:1 ratio.

```python
indices = np.random.permutation(DataX_Numpy.shape[0])
Train_idx, Test_idx = indices[:int(DataX_Numpy.shape[0]*0.8)], indices[80:(DataX_Numpy.shape[0] - int(DataX_Numpy.shape[0]*0.8))]
TrainX, TestX = DataX_Numpy[Train_idx,:], DataX_Numpy[Test_idx,:]
TrainY, TestY = DataY_Numpy[Train_idx], DataY_Numpy[Test_idx]
```

### Using the pre-built Sklearn models

We apply a variety of classification techniques using the Python package Scikit Learn (sklearn) for both training and testing purposes. Our approach encompasses nine distinct models: nearest neighbor, linear SVM, RBF SVM, decision tree, random forest, neural network, Adaboost, Naive Bayes, and QDA. The process involves initially feeding training data to refine these models, followed by using test data to evaluate their predictive capabilities. The effectiveness of these models is assessed by comparing the predicted outcomes against actual labels, using a tool known as a [confusion matrix](https://en.wikipedia.org/wiki/Confusion_matrix). This matrix helps us visualize the accuracy of each model in predicting various combinations of labels.

```python
classifier_names = [
    "Nearest Neighbors",
    "Linear SVM",
    "RBF SVM",
    "Decision Tree",
    "Random Forest",
    "Neural Net",
    "AdaBoost",
    "Naive Bayes",
    "QDA",
]

classifiers = [
    KNeighborsClassifier(3),
    SVC(kernel="linear", C=0.025),
    SVC(gamma=2, C=1),
    DecisionTreeClassifier(max_depth=5),
    RandomForestClassifier(max_depth=5, n_estimators=10, max_features=1),
    MLPClassifier(alpha=1, max_iter=1000),
    AdaBoostClassifier(),
    GaussianNB(),
    QuadraticDiscriminantAnalysis(),
]

fig, axes = plt.subplots(3,3,figsize=(18, 13.5))
for i, model in enumerate(classifiers):
    model.fit(TrainX, TrainY)
    Result = model.predict(TestX)
    mat = confusion_matrix(TestY, Result)
    sns.heatmap(mat.T, square=True, annot=True, fmt='d', cbar=False,
                xticklabels=Label, yticklabels=Label, ax = axes[i//3][i%3])
    axes[i//3][i%3].set_title("{},  Accuracy : {}".format(classifier_names[i], round(accuracy_score(Result, TestY), 3)), fontweight="bold", size=13)
    axes[i//3][i%3].set_xlabel('true label', fontsize = 10.0)
    axes[i//3][i%3].set_ylabel('predicted label', fontsize = 10.0)
plt.tight_layout()
```

![Python output](figures/6-1-3-4.png)

In our exploration of these nine models, we observed that the RBF SVM model stands out by achieving a classification accuracy of nearly 70%. It's noteworthy that this result was attained using unprocessed data. For those interested in delving deeper into the classifier's potential, further investigation and data analysis can enhance its proficiency in categorizing diverse data types.

## Case 2: Clustering of Air Quality Sensors

In this case, we're working with the air quality data from Taiwan's EPA monitoring stations, which is part of the Civil IoT Taiwan initiative. Our focus is on examining the historical data to see how similar the data trends are among different groups of stations.

### Data download and preprocessing

To gather all the air quality data from Taiwan's EPA monitoring stations for 2021, we downloaded it from the Civil IoT Taiwan Data Service Platform's historical database. After downloading, we unpacked the files into the `/content` directory.

```python
!wget -O 'EPA_OD_2021.zip' -q "https://history.colife.org.tw/?r=/download&path=L%2Bepuuawo%2BWTgeizqi%2FnkrDkv53nvbJf5ZyL5a6256m65ZOB5ris56uZL0VQQV9PRF8yMDIxLnppcA%3D%3D"
!unzip -q 'EPA_OD_2021.zip' && rm 'EPA_OD_2021.zip' 
!unzip -q '/content/EPA_OD_2021/EPA_OD_202112.zip' -d '/content'
!rm -rf '/content/EPA_OD_2021'
```

Our first step was to focus on December 2021 data. We removed fields we didn't need: `Pollutant`, `SiteId`, `Status`, `SO2_AVG`. Next, we converted the data type of the sensor readings to floating-point numbers, making it easier to work with them later.

```python
Dataframe = pd.read_csv("/content/EPA_OD_202112.csv", parse_dates=['PublishTime'])
Dataframe = Dataframe.drop(columns=["Pollutant", "SiteId", "Status", "SO2_AVG"])
Numerical_ColumnNames = list(Dataframe.columns.values)
for ColumnName in ['SiteName', 'County', 'PublishTime']:
    Numerical_ColumnNames.remove(ColumnName)
for Numerical_ColumnName in Numerical_ColumnNames:
    Dataframe[Numerical_ColumnName] = pd.to_numeric(Dataframe[Numerical_ColumnName], errors='coerce').astype('float64')
Dataframe = Dataframe.dropna()
Dataframe.head()
```

![Python output](figures/6-1-4-1.png)

Considering the huge volume of data for the entire month, we aimed to speed up our sample program. To do this, we took a subset of data, specifically from December 13 to 17, 2021, and labeled this `FiveDay_Dataframe`. In the example below, you'll see how we combined this data using the `Country` and `SiteName` fields. Finally, we arranged the data in order of their publication times.

```python
FiveDay_Dataframe = Dataframe.loc[(Dataframe['PublishTime'] <= '2021-12-17 23:00:00') & (Dataframe['PublishTime'] >= '2021-12-13 00:00:00')]
FiveDay_Dataframe['CountyAndSiteName'] = FiveDay_Dataframe['County'] + FiveDay_Dataframe['SiteName']
FiveDay_Dataframe = FiveDay_Dataframe.drop(columns=["County", "SiteName"])
FiveDay_Dataframe = FiveDay_Dataframe.sort_values(by=['CountyAndSiteName','PublishTime'])
FiveDay_Dataframe = FiveDay_Dataframe.set_index(keys = ['CountyAndSiteName'])
FiveDay_Dataframe
```

![Python output](figures/6-1-4-2.png)

### Dynamic Time Warping (DTW)

Next, we assess how "similar" two sites are and turn this similarity into a numerical value. The main way we do this is by lining up the data from two monitoring stations based on the time they were recorded and then comparing the differences in their air quality readings. However, since air pollution can happen at different times and last for varying lengths at each site, we need a more adaptable approach to gauge their similarity. That's why we use the dynamic time warping (DTW) method. The closer the DTW distance is between two stations, the more similar they are.

```python
Site_TimeSeriesData = dict()
for Site in np.unique(FiveDay_Dataframe.index.values):
    tmp = FiveDay_Dataframe[FiveDay_Dataframe.index == Site]
    tmp = tmp.groupby(['CountyAndSiteName', 'PublishTime'], as_index=False).mean()
    tmp = tmp.loc[:,~tmp.columns.isin(['CountyAndSiteName', 'PublishTime'])]
    Site_TimeSeriesData[Site] = tmp.to_numpy()

DictKeys = Site_TimeSeriesData.keys()
Sites_DTW = dict()
for i, key1 in enumerate(DictKeys):
    for j, key2 in enumerate(DictKeys):
        if i >= j: 
            continue
        else:
            Sites_DTW[str(key1)+" "+str(key2)] = fastdtw(Site_TimeSeriesData[key1][:,:-4], Site_TimeSeriesData[key2][:,:-4], dist=euclidean)[0]
Sites_DTW_keys = np.array(list(Sites_DTW.keys()))
Site_DTW_Numpy = np.array([[value] for _, value in Sites_DTW.items()])
```

In the figure below, we've charted the DTW distances between all the sites, arranged from smallest to largest. To delve deeper, we'll next start analyzing this data using clustering algorithms.

```python
fig = plt.figure(figsize=(4, 3), dpi=150)
ax = fig.add_subplot(1,1,1)
ax.scatter(Site_DTW_Numpy[:,0], [1]*len(Sites_DTW.items()), s=0.05)
ax.get_yaxis().set_visible(False)
fig.text(0.5, 0.01, 'DTW distance', ha='center', va='center')
fig.text(0.06, 0.5, 'Group number', ha='center', va='center', rotation='vertical')
```

![Python output](figures/6-1-4-3.png)

### K-Mean clustering

We applied a technique called "K-Means clustering," which is part of the sklearn software, to organize our data into groups. This method requires us to decide on the number of groups (or clusters) beforehand, so we chose to start with 3. To do this, we used a specific code, and then we visually represented our findings. In our chart, we displayed the group number on the vertical (Y) axis and the level of similarity to other data, calculated using a method called DTW, on the horizontal (X) axis.

```python
from sklearn.cluster import KMeans

model = KMeans(n_clusters=3, random_state=0).fit([[value] for _, value in Sites_DTW.items()])
Result = model.labels_
for i in np.unique(Result):
    print("Number of Cluster{} : {}".format(i,len(Result[Result==i])))

fig = plt.figure(figsize=(4, 3), dpi=150)
ax = fig.add_subplot(1,1,1)
for i in np.unique(Result):
    ax.scatter(Site_DTW_Numpy[Result==i][:,0],[i]*len(Site_DTW_Numpy[Result==i]), s=0.05)
ax.get_yaxis().set_visible(False)
fig.text(0.5, 0.01, 'DTW distance', ha='center', va='center')
fig.text(0.06, 0.5, 'Group number', ha='center', va='center', rotation='vertical')
```

```
Number of Cluster0 : 1165
Number of Cluster1 : 994
Number of Cluster2 : 542
```

![Python output](figures/6-1-4-4.png)

Our results from using the K-Means algorithm revealed that our initial data was separated into three distinct groups, containing 1,165, 994, and 542 items of data each. To get a better understanding of why these particular groups were formed, we're now looking into each possible factor that might have influenced their creation.

### Correlation between data clusters and geographical locations

We started by considering that air quality changes might be specific to certain areas. To investigate this, we looked at whether the patterns we found in air quality data were connected to where the air quality monitoring stations are located. First, we got the GPS coordinates of these stations and calculated how far apart they are from each other. 

Next, we grouped the data from these stations based on similarities and differences in their readings. Using this grouping, we then analyzed how the distances between stations in each group varied. We've illustrated these findings in the image below.

```python
Dist_for_Clusters = [None]*len(np.unique(Result))
for i in np.unique(Result):
    Dist_for_Cluster = []
    Cluster = Sites_DTW_keys[Result==i]
    for Sites in Cluster:
        Site1, Site2 = Sites.split(' ')
        coord1 = Site_TimeSeriesData[Site1][0,-1], Site_TimeSeriesData[Site1][0,-2]
        coord2 = Site_TimeSeriesData[Site2][0,-1], Site_TimeSeriesData[Site2][0,-2]
        Dist_for_Cluster.append(geopy.distance.geodesic(coord1, coord2).km)
    Dist_for_Cluster = np.array(Dist_for_Cluster)
    Dist_for_Clusters[i] = Dist_for_Cluster
Dist_for_Clusters = np.array(Dist_for_Clusters)
# for Dist_for_Cluster in Dist_for_Clusters:
#     print(np.mean(Dist_for_Cluster))

fig = plt.figure(figsize=(4, 3), dpi=150)
ax = fig.add_subplot(1,1,1)
for i in np.unique(Result):
    gtMean = Dist_for_Clusters[i][Dist_for_Clusters[i]>np.mean(Dist_for_Clusters[i])]
    ltMean = Dist_for_Clusters[i][Dist_for_Clusters[i]<np.mean(Dist_for_Clusters[i])]
    print("Mean Distance between Sites for Cluster{} : {}".format(i, np.mean(Dist_for_Clusters[i])))
    print("In Cluster{} there are {:.2%} less than mean, and {:.2%} greater than mean.\n".format(i, len(ltMean)/len(Dist_for_Clusters[i]), len(gtMean)/len(Dist_for_Clusters[i])))
    ax.scatter(gtMean, [i]*len(gtMean), s=0.05, color="orange")
    ax.scatter(ltMean, [i]*len(ltMean), s=0.05, color="pink")
    ax.axvline(np.mean(Dist_for_Clusters[i]), ymin = 0.45*i, ymax = 0.45*i+0.1, color = "red", linewidth=0.5)
ax.get_yaxis().set_visible(False)
fig.text(0.5, 0.01, 'DTW distance', ha='center', va='center')
fig.text(0.06, 0.5, 'Group number', ha='center', va='center', rotation='vertical')
```

```
Mean Distance between Sites for Cluster0 : 84.34126465234523
In Cluster0 there are 60.09% less than mean, and 39.91% greater than mean.

Mean Distance between Sites for Cluster1 : 180.26230465399215
In Cluster1 there are 54.53% less than mean, and 45.47% greater than mean.

Mean Distance between Sites for Cluster2 : 234.89206124762546
In Cluster2 there are 39.48% less than mean, and 60.52% greater than mean.
```

![Python output](figures/6-1-4-5.png)

Our analysis revealed that groups with higher DTW values (meaning the air quality data from these stations was less similar over time) tended to have stations that were further apart. Conversely, groups with more similar air quality data had stations that were closer together. This suggests that the spread of air pollutants is indeed influenced by how far apart these monitoring stations are, supporting our original idea.

### Correlation between data clusters and wind directions

First, we considered that changes in air quality might be influenced by the direction of the wind in the area. So, we looked into whether there's a link between the patterns we found in the air quality data and the wind direction at the air quality monitoring station. We started by getting the GPS coordinates of the station and then worked out the directional relationship between two places on the map. Next, we examined how the patterns in the data were connected to the wind direction at the station's location. After this, we did some basic statistical analysis and presented our findings in the figure below.

```python
def get_bearing(lat1, long1, lat2, long2):
    dLon = (long2 - long1)
    x = math.cos(math.radians(lat2)) * math.sin(math.radians(dLon))
    y = math.cos(math.radians(lat1)) * math.sin(math.radians(lat2)) - math.sin(math.radians(lat1)) * math.cos(math.radians(lat2)) * math.cos(math.radians(dLon))
    brng = np.arctan2(x,y)
    brng = np.degrees(brng)
    return brng

def Check_Wind_Dirc(brng, wind_dirc):
    if brng > 180:
        return ((brng < wind_dirc + 45) and (brng > wind_dirc - 45)) or ((brng - 180 < wind_dirc + 45) and (brng - 180 > wind_dirc - 45))
    else:
        return ((brng < wind_dirc + 45) and (brng > wind_dirc - 45)) or ((brng + 180 < wind_dirc + 45) and (brng + 180 > wind_dirc - 45))

Brng_for_Clusters = [None]*len(np.unique(Result))
Boolean_WindRelated_for_Clusters = [None]*len(np.unique(Result))
for i in np.unique(Result):
    Brng_for_Cluster = []
    Boolean_WindRelated_for_Cluster = []
    Cluster = Sites_DTW_keys[Result==i]
    for Sites in Cluster:
        Site1, Site2 = Sites.split(' ')
        coord1 = Site_TimeSeriesData[Site1][0,-1], Site_TimeSeriesData[Site1][0,-2]
        coord2 = Site_TimeSeriesData[Site2][0,-1], Site_TimeSeriesData[Site2][0,-2]
        Brng_Between_Site = get_bearing(coord1[0], coord1[1], coord2[0], coord2[1])
        Brng_for_Cluster.append(Brng_Between_Site)
        MeanWindDirc1 = np.mean(Site_TimeSeriesData[Site1][:,-3])
        MeanWindDirc2 = np.mean(Site_TimeSeriesData[Site2][:,-3])
        Boolean_WindRelated_for_Cluster.append(Check_Wind_Dirc(Brng_Between_Site, MeanWindDirc1) or Check_Wind_Dirc(Brng_Between_Site, MeanWindDirc2))
    Brng_for_Cluster = np.array(Brng_for_Cluster)
    Boolean_WindRelated_for_Cluster = np.array(Boolean_WindRelated_for_Cluster)
    Boolean_WindRelated_for_Clusters[i] = Boolean_WindRelated_for_Cluster
    Brng_for_Clusters[i] = Brng_for_Cluster
Brng_for_Clusters = np.array(Brng_for_Clusters)
Boolean_WindRelated_for_Clusters = np.array(Boolean_WindRelated_for_Clusters)

fig = plt.figure(figsize=(4, 3), dpi=150)
ax = fig.add_subplot(1,1,1)
for i in np.unique(Result):
    print("Relevance for Cluster{} : {:.2%}".format(i, len(Dist_for_Clusters[i][Boolean_WindRelated_for_Clusters[i] == True])/len(Dist_for_Clusters[i])))
    ax.scatter(Dist_for_Clusters[i][Boolean_WindRelated_for_Clusters[i] == True],\
               [i]*len(Dist_for_Clusters[i][Boolean_WindRelated_for_Clusters[i] == True]), s=2, color="green")
    ax.scatter(Dist_for_Clusters[i][Boolean_WindRelated_for_Clusters[i] == False],\
               [i]*len(Dist_for_Clusters[i][Boolean_WindRelated_for_Clusters[i] == False]), s=0.05, color="violet")
    ax.axvline(np.mean(Dist_for_Clusters[i]), ymin = 0.45*i, ymax = 0.45*i+0.1, color = "red", linewidth=2)
ax.get_yaxis().set_visible(False)
fig.text(0.5, 0.01, 'DTW distance', ha='center', va='center')
fig.text(0.06, 0.5, 'Group number', ha='center', va='center', rotation='vertical')
```

```
Relevance for Cluster0 : 54.08%
Relevance for Cluster1 : 39.24%
Relevance for Cluster2 : 22.69%
```

![Python output](figures/6-1-4-6.png)

Our analysis indicates that when the DTW (Dynamic Time Warping) value is lower, there's a stronger link between the direction from one station to another and the local wind direction — and the opposite is true when the DTW value is higher. This shows that the way air pollution spreads is indeed influenced by the wind direction around the area, supporting our initial theory.

## Case 3: Clustering and Classification Application of Water and Meteorological Data

This example demonstrates how we use two types of data: rainfall data (from the Central Weather Bureau) and flood sensor data (from the Water Resource Agency), both sourced from Civil IoT Taiwan. We analyze past data using a method called data clustering. This method helps us identify which river water level stations are most affected by changes in rainfall. Then, we apply a technique known as data classification to predict if a certain area might experience flooding, based solely on rainfall data.

### Data download and preprocessing

To gather all the 2021 data from rain gauges (Central Weather Bureau) and flood sensors (Water Resource Agency), we use specific codes to download from the [historical database](https://history.colife.org.tw/) of the Civil IoT Taiwan Data Service Platform. After downloading, we unpack the files into the `/content` directory.

```python
!wget -O 'Rain_2021.zip' -q "https://history.colife.org.tw/?r=/download&path=L%2Bawo%2BixoS%2FkuK3lpK7msKPosaHlsYBf6Zuo6YeP56uZLzIwMjEuemlw"
!wget -O 'Rain_Stataion.csv' -q "https://history.colife.org.tw/?r=/download&path=L%2Bawo%2BixoS%2FkuK3lpK7msKPosaHlsYBf6Zuo6YeP56uZL3JhaW5fc3RhdGlvbi5jc3Y%3D"
!unzip -q 'Rain_2021.zip' && rm 'Rain_2021.zip' 
!find '/content/2021' -name '*.zip'  -exec unzip -q {} -d '/content/Rain_2021_csv' \;
!rm -rf '/content/2021'

!wget -O 'Flood_2021.zip' -q "https://history.colife.org.tw/?r=/download&path=L%2BawtOizh%2Ba6kC%2FmsLTliKnnvbLvvIjoiIfnuKPluILmlL%2FlupzlkIjlu7rvvIlf5re55rC05oSf5ris5ZmoLzIwMjEuemlw"
!wget -O 'Flood_Stataion.csv' -q "https://history.colife.org.tw/?r=/download&path=L%2BawtOizh%2Ba6kC%2FmsLTliKnnvbLvvIjoiIfnuKPluILmlL%2FlupzlkIjlu7rvvIlf5re55rC05oSf5ris5ZmoL3N0YXRpb25f5rC05Yip572y77yI6IiH57ij5biC5pS%2F5bqc5ZCI5bu677yJX%2Ba3ueawtOaEn%2Ba4rOWZqC5jc3Y%3D"
!unzip -q 'Flood_2021.zip' && rm 'Flood_2021.zip' 
!find '/content/2021' -name '*_QC.zip'  -exec unzip -q {} -d '/content/Flood_2021_csv' \;
!rm -rf '/content/2021'
```

Starting with the rain gauge data, we first remove fields we don't need: `MIN_10`, `HOUR_6`, `HOUR_12`, and `NOW`. Next, we discard any data recorded after November, keeping the rest in the `Rain_df`. Additionally, we bring in details about the rain gauges and save this information in `Rain_Station_df`. Please note that this process involves a large amount of data and may take some time to complete.

```python
csv_files = glob.glob(os.path.join("/content/Rain_2021_csv", "*.csv"))
csv_files.sort()
Rain_df = pd.DataFrame()

for csv_file in tqdm(csv_files):
    tmp_df = pd.read_csv(csv_file, parse_dates=['obsTime'])
    tmp_df.drop(['MIN_10','HOUR_6', 'HOUR_12', 'NOW'], axis=1, inplace=True)
    try:
        tmp_df = tmp_df.loc[tmp_df['obsTime'].dt.minute == 00]
        Rain_df =  pd.concat([Rain_df, tmp_df])
    except:
        print(csv_file)
        continue
Rain_df = Rain_df.loc[Rain_df['obsTime'] < "2021-11-01 00:00:00"]
num = Rain_df._get_numeric_data()
num[num < 0] = 0
Rain_df.dropna(inplace=True)
Rain_df.sort_values(by=['station_id','obsTime'], inplace=True)
Rain_Station_df = pd.read_csv('/content/Rain_Stataion.csv')
Rain_df
```

![Python output](figures/6-1-5-1.png)

Similarly, for the flood sensor data, we exclude any records after November. We also remove entries with missing information, storing the remaining data in `Flood_df`. Information about the flood sensors is also imported and saved in `Flood_Station_df`. Like before, this step involves processing a significant volume of data, so a considerable amount of time might be needed.

```python
csv_files = glob.glob(os.path.join("/content/Flood_2021_csv", "*_QC.csv"))
csv_files.sort()
Flood_df = pd.DataFrame()

for csv_file in tqdm(csv_files):
    tmp_df = pd.read_csv(csv_file, parse_dates=['timestamp'])
    tmp_df = tmp_df.loc[(tmp_df['PQ_unit'] == 'cm')]
    Flood_df = pd.concat([Flood_df,tmp_df], axis=0, ignore_index=True)
Flood_df = Flood_df.loc[Flood_df['timestamp'] < "2021-11-01 00:00:00"]
Flood_df.replace(-999.0,0.0, inplace=True)
Flood_df.dropna(inplace=True)
Flood_df.sort_values(by=['timestamp'], inplace=True)
Flood_Station_df = pd.read_csv('/content/Flood_Stataion.csv')
Flood_df
```

![Python output](figures/6-1-5-2.png)

### Correlation of flood sensors and rain gauges

We're dealing with a lot of data from flood sensors. To make it easier to understand, let's focus on one specific flood sensor, number `43b2aec1-69b0-437b-b2a2-27c76a3949e8`, which is located in Yunlin County. We'll store the data from this sensor in an object called `Flood_Site_df`. This will be our example for further analysis.

```python
Flood_Site_df = Flood_df.loc[Flood_df['station_id'] == '43b2aec1-69b0-437b-b2a2-27c76a3949e8']
Flood_Site_df.head()
```

![Python output](figures/6-1-5-3.png)

Next, we want to find out how closely this flood sensor's data matches with the data from rain gauges. To do this, we use a method called Dynamic Time Warping (DTW). DTW helps us measure how similar two sets of data are. The lower the DTW value, the more similar the data. To make this easier to understand, we'll define similarity as the opposite of the DTW value. This means a smaller DTW value indicates more similarity. We'll calculate this similarity between our chosen flood sensor and the data from all the rain gauges.

```python
Flood_Sensor_np = np.array([[v,v,v] for v in Flood_Site_df['value'].to_numpy()])
Site_dtw_Dist = dict()
Rain_tmp_df = Rain_df.loc[(Rain_df['obsTime'].dt.hour == 1)]
for Site in tqdm(np.unique(Rain_Station_df.loc[Rain_Station_df['city']=='雲林縣']['station_id'].to_numpy())):
    tmp_df = Rain_tmp_df.loc[(Rain_tmp_df['station_id'] == Site)]
    if tmp_df.empty:
        continue
    tmp_np = tmp_df[['RAIN','HOUR_3','HOUR_24']].to_numpy()
    Site_dtw_Dist[Site] = (1/fastdtw(Flood_Sensor_np, tmp_np, dist=euclidean)[0])
Site_dtw_Dist = dict(sorted(Site_dtw_Dist.items(), key=lambda item: item[1]))
print(json.dumps(Site_dtw_Dist, indent=4, ensure_ascii=False))
```

```
{
    "88K590": 4.580649481044748e-05,
    "C0K560": 4.744655320519647e-05,
    "C0K490": 4.79216996101994e-05,
    "C0K520": 5.038234963332513e-05,
    "C0K420": 5.0674082994877385e-05,
    "C0K250": 5.1021366345465985e-05,
    "C0K280": 5.118406054309105e-05,
    "A0K420": 5.1515699268157996e-05,
    "C0K400": 5.178763059243615e-05,
    "O1J810": 5.2282255279259976e-05,
    "C0K240": 5.2470804312991397e-05,
    "01J960": 5.334885670256585e-05,
    "C0K470": 5.438256498969844e-05,
    "81K580": 5.45854441959214e-05,
    "C0K410": 5.5066753408217084e-05,
    "A2K570": 5.520214274022887e-05,
    "01J970": 5.529887546186233e-05,
    "C0K480": 5.5374254960644355e-05,
    "C0K460": 5.5657892623955056e-05,
    "72K220": 5.5690175197363816e-05,
    "C0K510": 5.5742273217039165e-05,
    "C1K540": 5.618025674136218e-05,
    "C0K550": 5.621240903075098e-05,
    "C0K450": 5.62197509062689e-05,
    "C0K291": 5.6380522616008906e-05,
    "C0K330": 5.638960953991442e-05,
    "C0K530": 5.6525582441919285e-05,
    "C0K500": 5.6825555408648244e-05,
    "C0K440": 5.692254536595e-05,
    "C0K430": 5.697351917955081e-05,
    "01J930": 5.7648109890427854e-05,
    "C0K580": 5.770344946580767e-05,
    "C0K390": 5.782553930260475e-05,
    "01J100": 5.7933240408734325e-05,
    "01K060": 5.8343415644572526e-05
}
```

### Data clustering to find highly correlated rain gauges

We grouped the rain gauges into three categories using a technique known as clustering, which groups items based on how similar they are to each other. Our focus was to identify the group most closely matching the flood sensor data. In our case, we ended up with three groups containing 9, 23, and 3 rain gauges respectively. The second group, with 23 gauges, had rainfall patterns most closely resembling the data from the flood sensors.

```python
cluster_model = KMeans(n_clusters=3).fit([[value] for _, value in Site_dtw_Dist.items()])
Result = cluster_model.labels_
for i in np.unique(Result):
    print("Number of Cluster {} : {}".format(i,len(Result[Result==i])))
fig = plt.figure(figsize=(4, 3), dpi=150)
ax = fig.add_subplot(1,1,1)
fig.text(0.5, 0.01, 'DTW distance', ha='center', va='center')
fig.text(0.06, 0.5, 'Group number', ha='center', va='center', rotation='vertical')

Site_DTW_Numpy = np.array([value for _, value in Site_dtw_Dist.items()])
Site_Name_Numpy = np.array([key for key, _ in Site_dtw_Dist.items()])
Mean_Dis_For_Cluster = [None] * len(np.unique(Result))
for i in np.unique(Result):
    Mean_Dis_For_Cluster[i] = (np.mean(Site_DTW_Numpy[Result==i]))
    ax.scatter(Site_DTW_Numpy[Result==i],[i]*len(Site_DTW_Numpy[Result==i]), s=10)
    print("Mean Distance of Cluster {} : {}".format(i,Mean_Dis_For_Cluster[i]))
ax.get_yaxis().set_visible(False)
Best_Cluster = np.where(Mean_Dis_For_Cluster == np.amax(Mean_Dis_For_Cluster))[0]
Best_Site = Site_Name_Numpy[Result == Best_Cluster]
print(Best_Site)
```

```
Number of Cluster0 : 23
Number of Cluster1 : 3
Number of Cluster2 : 9
Mean Similarity of Cluster0 : 5.6307994901628334e-05
Mean Similarity of Cluster1 : 4.7058249208614454e-05
Mean Similarity of Cluster2 : 5.1629678408018994e-05
['C0K470' '81K580' 'C0K410' 'A2K570' '01J970' 'C0K480' 'C0K460' '72K220'
 'C0K510' 'C1K540' 'C0K550' 'C0K450' 'C0K291' 'C0K330' 'C0K530' 'C0K500'
 'C0K440' 'C0K430' '01J930' 'C0K580' 'C0K390' '01J100' '01K060']
```

![Python output](figures/6-1-5-4.png)

To delve deeper into the relationship between the flood sensor and these 23 rain gauges, we arranged the data from the chosen flood sensor in a time sequence.

```python
tmp= Flood_Site_df['value'].to_numpy()
fig= plt.figure(figsize=(6, 3), dpi=150)
ax= fig.add_subplot(1,1,1)
ax.plot(range(len(tmp)),tmp, linewidth=0.5)
ax.set_xlabel('Time sequence')
ax.set_ylabel('Water level')
```

![Python output](figures/6-1-5-5.png)

Next, we created 23 separate charts, each showing the hourly rainfall recorded by one of the 23 rain gauges in the group we identified as most similar. These charts revealed a notable pattern: when the flood sensor readings were high, the rainfall measurements from the rain gauges also tended to increase. This trend between the two sets of data was strikingly similar, aligning well with what we would logically expect.

```python
fig = plt.figure(figsize=(8, 2*(len(Best_Site)//4+1)), dpi=150)
for i, Site in enumerate(Best_Site):
    tmp = Rain_df.loc[Rain_df['station_id']==Site]['RAIN']
    ax = fig.add_subplot(len(Best_Site)//4+1,4,i+1)
    ax.plot(range(len(tmp)),tmp, linewidth=0.5)
```

![Python output](figures/6-1-5-6.png)

### Data classification to predict flooding by rainfall data

To start, we're using data from flood sensors to determine if a flood happened at a certain location. This data is treated as our label. We also use information from closely related rain gauges to create a simple tool (classifier) to predict flooding. We've divided data from January 2021 to October 2021 into two parts: training data (first seven months) and testing data (next two months). In our system, if a flood sensor records a value above 0, we consider it a flood event; if it records 0, we mark it as no flood event. This sorted data is saved in an object called `Train_DataSet`.

Please note that, for simplicity, we're using a basic rule where any water level above 0 indicates a flood. However, real-world criteria for defining floods are more complex. For accurate applications, please refer to relevant laws and regulations.

```python
Flooding = Flood_Site_df.loc[(Flood_Site_df['value'] > 0.0) & (Flood_Site_df['timestamp'] < "2021-08-01 00:00:00")][['timestamp', 'value']].values
Not_Flooding = Flood_Site_df.loc[(Flood_Site_df['value'] == 0.0) & (Flood_Site_df['timestamp'] < "2021-08-01 00:00:00")][['timestamp', 'value']]\
                .sample(n=10*len(Flooding)).values

Train_DataSet = {'x':[], 'y':[]}
for timestamp, _ in  tqdm(Flooding):
    tmp_x = []
    tmp_df = Rain_df.loc[(Rain_df['obsTime'] < (timestamp - timedelta(hours=1))) & (Rain_df['obsTime'] > (timestamp - timedelta(hours=60)))]
    for Site in Best_Site:
        Site_tmp_df = tmp_df.loc[(tmp_df['station_id']==Site)]
        if not Site_tmp_df.empty:
            tmp_x.append(Site_tmp_df.tail(24)[['RAIN', 'HOUR_3']].values.flatten())
    while len(tmp_x) < len(Best_Site):
        tmp_x.append(tmp_x[0])
    tmp_x = np.array(tmp_x).flatten()
    if len(tmp_x) == 24*len(Best_Site)*2:
        Train_DataSet['x'].append(tmp_x)
        Train_DataSet['y'].append(1)

for timestamp, _ in  tqdm(Not_Flooding):
    tmp_x = []
    tmp_df = Rain_df.loc[(Rain_df['obsTime'] < (timestamp - timedelta(hours=1))) & (Rain_df['obsTime'] > (timestamp - timedelta(hours=60)))]
    for Site in Best_Site:
        Site_tmp_df = tmp_df.loc[(tmp_df['station_id']==Site)]
        if not Site_tmp_df.empty:
            tmp_x.append(Site_tmp_df.tail(24)[['RAIN', 'HOUR_3']].values.flatten())
    tmp_x = np.array(tmp_x).flatten()
    if len(tmp_x) == 24*len(Best_Site)*2:
        Train_DataSet['x'].append(tmp_x)
        Train_DataSet['y'].append(0)
```

Similarly, for data from August 2021 to October 2021, a value above 0 in a flood sensor is marked as a flood, and a value of 0 as no flood. This sorted data is stored in another object called `Test_DataSet`.

```python
Flooding = Flood_Site_df.loc[(Flood_Site_df['value'] > 0.0) & (Flood_Site_df['timestamp'] > "2021-08-01 00:00:00")][['timestamp', 'value']].values
Not_Flooding = Flood_Site_df.loc[(Flood_Site_df['value'] == 0.0) & (Flood_Site_df['timestamp'] > "2021-08-01 00:00:00")][['timestamp', 'value']]\
                .sample(n=2*len(Flooding)).values

Test_DataSet = {'x':[], 'y':[]}
for timestamp, _ in  tqdm(Flooding):
    tmp_x = []
    tmp_df = Rain_df.loc[(Rain_df['obsTime'] < (timestamp - timedelta(hours=1))) & (Rain_df['obsTime'] > (timestamp - timedelta(hours=60)))]
    for Site in Best_Site:
        Site_tmp_df = tmp_df.loc[(tmp_df['station_id']==Site)]
        if not Site_tmp_df.empty:
            tmp_x.append(Site_tmp_df.tail(24)[['RAIN', 'HOUR_3']].values.flatten())
    while len(tmp_x) < len(Best_Site):
        tmp_x.append(tmp_x[0])
    tmp_x = np.array(tmp_x).flatten()
    if len(tmp_x) == 24*len(Best_Site)*2:
        Test_DataSet['x'].append(tmp_x)
        Test_DataSet['y'].append(1)

for timestamp, _ in  tqdm(Not_Flooding):
    tmp_x = []
    tmp_df = Rain_df.loc[(Rain_df['obsTime'] < (timestamp - timedelta(hours=1))) & (Rain_df['obsTime'] > (timestamp - timedelta(hours=60)))]
    for Site in Best_Site:
        Site_tmp_df = tmp_df.loc[(tmp_df['station_id']==Site)]
        if not Site_tmp_df.empty:
            tmp_x.append(Site_tmp_df.tail(24)[['RAIN', 'HOUR_3']].values.flatten())
    tmp_x = np.array(tmp_x).flatten()
    if len(tmp_x) == 24*len(Best_Site)*2:
        Test_DataSet['x'].append(tmp_x)
        Test_DataSet['y'].append(0)
```

For training and testing, we're using a Python package called Scikit Learn (sklearn). We're trying out nine different methods, including nearest neighbor, linear SVM, RBF SVM, decision tree, random forest, neural network, Adaboost, Naive Bayes, and QDA. We feed in the training data for fine-tuning and then the test data for predictions. To evaluate how well our model works, we compare the test data with predicted results using a tool known as a [confusion matrix](https://en.wikipedia.org/wiki/Confusion_matrix).

```python
names = [
    "Nearest Neighbors",
    "Linear SVM",
    "RBF SVM",
    "Decision Tree",
    "Random Forest",
    "Neural Net",
    "AdaBoost",
    "Naive Bayes",
    "QDA",
]

classifiers = [
    KNeighborsClassifier(3),
    SVC(kernel="linear", C=0.025),
    SVC(gamma=2, C=1),
    DecisionTreeClassifier(max_depth=5),
    RandomForestClassifier(max_depth=5, n_estimators=10, max_features=1),
    MLPClassifier(alpha=1, max_iter=1000),
    AdaBoostClassifier(),
    GaussianNB(),
    QuadraticDiscriminantAnalysis(),
]

fig, axes = plt.subplots(3,3,figsize=(18, 13.5))
for i, model in enumerate(classifiers):
    model.fit(Train_DataSet['x'], Train_DataSet['y'])
    Result = model.predict(Test_DataSet['x'])
    mat = confusion_matrix(Test_DataSet['y'], Result)
    sns.heatmap(mat.T, square=True, annot=True, fmt='d', cbar=False,
                xticklabels=["不淹水","淹水"], yticklabels=["不淹水","淹水"], ax = axes[i//3][i%3])
    axes[i//3][i%3].set_title("{},  Accuracy : {}".format(names[i], round(accuracy_score(Result, Test_DataSet['y']), 3)), fontweight="bold", size=13)
    axes[i//3][i%3].set_xlabel('true label', fontsize = 10.0)
    axes[i//3][i%3].set_ylabel('predicted label', fontsize = 10.0)
plt.tight_layout()
```

![Python output](figures/6-1-5-7.png)

In our experiment, the nearest neighbor method showed a success rate of about 7.3% in classifying floods. This is just a basic attempt, using unprocessed data. With more detailed analysis and research on the data, we can likely improve our success rate. For those interested in data classification, further study in this area can help enhance the effectiveness of these classifiers in dealing with various types of data.

## References

- Ibrahim Saidi, Your First Machine Learning Project in Python, Medium, Jan, 2022, ([https://ibrahimsaidi.com.au/your-first-machine-learning-project-in-python-e3b90170ae41](https://ibrahimsaidi.com.au/your-first-machine-learning-project-in-python-e3b90170ae41))
- Esmaeil Alizadeh, An Illustrative Introduction to Dynamic Time Warping, Medium, Oct. 2020, ([https://towardsdatascience.com/an-illustrative-introduction-to-dynamic-time-warping-36aa98513b98](https://towardsdatascience.com/an-illustrative-introduction-to-dynamic-time-warping-36aa98513b98))
- Jason Brownlee, 10 Clustering Algorithms With Python, Machine Learning Mastery, Aug. 2020, ([https://machinelearningmastery.com/clustering-algorithms-with-python/](https://machinelearningmastery.com/clustering-algorithms-with-python/))
- Alexandra Amidon, How to Apply K-means Clustering to Time Series Data, TOwards Data Science, July 2020, ([https://towardsdatascience.com/how-to-apply-k-means-clustering-to-time-series-data-28d04a8f7da3](https://towardsdatascience.com/how-to-apply-k-means-clustering-to-time-series-data-28d04a8f7da3))
- scikit-learn Tutorials ([https://scikit-learn.org/stable/tutorial/index.html](https://scikit-learn.org/stable/tutorial/index.html))
- scikit-learn Classifier comparison ([https://scikit-learn.org/stable/auto_examples/classification/plot_classifier_comparison.html](https://scikit-learn.org/stable/auto_examples/classification/plot_classifier_comparison.html))
- FastDTW - A Python implementation of FastDTW, ([https://github.com/slaypni/fastdtw](https://github.com/slaypni/fastdtw))
- Nearest Neighbors - Wikipedia, ([https://en.wikipedia.org/wiki/K-nearest_neighbors_algorithm](https://en.wikipedia.org/wiki/K-nearest_neighbors_algorithm))
- SVM Classifier - Wikipedia, ([https://en.wikipedia.org/wiki/Support-vector_machine](https://en.wikipedia.org/wiki/Support-vector_machine))
- Decision Tree - Wikipedia, ([https://en.wikipedia.org/wiki/Decision_tree](https://en.wikipedia.org/wiki/Decision_tree))
- Random Forest - Wikipedia, ([https://en.wikipedia.org/wiki/Random_forest](https://en.wikipedia.org/wiki/Random_forest))
- K-Means - Wikipedia, ([https://en.wikipedia.org/wiki/K-means_clustering](https://en.wikipedia.org/wiki/K-means_clustering))
- DBSCAN - Wikipedia, ([https://en.wikipedia.org/wiki/DBSCAN](https://en.wikipedia.org/wiki/DBSCAN))
- Hierarchical Clustering - Wikipedia, ([https://en.wikipedia.org/wiki/Hierarchical_clustering](https://en.wikipedia.org/wiki/Hierarchical_clustering))
- BIRCH - Wikipedia, ([https://en.wikipedia.org/wiki/BIRCH](https://en.wikipedia.org/wiki/BIRCH))
- Confusion matrix - Wikipedia, ([https://en.wikipedia.org/wiki/Confusion_matrix#Confusion_matrices_with_more_than_two_categories](https://en.wikipedia.org/wiki/Confusion_matrix#Confusion_matrices_with_more_than_two_categories))
