---
title : "4.3. Time Series Data Clustering"
weight : 30
socialshare : true
description : "We introduce advanced data grouping analysis. We first present two time-series feature extraction methods, Fourier transform and wavelet transform, and briefly explain the similarities and differences between the two transform methods. We introduce two different time series comparison methods, Euclidean distance and dynamic time warping (DTW), and apply existing clustering algorithms accordingly. We then discuss data clustering with different temporal resolutions, as well as their representation and potential applications in the real world."
tags: ["Python", "Air" ]
levels: ["advanced"]
authors: ["Yu-Chi Peng"]
---



[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/drive/1n01pJ_3HlUN3iOru3G1E9onkUlSfu-Z6?usp=sharing)

{{< toc >}}

Cluster analysis is a popular technique in data science aimed at identifying groups, or 'clusters', of similar data points. This method helps organize data by grouping similar items together, making it easier for researchers to conduct more detailed analyses on these sets. In the context of Civil IoT Taiwan, where the data from sensors is in the form of time series (data points collected over time), this approach is especially useful. To effectively group numerous sensors for comprehensive data analysis, standard methods for extracting key features are used. These methods help in classifying time series data into clusters for more insightful analysis.

## Goal

- Understand how to apply Fast Fourier Transform (FFT) and Wavelet Transform for feature extraction in time series data. These techniques help in breaking down complex data patterns into simpler components, making it easier to analyze and understand the data.
- Employ unsupervised learning methods for clustering time series data. This approach doesn't rely on pre-labeled data; instead, it discovers natural groupings within the data based on the similarities and differences in the data points.

## Package Installation and Importing

In this guide, we'll be using several pre-installed packages on our development platform, Google Colab. These include pandas, matplotlib, numpy, and pywt, all of which are ready to use without any manual installation. However, there's an additional package, tslearn, that isn't pre-installed on Colab. We'll need to install this package separately. To do so, we'll follow a specific installation process which I'll describe next.

```python
!pip install --upgrade pip # This command upgrades 'pip' to the latest version. 'pip' is the package installer for Python, used to install and manage software packages.
!pip install tslearn # This command installs the 'tslearn' package. 'tslearn' is a machine learning library for time series analysis in Python.
```

Once the installation is finished, we can start importing the necessary packages to set up our workspace for this tutorial. Here's how we'll do it: we'll use a specific syntax to import each of these packages into our coding environment. This step is crucial as it prepares our platform with all the tools we need for the tasks ahead in this guide.

```python
import numpy as np # Imports NumPy, a library for numerical operations. 'np' is a common alias used for NumPy.
import pandas as pd # Imports Pandas, a library for data manipulation and analysis. 'pd' is a standard alias for Pandas.
import pywt # Imports PyWavelets, a library for wavelet transformation, often used for signal processing.
import os, zipfile # Imports 'os' and 'zipfile' for operating system interactions and handling zip files, respectively.
import matplotlib.pyplot as plt # Imports Matplotlib's 'pyplot', a plotting library. 'plt' is a conventional alias.

from datetime import datetime, timedelta # Imports specific classes from the 'datetime' module for handling dates and times.
from numpy.fft import fft, ifft # Imports 'fft' and 'ifft' from NumPy's Fast Fourier Transform (FFT) sub-module for frequency domain transformations.
from pywt import cwt # Imports 'cwt' (Continuous Wavelet Transform) from PyWavelets for time-frequency analysis.
from tslearn.clustering import TimeSeriesKMeans # Imports TimeSeriesKMeans for clustering time series data from the 'tslearn' library.
```

## Data Access and Preprocessing

In this tutorial, we're focusing on using long-term historical data. Instead of accessing data through the standard methods provided by the pyCIOT package, we'll take a different approach. We'll directly download the 2021 data archive of the "Academia Sinica - Micro Air Quality Sensors" from the historical database of the Civil IoT Taiwan Data Service Platform. Once downloaded, this data will be stored in a folder named `Air`. This method allows us to work with a comprehensive dataset, giving us a broader view of the historical trends and patterns.

```python
!mkdir Air CSV_Air # Creates two directories: 'Air' and 'CSV_Air'. These will be used to store downloaded and extracted data.

# This command uses 'wget' to download a file from the provided URL. 
# The '-O Air/2021.zip' option specifies that the downloaded file should be named '2021.zip' and saved in the 'Air' directory.
# The '-q' option runs the command in quiet mode, suppressing most of the command output.
!wget -O Air/2021.zip -q "https://history.colife.org.tw/?r=/download&path=L%2Bepuuawo%2BWTgeizqi%2FkuK3noJTpmaJf5qCh5ZyS56m65ZOB5b6u5Z6L5oSf5ris5ZmoLzIwMjEuemlw"

!unzip Air/2021.zip -d Air # This command unzips the contents of '2021.zip' into the 'Air' directory.
```
As we proceed, it's important to note that the data we download will be in a zip file format. This means our first step is to unzip this file. Doing so will reveal a collection of daily files, each also compressed. Specifically, we'll focus on the data from August 2021. Our task is to decompress these daily files for this month and then store the contents from these unzipped CSV files into a dataframe, which we will refer to as `air_month`. This process enables us to organize and access the data efficiently for our analysis.

```python
folder = 'Air/2021/202108' # Sets the folder path where the data is stored.
extension_zip = 'zip' # Defines the extension for zip files.
extension_csv = 'csv' # Defines the extension for CSV files.

# Loop to extract files from zip archives in the specified folder
for item in os.listdir(folder): 
  if item.endswith(extension_zip): # Checks if the file is a zip file.
    file_name = f'{folder}/{item}' # Creates the full path for the zip file.
    zip_ref = zipfile.ZipFile(file_name) # Opens the zip file.
    zip_ref.extractall(folder) # Extracts all contents to the folder.
    zip_ref.close() # Closes the zip file.

air_month = pd.DataFrame() # Initializes an empty DataFrame to store the combined data.

# Loop to read CSV files and append them to the DataFrame
for item in os.listdir(folder):
  if item.endswith(extension_csv): # Checks if the file is a CSV file.
    file_name = f'{folder}/{item}' # Creates the full path for the CSV file.
    df = pd.read_csv(file_name, parse_dates=['timestamp']) # Reads the CSV file into a DataFrame, parsing 'timestamp' as dates.
    air_month = air_month.append(df) # Appends the DataFrame to 'air_month'.

# Setting 'timestamp' as the index and sorting the DataFrame
air_month.set_index('timestamp', inplace=True) # Sets 'timestamp' as the index of the DataFrame.
air_month.sort_values(by='timestamp', inplace=True) # Sorts the DataFrame by 'timestamp'.
```
Currently, the `air_month` dataframe isn't structured in a way that suits our analysis needs. We need to rearrange it so that the data from different sites are in separate columns, and time data is organized in rows. To start this reformatting process, our first step is to identify all the unique sites represented in our data. We'll gather this information and store it in a sequence. This step is essential for understanding the range of locations our data covers, and it will help us in efficiently restructuring the `air_month` dataframe for more effective data analysis.

