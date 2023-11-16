---
title : "5.1. Geospatial Filtering"
weight : 10
socialshare: true
description : "We use Civil IoT Taiwan's earthquake and disaster prevention and relief data and filter the data of specific administrative areas by overlaying the administrative area boundary map obtained from the government open data platform. Then, we generate the image file of the data distribution location after the superimposed map. In addition, we also demonstrate how to Nest specific geometric topological regions and output nested results to files for drawing operations."
tags: ["Python", "Water", "Air" ]
levels: ["beginner" ]
authors: ["Ming-Kuang Chung", "Tze-Yu Sheng"]
---

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/drive/1qSeWVhJr8usqQQzJwRTS-6hp8KV4bbz9?usp=sharing)


{{< toc >}}

In the past, if we wanted to learn about things like air quality, earthquakes, or floods, we usually depended on information from public sources or experts to understand how widespread and intense these events were.

Imagine you want to check the air quality around your home. The Taiwan Air Quality Monitoring Network, managed by the Environmental Protection Administration Executive Yuan, is a key source of this info. But these high-tech weather stations are expensive and not very numerous, so the closest one might be 10 kilometers away. This distance leads to a question: does the air quality stay the same over such a distance? On the other hand, smaller, less expensive sensors used in the Internet of Things (IoT) can be placed closer to where we live. These sensors can give us a clearer picture of how air quality is affected by local elements like schools, busy streets, temples, or even home cooking. So, how do we find and use data from the right sensor stations?

Every IoT station has a specific location. Stations closer to each other often record similar data because they are in similar environments. This idea is summarized in the first law of geography by Waldo R. Tobler: “All things are related, but nearby things are more related than distant things.”

However, different factors around each station can cause variations in the data. To ensure we get reliable data, we need to look at each station as a central point, choose nearby stations based on their administrative area or a specific distance (like a radius), and then display this information in charts or maps.

In this chapter, we’ll learn how to select and use spatial data from air quality monitoring stations (Environmental Protection Administration, EPA), weather stations (Central Weather Bureau, CWB), and flood sensors (Water Resources Agency, WRA).

```python
import matplotlib.pyplot as plt
import seaborn as sns
import pandas as pd
import numpy as np
import urllib.request
import ssl
import json
#install geopython libraries
!apt install gdal-bin python-gdal python3-gdal
#install python3-rtree - Geopandas requirement
!apt install python3-rtree 
#install geopandas
!pip install geopandas

#install descartes - Geopandas requirement
!pip install descartes
import geopandas as gpd
!pip install pyCIOT
import pyCIOT.data as CIoT
```

```python
# downlaod the county boundary shpfile from open database
!wget -O "shp.zip" -q "https://data.moi.gov.tw/MoiOD/System/DownloadFile.aspx?DATA=72874C55-884D-4CEA-B7D6-F60B0BE85AB0"
!unzip shp.zip -d shp
```

```python
# get flood sensors' data by pyCIOT
wa = CIoT.Water().get_data(src="FLOODING:WRA")
wa2 = CIoT.Water().get_data(src="FLOODING:WRA2")
wea_list = CIoT.Weather().get_station('GENERAL:CWB')
county = gpd.read_file('county.shp')
basemap = county.loc[county['COUNTYNAME'].isin(["嘉義縣","嘉義市"])]

# convert to geopandas.GeoDataFrame
flood_list = wa + wa2

flood_df = pd.DataFrame([],columns = ['name',  'Observations','lon', 'lat'])
for i in flood_list:
    #print(i['data'][0])
    if len(i['data'])>0:
        df = pd.DataFrame([[i['properties']['stationName'],i['data'][0]['values'][0]['value'],i['location']['longitude'],i['location']['latitude']]],columns = ['name',  'Observations','lon', 'lat'])
    else :
        df = pd.DataFrame([[i['properties']['stationName'],-999,-999,-999]],columns = ['name',  'Observations','lon', 'lat'])
    flood_df = pd.concat([flood_df,df])
    #print(df)

result_df = flood_df.drop_duplicates(subset=['name'], keep='first')
station=result_df.sort_values(by=['lon', 'lat'])
station = station[station.lon!=-999]
station.reset_index(inplace=True, drop=True)
gdf_flood = gpd.GeoDataFrame(
    station, geometry=gpd.points_from_xy(station.lon, station.lat),crs="EPSG:4326")

weather_df = pd.DataFrame([],columns = ['name','lon', 'lat'])
for i in wea_list:
    #print(i['data'][0])
    df = pd.DataFrame([[i['name'],i['location']['longitude'],i['location']['latitude']]],columns = ['name','lon', 'lat'])

    weather_df = pd.concat([weather_df,df])
    #print(df)

result_df = weather_df.drop_duplicates(subset=['name'], keep='first')
station=result_df.sort_values(by=['lon', 'lat'])
station.reset_index(inplace=True, drop=True)
gdf_weather = gpd.GeoDataFrame(
    station, geometry=gpd.points_from_xy(station.lon, station.lat),crs="EPSG:4326")
```

## Intersect

In simple terms, we can focus on areas like villages or towns as our main points of interest. By crossing different sets of data, we can identify the specific station IDs located within one of these areas. With the help of an API, we're able to access real-time data, as well as averages calculated over an hour, a day, or even a week, from these stations. Additionally, this process allows us to compare stations within the same area to see if their data trends are similar. We can also spot any stations that are reporting data that stands out or is very different from the rest.

