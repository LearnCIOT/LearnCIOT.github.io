---
title : "7.2. Tableau Application"
weight : 20
socialshare: true
description : "We introduce the use of Tableau tools to render Civil IoT Taiwan data and conduct two example cases using air quality data and disaster notification data. We demonstrate how worksheets, dashboards, and stories can be used to create interactive data visualizations for users to explore data. We also provide a wealth of reference materials for people to further study reference."
tags: ["Air", "Disaster" ]
levels: ["beginner" ]
authors: ["Yu-Shen Cheng", "Ming-Kuang Chung"]
---

{{< toc >}}

Data visualization transforms data into graphical formats, making it easier to understand and interpret. Different graph types are better suited for various data sets. For instance, map-type charts are often used for data with latitude and longitude coordinates, while line charts or histograms are ideal for time series data. However, many real-world data sets have multiple characteristics. Take the sensor data from Civil IoT Taiwan, for example; it includes both geographical coordinates and time-series elements. Hence, to effectively convey the insights of such data, we might need to use multiple types of charts. In this chapter, we introduce Tableau, a widely-used software, to help us effectively present these complex types of data.

Tableau is a user-friendly platform for visual analysis. It enables quick creation of diverse charts, and its dashboard feature simplifies the presentation and integration of different data visualizations. Dashboards in Tableau also offer interactive elements with graphs, enhancing both the ease and speed of understanding data, as well as simplifying data analysis. Originally a paid commercial software, Tableau also offers [Tableau Public](https://public.tableau.com/), a free version accessible to everyone. It's important to note, however, that any work created using the free version of Tableau must be publicly available.

{{% notice style="note" %}}
Please note: The version of Tableau Public used in this article is Tableau Desktop Public Edition (2022.2.2 (20222.22.0916.1526) 64-bit). However, we only utilize the basic functions of the software, which are generally consistent across different versions. Therefore, if you are using another version of the software, you should still be able to follow along and execute the processes described here without issues.
{{% /notice %}}

## Goal

- Load data into Tableau for visual representation.
- Use Worksheets in Tableau to design various data charts.
- Develop interactive visuals and reports by employing dashboards and stories within Tableau Public.

## Data Source

- Civil IoT Taiwan Historical Data: Academia Sinica - Micro Air Quality Sensors ([https://history.colife.org.tw/#/?cd=%2F空氣品質%2F中研院_校園空品微型感測器](https://history.colife.org.tw/#/?cd=%2F空氣品質%2F中研院_校園空品微型感測器))
- Civil IoT Taiwan Historical Data: National Fire Agency - Disaster Notifications ([https://history.colife.org.tw/#/?cd=%2F災害示警與災情通報%2F消防署_災情通報](https://history.colife.org.tw/#/?cd=%2F%E7%81%BD%E5%AE%B3%E7%A4%BA%E8%AD%A6%E8%88%87%E7%81%BD%E6%83%85%E9%80%9A%E5%A0%B1%2F%E6%B6%88%E9%98%B2%E7%BD%B2_%E7%81%BD%E6%83%85%E9%80%9A%E5%A0%B1))

## Tableau Basic Operation

### Data Import

Tableau is capable of reading text files (like csv) and spatial information files (such as shp and geojson). To import data, you need to go to Data > Add Data Source and then select the data file you wish to import.

![Tableau screenshot](figures/7-2-3-1.png)

Depending on the type of file you're working with, we have different examples to guide you:

- For Geospatial Data Formats (shp, geojson):
    1. First, click on the data source in the lower left corner to view the currently imported data fields.
        
      ![Tableau screenshot](figures/7-2-3-2.png)
 
    2. Next, click the icon above the field name to modify the properties of that field.
        
       ![Tableau screenshot](figures/7-2-3-3.png)
 
    3. Lastly, assign latitude and longitude coordinates to geographic roles to prepare for further operations.
        
       ![Tableau screenshot](figures/7-2-3-4.png)

- For Text Data Formats (CSV):
    1. After importing the PM 2.5 records and station locations into Tableau, establish a relationship between these data tables.
 
       ![Tableau screenshot](figures/7-2-3-5.png)
 
    2. To link the PM 2.5 station data with station coordinates, first double-click on the station data to enter the connection canvas. Then, drag another data table onto this canvas.
        
       ![Tableau screenshot](figures/7-2-3-6.png)
 
    3. Now, click the link icon in the middle to set the type of link and define how they interact.
        
       ![Tableau screenshot](figures/7-2-3-7.png)
 
    4. Once set up, you'll notice that the Station id in the lower right corner is sourced from the Location data table, linking the PM 2.5 data with the station location data.

### Worksheet Introduction

A worksheet in Tableau is the space where you can create visual representations of your data. It's not limited to traditional graph types like pie, bar, and line graphs; it can also be used to plot geographic information. We'll guide you through how to create various visualizations.

Once you've prepared your data, click on 'New Worksheet' in the lower left corner. Upon entering the worksheet, you'll be presented with the interface we're about to discuss.

![Tableau screenshot](figures/7-2-3-8.png)

In the upper right corner of the worksheet in Tableau, there's a "Show Me" button. When you click on it, a variety of chart types will be displayed. The system automatically identifies which types of charts can be created based on your data. Those that aren't suitable for your data will be grayed out or highlighted. You can then click on the chart type you want to use to change the visualization accordingly.

![Tableau screenshot](figures/7-2-3-9.png)

## Tableau Example 1: Spatio-temporal Distribution of Air Quality Data

### Spatial Distribution Graph

Before you begin creating your visualization, it's important to change the latitude and longitude fields from being a 'measure' to a 'dimension'. Understanding the difference between a measure and a dimension is crucial in Tableau. For a more detailed explanation of these concepts, please refer to the [reference materials](https://help.tableau.com/current/pro/desktop/en-us/datafields_typesandroles.htm). The specific steps to change these fields in Tableau are demonstrated in the following animation:

![Tableau screenshot](figures/7-2-4-1.gif)


Next, you'll need to drag the latitude and longitude fields to the respective column and row areas in Tableau. After that, drag the PM 2.5 value to the indicated area for it to be included in the visualization. Then, change the mark type to 'Color' to create a distribution map showing PM 2.5 levels across Taiwan. This process is demonstrated in the animation below.

![Tableau screenshot](figures/7-2-4-2.gif)

You have the option to click on the color section to alter the color of the dots on your map. Additionally, the filter tool can be used to narrow down the PM 2.5 values based on specific conditions. For instance, by selecting PM 2.5 > Metric, you can choose to display the mean, sum, or median values of PM 2.5. The following steps will guide you on how to specifically display the PM 2.5 values for New Taipei City.

1. Drag “Site Name” to filter
2. Click on wildcards, click “Include” and enter New Taipei City
3. Click “OK”

![Tableau screenshot](figures/7-2-4-3.png)

### Time Series Graph

To create a time-based visualization, start by dragging "Timestamp" to the column area and both "Site Name" and the PM 2.5 value to the row area. Next, click on "Timestamp" to adjust the time interval. For instance, in the example below, the interval is set to 1 hour. Similar to plotting spatial distribution, you can also use the filter function to select specific stations for display. In the animation provided below, we illustrate how to showcase the daily monitoring values of the PM2.5 station in Kaohsiung.

![Tableau screenshot](figures/7-2-4-4.gif)

## Tableau Example 2: Disaster Notification Dashboard

Dashboards in Tableau offer the ability to amalgamate various worksheets (charts) into a single view, presenting a more comprehensive set of information and enabling users to interact with the data. In the upcoming example, we'll guide you through creating a simple dashboard utilizing rainstorm disaster data from the Civil IoT Taiwan project. This will demonstrate how you can effectively use dashboards to display and interact with complex data sets.

### Disaster data format conversion

For our example, we'll focus on the flood event that happened on August 23, 2018, known as the "823 flood." The original data for this event is in XML format. To use this data in Tableau, we first need to convert it into a CSV file. You can do this conversion using an online tool available at [https://www.convertcsv.com/xml-to-csv.htm](https://www.convertcsv.com/xml-to-csv.htm). The process of converting the XML file to a CSV format using this website is demonstrated in the animation below.

![Tableau screenshot](figures/7-2-5-1.gif)

### Dashboard size

Once you click on 'Dashboard' in the Tableau menu, you'll have the option to set the size of your dashboard. This can be done using the tool list located on the left-hand side. This feature allows you to customize the layout and size of your dashboard to best fit the data visualizations you plan to include.

![Tableau screenshot](figures/7-2-5-2.png)

### Add worksheets to a dashboard

Next, you'll see the worksheet(s) you created earlier listed in the tool list on the left side of the dashboard interface in Tableau. To add any of these worksheets to your dashboard, simply drag and drop them into the empty space of the dashboard. This intuitive feature allows you to easily organize and display the various data visualizations you've created in a cohesive and interactive dashboard layout.

![Tableau screenshot](figures/7-2-5-3.png)

Once you've added a worksheet to your Tableau dashboard, the software automatically processes the data from that worksheet. It reads the information contained within and automatically generates an initial chart based on that data. This feature simplifies the initial steps of data visualization, providing you with a starting point that you can then customize and refine according to your needs and preferences.

![Tableau screenshot](figures/7-2-5-4.png)

The worksheet you've just dragged into your Tableau dashboard comes with a preset size. If you want to adjust this size, look for a downward arrow at the top right corner of the worksheet within the dashboard. By clicking on this arrow and selecting the "Float" option, you gain the flexibility to resize the graph as needed. This feature allows you to customize the layout of your dashboard more dynamically, ensuring that each visual element fits and complements the overall design.

![Tableau screenshot](figures/7-2-5-5.png)

### Add interactions

Next, we’ll show you how to create an interactive interface that lets users choose a specific point in time from the disaster data for observation.

Begin by clicking the downward arrow located in the upper right corner of the sheet. From the dropdown menu, select ‘Filters,’ and then choose ‘Case Time.’

![Tableau screenshot](figures/7-2-5-6.png)

Once you click, a selection field will appear in the worksheet. From there, you can choose the date you want to observe. After selecting the date, you’ll be able to view the disaster situation for that specific day.

![Tableau screenshot](figures/7-2-5-7.png)

To modify the date selection method, you can also click the downward arrow at the top right corner of the selection field. This will allow you to adjust the appearance of the selection field.

![Tableau screenshot](figures/7-2-5-8.png)

As an illustration, we can switch the selection method from the initial list format to the slider format in the following manner.

![Tableau screenshot](figures/7-2-5-9.png)

### Interacting multiple worksheet

The interaction buttons mentioned above are applicable to a single worksheet only. If you wish to have multiple worksheets on the dashboard share the same interactive button simultaneously, consider the following steps:

1. Start by creating an interactive button field using the method described above.
2. Next, click the downward arrow at the top right corner of the interactive button field. From the dropdown menu, choose ‘Apply to worksheet’ and select the desired worksheet.
   
   ![Tableau screenshot](figures/7-2-5-10.png)
 
3. Finally, apply the settings to the selected worksheet.
    
   ![Tableau screenshot](figures/7-2-5-11.png)
 

### Additional information

If you’d like to include additional information on the dashboard, look for the object column located at the bottom left of the toolbar. This column houses various elements such as text, images, and other objects. Simply drag and drop the necessary items to the top of the dashboard to display them there. You can then add text, pictures, and any other relevant objects.

![Tableau screenshot](figures/7-2-5-12.png)

## Story

A story in this context is a collection of multiple dashboards or sheets that come together to create a slideshow-like presentation. Adding a story to a dashboard or sheet is quite similar to adding a sheet from a dashboard. You simply need to drag the sheet or dashboard you’ve created on the right and place it into the empty space designated for the story.

![Tableau screenshot](figures/7-2-5-13.png)

If you want to add a new page to your story, just click on ‘New Story’ at the top. This will create a fresh blank page. Alternatively, you can duplicate an existing page to create a new one.

![Tableau screenshot](figures/7-2-5-14.png)

Lastly, if you want the text information to better align with its content, you can modify the title of the page by double-clicking the box above the text area.

![Tableau screenshot](figures/7-2-5-15.png)

## Conclusion

In this chapter, we’ll provide a brief overview of the fundamental operations in Tableau. We’ll explore how to create interactive presentations and charts that engage users. However, it’s essential to note that Tableau offers a wealth of functionalities beyond what we cover here. Many classic examples showcasing Tableau’s capabilities can be found online. If you’re interested in diving deeper, consider exploring the additional resources listed below.

## References

- Tableau Public ([https://public.tableau.com/](https://public.tableau.com/))
- Tableau - About data field roles and types ([https://help.tableau.com/current/pro/desktop/en-us/datafields_typesandroles.htm](https://help.tableau.com/current/pro/desktop/en-us/datafields_typesandroles.htm))
- Get Started with Tableau ([https://help.tableau.com/current/guides/get-started-tutorial/en-us/get-started-tutorial-home.htm](https://help.tableau.com/current/guides/get-started-tutorial/en-us/get-started-tutorial-home.htm))
- Tableau Tutorial — Learn Data Visualization Using Tableau ([https://medium.com/edureka/tableau-tutorial-71ef4c122e55](https://medium.com/edureka/tableau-tutorial-71ef4c122e55))
- YouTube: Tableau For Data Science 2022 ([https://www.youtube.com/watch?v=Wh4sCCZjOwo](https://www.youtube.com/watch?v=Wh4sCCZjOwo))
- YouTube: Tableau Online Training ([https://www.youtube.com/watch?v=ttCDqyfrcEc](https://www.youtube.com/watch?v=ttCDqyfrcEc))