```python
# Extract the 'device_id' column from the 'air_month' DataFrame and convert it to a NumPy array.
# 'device_id' contains the IDs of the devices used for air quality measurement.
id_list = air_month['device_id'].to_numpy()

# Use NumPy's 'unique' function to find all the unique device IDs in the array.
# This removes any duplicate IDs, giving us a list of distinct devices used in the data collection.
id_uniques = np.unique(id_list)

# The variable 'id_uniques' now holds the unique device IDs from your dataset.
id_uniques
```

```
array(['08BEAC07D3E2', '08BEAC09FF12', '08BEAC09FF22', ...,
       '74DA38F7C648', '74DA38F7C64A', '74DA38F7C64C'], dtype=object)
```

Next, we'll organize the data from each unique site into its own column, compiling all these columns into a new dataframe named `air`. This structure will make the data more accessible and easier to analyze, with each site's data clearly separated and organized chronologically.

After completing this step, it's important to clean up our workspace. We'll do this by deleting all the originally downloaded and unpacked data files. This cleanup is crucial for conserving cloud storage space, ensuring our workspace remains efficient and uncluttered. This practice is not only good for organization but also essential in a cloud computing environment where storage space and neatness are key.

```python
# Initialize an empty DataFrame 'air' to store aggregated data.
air = pd.DataFrame()

# Iterate over each unique device ID.
for i in range(len(id_uniques)):
    # Construct a query string to filter data for the current device ID.
    # This step is essential for processing data device-wise.
    query = air_month.query('device_id=="' + id_uniques[i] + '"')

    # Sort the filtered data by 'timestamp' to ensure chronological order.
    query.sort_values(by='timestamp', inplace=True)

    # Resample the data to an hourly frequency and compute the mean for each hour.
    # This step aggregates the data, making it more manageable and easier to analyze.
    query_mean = query.resample('H').mean()

    # Rename the 'PM25' column to the current device ID for clarity.
    query_mean.rename(columns={'PM25': id_uniques[i]}, inplace=True)

    # Concatenate the processed data for the current device to the 'air' DataFrame.
    # The 'axis=1' parameter indicates a horizontal concatenation (column-wise).
    air = pd.concat([air, query_mean], axis=1)

!rm -rf Air
```
To quickly inspect the contents of our newly organized `air` dataframe, we can use a specific syntax. This command will display a snapshot of the dataframe, allowing us to verify its structure and the data it contains. This step is important for ensuring that our data reorganization was successful and that the dataframe is ready for further analysis.

```python
# Displaying the structure and summary information of the 'air' DataFrame.
air.info()

# Printing the first five rows of the 'air' DataFrame to get a quick overview of the data.
print(air.head())
```

```
<class 'pandas.core.frame.DataFrame'>
DatetimeIndex: 745 entries, 2021-08-01 00:00:00 to 2021-09-01 00:00:00
Freq: H
Columns: 1142 entries, 08BEAC07D3E2 to 74DA38F7C64C
dtypes: float64(1142)
memory usage: 6.5 MB
                     08BEAC07D3E2  08BEAC09FF12  08BEAC09FF22  08BEAC09FF2A  \
timestamp                                                                     
2021-08-01 00:00:00      2.272727      1.250000     16.818182     12.100000   
2021-08-01 01:00:00      1.909091      1.285714     13.181818     14.545455   
2021-08-01 02:00:00      2.000000      1.125000     12.727273     16.600000   
2021-08-01 03:00:00      2.083333      3.000000     11.800000     12.090909   
2021-08-01 04:00:00      2.000000      2.600000     10.090909      8.545455   

                     08BEAC09FF34  08BEAC09FF38  08BEAC09FF42  08BEAC09FF44  \
timestamp                                                                     
2021-08-01 00:00:00      2.727273      0.000000      1.181818           NaN   
2021-08-01 01:00:00      2.000000      0.545455      0.909091           NaN   
2021-08-01 02:00:00      4.090909      1.583333      0.636364           NaN   
2021-08-01 03:00:00      0.545455      1.454545      1.181818           NaN   
2021-08-01 04:00:00      1.363636      1.363636      2.454545           NaN   

                     08BEAC09FF46  08BEAC09FF48  ...  74DA38F7C62A  \
timestamp                                        ...                 
2021-08-01 00:00:00      2.636364      2.545455  ...      6.777778   
2021-08-01 01:00:00      1.636364      2.272727  ...      7.800000   
2021-08-01 02:00:00      1.400000      3.100000  ...      7.300000   
2021-08-01 03:00:00      2.181818      4.000000  ...     12.000000   
2021-08-01 04:00:00      1.909091      2.100000  ...      9.000000   

                     74DA38F7C630  74DA38F7C632  74DA38F7C634  74DA38F7C63C  \
timestamp                                                                     
2021-08-01 00:00:00      9.800000     13.200000           5.0      6.200000   
2021-08-01 01:00:00     13.000000     15.700000           5.2      6.800000   
2021-08-01 02:00:00     12.800000     19.300000           5.0      7.300000   
2021-08-01 03:00:00      8.444444     15.200000           5.1      6.777778   
2021-08-01 04:00:00      6.500000     10.222222           4.9      6.100000   

                     74DA38F7C63E  74DA38F7C640  74DA38F7C648  74DA38F7C64A  \
timestamp                                                                     
2021-08-01 00:00:00           NaN      7.500000      5.000000      9.000000   
2021-08-01 01:00:00           NaN     10.200000      4.900000      6.600000   
2021-08-01 02:00:00           NaN     10.500000      3.666667      7.600000   
2021-08-01 03:00:00           NaN      8.500000      8.400000      8.000000   
2021-08-01 04:00:00           NaN      5.571429      6.200000      5.666667   

                     74DA38F7C64C  
timestamp                          
2021-08-01 00:00:00      7.600000  
2021-08-01 01:00:00      7.700000  
2021-08-01 02:00:00      7.888889  
2021-08-01 03:00:00      6.400000  
2021-08-01 04:00:00      5.000000  
```

