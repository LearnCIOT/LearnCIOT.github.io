---
title : "7.1. QGIS Application"
weight : 10
description : "We introduce the presentation of geographic data using the QGIS system, and use the data from Civil IoT Taiwan as an example to perform geospatial analysis by clicking and dragging. We also discuss the advantages and disadvantages of QGIS software and when to use it."
tags: ["Air", "Disaster" ]
levels: ["beginner" ]
authors: ["Yu-Shen Cheng", "Ming-Kuang Chung"]
---


{{< toc >}}

QGIS, which stands for Quantum Geographic Information System, is an open-source desktop application that allows users to work with geographic data. Not only can you visualize the data collected by users in various geographic formats, but you can also process, analyze, and integrate geospatial information. One of its powerful features is the ability to create thematic maps.

In this article, we’ll explore how QGIS assists in analyzing and presenting PM2.5 data obtained from Civil IoT Taiwan. After completing the analysis, we’ll generate a thematic map to interpret the results. Additionally, we’ll demonstrate how to combine disaster prevention data from Civil IoT Taiwan to create a distribution map of disaster shelters. This functionality enables citizens to easily locate the nearest shelters in times of need.

{{% notice style="note" %}}
Please note: The version of QGIS used in this article is 3.16.8. However, the functions discussed here are fundamental features of the software. If you’re using a different version, you should still be able to use these functions without any issues.
{{% /notice %}}

## Goal

- Import data into QGIS
- Basic geographic and spatial data analysis in QGIS
- Export thematic maps from QGIS

## QGIS Basic Operation

After entering the QGIS software, you can see the following operation interface. In addition to the data frame in the middle area, the upper part is the standard toolbar, which provides various basic operation tools and functions. There are two sub-areas on the left, which are the data catalog window and layers; on the right is the analysis toolbar, which provides various analysis tools.

![QGIS](figures/7-1-1-1.png)

### Data Preparation

