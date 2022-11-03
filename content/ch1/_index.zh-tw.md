---
title: "1. 教學網站簡介"
weight: 1
chapter: true
socialshare: true
description : "教學網站簡介"
tags: ["概述" ]
levels: ["beginner" ]
authors: ["高慧君"]
---

# 1. 教學網站簡介

{{< toc >}}


## 環境感知

人類好奇自然界的各種變化，大概可以追溯自遠古農業文明仰望天象、夜觀星辰的活動，迨至西元十六世紀文藝復興時代，哥白尼依據既有的天文資料推導出「地動說」，隨後歷經伽利略與牛頓的接棒發展，更是近代科學的濫觴。來到現代，由於半導體製程的一日千里，人們用來感知生活環境變化的工具愈來愈多元、精密與微型化，搭上資訊科技日新月異的時代潮流，結合感測器的即時感測，以及網際網路的無遠弗屆資料傳輸，因而產生大量的觀測數據。面對如此汪洋的數據大海，科學家們如何從中找出環境（因子）變化的規律，探究這些規律與災害間的關聯，進而達到預測結果的目標，促進人們生活品質的提升、災害防救的效率，甚至人類與環境之間的友善互動，從而打造美好永續的生活環境，更成為當代無論個人、群體、社會、國家乃是全世界共同關注的主要議題。

## 開放資料