Moving forward, our next step involves cleaning the data in the `air` dataframe. We'll specifically focus on removing any parts of the data that contain missing values, often represented as 'NaN' (Not a Number). This step is crucial for ensuring the accuracy and reliability of our analysis.

Once we've cleaned the data by removing these missing values, the next step is to visualize the data. We'll do this by plotting the data in a graph. This visualization will help us to observe and understand the distribution and patterns within the data more clearly. Graphical representation is a powerful tool for gaining insights and making data more accessible and interpretable.

```python
# Removing the last row from the 'air' DataFrame.
air = air[:-1]

# Creating a new DataFrame 'air_clean' by dropping columns with any missing values.
air_clean = air.dropna(1, how='any')

# Plotting the data in 'air_clean' with a specified figure size and without displaying the legend.
air_clean.plot(figsize=(20, 15), legend=None)
```

![Python output](figures/4-3-3-1.png)

The original data from the sensor can show sudden and dramatic changes because of changes in the environment. To make this data more consistent and reliable, we apply a technique called the moving average method. This approach averages the sensor readings every ten instances. By doing this, the resulting data becomes smoother and more stable. It also more accurately shows the general trends in the area around the sensor, which is helpful for the next step of our analysis, known as cluster analysis.

```python
# Applying a rolling mean with a window of 10 and a minimum of 1 period to the 'air_clean' DataFrame.
air_clean = air_clean.rolling(window=10, min_periods=1).mean()

# Plotting the modified 'air_clean' data with a specified figure size and without a legend.
air_clean.plot(figsize=(20, 15), legend=None)
```

![Python output](figures/4-3-3-2.png)

## Data Clustering

In our current data setup, each weather station's data is placed in a separate column in a dataframe, with each row showing the sensor readings from all stations at a specific moment. This layout is good for analyzing data over time, but not as useful for grouping data. To prepare for data grouping, we first need to rearrange the data by transposing it, essentially switching the rows and columns.

Data clustering is a popular technique in data science, and we're using a specific method called KMeans clustering, provided by the `tslearn` package. This method falls under 'unsupervised learning' in machine learning. Unlike other methods where data is sorted into groups based on predefined criteria, KMeans clustering groups data based on how similar the data points are to each other. This makes it great for identifying unusual data points or for making predictions.

Here's how the KMeans clustering process works:

1. Choose the number of clusters, *k*. We decide how many groups we want to create.
2. Randomly pick *k* data points as the starting points for these clusters, known as 'cluster heads.'
3. Measure how close each data point is to these cluster heads, and assign each point to the nearest cluster.
4. Update the center of each cluster based on the points assigned to it, and repeat the process until the cluster heads don't change anymore.

We'll start by setting *k=10*, meaning we want to create 10 clusters. Using `TimeSeriesKMeans`, we'll divide the data into these 10 groups, labeled from 0 to 9.

```python
# Transposing the 'air_clean' DataFrame for time series clustering.
air_transpose = air_clean.transpose()

# Initializing the Time Series K-Means model with 10 clusters, using the DTW (Dynamic Time Warping) metric, and setting a maximum of 5 iterations.
# 'n_clusters' specifies the number of clusters, 'max_iter' sets the maximum number of iterations for the clustering process.
model = TimeSeriesKMeans(n_clusters=10, metric="dtw", max_iter=5) # n_cluster:分群數量, max_iter: 分群的步驟最多重複幾次

# Fitting the model to the transposed 'air_clean' DataFrame.
pre = model.fit(air_transpose)

# Retrieving the cluster labels for each time series in the dataset.
pre.labels_
```

```
array([3, 3, 6, 3, 3, 3, 3, 6, 9, 6, 3, 6, 3, 3, 3, 3, 3, 1, 3, 3, 6, 3,
3, 3, 3, 6, 6, 6, 6, 3, 3, 3, 3, 3, 3, 3, 3, 3, 6, 3, 9, 3, 3, 3,
3, 3, 6, 3, 2, 6, 3, 2, 3, 3, 3, 6, 9, 3, 3, 3, 3, 6, 6, 3, 3, 3,
6, 6, 3, 3, 3, 3, 3, 6, 3, 6, 3, 3, 3, 3, 3, 6, 6, 3, 3, 3, 6, 3,
3, 3, 6, 3, 3, 3, 3, 3, 3, 6, 9, 6, 7, 3, 2, 3, 6, 3, 3, 3, 6, 3,
3, 3, 3, 6, 3, 3, 3, 6, 3, 6, 6, 3, 6, 3, 3, 3, 3, 6, 3, 3, 3, 3,
3, 3, 3, 3, 3, 3, 3, 0, 3, 6, 3, 3, 3, 0, 6, 0, 6, 3, 0, 3, 3, 3,
6, 6, 6, 6, 3, 3, 6, 3, 3, 9, 6, 3, 6, 3, 6, 6, 3, 2, 3, 3, 3, 6,
9, 6, 6, 3, 3, 6, 3, 0, 3, 3, 6, 6, 6, 3, 0, 0, 6, 3, 6, 6, 6, 3,
6, 6, 3, 0, 3, 3, 0, 3, 3, 6, 3, 6, 6, 6, 3, 0, 3, 0, 0, 0, 9, 0,
0, 0, 0, 0, 0, 0, 0, 0, 0, 9, 9, 9, 9, 0, 0, 0, 9, 9, 0, 0, 0, 4,
9, 9, 9, 9, 0, 0, 0, 0, 0, 0, 0, 9, 0, 9, 0, 0, 0, 9, 9, 0, 9, 0,
0, 0, 5, 0, 0, 0, 0, 9, 9, 0, 1, 0, 9, 2, 9, 9, 0, 0, 5, 0, 9, 0,
9, 0, 0, 0, 3, 0, 0, 0, 0, 0, 3, 2, 3, 3, 0, 3, 0, 0, 0, 6, 3, 0,
8, 9, 3, 9, 9, 0, 5, 0, 3, 0, 9, 9, 5, 5, 5, 3, 5, 9, 3, 0, 0, 3,
0, 0, 0, 3, 5, 5, 3, 0, 5, 6, 5, 5, 5, 0, 9, 0, 6, 6, 6, 3, 0, 3,
5, 5, 9, 6, 3, 0, 0, 0, 0, 3, 5, 0, 5, 5, 0, 3, 3, 9, 5, 5, 9, 5,
9, 9, 9, 9, 5, 5, 5, 5, 5, 5, 9, 5, 5, 5, 5, 5, 5, 5, 5, 5, 9, 5,
5, 5])
```
After we apply the clustering method, sometimes we find that certain sensors form their own cluster because their data is unique. These sensors are outliers and generally don't provide useful information for further analysis. So, our next step is to check if any cluster contains only a single sensor. If it does, we consider this cluster irrelevant and remove it from our analysis. For instance, in our current case, we will eliminate any cluster that is made up of just one sensor, as shown below:

```python
# Creating a DataFrame 'df_cluster' to map each metric in 'air_clean' to its corresponding cluster label.
df_cluster = pd.DataFrame(list(zip(air_clean.columns, pre.labels_)), columns=['metric', 'cluster'])

cluster_metrics_dict = df_cluster.groupby(['cluster'])['metric'].apply(lambda x: [x for x in x]).to_dict() # Creating a dictionary 'cluster_metrics_dict' that maps each cluster to the metrics it contains.
cluster_len_dict = df_cluster['cluster'].value_counts().to_dict() # Creating a dictionary 'cluster_len_dict' to count the number of metrics in each cluster.
clusters_dropped = [cluster for cluster in cluster_len_dict if cluster_len_dict[cluster]==1] # Creating a list 'clusters_dropped' containing clusters with only one metric.
clusters_final = [cluster for cluster in cluster_len_dict if cluster_len_dict[cluster]>1] # Creating a sorted list 'clusters_final' containing clusters with more than one metric.
clusters_final.sort()

# Outputting the list of clusters that were dropped.
clusters_dropped
```

```
[7, 4, 8]
```

In the final step, we visually present the clusters that remain after removing the irrelevant ones. When we look at these clusters, it's clear that the data within each cluster are very similar to each other, while the data across different clusters show noticeable differences. To measure how similar the data is within each cluster, we introduce a new metric called `quality`.

The `quality` of a cluster is calculated by averaging the correlation between each piece of data within the cluster and all other data in that same cluster. The idea is that the lower the `quality` value, the more alike the data in that cluster is. In other words, a lower `quality` score indicates a high degree of similarity among the data points in a cluster.

```python
# Creating a subplot for each cluster in 'clusters_final' with specific dimensions and resolution.
fig, axes = plt.subplots(nrows=len(clusters_final), ncols=1, figsize=(20, 15), dpi=500)

# Iterating over each cluster to create individual plots.
for idx, cluster_number in enumerate(clusters_final):
  x_corr = air_clean[cluster_metrics_dict[cluster_number]].corr().abs().values # Calculating the absolute correlation matrix for the metrics in the current cluster.
  x_corr_mean = round(x_corr[np.triu_indices(x_corr.shape[0],1)].mean(),2) # Calculating the mean of the upper triangle of the correlation matrix, rounding to two decimal places.
  plot_title = f'cluster {cluster_number} (quality={x_corr_mean}, n={cluster_len_dict[cluster_number]})' # Creating a title for the plot with cluster number, quality (mean correlation), and number of metrics.
  air_clean[cluster_metrics_dict[cluster_number]].plot(ax=axes[idx], title=plot_title) # Plotting the metrics in the current cluster on the respective subplot.
  axes[idx].get_legend().remove() # Removing the legend from each subplot for clarity.

# Adjusting the layout for better visibility and aesthetics.
fig.tight_layout()

# Displaying the plot.
plt.show()
```

![Python output](figures/4-3-4-1.png)

In our analysis, we've shown how to cluster air quality sensor data based on specific characteristics in the data. However, it's important to note that the original data might have some noise - unwanted variations or disturbances. This noise can affect the accuracy of our clustering results. To minimize the impact of this noise, we use two advanced methods: the Fourier transform and the wavelet transform. These methods help us extract more refined features from the time series data, which in turn improves the quality of our clustering.

### Fast Fourier Transform

The Fourier transform is a widely used method in signal analysis. It transforms data from the time domain to the frequency domain, making it easier to extract features and analyze the data. This technique is essential in fields like engineering and mathematics and is particularly useful in analyzing sound and time series data.

However, the standard Fourier transform involves complex mathematical computations, making it impractical for large datasets due to its time-consuming nature. To address this, the Fast Fourier Transform (FFT) was developed. FFT is a quicker way to perform the Fourier transform, significantly reducing the complexity and computational time, especially when dealing with large amounts of data.

In our next step, we'll use the Fast Fourier Transform method to extract relevant features for data grouping. To start, we'll examine the time variation of a single station's data (identified by the ID ''`08BEAC07D3E2`'') by creating a plot. This will give us a clearer understanding of how this method works and its effectiveness in our analysis.

```python
# Plotting the time series data for the column '08BEAC07D3E2' in the 'air_clean' DataFrame.
air_clean['08BEAC07D3E2'].plot()
```

![Python output](figures/4-3-4-2.png)

Next, we'll use the Fast Fourier Transform (FFT) tool, specifically the `fft` function from the NumPy package, to transform the data from our chosen station. By applying FFT, we change the way we view the data, shifting from the time domain to the frequency domain. This shift is key in analyzing and understanding the underlying patterns and characteristics of the data.

Once we've applied the FFT to the station's data, we'll visually represent the transformed data. This involves creating a graph to show how the data is distributed in the frequency domain after the transformation. This graphical representation will help us better understand the nature of the data and the impact of the FFT, illustrating the key features and patterns that emerge when we view the data through the lens of frequency rather than time.

```python
# Performing Fast Fourier Transform (FFT) on the '08BEAC07D3E2' column of the 'air_clean' DataFrame.
X = fft(air_clean['08BEAC07D3E2'])
N = len(X)
n = np.arange(N)

# Determining the sampling rate, assuming each data point represents an hourly interval.
sr = 1 / (60*60)

# Calculating the total time duration of the dataset.
T = N/sr

# Computing the frequency array.
freq = n/T 

# Limiting the spectrum to one-sided (since FFT output is symmetric for real-valued inputs).
n_oneside = N // 2

# Considering only the one-sided frequencies.
n_oneside = N//2
# get the one side frequency
f_oneside = freq[:n_oneside]

# Plotting the one-sided spectrum.
plt.figure(figsize = (12, 6))
plt.plot(f_oneside, np.abs(X[:n_oneside]), 'b') # 'b' specifies the color blue for the plot.
plt.xlabel('Freq (Hz)')
plt.ylabel('FFT Amplitude |X(freq)|')
plt.show()
```

![Python output](figures/4-3-4-3.png)