Data source: Civil IoT Taiwan - Historical Data ([https://history.colife.org.tw/#/?cd=%2F空氣品質%2F中研院_校園空品微型感測器](https://history.colife.org.tw/#/?cd=%2F空氣品質%2F中研院_校園空品微型感測器))

In this section, we’ll begin by explaining how to import data into QGIS. When dealing with data that might be split across multiple tables during storage, it’s crucial to pay attention to whether this applies to the dataset you’re analyzing. If so, you’ll need to recombine the data into a single, cohesive table.

Let’s break down the process:

- Import data
  
    - Start by importing the CSV file directly downloaded from Civil IoT Taiwan Historical Data into QGIS.
    - Keep in mind that the data contains Chinese characters, which can sometimes appear garbled during the import process.
    - To import the data, follow these steps:
            - Go to the top menu and select Layer > Add Layer > Add Delimited Text Layer.
            - The import interface will appear, allowing you to configure the import settings.
![](figures/7-1-2-1.png)
    
- Join data

  Since the original data divides PM2.5 and station coordinates (latitude and longitude) into two files, it is necessary to join PM2.5 and latitude and longitude coordinates first.
  ![](figures/7-1-2-2.png)
  The join method is as follows, right-click the file to be joined > Properties > joins > click the + sign to enter Add Vector Join, as shown below
  ![](figures/7-1-2-3.png)
  There are four options after entering:
  - Join Layer: the layer you want to join
  - Join field: corresponding to the Target field, which is the reference for Join (similar to Primary key)
  - Target field: corresponding to the Join field, which is the reference for Join (similar to Foreign key)
  - Joined field: the fields you want to join
  ![](figures/7-1-2-4.png)
    

### GeoJSON Output


Next, we’ll utilize the built-in features of QGIS to transform the original CSV data into [GeoJSON](https://en.wikipedia.org/wiki/GeoJSON) files, which represent geographical data in JSON format. The process involves navigating to Processing > Toolbox > Create points from the table.

Remember, once you’ve clicked on that option, choose the table you want to import. Specify longitude for the X-coordinate and latitude for the Y-coordinate. Set the Target CRS to WGS 84 (which corresponds to latitude and longitude coordinates). Finally, provide a name for the output file, as demonstrated below."

![](figures/7-1-2-5.png)

Then select the file format you want to export, and click "Save".

![](figures/7-1-2-6.png)

### Data Filter and Change Colors

Next, we’ll showcase how to utilize QGIS for filtering the necessary stations and dynamically adjusting their colors based on the PM2.5 values.


- Use Intersection to filter the required stations by county

  Before proceeding with the screening process, you’ll need to download the shapefiles (shp files) for municipalities, counties, and cities from the [government data open platform](https://data.gov.tw/dataset/7442) open platform. Once you’ve obtained these files, import the shapefiles for county and city boundaries into QGIS, following the steps below:
  {{% notice style="note" %}}
  Note: As part of the Civil IoT Taiwan project, National Chi Nan University has deployed micro air quality sensors in Changhua and Nantou counties and cities. Consequently, the data collected this time does not include information from Changhua and Nantou.
  {{% /notice %}}
  ![](figures/7-1-2-7.png)
    1. Click on the ‘Select Feature’ icon.
    2. Then, select the desired county by clicking on its name. The selected counties will be highlighted in yellow. Let’s take New Taipei City as an example, as shown in the figure below:

  ![](figures/7-1-2-8.png)
    3. In the ‘Processing Toolbox’, search for the ‘Intersection’ function. Click on it to open the interface, which provides three input options: Input Layer, Overlay Layer, and Intersection (Output)
    4. Place the PM2.5 layer in the ‘Input Layer’, and add the county-city boundary layer to the ‘Overlay Layer’. Make sure to check the ‘Select features only’ option. This ensures that only the stations intersecting with the previously selected New Taipei City are filtered out.
    5. Finally, enter a name for the exported file in the ‘Intersection’ section. The supported export file formats are the same as the ones you’ve previously selected.

![](figures/7-1-2-9.png)
  Upon completing these steps, you’ll obtain the desired results."

![](figures/7-1-2-10.png)
    
- Display different colors according to the value of PM2.5
    
  Then we right click on the PM2.5 layer and choose Properties. In the Symbology section, you’ll find the dot color settings. Follow these steps for each PM2.5 station:
    - Change the original setting from “No Symbols” to “Graduated.”
    - Select PM25 in the “Value” part.
    - Click the “Classify” button at the bottom.
    - Set the number of colors in “Classes” (Remember, it’s best not to have too many categories).
    - Go to “Color ramp” and choose the color for each value.
    - Click “OK” when you’re done.

![](figures/7-1-2-11.png)
  Once you’ve completed these steps, the resulting image will display dots with colors corresponding to the PM 2.5 value. The layer on the right will show how different colors represent varying PM 2.5 levels.

  {{% notice style="note" %}}
  Note: If you’re using the new version of QGIS (after 3.16), you can easily add the OpenStreetMap basemap by clicking XYZ Tiles and selecting OpenStreetMap in the figure below.
  {{% /notice %}}
    
![](figures/7-1-2-12.png)
    

### Export Thematic Maps

- After configuring the previous settings, the next step involves exporting your QGIS project as a JPG image. 
- Click on Project and then select New Print Layout. A dialog for setting the layout name will appear.
- Once you’ve completed the layout name setting, the following screen will be displayed.

![](figures/7-1-2-13.png)

- On the left side, click “Add map” and choose the area on the drawing canvas where you want to include the PM 2.5 map.

![](figures/7-1-2-14.png)

![](figures/7-1-2-15.png)

- Next, still on the left side, click “Add Legend” and select a range to import the label. You can adjust font size, color, and other properties in the “Item Properties” on the right.
- To finish, add the title, scale bar, and compass to create a comprehensive thematic map.

![](figures/7-1-2-16.png)

- Lastly, in the upper left corner, click “Layout” and choose “Export as Image” to save your thematic map as an image file.

## Example 2: Distribution of Emergency Shelters

Data source: [https://data.gov.tw/dataset/73242](https://data.gov.tw/dataset/73242)

Government public information has neatly organized Taiwan’s emergency shelters into electronic files, making it convenient for citizens to download and utilize them. In this example, we’ll demonstrate how to locate the nearest shelter to your home using QGIS.

We start by obtaining the shelter information from the provided URL and load this information into QGIS following the method mentioned earlier.

![](figures/7-1-3-1.png)

Given the substantial number of shelters across Taiwan, this article will focus on analyzing shelters specifically in Taipei City. However, the same approach can be applied to other counties and cities. If you’re curious, feel free to explore this process for yourself.

We then begin by using the intersection method to identify shelters within Taipei City.

![](figures/7-1-3-2.png)

Next, we utilize Voronoi Polygons from the side toolbar to create a Voronoi diagram, and we ill in the layer of shelters within the Voronoi Polygons and click “Run” as depicted below.

![](figures/7-1-3-3.png)

The Voronoi Diagram’s characteristics will reveal the location of the closest shelter to your residence, as shown in the image.

![](figures/7-1-3-4.png)

Once the analysis is complete, you can generate a thematic map using the same method as before.

## Conclusion

In this chapter, we’ve covered the process of importing data into QGIS and utilizing its analysis tools to explore that data. Additionally, we’ve introduced the data exporting method, which allows you to transform your analyzed data into thematic maps for interpretation. Keep in mind that there are many other features in QGIS that we haven’t covered here. If you’re keen on diving deeper into QGIS, we recommend exploring additional resources below.


## References

- QGIS: A Free and Open Source Geographic Information System ([https://qgis.org/](https://qgis.org/))
- QGIS Tutorials and Tips ([https://www.qgistutorials.com/en/](https://www.qgistutorials.com/en/))
- QGIS Documentation ([https://www.qgis.org/en/docs/index.html](https://www.qgis.org/en/docs/index.html))