```python
import matplotlib.pyplot as plt
import seaborn as sns

fig, ax = plt.subplots(figsize=(6, 10))
ax = sns.scatterplot(x='lon', y='lat', data=gdf_weather) 
# this is plotting the datapoints from the EPA dataframe
basemap.plot(ax=ax, facecolor='none', edgecolor='purple');
# plotting the city's boundaries here, with facecolor = none to 
# remove the polygon's fill color
plt.tight_layout();
```

![Python output](figures/5-1-1-1.png)

```python
# selecting the polygon's geometry field to filter out points that 
basemap = basemap.set_crs(4326,allow_override=True)
intersected_data = gpd.overlay(gdf_weather, basemap, how='intersection') 

```

```python
fig, ax = plt.subplots(figsize=(6, 10))
ax = sns.scatterplot(x='lon', y='lat', data=intersected_data) 
basemap.plot(ax=ax, facecolor='none', edgecolor='purple');
plt.tight_layout();
```

![Python output](figures/5-1-1-2.png)

## Buffer

Additionally, some monitoring stations might be right on the edge of different areas. If we just use the official area borders to choose stations, our results might not be very accurate. To solve this, we use a method called 'buffering'. This means we pick a station, then draw an imaginary circle around it using a set distance. This circle helps us include stations that are close by, even if they're just outside the area's official border.

With this 'buffering' idea, we can also pick important places like schools, parks, or factories as the center points and then look for stations near them. Also, if we use things like roads, rivers, or even larger areas like parks or industrial zones as our starting points, we can create a search area. This helps us find the exact monitoring station we need, in a much more precise way.

![圖1：不同緩衝區的概念示意圖](figures/5-1-2-1.png)


```python
# set the buffer distance for  0.05 degree
fig, ax = plt.subplots(figsize=(6, 10))
buffer = intersected_data.buffer(0.05)
buffer.plot(ax=ax, alpha=0.5)
intersected_data.plot(ax=ax, color='red', alpha=0.5)
basemap.plot(ax=ax, facecolor='none', edgecolor='purple');
plt.tight_layout();
```

![Python output](figures/5-1-2-2.png)

## Multi-ring buffer

We can create circles with different sizes around a central point, like targets with multiple rings. This helps us group stations that are close to each other based on how far away they are. We can then see if stations that are closer together have more similar readings or trends.

![圖2：多重緩衝區的概念圖](figures/5-1-3-1.png)


```python
# set the buffer distance 0.05 degree=blue；0.1 degree=green；0.2 degree=orange；0.3 degree=red
fig, ax = plt.subplots(figsize=(6, 10))
buffer_03 = intersected_data.buffer(0.3)
buffer_03.plot(ax=ax, color='red', alpha=1)
buffer_02 = intersected_data.buffer(0.2)
buffer_02.plot(ax=ax, color='orange', alpha=1)
buffer_01 = intersected_data.buffer(0.1)
buffer_01.plot(ax=ax, color='green', alpha=1)
buffer_005 = intersected_data.buffer(0.05)
buffer_005.plot(ax=ax, alpha=1)
intersected_data.plot(ax=ax, color='black', alpha=0.5)

# intersect with the flood sensors
buffer = gpd.GeoDataFrame(buffer_03,geometry=buffer_03)
buffer = buffer.to_crs(4326)
intersected_flood = gpd.overlay(gdf_flood, buffer, how='intersection') 
intersected_flood.plot(ax=ax, color='lightgray', alpha=0.5)
basemap.plot(ax=ax, facecolor='none', edgecolor='purple');
plt.tight_layout();
```

![Python output](figures/5-1-3-2.png)

## Distance matrix

Each observation station has its own set of precise location coordinates. By using these coordinates along with trigonometric functions, we can calculate the exact distance between any two stations. This calculation lets us create what's known as a distance matrix for all the stations. A distance matrix is simply a way to see at a glance how far apart different stations are from each other. This is especially useful for quickly checking if two stations are geographically close to one another (as shown in graph 3). For instance, in Taipei, we can apply this method to the locations of flood sensors. By converting their locations into a distance matrix, we can easily determine if these sensors are located near each other during flood events.

![圖3：距離矩陣的概念圖](figures/5-1-4-1.png)


```python
gdf_weather["ID"] = gdf_weather.index
df = pd.DataFrame(gdf_weather, columns=['ID','lon', 'lat'])
df.set_index('ID')
```

![Python output](figures/5-1-4-2.png)

```python
# convert to the dustance matrix
from scipy.spatial import distance_matrix

pd.DataFrame(distance_matrix(df.values, df.values), 
							index=df.index, columns=df.index)
```

![Python output](figures/5-1-4-3.png)

## Brief Summary

Using the methods described earlier, we can develop a station selection mechanism based on administrative regions or topological concepts. Additionally, we can explore the relationships between sensor values by analyzing the geospatial features of these stations.

# References

- Geopanda documentation ([https://geopandas.org/en/stable/docs.html](https://geopandas.org/en/stable/docs.html))
- Introduction to the shp file ([https://en.wikipedia.org/wiki/Shapefile](https://en.wikipedia.org/wiki/Shapefile))
- Taiwan's  Datum and coordinate system ([https://wiki.osgeo.org/wiki/Taiwan_datums](https://wiki.osgeo.org/wiki/Taiwan_datums))
- Introduction to WGS 84 (****EPSG:4326****) coordinate system ([https://epsg.io/4326](https://epsg.io/4326))
- Introduction to TWD 97 (****EPSG:3826****) coordinate system ([https://epsg.io/3826](https://epsg.io/3826))