After successfully applying the Fast Fourier Transform (FFT) to the data from one station and analyzing its frequency domain representation, our next step is to extend this transformation to all the sensors in our dataset. We will apply the FFT tool from the NumPy package to each sensor's data, converting their time-domain data into the frequency domain.

Once we've transformed the data from all sensors using FFT, we will collect and store these results in a variable named `air_fft`. This variable will hold the frequency domain data for each sensor, allowing us to analyze and compare the transformed data across all sensors. This comprehensive approach helps us gain a deeper understanding of the entire dataset, revealing patterns and insights that might not be evident when looking at individual sensors or the time-domain data alone.

```python
# Initializing an empty DataFrame to store the FFT results.
air_fft = pd.DataFrame()

# Getting the column names from the 'air_clean' DataFrame.
col_names = list(air_clean.columns)

# Looping through each column in 'air_clean'.
for name in col_names:
  # Performing FFT on the column.
  X = fft(air_clean[name])
  N = len(X)
  n = np.arange(N)

  # Determining the sampling rate (assuming hourly data).
  sr = 1 / (60*60) # 1 hour in seconds

  # Calculating the total time duration.
  T = N/sr

  # Computing the frequency array.
  freq = n/T 

  # Limiting the FFT output to one-sided.
  n_oneside = N//2

  # Getting the one-sided frequency array.
  f_oneside = freq[:n_oneside]

  # Storing the amplitude of the one-sided FFT in the 'air_fft' DataFrame.
  air_fft[name] = np.abs(X[:n_oneside])

# Printing the first five rows of the 'air_fft' DataFrame to verify the results.
print(air_fft.head())
```

```
08BEAC07D3E2  08BEAC09FF22  08BEAC09FF2A  08BEAC09FF42  08BEAC09FF48  \
0  12019.941563  11255.762431  13241.408211  12111.740447  11798.584351   
1   3275.369377   2640.372699   1501.672853   3096.493565   3103.006928   
2   1109.613670   1201.362571    257.659947   1085.384353   1128.373457   
3   2415.899146   1631.128345    888.838822   2281.597031   2301.936400   
4   1130.973327    446.032078    411.940722   1206.512460   1042.054041   

   08BEAC09FF66  08BEAC09FF80  08BEAC09FF82  08BEAC09FF8C  08BEAC09FF9C  ...  \
0  12441.845754  11874.390693  12259.096742   3553.255916  13531.649701  ...   
1   3262.357287   2999.042917   1459.720167   1109.764942    646.846038  ...   
2   1075.877632   1005.445596    478.569869    368.572815   1163.425916  ...   
3   2448.646527   2318.870954    956.029693    272.486798    553.409732  ...   
4   1087.461354   1172.755489    437.920193    471.824176    703.557830  ...   

   74DA38F7C504  74DA38F7C514  74DA38F7C524  74DA38F7C554  74DA38F7C5BA  \
0  11502.841221  10589.689400  11220.068048  10120.220198  11117.146124   
1   2064.762890   1407.105290   1938.647888   1126.088084   2422.787262   
2   2163.528535   1669.014077   2054.586664   1759.326882   1782.523300   
3   1564.407983   1157.759192   1253.849261   1244.799149   1519.477057   
4   1484.232397   1177.909914   1318.704021   1106.349846   1373.167639   

   74DA38F7C5BC  74DA38F7C5E0  74DA38F7C60C  74DA38F7C62A  74DA38F7C648  
0  11243.094213     26.858333   9408.414826  11228.246949   8931.871618  
1   2097.343959     15.020106   1667.485473   1687.251179   1395.239491  
2   1806.524987     10.659603   1585.987276   1851.628868   1527.925427  
3   1521.392873      6.021244   1217.547879   1240.173667   1022.239794  
4   1393.469185      3.361938   1161.975844   1350.756178   1051.434944  

[5 rows x 398 columns]
```
After transforming all sensor data into the frequency domain using the Fast Fourier Transform (FFT), we proceed to the next step: clustering this transformed data. We use the same `TimeSeriesKMeans` method as before, but this time, we apply it to the frequency-domain data. Our goal is to divide this data into 10 distinct clusters.

Following the clustering, we'll review the composition of each cluster. Any cluster that contains only a single sensor will be considered irrelevant and removed from our analysis, just like we did earlier. This is because a cluster with only one sensor doesn't provide valuable comparative information.

In this specific example, after removing the single-sensor clusters, we end up with 9 clusters. We then document and display the cluster each sensor belongs to by printing their cluster codes. This process helps us understand how the sensors group together based on their frequency-domain data, offering insights that differ from the time-domain analysis. By comparing these clusters, we can gain a more nuanced understanding of the similarities and differences among the sensors in our study.

```python
# Transposing the 'air_fft' DataFrame for time series clustering.
fft_transpose = air_fft.transpose()

# Initializing the Time Series K-Means model with 10 clusters, using the DTW (Dynamic Time Warping) metric, and setting a maximum of 5 iterations.
# 'n_clusters' specifies the number of clusters, 'max_iter' sets the maximum number of iterations for the clustering process.
model = TimeSeriesKMeans(n_clusters=10, metric="dtw", max_iter=5)
pre = model.fit(fft_transpose) # Fitting the model to the transposed FFT data.

# Creating a DataFrame 'df_cluster' to map each metric in 'air_fft' to its corresponding cluster label.
df_cluster = pd.DataFrame(list(zip(air_fft.columns, pre.labels_)), columns=['metric', 'cluster'])


cluster_metrics_dict = df_cluster.groupby(['cluster'])['metric'].apply(lambda x: [x for x in x]).to_dict() # Creating a dictionary 'cluster_metrics_dict' mapping each cluster to the metrics it contains.
cluster_len_dict = df_cluster['cluster'].value_counts().to_dict() # Creating a dictionary 'cluster_len_dict' to count the number of metrics in each cluster.
clusters_dropped = [cluster for cluster in cluster_len_dict if cluster_len_dict[cluster]==1] # Creating a list 'clusters_dropped' containing clusters with only one metric.
clusters_final = [cluster for cluster in cluster_len_dict if cluster_len_dict[cluster]>1] # Creating a sorted list 'clusters_final' containing clusters with more than one metric.
clusters_final.sort()

# Printing the first 10 rows of the 'df_cluster' DataFrame.
print(df_cluster.head(10))
```

