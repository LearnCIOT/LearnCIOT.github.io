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

自古以來，人類對自然界的好奇心始於農業文明時期，那時的人們會仰望天空，觀察星象。到了西元十六世紀的文藝復興時代，哥白尼根據當時的天文觀測資料提出「地動說」，伽利略和牛頓則接續其思路，為現代科學奠定了基石。在當今社會，隨著半導體技術的快速進步，我們使用的感知生活環境的工具日趨多樣、精確和微型。當這些微型感測器結合了資訊科技的演進，以及網際網路的即時資料傳輸，我們獲得了前所未有的大量觀測數據。

面對這浩如瀚海的數據，科學家正努力從中分析環境變化的模式，研究這些模式與災害之間的連結。他們的目的是為了更好地預測災害、提高我們的生活品質、強化災害防救效果，並促進人與環境的和諧共生。這不僅是現今個人、社群、國家乃至全球共同關心的議題，也是努力追求美好、永續生活環境的核心目標。

## 開放資料

「開放資料」是一種策略，將各式電子化資料 —— 無論是文字、數據、圖片、影片還是聲音等，都經過特定的程序和格式整理，然後公開提供，並鼓勵各方自由使用。依據 [Open Knowledge Foundation 的定義](https://opendefinition.org/od/2.1/en/)，要被認定為開放資料，必須符合以下幾項要件：


- 開放授權或開放狀態 (Open license or status)：開放資料應屬於公眾領域，且須以開放授權的方式釋出，確保不受額外的使用限制。
- 自由存取 (Access)：人們應能在網路上輕鬆取得開放資料。儘管在某些合理情境下，一次性的取用費用或特定的使用條款可能是允許的。
- 機器可讀 (Machine readability)：開放資料的格式應該容易讓電腦進行讀取、分析和修改。
- 開放格式 (Open format)：資料必須採用一種開放的標準格式，確保使用者可以透過免費或開放原始碼的軟體來取得和使用它。

隨著開放原始碼、開放政府以及開放科學等理念的普及，開放資料已經逐步成為政府和科學界制定政策和進行學術研究的重要原則。特別是近來受到廣大關注的環境感知物聯網，因為其廣泛分布於公共空間，加上所監測的環境信息與大眾的日常生活密切相關，更被視為最受期待的開放資料來源之一。






目前網路上常見的開放資料多使用 [JSON、](https://en.wikipedia.org/wiki/JSON)[CSV](https://en.wikipedia.org/wiki/Comma-separated_values) 或 [XML](https://en.wikipedia.org/wiki/XML) 的資料格式：

- JSON：全名為 Javascript Object Notation，這是一個輕量化的資料交換格式。它由屬性和值所組成，不僅機器友善，人們也能輕鬆閱讀和解讀。這格式常被選用於網站的資料展示、傳輸和存儲。

- CSV：全名為 Comma-Separated Values，正如其名，它是一種以文字方式儲存表格式資料的方法。在這格式中，每筆資料都是一行，而每個屬性則是一列。當資料儲存時，所有的屬性都會按照特定的順序進行排列，並用特定的符號分隔。這種格式廣泛應用於軟體的資料匯入 / 匯出、網路資料傳送以及歷史資料存儲。

- XML：全名為 Extensible Markup Language，它基於標準通用標記語言 (Standard Generalized Markup Language, [SGML](https://en.wikipedia.org/wiki/Standard_Generalized_Markup_Language)) 而來，是一個可擴展的標記式語言。它允許使用者自訂他們需要的標籤，適合建立包含結構化資料的文件，並廣泛用於文件的呈現和網上資料交換。



目前臺灣常用的開放資料，多彙整於下列平台：

- 政府資料開放平臺 ([https://data.gov.tw/](https://data.gov.tw/))
- 氣象資料開放平臺 ([https://opendata.cwb.gov.tw/devManual/insrtuction](https://opendata.cwb.gov.tw/devManual/insrtuction))
- 環保署環境資料開放平臺 ([https://data.epa.gov.tw/](https://data.epa.gov.tw/))

## 民生公共物聯網

鑑於環境感知的關鍵性及相應技術的快速進展，政府為更貼近民眾的生活與提供綜合性的公共服務，確認了四大民生議題：空氣品質、地震、水資源和災防。於是，在民國 106 年的「前瞻基礎建設 - 數位建設」計畫中，集結國科會、交通部、經濟部、內政部、環保署、中央研究院、農委會的資源，共同推動了一個跨部門的大型政府計畫－「民生公共物聯網」。此計畫融合大數據、人工智慧和物聯網技術，開發各式智慧生活服務系統，協助政府及民眾一同迎戰環境的變遷挑戰。此外，考量到不同的使用者群像，如政府部門、學術界、產業和大眾，這計畫也旨在促進政府的智慧治理、支持學界和產業的發展，並提升民眾的生活質量。

民生公共物聯網目前著重於以下四大領域：
- 水資源：民生公共物聯網結合中央與地方政府的水利單位，研發佈建多元化的水資源感測器，並且建置各式水文觀測設施與農田灌溉水情感測器，有效整合地面、地下與新興水源資訊，以及相關監測資訊，建立水資源物聯網入口網與灌溉配水動態分析管理平台，透過大數據收集與雲端分析運算，強化各河川局防汛作業系統，並建立污水下水道雲端管理雲，提供各級相關單位進行遠端、自動化及智慧化管理，以達到聯合運用的目標，促進水源智慧調控管理、灌溉配水動態分析管理、污水下水道雲加值應用、路面淹水示警等多元應用，同時透過資訊公開與資料開放，帶動公私協力的水資源資料應用與決策輔助進展。
- 空氣品質：民生公共物聯網結合環保署、經濟部、國科會與中央研究院，從空氣品質感測的基礎設施佈建開始，研發空氣品質感測器，建立感測器性能測試驗證中心，並於全台廣佈大量不同目的用途的空氣品質感測器，透過大量更小時間與空間尺度的感測資料收集，建立空品物聯網運算營運平台與智慧環境感測數據中心，並搭建空品感測資料展示平台，一方面促進空氣品質感測物聯網的產業開展，另一方面提供智慧環保稽查的事證資料。藉由智慧化環境監測，提供國人即時且在地的周遭環境空氣品質感測資料，並且利用高解析度感測數據，協助執法人員鑑別污染熱區，有效進行環境稽查與環境治理，並於計畫期間同時強化國內自有技術研發能量，確立國產自主化空品感測技術的發展。
- 地震：民生公共物聯網針對台灣常見的地震活動，除了大幅增加現地型地震速報主站，以提供高密度且高品質的地震速報資訊外，也同時增設與升級地震與地球物理觀測站，提升包含全球導航衛星系統 (Global Navigation Satellite System, GNSS) 站、井下地震站、強震站、地磁站、地下水站等觀測資料的品質與解析度；此外，針對大屯火山的區域監測，也強化其全球導航衛星系統站、井下地震站及無人機 (Unmanned Aerial Vehicle, UAV) 觀測等項目，並且也針對台灣特有的海島特性，強化強震與海嘯測報效能，擴建海底纜線與相關海底和陸上測站。另外，透過大數據的整合運用，除了建立台灣地震與地球物理資料整合與查詢平台，並且建立整合現地型與區域型地震速報資訊的複合式地震速報平台，可以強化台灣斷層帶及大屯火山區的調查觀測，並且更快速完整地傳遞地震速報資訊，提供民眾強震即時警報，進而促進地震防災產業的開展。
- 防救災：透過民生公共物聯網的介接整合，將全台包含空、水、地、災與民生各類別共計達58種示警資料集結在單一「民生示警公開資訊平台」，提供民眾即時防救災資訊，並透過應變管理資訊雲端系統 (EMIC2.0)，以及相關的決策圖台，提供防災人員各項災情、通報與救災資源，輔助各項決策工作，同時將相關歷史資料利用統一個資料格式彙整與釋出，提供防災產業分析使用，促進災害情資產業鏈的發展。

這些創新措施不僅優化了資源和環境管理，也為政府、企業和公眾提供了即時、可靠的資訊，從而提升了人民的生活品質和安全保障。


## 民生公共物聯網資料服務平台


為了管理和運用民生公共物聯網系統產生的龐大資料，民生公共物聯計畫中也創建了一個專門的資料服務平台，確保提供穩定且高品質的感測資料來支持各類環境管理活動。這平台的目的也在於減少環境資訊的落差，讓所有人都能獲得即時和全面的環境數據，掌握周遭環境的即時變化。

「民生公共物聯網資料服務平台」將開放資料的理念融入其中，使用標準化的資料格式來提供即時資料連接和歷史資料查詢服務。該平台致力於增加使用者的資料瀏覽和搜索速度，並建立一個有效的感測資料存儲機制，以支持模擬分析和人工智慧的應用。這個平台不僅讓民眾能隨時隨地查詢到關於周邊環境的詳細資訊，也為產業提供了一個基礎，促進民間創新，並開發出能解決民生問題的優質服務，同時將資訊和技術資源開放給所有人，讓社會各界能共同參與，創建一個更美好、更可持續的生活環境。

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


  {{< youtube id="HGFTr7aTdJc" title="民生公共物聯網資料應用 - 1 教學網站簡介" >}}

