---
title: "1. Introduction"
weight: 1
chapter: true
socialshare: true
description : "Introduction"
tags: ["Introduction" ]
levels: ["beginner" ]
authors: ["Huei-Jun Kao"]
---


# 1. Introduction

{{< toc >}}

## Environment Perception


Since ancient times, human curiosity about the natural world began during the agricultural civilization era, when people would look up to the sky and observe the stars. During the Renaissance in the 16th century, Copernicus proposed the heliocentric theory based on the astronomical data available at the time. Galileo and Newton followed his line of thought, laying the foundation for modern science. In today's society, with the rapid advancement of semiconductor technology, the tools we use to perceive our living environment are becoming increasingly diverse, precise, and miniature. When these miniature sensors are combined with the evolution of information technology and the real-time data transmission of the internet, we obtain an unprecedented amount of observational data.

Faced with this vast ocean of data, scientists are striving to analyze patterns of environmental change and study the connections between these patterns and disasters. Their goal is to better predict disasters, improve our quality of life, enhance disaster prevention and mitigation, and promote harmonious coexistence between humans and the environment. This is not only an issue of concern for individuals, communities, nations, and the globe today, but it is also the core objective in the pursuit of a beautiful and sustainable living environment.

## Open Data

"Open data" is a strategy that involves organizing various types of digital data — whether text, numbers, images, videos, or sound — in specific processes and formats, then making them publicly available and encouraging their free use by all. According to the definition provided by the Open Knowledge Foundation ([https://opendefinition.org/od/2.1/en/](https://opendefinition.org/od/2.1/en/)), to be classified as open data, it must meet several criteria.

- Open license or status: The open data content must be in the public domain and released with an open license without any additional restrictions.
- Access: Open data content must be freely accessible from the Internet. However, access to open data may also allow a one-time fee and conditional incidental terms under reasonable conditions.
- Machine readability: Open data content must be easily accessible, processed, and modified by computers.
- Open format: Open data content must be in an available form, accessed using free or open software.

As the concepts of open source, open government, and open science become more widespread, open data has gradually become an important principle in policy-making and academic research for governments and the scientific community. Especially notable in recent times is the environmental sensing Internet of Things (IoT), which, due to its wide distribution in public spaces and the close relevance of its monitored environmental information to the daily lives of the public, is considered one of the most anticipated sources of open data.

Common open data formats found on the internet today include JSON, CSV, and XML:

- JSON (Javascript Object Notation): This is a lightweight data interchange format. It consists of attributes and values, and is not only machine-friendly but also easy for humans to read and interpret. This format is often chosen for data display, transmission, and storage on websites.
- CSV (Comma-Separated Values): As the name suggests, this is a method for storing tabular data in text format. In this format, each piece of data is a row, and each attribute is a column. When data is stored, all attributes are arranged in a specific order and separated by a specific symbol. This format is widely used for data import/export in software, web data transmission, and historical data storage.
- XML (Extensible Markup Language): Based on Standard Generalized Markup Language (SGML), XML is an extensible markup language. It allows users to customize their required tags and is suitable for creating documents containing structured data. XML is widely used in document presentation and online data exchange.

Currently, open data commonly used in Taiwan are aggregated on the following platforms:

- Taiwan government open data platform ([https://data.gov.tw/en/](https://data.gov.tw/en/))
- Taiwan open weather data ([https://opendata.cwb.gov.tw/devManual/insrtuction](https://opendata.cwb.gov.tw/devManual/insrtuction))
- Taiwan EPA open data platform ([https://data.epa.gov.tw/en](https://data.epa.gov.tw/en))

## Civil IoT Taiwan


The "Civil IoT Taiwan" initiative is a major collaborative effort aimed at addressing four critical public needs, focusing on areas like air quality, earthquakes, water resources, and disaster prevention. This initiative is a part of the "Digital Technology" category within the larger "Forward-looking Infrastructure Development Program." Key participants in this project include the Ministry of Science and Technology, the Ministry of Communications, the Ministry of Economic Affairs, the Ministry of the Interior, the Environmental Protection Agency, Academia Sinica, and the Council of Agriculture.

The core idea of this project is to harness the power of big data, artificial intelligence, and Internet of Things technologies to create advanced systems that enhance everyday life. These systems are designed to help both the government and the public tackle the challenges posed by environmental changes.

One of the key aspects of the Civil IoT Taiwan project is its inclusive approach, considering the needs and experiences of various users, including government agencies, academic institutions, businesses, and the general public. The overarching goals are to improve government efficiency, support the growth of academia and industry, and ultimately increase public well-being.

The Civil IoT Taiwan project encompasses four main areas:

- Water Resources: In partnership with central and local government water conservation agencies, the Civil IoT Taiwan project has set out to develop and deploy a range of water resource sensors. These include sensors for hydrological observation and for monitoring farmland irrigation. The goals are comprehensive:
  - To integrate information on surface and groundwater as well as new water sources.
  - To create an IoT portal and a dynamic analysis and management platform for managing irrigation water, thereby enhancing flood control systems for river bureaus.
  - To establish a cloud-based sewer management platform enabling remote, automated, and intelligent control for various levels of agencies.
  - To facilitate combined usage of water resources and support applications like intelligent water management, dynamic irrigation analysis, sewage and sewer value-added applications, and flood warning systems for roads. Opening up data access aims to boost the development of water resource applications and collaborative decision-making between the public and private sectors.

- Air Quality: Collaborating with the Environmental Protection Agency, the Ministry of Economic Affairs, the Ministry of Science and Technology, and Academia Sinica, this project has launched an ambitious initiative to improve air quality monitoring.
  - It involves deploying a wide network of air quality sensors across Taiwan, establishing a sensor performance testing center, and developing various types of air quality sensors.
  - A computing operation platform for the IoT and an intelligent environment sensing data center are being established to gather extensive data.
  - A visualization platform for air quality data is being built to enhance IoT-based air quality monitoring and provide high-resolution data for intelligent environmental inspections, helping to pinpoint pollution hotspots for effective governance.
  - The project also focuses on bolstering domestic technology in sensor development, aiming to establish Taiwan's own airborne product sensing technology.

- Earthquakes: Recognizing Taiwan's seismic activity, the project has significantly increased the number of quick-reporting earthquake stations for dense, high-quality data.
  - It has added and upgraded seismic and geophysical observatories, including GNSS stations, underground seismic stations, strong earthquake stations, geomagnetic stations, and groundwater stations.
  - Enhancements are being made to monitor the Datun Volcano region, including advanced GNSS and seismic stations and drone surveillance.
  - The project has also improved forecasting capabilities for earthquakes and tsunamis, expanded submarine optical cables, and established related stations.
  - An integrated data management system for seismic and geophysical data is being developed to consolidate earthquake information, enhancing fault area monitoring and providing rapid, comprehensive earthquake reports for public early warnings and supporting earthquake disaster prevention industries.

- Disaster Prevention and Relief: A comprehensive "Civil Alert Information Platform" has been created, gathering 58 types of warning data (air, water, land, disaster, and livelihood) in one place.
  - This platform offers real-time disaster prevention and relief information to the public.
  - Incorporating the EMIC2.0 system and other decision-assisting tools, it provides disaster personnel with crucial information on various disaster scenarios, notifications, and resources for effective decision-making.
  - Historical data is being collected, standardized, and released to support the disaster prevention industry, enabling analysis and development of the disaster information chain.

## Civil IoT Taiwan Data Service Platform

Additionally, the Civil IoT Taiwan project is establishing the Civil IoT Taiwan Data Service Platform, a key component designed to manage the diverse data generated by the project. This platform aims to offer stable and high-quality data for various environmental management needs. Embracing the concept of open data, the platform will:

- Use a standardized data format to ensure consistency and ease of use.
- Provide real-time data interfaces and historical data query services, making it easier for users to access current and past data.
- Enhance user experience by improving browsing and search speeds.
- Implement sensor data storage mechanisms that support scientific computing and artificial intelligence applications.

The goal of the data service platform is to bridge the gap in environmental information availability. It will deliver more immediate and comprehensive environmental data, enabling the public to easily access and understand changes in their environment in real-time. This platform is not just for public awareness; it will also serve as a foundation for industrial innovation. The data it provides can be utilized for creating added value in various industries, fostering creativity among the populace, and offering high-quality solutions to their environmental challenges.

Currently, the Civil IoT Taiwan Data Service Platform uses the open data format of the [OGC SensorThings API](https://developers.sensorup.com/docs/). Please refer to the following slides and pictures for relevant data format descriptions and data content in various fields:

- Introduction to Civil IoT Taiwan Data Service Platform
    - [PPT] [Introduction to Civil IoT Taiwan Data Service Platform and OGC SensorThings API](https://ci.taiwan.gov.tw/creativity/file/0-2.%E6%B0%91%E7%94%9F%E5%85%AC%E5%85%B1%E7%89%A9%E8%81%AF%E7%B6%B2%E8%B3%87%E6%96%99%E6%87%89%E7%94%A8%E7%AB%B6%E8%B3%BD_%E8%B3%87%E6%96%99%E6%9C%8D%E5%8B%99%E5%B9%B3%E5%8F%B0_%E8%AA%AA%E6%98%8E%E7%B0%A1%E5%A0%B1.pdf) [[Video](https://youtu.be/Jk9uaqEcIdQ)] (in Chinese)
- Water Resources
    - [PPT] [Water Resources IoT](https://ci.taiwan.gov.tw/creativity/file/7.%E6%B0%91%E7%94%9F%E5%85%AC%E5%85%B1%E7%89%A9%E8%81%AF%E7%B6%B2%E8%B3%87%E6%96%99%E6%87%89%E7%94%A8%E7%AB%B6%E8%B3%BD_%E6%B0%B4%E8%B3%87%E6%BA%90%E7%89%A9%E8%81%AF%E7%B6%B2_%E8%AA%AA%E6%98%8E%E7%B0%A1%E5%A0%B1.pdf) [[Video](https://youtu.be/NL2zUgZzsIY)] (in Chinese)
- Air Quality
    - [PPT] [Environment Quality Sensing IoT](https://ci.taiwan.gov.tw/creativity/file/1.%E6%B0%91%E7%94%9F%E5%85%AC%E5%85%B1%E7%89%A9%E8%81%AF%E7%B6%B2%E8%B3%87%E6%96%99%E6%87%89%E7%94%A8%E7%AB%B6%E8%B3%BD_%E7%92%B0%E5%A2%83%E5%93%81%E8%B3%AA%E6%84%9F%E6%B8%AC%E7%89%A9%E8%81%AF%E7%B6%B2_%E8%AA%AA%E6%98%8E%E7%B0%A1%E5%A0%B1.pdf) [[Video](https://youtu.be/O3i_2STiQlc)] (in Chinese)
    - [PPT] [Deployment of Micro PM2.5 Sensors](https://ci.taiwan.gov.tw/creativity/file/2.%E6%B0%91%E7%94%9F%E5%85%AC%E5%85%B1%E7%89%A9%E8%81%AF%E7%B6%B2%E8%B3%87%E6%96%99%E6%87%89%E7%94%A8%E7%AB%B6%E8%B3%BD_PM2.5%E5%BE%AE%E5%9E%8B%E6%84%9F%E6%B8%AC%E5%99%A8%E5%B8%83%E5%BB%BA_%E8%AA%AA%E6%98%8E%E7%B0%A1%E5%A0%B1.pdf) [[Video](https://youtu.be/WVZoTsN_bzI)] (in Chinese)
- Earthquake
    - [PPT] [Joint Sea-Land Earthquake Observation](https://ci.taiwan.gov.tw/creativity/file/3.%E6%B0%91%E7%94%9F%E5%85%AC%E5%85%B1%E7%89%A9%E8%81%AF%E7%B6%B2%E8%B3%87%E6%96%99%E6%87%89%E7%94%A8%E7%AB%B6%E8%B3%BD_%E6%B5%B7%E9%99%B8%E5%9C%B0%E9%9C%87%E8%81%AF%E5%90%88%E8%A7%80%E6%B8%AC_%E8%AA%AA%E6%98%8E%E7%B0%A1%E5%A0%B1.pdf) [[Video](https://youtu.be/PvhaT8GR430)] (in Chinese)
    - [PPT] [Composite Earthquake Quick Report](https://ci.taiwan.gov.tw/creativity/file/4.%E6%B0%91%E7%94%9F%E5%85%AC%E5%85%B1%E7%89%A9%E8%81%AF%E7%B6%B2%E8%B3%87%E6%96%99%E6%87%89%E7%94%A8%E7%AB%B6%E8%B3%BD_%E8%A4%87%E5%90%88%E5%BC%8F%E5%9C%B0%E9%9C%87%E9%80%9F%E5%A0%B1_%E8%AA%AA%E6%98%8E%E7%B0%A1%E5%A0%B1.pdf) [[Video](https://youtu.be/9WX7gcpwngw)] (in Chinese)
- Disaster prevention and relief
    - [PPT] [Civil Alert Open Data](https://ci.taiwan.gov.tw/creativity/file/5.%E6%B0%91%E7%94%9F%E5%85%AC%E5%85%B1%E7%89%A9%E8%81%AF%E7%B6%B2%E8%B3%87%E6%96%99%E6%87%89%E7%94%A8%E7%AB%B6%E8%B3%BD_%E6%B0%91%E7%94%9F%E7%A4%BA%E8%AD%A6%E5%85%AC%E9%96%8B%E8%B3%87%E6%96%99_%E8%AA%AA%E6%98%8E%E7%B0%A1%E5%A0%B1pdf.pdf) [[Video](https://youtu.be/KQFQElGM3pY)] (in Chinese)
    - [PPT] [Integration of Disaster Prevention and Relief Information Systems](https://ci.taiwan.gov.tw/creativity/file/6.%E6%B0%91%E7%94%9F%E5%85%AC%E5%85%B1%E7%89%A9%E8%81%AF%E7%B6%B2%E8%B3%87%E6%96%99%E6%87%89%E7%94%A8%E7%AB%B6%E8%B3%BD_%E7%81%BD%E5%AE%B3%E9%98%B2%E6%95%91%E8%B3%87%E8%A8%8A%E7%B3%BB%E7%B5%B1%E6%95%B4%E5%90%88_%E8%AA%AA%E6%98%8E%E7%B0%A1%E5%A0%B1.pdf) [[Video](https://youtu.be/dZnvI9HHjZs)] (in Chinese)

Since its inception in 2018, the Civil IoT Taiwan project has actively engaged the public and fostered innovation through a variety of initiatives:

- Data Application Competitions: These competitions encourage participants to explore creative uses of the data generated by the Civil IoT Taiwan project, showcasing their skills in data manipulation and interpretation.
- Data Innovation Hackathons: These events bring together programmers, designers, and others interested in data innovation to collaborate intensively on software projects, using Civil IoT Taiwan data to create new solutions and applications.
- Physical and Virtual Exhibitions: These exhibitions provide platforms for participants to showcase their projects and innovations, allowing a broader audience to engage with and understand the potential applications of Civil IoT Taiwan data.

In addition to these events, the project has developed comprehensive training materials and business coaching programs. These resources cover various aspects, including team building, idea development, and application services, to help participants effectively use the Civil IoT Taiwan Data Platform.

Over the years, these efforts have culminated in several successful case studies. These cases demonstrate that the Civil IoT Taiwan project has evolved beyond just a government hardware initiative. It has become a foundational infrastructure for improving people's livelihoods. By continuously providing high-quality sensor data, the project enhances everyday life and paves the way for more innovative, convenient, and compassionate information services.

For detailed examples and more information about applications and solutions utilizing Civil IoT Taiwan data in various fields, you can visit specific website resources:

- [Civil IoT Taiwan Service and Solution Guide: Water resources](https://www.civiliottw.tadpi.org.tw/water.html)
- [Civil IoT Taiwan Service and Solution Guide: Air qualuty](https://www.civiliottw.tadpi.org.tw/air.html)
- [Civil IoT Taiwan Service and Solution Guide: Earthquake](https://www.civiliottw.tadpi.org.tw/earthquake.html)
- [Civil IoT Taiwan Service and Solution Guide: Disaster prevention and relief](https://www.civiliottw.tadpi.org.tw/disaster.html)

## References

- Open Definition: defining open in open data, open content, and open knowledge. Open Knowledge Foundation ([https://opendefinition.org/od/2.1/en/](https://opendefinition.org/od/2.1/en/))
- Civil IoT Taiwan ([https://ci.taiwan.gov.tw](https://ci.taiwan.gov.tw/))
- Civil IoT Taiwan Virtual Expo: Dialogue in Civil IoT ([https://ci.taiwan.gov.tw/dialogue-in-civil-iot](https://ci.taiwan.gov.tw/dialogue-in-civil-iot))
- Civil IoT Taiwan Service and Solution Guide ([https://www.civiliottw.tadpi.org.tw](https://www.civiliottw.tadpi.org.tw/))
- Civil IoT Taiwan Data Service Platform ([https://ci.taiwan.gov.tw/dsp/](https://ci.taiwan.gov.tw/dsp/))
- XML - Wikipedia ([https://en.wikipedia.org/wiki/XML](https://en.wikipedia.org/wiki/XML))
- JSON - Wikipedia ([https://en.wikipedia.org/wiki/JSON](https://en.wikipedia.org/wiki/JSON))
- Comma-separated values - Wikipedia ([https://en.wikipedia.org/wiki/Comma-separated_values](https://en.wikipedia.org/wiki/Comma-separated_values))
- Standard Generalized Markup Language - Wikipedia ([https://en.wikipedia.org/wiki/Standard_Generalized_Markup_Language](https://en.wikipedia.org/wiki/Standard_Generalized_Markup_Language))
- OGC SensorThings API Documentation ([https://developers.sensorup.com/docs/](https://developers.sensorup.com/docs/))