```
         metric  cluster
0  08BEAC07D3E2        7
1  08BEAC09FF22        3
2  08BEAC09FF2A        0
3  08BEAC09FF42        7
4  08BEAC09FF48        7
5  08BEAC09FF66        7
6  08BEAC09FF80        7
7  08BEAC09FF82        0
8  08BEAC09FF8C        2
9  08BEAC09FF9C        8
```
In the final stage of our analysis, we create individual plots for the sensor data of each of the nine remaining clusters. These plots will be based on the frequency-domain data we obtained after applying the Fast Fourier Transform (FFT).

By visually representing the data in this way, we can observe and compare the characteristics of the sensors within each cluster. What we expect to find is that the sensors grouped into the same cluster will display very similar patterns in the frequency domain. This similarity indicates that these sensors are responding to environmental factors in a similar way.

Conversely, when we compare the frequency-domain data of sensors from different clusters, we anticipate seeing more significant differences. This contrast highlights the distinct behaviors or responses of sensors in different environmental conditions or locations.

These plots are crucial for a comprehensive understanding of our data. They not only validate the effectiveness of our clustering approach but also provide valuable insights into the similarities and differences among the sensors, based on how they react to various environmental factors as reflected in their frequency-domain data.

```python
# Creating a figure with a subplot for each cluster in 'clusters_final', setting specific size and resolution.
fig, axes = plt.subplots(nrows=len(clusters_final), ncols=1, figsize=(20, 15), dpi=500)

# Iterating over each cluster in 'clusters_final'.
for idx, cluster_number in enumerate(clusters_final):
  x_corr = air_fft[cluster_metrics_dict[cluster_number]].corr().abs().values # Calculating the absolute correlation matrix for the metrics in the current cluster.
  x_corr_mean = round(x_corr[np.triu_indices(x_corr.shape[0],1)].mean(),2) # Calculating the mean of the upper triangle of the correlation matrix, rounding to two decimal places.
  plot_title = f'cluster {cluster_number} (quality={x_corr_mean}, n={cluster_len_dict[cluster_number]})' # Creating a title for the subplot with cluster number, quality (mean correlation), and number of metrics.
  air_fft[cluster_metrics_dict[cluster_number]].plot(ax=axes[idx], title=plot_title) # Plotting the metrics in the current cluster on the respective subplot.
  axes[idx].get_legend().remove() # Removing the legend from each subplot for clarity.

fig.tight_layout() # Adjusting the layout for better visibility and aesthetics.
plt.show() # Displaying the complete figure.
```

![Python output](figures/4-3-4-4.png)

The wavelet transform is another valuable method for analyzing data, complementing the Fourier transform. While the Fourier transform converts time-domain data to the frequency domain, the wavelet transform offers additional perspectives on this frequency-domain data. Its ability to provide more detailed insights makes the wavelet transform particularly useful for analyzing complex data types such as video, audio, and time series.

For performing wavelet transforms, we utilize the `pywt` package, a Python library designed for this purpose. The key to wavelet transform is the use of a "mother wavelet" – a kind of base function that helps in extracting features from time series data.

Before we proceed with the transformation, we need to identify which mother wavelet is most suitable for our data. In `pywt`, there are various mother wavelets available, each with unique characteristics that make them suited for different types of data. To find out which mother wavelets we can use, we'll start with the following syntax in Python, which lists the names of all available mother wavelets in the `pywt` package. This step is crucial as the choice of mother wavelet can significantly impact the quality and relevance of our analysis.
```python
# Using PyWavelets to get a list of all available continuous wavelets.
wavlist = pywt.wavelist(kind="continuous")

# Printing the list of continuous wavelets.
print(wavlist)
```

```
['cgau1', 'cgau2', 'cgau3', 'cgau4', 'cgau5', 'cgau6', 'cgau7', 'cgau8', 'cmor', 'fbsp', 'gaus1', 'gaus2', 'gaus3', 'gaus4', 'gaus5', 'gaus6', 'gaus7', 'gaus8', 'mexh', 'morl', 'shan']
```
The wavelet transform offers a unique advantage over the Fourier transform. While the Fourier transform focuses solely on frequency changes, the wavelet transform can adjust, or 'scale,' a finite mother wavelet to extract specific features from segments of data. This scaling ability allows for a more nuanced analysis of the data, particularly when dealing with non-stationary signals where frequency components vary over time.

An important part of using the wavelet transform is selecting the right mother wavelet for your data. Each mother wavelet has its own shape and properties, making some better suited for certain types of data than others. To make an informed choice, it's often helpful to visually inspect the wavelet.

For instance, if we consider using the 'morl' (Morlet) mother wavelet, we can first draw its shape to understand its characteristics. The Morlet wavelet, known for its good balance between time and frequency localization, is often used in signal processing. To visualize the Morlet wavelet, we would use specific syntax in Python to create a graph that represents it. This visual representation helps in deciding whether the 'morl' wavelet is the appropriate choice for our data analysis, allowing us to see how it might interact with the data's features.

```python
# Initializing the Morlet wavelet object.
wav = pywt.ContinuousWavelet("morl")

# Setting the scale for the wavelet. In this case, it's set to 1, but can be adjusted as needed.
scale = 1

# Integrating the wavelet function with a specified precision.
int_psi, x = pywt.integrate_wavelet(wav, precision=10)

# Normalizing the integrated wavelet function so its absolute maximum value is 1.
int_psi /= np.abs(int_psi).max()

# Reversing the wavelet filter to match the convention for convolution operations.
wav_filter = int_psi[::-1]

nt = len(wav_filter) # Getting the number of time samples in the wavelet filter.
t = np.linspace(-nt // 2, nt // 2, nt) # Creating a time array for plotting.
plt.plot(t, wav_filter.real) # Plotting the real part of the wavelet filter against time.
plt.ylim([-1, 1]) # Setting the y-axis limits to [-1, 1].
plt.xlabel("time (samples)") # Labeling the x-axis.
```

![Python output](figures/4-3-4-5.png)

To proceed with the wavelet transform using the 'morl' (Morlet) mother wavelet, we start by setting up the necessary parameters. This includes specifying 'morl' as our mother wavelet of choice.

For the sensor data we are focusing on, we need to determine the appropriate scale range for the Morlet wavelet. In this case, we decide to vary the scale from 1 to 31 times. This range allows us to observe how the wavelet, when scaled to different sizes, extracts features from the sensor data.

Once the parameters are set, we perform the wavelet transform on the selected sensor's data. This process involves applying the Morlet wavelet at each scale within our specified range and analyzing how the wavelet interacts with the data.

After the wavelet transformation is complete, we plot the results. These plots are crucial as they visually represent the outcome of the wavelet transform. They show us the different features the Morlet wavelet has extracted from the data at various scales. This kind of analysis is particularly useful for understanding complex time-series data, as it reveals both the frequency and time-related characteristics of the data, offering a more comprehensive view than what is possible with the Fourier transform alone.

