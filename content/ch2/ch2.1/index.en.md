---
title: "2.1. Material Architecture"
weight : 10
description : "Introduction of the material architecture"
socialshare: true
tags: ["Introduction" ]
levels: ["beginner" ]
authors: ["Huei-Jun Kao", "Ling-Jyh Chen"]
---

This comprehensive set of materials on Civil IoT Taiwan data application is organized into seven distinct themes, each focusing on different aspects of data utilization and analysis. Here's an overview of each theme:

1. Introduction: This section lays the foundation by introducing the Civil IoT Taiwan Project. It highlights the project's achievements over the years and its significant impact on water resources, air quality, earthquake, and disaster prevention and relief. Through various materials, videos, and success stories, learners gain a profound understanding of the project's relevance to everyday life.

2. Overview of the Materials: Here, we outline the structure of the entire course, focusing on Python and the Google Colab platform. The section guides learners through the material's layout and points them to additional resources for further exploration of related technologies.

3. Data Access Method: This critical part teaches how to access Civil IoT Taiwan Data using Python, specifically through a custom-developed tool, pyCIOT. It's divided into two segments:
    - Basic Data Access: Covers how to retrieve the latest sensor data from various fields and lists of all monitoring stations.
    - Specific Spatio-temporal Data Access: Focuses on accessing data from a particular station at specific times or intervals and locating the latest data from stations near given coordinates.

Alongside these methods, the section incorporates basic exploratory data analysis (EDA) and uses common statistical methods to describe data characteristics, concluding with simple data visualization techniques.

4. Data Analysis in Time Dimension: This section delves into time-series data analysis using Civil IoT Taiwan data, divided into three parts:
    - Time Series Data Processing: Involves understanding moving averages, performing periodic analyses, disassembling time series data, and applying change point and outlier detection techniques.
    - Time Series Data Forecast: Uses sensory data to compare different prediction methods and discusses the implications and potential applications of these forecasts.
    - Time Series Data Clustering: Introduces advanced clustering analysis, including Fourier and wavelet transforms, and discusses the use of Euclidean and Dynamic Time Warping (DTW) distance for clustering.

5. Data Analysis in Space Dimension: This theme focuses on spatial data analysis, covering:
    - Geospatial Filtering: Demonstrates overlaying Civil IoT Taiwan data with administrative region maps and performing geospatial data distribution analyses.
    - Geospatial Analysis: Teaches advanced techniques like constructing Convex Hulls, applying the Voronoi Diagram algorithm, and performing spatial interpolation.

6. Data Applications: This practical segment explores the extended applications of Civil IoT Taiwan data:
    - Introduction to Machine Learning: Combines Civil IoT data with other datasets for predictive analytics, addressing machine learning processes, effectiveness evaluation, and avoiding biases.
    - Anomaly Detection: Implements anomaly detection algorithms in air quality data, covering the entire process from data preparation to feature extraction and analysis.
    - Dynamic Calibration Model: Demonstrates calibration models for air quality sensors, combining basic data analysis with machine learning.

7. System Applications: This final theme integrates Civil IoT Taiwan data with various application software:
   - QGIS Applications: Uses QGIS for organizing, analyzing, and presenting geospatial data from Civil IoT Taiwan.
   - Tableau Applications: Demonstrates using Tableau for data visualization and integration with Civil IoT Taiwan data.
   - Leafmap Applications: Shows how to use Leafmap, combined with Google Earth Engine, for advanced GIS applications and introduces the integration with Streamlit for web-based GIS services.

This course material is designed to provide a thorough understanding and practical application of Civil IoT Taiwan data, leveraging Python and various data science tools to address public welfare challenges.


## References

- Civil IoT Taiwan. [https://ci.taiwan.gov.tw](https://ci.taiwan.gov.tw/)
- Civil IoT Taiwan Data Platform. [https://ci.taiwan.gov.tw/dsp/](https://ci.taiwan.gov.tw/dsp/)
- QGIS - A Free and Open Source Geographic Information System. [https://qgis.org/](https://qgis.org/)
- Tableau - Business Intelligence and Analytics Software. [https://www.tableau.com/](https://www.tableau.com/)
- Leafmap - A Python package for geospatial analysis and interactive mapping in a Jupyter environment. [https://leafmap.org/](https://leafmap.org/)
- Streamlit - The fastest way to build and share data apps. [https://streamlit.io/](https://streamlit.io/)
- Google Colaboratory. [https://colab.research.google.com/](https://colab.research.google.com/)