「開放資料」使一種將電子化的資料（包含但不限於文字、數據、圖片、影片、聲音等），透過一定的程序與格式，公開允許並鼓勵各界使用。根據 [Open Knowledge Foundation 的定義](https://opendefinition.org/od/2.1/en/)，開放資料必須滿足下列的條件：

- 開放授權或開放狀態 (Open license or status)：開放資料的內容必須在公眾領域的範疇，並且使用開放授權對外釋出，不存在任何額外的限制。
- 自由存取 (Access)：開放資料的內容必須可以從網際網路上自由存取，但在合理的條件規範下，開放資料的存取也可以允許一次性的收費，並且允許有條件的附帶條款。
- 機器可讀 (Machine readability)：開放資料的內容必須讓電腦可以容易存取、處理與修改。
- 開放格式 (Open format)：開放資料的內容必須使用開放的格式，可以利用免費、自由或開放的軟體進行存取。

隨著開放原始碼、開放政府、開放科學等開放潮流的盛行，開放資料也逐漸成為政府與科學界在從事各項政策推動與學術研究工作時所奉行的圭臬。而近期盛起的環境感知物聯網，由於其廣佈於公眾領域，且其所觀測的環境資訊與民眾息息相關，更成為眾所期待的開放資料項目之一。

目前網路上常見的開放資料多使用 [JSON、](https://en.wikipedia.org/wiki/JSON)[CSV](https://en.wikipedia.org/wiki/Comma-separated_values) 或 [XML](https://en.wikipedia.org/wiki/XML) 的資料格式：

- JSON 格式的全名為 Javascript Object Notation，是一種輕量級的結構化資料交換標準，其內容由屬性與值組成，易於讓電腦或使用者閱讀與使用，常用於網站上的資料呈現、傳輸與儲存。
- CSV 格式的全名為 Comma-Separated Values，顧名思義是一種純文字形式的儲存表格資料，其中每一筆資料為一列，資料中的每一個屬性為一欄，在儲存時所有的屬性必須按照一定的順序排列，並將每個屬性的值以純文字方式呈現，並在不同屬性間插入特定符號作為間隔，常用於應用程式的檔案匯入與匯出、網路資料的傳遞，以及歷史資料的儲存。
- XML 格式的全名為 Extensible Markup Language，是一種從標準通用標記語言 (Standard Generalized Markup Language, [SGML](https://en.wikipedia.org/wiki/Standard_Generalized_Markup_Language)) 所簡化衍生的一種可延伸標記式語言，允許使用者自行定義所需的標籤 (tags) ，可用來建立包含結構化資訊的文件，常用於文件檔案資料的呈現，以及網路資料的交換。

目前臺灣常用的開放資料，多彙整於下列平台：

- 政府資料開放平臺 ([https://data.gov.tw/](https://data.gov.tw/))
- 氣象資料開放平臺 ([https://opendata.cwb.gov.tw/devManual/insrtuction](https://opendata.cwb.gov.tw/devManual/insrtuction))
- 環保署環境資料開放平臺 ([https://data.epa.gov.tw/](https://data.epa.gov.tw/))

## 民生公共物聯網

有鑒於環境感知的重要與相關科技的發展趨勢，政府為整合與貼近民生公共相關服務，擬定民眾四大迫切需求，包括空氣品質、地震、水資源，以及災防等議題，於民國 106 年政府的「前瞻基礎建設 - 數位建設」計畫中，集結國科會、交通部、經濟部、內政部、環保署、中央研究院、農委會，共同建構的政府大型跨部會計畫 - 「民生公共物聯網」，應用大數據、人工智慧、物聯網技術，建置各項智慧生活服務系統，協助政府與民眾共同面對環境變化所帶來的挑戰；同時，此計畫亦考量不同使用者的經驗，包括政府決策單位、學界、產業，以及一般民眾，以提供政府智慧化治理目標，協助產業/學界的發展，提升民眾的幸福感。

目前民生公共物聯網主要涵蓋的四大領域分別是：

- 水資源：民生公共物聯網結合中央與地方政府的水利單位，研發佈建多元化的水資源感測器，並且建置各式水文觀測設施與農田灌溉水情感測器，有效整合地面、地下與新興水源資訊，以及相關監測資訊，建立水資源物聯網入口網與灌溉配水動態分析管理平台，透過大數據收集與雲端分析運算，強化各河川局防汛作業系統，並建立污水下水道雲端管理雲，提供各級相關單位進行遠端、自動化及智慧化管理，以達到聯合運用的目標，促進水源智慧調控管理、灌溉配水動態分析管理、污水下水道雲加值應用、路面淹水示警等多元應用，同時透過資訊公開與資料開放，帶動公私協力的水資源資料應用與決策輔助進展。
- 空氣品質：民生公共物聯網結合環保署、經濟部、國科會與中央研究院，從空氣品質感測的基礎設施佈建開始，研發空氣品質感測器，建立感測器性能測試驗證中心，並於全台廣佈大量不同目的用途的空氣品質感測器，透過大量更小時間與空間尺度的感測資料收集，建立空品物聯網運算營運平台與智慧環境感測數據中心，並搭建空品感測資料展示平台，一方面促進空氣品質感測物聯網的產業開展，另一方面提供智慧環保稽查的事證資料。藉由智慧化環境監測，提供國人即時且在地的周遭環境空氣品質感測資料，並且利用高解析度感測數據，協助執法人員鑑別污染熱區，有效進行環境稽查與環境治理，並於計畫期間同時強化國內自有技術研發能量，確立國產自主化空品感測技術的發展。
- 地震：民生公共物聯網針對台灣常見的地震活動，除了大幅增加現地型地震速報主站，以提供高密度且高品質的地震速報資訊外，也同時增設與升級地震與地球物理觀測站，提升包含全球導航衛星系統 (Global Navigation Satellite System, GNSS) 站、井下地震站、強震站、地磁站、地下水站等觀測資料的品質與解析度；此外，針對大屯火山的區域監測，也強化其全球導航衛星系統站、井下地震站及無人機 (Unmanned Aerial Vehicle, UAV) 觀測等項目，並且也針對台灣特有的海島特性，強化強震與海嘯測報效能，擴建海底纜線與相關海底和陸上測站。另外，透過大數據的整合運用，除了建立台灣地震與地球物理資料整合與查詢平台，並且建立整合現地型與區域型地震速報資訊的複合式地震速報平台，可以強化台灣斷層帶及大屯火山區的調查觀測，並且更快速完整地傳遞地震速報資訊，提供民眾強震即時警報，進而促進地震防災產業的開展。
- 防救災：透過民生公共物聯網的介接整合，將全台包含空、水、地、災與民生各類別共計達58種示警資料集結在單一「民生示警公開資訊平台」，提供民眾即時防救災資訊，並透過應變管理資訊雲端系統 (EMIC2.0)，以及相關的決策圖台，提供防災人員各項災情、通報與救災資源，輔助各項決策工作，同時將相關歷史資料利用統一個資料格式彙整與釋出，提供防災產業分析使用，促進災害情資產業鏈的發展。

## 民生公共物聯網資料服務平台

此外，為了收納所有民生公共物聯網系統所產出的各式資料，提供穩定、高品質的感測資料作為各項環境治理用途；同時也為了降低環境資訊落差，提供更即時與全面的環境資料數據，使民眾可隨時查詢生活周遭環境的即時感測資訊和時空變化，並做為產業加值應用開發的基礎，讓民間創意能量得以發揮，產出能解決民眾問題之優質服務。在「民生公共物聯網」計畫中也特別規劃「民生公共物聯網資料服務平台」，以開放資料的精神，使用統一的資料格式，提供即時資料介接與歷史資料查詢服務，並且提高使用者瀏覽與搜尋的速度，建立感測資料儲存機制，提供模擬分析或人工智慧之應用。

目前民生公共物聯網資料服務平台使用 [OGC SensorThings API](https://developers.sensorup.com/docs/) 的開放資料格式，相關的資料格式說明，以及各領域資料內容說明，可參考下列的投影片與影像說明：

- 資料服務平台簡介
    - [PPT] [資料服務平台與SensorThings API簡介](https://ci.taiwan.gov.tw/creativity/file/0-2.%E6%B0%91%E7%94%9F%E5%85%AC%E5%85%B1%E7%89%A9%E8%81%AF%E7%B6%B2%E8%B3%87%E6%96%99%E6%87%89%E7%94%A8%E7%AB%B6%E8%B3%BD_%E8%B3%87%E6%96%99%E6%9C%8D%E5%8B%99%E5%B9%B3%E5%8F%B0_%E8%AA%AA%E6%98%8E%E7%B0%A1%E5%A0%B1.pdf) [[Video](https://youtu.be/Jk9uaqEcIdQ)]
- 水資源領域
    - [PPT] [水資源物聯網](https://ci.taiwan.gov.tw/creativity/file/7.%E6%B0%91%E7%94%9F%E5%85%AC%E5%85%B1%E7%89%A9%E8%81%AF%E7%B6%B2%E8%B3%87%E6%96%99%E6%87%89%E7%94%A8%E7%AB%B6%E8%B3%BD_%E6%B0%B4%E8%B3%87%E6%BA%90%E7%89%A9%E8%81%AF%E7%B6%B2_%E8%AA%AA%E6%98%8E%E7%B0%A1%E5%A0%B1.pdf) [[Video](https://youtu.be/NL2zUgZzsIY)]
- 空氣品質領域
    - [PPT] [環境品質感測物聯網](https://ci.taiwan.gov.tw/creativity/file/1.%E6%B0%91%E7%94%9F%E5%85%AC%E5%85%B1%E7%89%A9%E8%81%AF%E7%B6%B2%E8%B3%87%E6%96%99%E6%87%89%E7%94%A8%E7%AB%B6%E8%B3%BD_%E7%92%B0%E5%A2%83%E5%93%81%E8%B3%AA%E6%84%9F%E6%B8%AC%E7%89%A9%E8%81%AF%E7%B6%B2_%E8%AA%AA%E6%98%8E%E7%B0%A1%E5%A0%B1.pdf) [[Video](https://youtu.be/O3i_2STiQlc)]
    - [PPT] [PM2.5微型感測器布建](https://ci.taiwan.gov.tw/creativity/file/2.%E6%B0%91%E7%94%9F%E5%85%AC%E5%85%B1%E7%89%A9%E8%81%AF%E7%B6%B2%E8%B3%87%E6%96%99%E6%87%89%E7%94%A8%E7%AB%B6%E8%B3%BD_PM2.5%E5%BE%AE%E5%9E%8B%E6%84%9F%E6%B8%AC%E5%99%A8%E5%B8%83%E5%BB%BA_%E8%AA%AA%E6%98%8E%E7%B0%A1%E5%A0%B1.pdf) [[Video](https://youtu.be/WVZoTsN_bzI)]
- 地震領域
    - [PPT] [海陸地震聯合觀測](https://ci.taiwan.gov.tw/creativity/file/3.%E6%B0%91%E7%94%9F%E5%85%AC%E5%85%B1%E7%89%A9%E8%81%AF%E7%B6%B2%E8%B3%87%E6%96%99%E6%87%89%E7%94%A8%E7%AB%B6%E8%B3%BD_%E6%B5%B7%E9%99%B8%E5%9C%B0%E9%9C%87%E8%81%AF%E5%90%88%E8%A7%80%E6%B8%AC_%E8%AA%AA%E6%98%8E%E7%B0%A1%E5%A0%B1.pdf) [[Video](https://youtu.be/PvhaT8GR430)]
    - [PPT] [複合式地震速報](https://ci.taiwan.gov.tw/creativity/file/4.%E6%B0%91%E7%94%9F%E5%85%AC%E5%85%B1%E7%89%A9%E8%81%AF%E7%B6%B2%E8%B3%87%E6%96%99%E6%87%89%E7%94%A8%E7%AB%B6%E8%B3%BD_%E8%A4%87%E5%90%88%E5%BC%8F%E5%9C%B0%E9%9C%87%E9%80%9F%E5%A0%B1_%E8%AA%AA%E6%98%8E%E7%B0%A1%E5%A0%B1.pdf) [[Video](https://youtu.be/9WX7gcpwngw)]
- 防救災領域
    - [PPT] [民生示警公開資料](https://ci.taiwan.gov.tw/creativity/file/5.%E6%B0%91%E7%94%9F%E5%85%AC%E5%85%B1%E7%89%A9%E8%81%AF%E7%B6%B2%E8%B3%87%E6%96%99%E6%87%89%E7%94%A8%E7%AB%B6%E8%B3%BD_%E6%B0%91%E7%94%9F%E7%A4%BA%E8%AD%A6%E5%85%AC%E9%96%8B%E8%B3%87%E6%96%99_%E8%AA%AA%E6%98%8E%E7%B0%A1%E5%A0%B1pdf.pdf) [[Video](https://youtu.be/KQFQElGM3pY)]
    - [PPT] [災害防救資訊系統整合](https://ci.taiwan.gov.tw/creativity/file/6.%E6%B0%91%E7%94%9F%E5%85%AC%E5%85%B1%E7%89%A9%E8%81%AF%E7%B6%B2%E8%B3%87%E6%96%99%E6%87%89%E7%94%A8%E7%AB%B6%E8%B3%BD_%E7%81%BD%E5%AE%B3%E9%98%B2%E6%95%91%E8%B3%87%E8%A8%8A%E7%B3%BB%E7%B5%B1%E6%95%B4%E5%90%88_%E8%AA%AA%E6%98%8E%E7%B0%A1%E5%A0%B1.pdf) [[Video](https://youtu.be/dZnvI9HHjZs)]

為了擴大全民對於民生公共物聯網與其資料平台的參與，自 2018 年起，民生公共物聯網計畫也陸續舉辦了一系列的資料應用競賽、資料創新馬拉松、實體與虛擬展覽，同時也設計了一系列的訓練教材與商業輔導，從團隊的建立與養成，創意的發起到成形，應用服務的構思到落地，在過去這幾年間已累積多年的能量，並且成功發展出成功的案例，讓民生公共物聯網不僅僅只是政府單位的硬體佈建，更已成功轉化為民生公共的基礎資訊建設，提供源源不絕高品質的感測資料，便利民眾的生活，開創更多便民、有感、體貼的資訊服務。

有關民生公共物聯網資料應用服務在各個領域的解決方案範例，可參考下列的網站資源：

- [民生公共物聯網資料應用服務與解決方案－水資源](https://www.civiliottw.tadpi.org.tw/water.html)
- [民生公共物聯網資料應用服務與解決方案－空氣品質](https://www.civiliottw.tadpi.org.tw/air.html)
- [民生公共物聯網資料應用服務與解決方案－地震](https://www.civiliottw.tadpi.org.tw/earthquake.html)
- [民生公共物聯網資料應用服務與解決方案－防救災](https://www.civiliottw.tadpi.org.tw/disaster.html)

## 參考資料

- Open Definition: defining open in open data, open content and open knowledge. Open Knowledge Foundation. ([https://opendefinition.org/od/2.1/en/](https://opendefinition.org/od/2.1/en/))
- 民生公共物聯網 ([https://ci.taiwan.gov.tw](https://ci.taiwan.gov.tw/))
- 民生公共物聯網線上虛擬特展：在萬物相連中對話 ([https://ci.taiwan.gov.tw/dialogue-in-civil-iot](https://ci.taiwan.gov.tw/dialogue-in-civil-iot))
- 民生公共物聯網資料應用服務與解決方案指南 ([https://www.civiliottw.tadpi.org.tw](https://www.civiliottw.tadpi.org.tw/))
- 民生公共物聯網資料服務平台 ([https://ci.taiwan.gov.tw/dsp/](https://ci.taiwan.gov.tw/dsp/))
- XML - Wikipedia ([https://en.wikipedia.org/wiki/XML](https://en.wikipedia.org/wiki/XML))
- JSON - Wikipedia ([https://en.wikipedia.org/wiki/JSON](https://en.wikipedia.org/wiki/JSON))
- Comma-separated values - Wikipedia ([https://en.wikipedia.org/wiki/Comma-separated_values](https://en.wikipedia.org/wiki/Comma-separated_values))
- Standard Generalized Markup Language - Wikipedia ([https://en.wikipedia.org/wiki/Standard_Generalized_Markup_Language](https://en.wikipedia.org/wiki/Standard_Generalized_Markup_Language))
- OGC SensorThings API Documentation ([https://developers.sensorup.com/docs/](https://developers.sensorup.com/docs/))