```python
# Setting the sampling frequency to 1 sample per hour.
F = 1  # Samples per hour
hours = 744 # Specifying the total number of hours (for a month with 31 days).
nos = np.int(F*hours) # Calculating the number of samples in 31 days.

x = air_clean['08BEAC09FF2A'] # Selecting the time series data from the 'air_clean' DataFrame.
scales = np.arange(1, 31, 1) # Setting the range of scales to use in the CWT.
coef, freqs = cwt(x, scales, 'morl') # Performing the Continuous Wavelet Transform using the Morlet wavelet.

# Creating a figure for the scalogram.
plt.figure(figsize=(15, 10))

# Displaying the scalogram with specific settings for visualization.
plt.imshow(abs(coef), extent=[0, 744, 30, 1], interpolation='bilinear', cmap='viridis',
           aspect='auto', vmax=abs(coef).max(), vmin=abs(coef).min())

# Inverting the y-axis so that the scale increases upwards.
plt.gca().invert_yaxis()

# Setting the ticks for the y-axis (scales).
plt.yticks(np.arange(1, 31, 1))

# Setting the ticks for the x-axis (hours), dividing the month into 20 equal intervals.
plt.xticks(np.arange(0, nos/F, nos/(20*F)))

# Labeling the y-axis as 'scales' and the x-axis as 'hour'.
plt.ylabel("scales")
plt.xlabel("hour")

# Adding a colorbar to indicate the magnitude of the wavelet coefficients.
plt.colorbar()

# Displaying the plot.
plt.show()
```

![Python output](figures/4-3-4-6.png)

When we analyze the results of the wavelet transform, particularly with the Morlet wavelet ('morl'), we observe a distinct pattern in the visualization. The scale of the wavelet, which we varied from 1 to 31 times, is reflected in the color intensity of the plot. Generally, the larger the scale, the more the color tends toward yellow-green. This color change indicates a closer resemblance between the extracted features and the mother wavelet. In contrast, smaller scales, where the features are less similar to the mother wavelet, exhibit different color intensities.

This observation is key for understanding how different scales of the wavelet reveal different aspects of the data. The resemblance to the mother wavelet at various scales gives us insight into the frequency and time characteristics of the sensor data.

However, the wavelet transform converts each site's data into two-dimensional feature values. To prepare this data for clustering, we need to reformat it from two dimensions into one dimension. This step is crucial for the upcoming data grouping process. We will consolidate the transformed data fields into a single-dimensional format and store them in a variable named `air_cwt`.

By converting the two-dimensional wavelet transform results into a one-dimensional format, we make the data compatible with clustering algorithms like `TimeSeriesKMeans`. This process ensures that the rich, multi-scale information extracted by the wavelet transform is effectively utilized in our subsequent data grouping analysis.

```python
air_cwt = pd.DataFrame() # Initializing an empty DataFrame to store the CWT results.
scales = np.arange(28, 31, 2) # Setting the range of scales for the CWT.
col_names = list(air_clean.columns) # Extracting the column names from the 'air_clean' DataFrame.

# Looping through each column in 'air_clean'.
for name in col_names:
  # Performing the Continuous Wavelet Transform using the Morlet wavelet.
  coef, freqs = cwt(air_clean[name], scales, 'morl')

  # Reshaping the coefficients into a one-dimensional array and storing their absolute values in 'air_cwt'.
  air_cwt[name] = np.abs(coef.reshape(coef.shape[0]*coef.shape[1]))

# Printing the first five rows of the 'air_cwt' DataFrame to verify the results.
print(air_cwt.head())
```

```
08BEAC07D3E2  08BEAC09FF22  08BEAC09FF2A  08BEAC09FF42  08BEAC09FF48  \
0      0.778745      6.336664      2.137342      1.849035      0.778745   
1      0.929951      8.348031      2.977576      1.829540      0.958915   
2      1.190476     11.476048      3.223773      1.832544      1.292037   
3      1.227021     12.890554      4.488244      1.634717      1.338655   
4      1.252126     14.103178      5.715656      1.430744      1.376562   

   08BEAC09FF66  08BEAC09FF80  08BEAC09FF82  08BEAC09FF8C  08BEAC09FF9C  ...  \
0      0.555941      0.334225      8.110581      2.287378      0.409913  ...   
1      0.596532      0.201897      8.774271      1.706411      0.311242  ...   
2      0.771641      0.294242      8.327808      1.018296      2.754100  ...   
3      0.713931      0.065011      8.669036      0.249029      3.278048  ...   
4      0.670639      0.193083      8.795376      0.624988      3.628597  ...   

   74DA38F7C504  74DA38F7C514  74DA38F7C524  74DA38F7C554  74DA38F7C5BA  \
0      1.875873      1.090635      0.284901      1.111999      1.049112   
1      0.643478      1.446061      0.420701      0.974641      0.478931   
2      1.101801      2.404254      1.389941      1.190141      0.511454   
3      2.370436      2.510728      1.927692      0.703424      1.054164   
4      3.563873      2.387982      2.463458      0.098485      1.448078   

   74DA38F7C5BC  74DA38F7C5E0  74DA38F7C60C  74DA38F7C62A  74DA38F7C648  
0      1.107551      0.000918      0.080841      0.859855      0.415574  
1      0.245802      0.005389      1.053260      1.762642      0.190458  
2      1.162331      0.009754      2.667145      3.210326      0.516525  
3      1.993138      0.014496      3.669712      3.889207      0.666492  
4      2.672666      0.020433      4.482730      4.311888      0.677992  

[5 rows x 398 columns]
```

In transforming the two-dimensional wavelet transform results into a one-dimensional format, a significant increase occurs in the number of characteristic values (features) for each piece of data. This expansion adds complexity to the calculations we need to perform in the subsequent steps of our analysis. To manage this complexity and ensure efficient processing, we adopt a strategy of simplification: we select only the top 100 characteristic values to represent each sensor's data.

This selection of the top 100 features is a balancing act. It reduces computational demands while still retaining the most critical information extracted by the wavelet transform. With this simplified dataset, we then apply the KMeans method for data clustering.

Given that the wavelet transform can uncover more nuanced features in the data, we set an initial goal of forming 20 clusters in this data clustering phase. This number is a starting point, and we encourage readers to experiment with different numbers of clusters to see how it affects the results. Such experimentation can offer insights into the data's structure and the clustering algorithm's behavior.

As we proceed with the clustering, it's also essential to monitor the composition of these clusters. Specifically, we look for and eliminate any small clusters that contain only a single sensor. This step is crucial because a few sensors with unusual conditions could skew the overall results of our data grouping. By removing these outlier clusters, we aim to achieve a more accurate and representative clustering outcome, focusing on broader trends and patterns in the data rather than exceptions.

```python
air_cwt_less = air_cwt.iloc[:, 0:101] # Selecting the first 101 columns from the 'air_cwt' DataFrame for clustering.
cwt_transpose = air_cwt_less.transpose() # Transposing the reduced 'air_cwt' DataFrame for time series clustering.

# Initializing the Time Series K-Means model with 20 clusters, using the DTW (Dynamic Time Warping) metric, 
# setting a maximum of 10 iterations, enabling verbose output, and using all available CPU cores.
model = TimeSeriesKMeans(n_clusters=20, metric="dtw", max_iter=10, verbose=1, n_jobs=-1)

pre = model.fit(cwt_transpose) # Fitting the model to the transposed CWT data.
df_cluster = pd.DataFrame(list(zip(air_cwt.columns, pre.labels_)), columns=['metric', 'cluster']) # Creating a DataFrame 'df_cluster' to map each metric in 'air_cwt' to its corresponding cluster label.

cluster_metrics_dict = df_cluster.groupby(['cluster'])['metric'].apply(lambda x: [x for x in x]).to_dict() # Creating a dictionary 'cluster_metrics_dict' mapping each cluster to the metrics it contains.
cluster_len_dict = df_cluster['cluster'].value_counts().to_dict() # Creating a dictionary 'cluster_len_dict' to count the number of metrics in each cluster.
clusters_dropped = [cluster for cluster in cluster_len_dict if cluster_len_dict[cluster]==1] # Creating a list 'clusters_dropped' containing clusters with only one metric.
clusters_final = [cluster for cluster in cluster_len_dict if cluster_len_dict[cluster]>1] # Creating a sorted list 'clusters_final' containing clusters with more than one metric.
clusters_final.sort()

# Printing the first 10 rows of the 'df_cluster' DataFrame.
print(df_cluster.head(10))
```

```
metric  cluster
0  08BEAC07D3E2        7
1  08BEAC09FF22        3
2  08BEAC09FF2A        6
3  08BEAC09FF42       10
4  08BEAC09FF48        7
5  08BEAC09FF66        7
6  08BEAC09FF80        7
7  08BEAC09FF82        6
8  08BEAC09FF8C       15
9  08BEAC09FF9C       19
```

In our example, following the process of selecting the top 100 characteristic values and applying the KMeans method to the wavelet-transformed data, we end up with 14 distinct clusters. This is after we've removed any clusters that consisted of only a single sensor, as these are considered outliers and not representative of general trends.

The next step is to visually represent the raw sensor data for each of these 14 clusters. This visualization is crucial as it allows us to observe the consistency within each cluster and the distinctions between different clusters. When we plot the raw data of the sensors in each cluster, we notice a clear pattern: the data within the same cluster show a high degree of similarity.

Moreover, the differences between clusters are more pronounced and detailed compared to the results we obtained using either the raw data or the Fourier transform. This indicates that the wavelet transform, combined with our clustering approach, provides a more nuanced understanding of the data. It captures subtler features and patterns that might be missed with other methods.

These findings underscore the effectiveness of the wavelet transform in extracting intricate features from time-series data. The resulting clusters reveal a more refined and detailed grouping of the sensors, offering deeper insights into the underlying dynamics and characteristics of the environmental data we are analyzing.

```python
# Creating a figure with a subplot for each cluster in 'clusters_final', setting specific size and resolution.
fig, axes = plt.subplots(nrows=len(clusters_final), ncols=1, figsize=(20, 15), dpi=500)

# Iterating over each cluster in 'clusters_final'.
for idx, cluster_number in enumerate(clusters_final):
  x_corr = air_cwt[cluster_metrics_dict[cluster_number]].corr().abs().values # Calculating the absolute correlation matrix for the metrics in the current cluster.
  x_corr_mean = round(x_corr[np.triu_indices(x_corr.shape[0],1)].mean(),2) # Calculating the mean of the upper triangle of the correlation matrix, rounding to two decimal places.
  plot_title = f'cluster {cluster_number} (quality={x_corr_mean}, n={cluster_len_dict[cluster_number]})' # Creating a title for the subplot with cluster number, quality (mean correlation), and number of metrics.
  air_cwt[cluster_metrics_dict[cluster_number]].plot(ax=axes[idx], title=plot_title) # Plotting the metrics in the current cluster on the respective subplot.
  axes[idx].get_legend().remove() # Removing the legend from each subplot for clarity.

# Adjusting the layout for better visibility and aesthetics.
fig.tight_layout()

# Displaying the complete figure.
plt.show()
```

![Python output](figures/4-3-4-7.png)

## References

- Civil IoT Taiwan: Historical Data ([https://history.colife.org.tw/](https://history.colife.org.tw/#/))
- Time Series Clustering — Deriving Trends and Archetypes from Sequential Data | by Denyse | Towards Data Science ([https://towardsdatascience.com/how-to-apply-k-means-clustering-to-time-series-data-28d04a8f7da3](https://towardsdatascience.com/how-to-apply-k-means-clustering-to-time-series-data-28d04a8f7da3))
- How to Apply K-means Clustering to Time Series Data | by Alexandra Amidon | Towards Data Science ([https://towardsdatascience.com/how-to-apply-k-means-clustering-to-time-series-data-28d04a8f7da3](https://towardsdatascience.com/how-to-apply-k-means-clustering-to-time-series-data-28d04a8f7da3))
- Understanding K-means Clustering: Hands-On with SciKit-Learn | by Carla Martins | May, 2022 | Towards AI ([https://pub.towardsai.net/understanding-k-means-clustering-hands-on-with-scikit-learn-b522c0698c81](https://pub.towardsai.net/understanding-k-means-clustering-hands-on-with-scikit-learn-b522c0698c81))
- Fast Fourier Transform. How to implement the Fast Fourier… | by Cory Maklin | Towards Data Science ([https://towardsdatascience.com/fast-fourier-transform-937926e591cb](https://towardsdatascience.com/fast-fourier-transform-937926e591cb))
- PyWavelets/pywt: PyWavelets - Wavelet Transforms in Python ([https://github.com/PyWavelets/pywt](https://github.com/PyWavelets/pywt))
