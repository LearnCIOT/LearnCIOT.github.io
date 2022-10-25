---
title: "2.1. 課程架構說明"
weight : 10
socialshare: true
description : "民生公共物聯網資料應用的整體架構介紹"
tags: ["概述" ]
levels: ["beginner" ]
author: ["高慧君", "陳伶志"]
---



本套課程包含「教學網站簡介」、「整體課程前言」、「資料取用方法」、「時間維度資料分析」、「空間維度資料分析」、「資料應用案例」與「系統整合案例」等七大主題，其具體內容分別說明如下：

1. 教學網站簡介
我們介紹這整個計畫的核心 - 「民生公共物聯網」，介紹台灣過去幾年在「民生公共物聯網」的各項成果，以及「民生公共物聯網開放資料」涵蓋水、空、地、災四大面向資料，所能帶來在日常生活上的各項應用與服務。我們同時呈現各項既有的文章、影片與成功案例，讓學員與閱讀者能深刻體認「民生公共物聯網」對民生用途的重要性。
2. 整體課程前言
我們介紹這整套課程的架構，以及本套課程所使用的 Python程式語言與 Google Colab程式開發平台，透過淺顯的文字介紹，引導學員快速了解整體課程的架構，並且提供豐富的外部資源清單，讓有興趣與需求的學員，能更進一步深入探索相關的技術。
3. 資料取用方法
我們介紹如何透過我們開發的工具，可以使用簡易的 Python 語法，直接取用民生公共物聯網開放資料平台的資料，並且根據不同的需求，切割出兩個單元進行更深入的介紹：
    - 基本資料存取方法：我們介紹如何取用民生公共物聯網開放資料平台中，有關水、空、地、災不同面向單一測站的最新一筆感測資料，如何獲取所有測站的列表，以及如何獲取所有測站當下最新的一筆感測資料。
    - 存取特定時空條件的資料：我們介紹如何獲取民生公共物聯網資料平台中某測站特定時間或時間區段的資料、尋找最鄰近的測站當下最新的一筆資料，以及尋找某位置座標周圍固定區域所有測站當下最新的一筆資料等應用。
    
    在這兩個單元中，除了介紹資料的存取方法外，我們也穿插基本的探索式資料分析 (Exploratory Data Analysis, EDA) 方法，藉由常用的統計方式描述資料不同面向的資料特性，並繪製簡易的圖表，讓讀者透過動手做、做中學的方式，體驗掌握資料與資料分析的成就感。
    
4. 時間維度資料分析
我們針對物聯網資料所具備的時序特性，設計三個單元介紹一系列的時間維度資料分析方法。
    - 時間序列資料處理：我們使用民生公共物聯網資料平台的感測資料，引導讀者了解移動式平均 (Moving Average) 的使用方法，以及進行時序資料的週期性分析，並進而將時序資料拆解出長期趨勢、季節變動、循環變動與隨機波動四大部分，同時套用既有常見的 Python 語言套件，進行變點檢測 (Change Point Detection) 與異常值檢測 (Outlier Detection)，用以檢視現有民生公共物聯網資料中，有關變點檢測與異常值檢測，其背後所可能隱含的意義。
    - 時間序列資料預測：我們使用民生公共物聯網資料平台的感測資料，套用現有的 Python 資料科學套件（例如 scikit-learn、Kats 等），用動手實作的方式，比較各種套件所內建的不同資料預測模型的使用方法與預測結果，用製圖的方式進行資料呈現，並且探討不同的資料集與不同時間解析度的資料預測，在真實場域所代表的意義，以及可能衍生的應用。
    - 時間序列屬性分群：我們介紹較為進階的資料分群分析。我們首先介紹兩種時間序列的特徵擷取方法，分別是傅立葉轉換 (Fourier Transform) 和小波轉換 (Wavelet Transform)，並且以簡要的方式說明兩種轉換方式之異同。我們介紹兩種不同的時間序列比較方法，分別是幾何距離 (Euclidean Distance) 與動態時間規整 (Dynamic Time Warping, DTW) 距離，並根據所使用的距離函數，套用既有的分群演算法套件，探討資料集在不同時間解析度所代表的意義，以及可能衍生的應用。
5. 空間維度資料分析
我們針對物聯網資料所具備的地理空間特性，根據不同的分析需求與應用，介紹一系列的空間維度資料處理與資料分析方法。
    - 地理空間篩選：我們使用民生公共物聯網資料平台的地震和防救災資料，套疊從政府開放資料平臺取得的行政區域界線圖資，篩選特定行政區域內的資料，以及產製套疊地圖後的資料分布位置圖片檔案。除此之外，我們同時示範如何套疊特定的幾何拓撲區域，並將套疊的成果輸出成檔案與進行繪圖動作。
    - 地理空間分析：我們使用民生公共物聯網資料平台的感測資料，介紹較為進階的地理空間分析，利用測站資訊中的 GPS 位置座標，首先利用尋找最大凸多邊形 (Convex Hull) 的套件，框定感測器所涵蓋的地理區域；接著套用 Voronoi Diagram 的套件，將地圖上的區域依照感測器的分布狀況，切割出每個感測器的勢力範圍。針對感測器與感測器之間的區域，我們利用空間內插的方式，套用不同的空間內插演算法，根據感測器的數值，進行空間地圖上的填值，並產製相對應的圖片輸出。
6. 資料應用案例
我們在這個主題中，將著重在存取民生公共物聯網開放資料後的衍生應用，透過其他函式庫套件與分析演算法的導入，強化民生公共物聯網開放資料的價值與應用服務。我們依照由淺入深的原則，依序發展下列三個子主題：
    - 機器學習初探：我們使用空品和水位類別資料，結合天氣觀測資料，利用資料集的時間欄位進行連結，帶入機器學習的套件，進行感測值的預測分析。我們將示範機器學習的標準流程，並且介紹預測分析的成效評估方法，以及如何避免機器學習容易產生的偏差 (Bias) 與過度學習 (Overfitting) 問題。
    - 感測器異常偵測：我們使用空品類別資料，示範台灣微型空品感測資料上常用的感測器異常偵測演算法，以做中學的方式，一步步從資料準備，特徵擷取，到資料分析、統計與歸納，重現異常偵測演算法的原理與實作過程，讓讀者體驗如何透過疊加基本的資料分析方法，逐步達成進階且實用的資料應用服務。
    - 感測器動態校正模型：我們使用空品類別資料，示範台灣微型空品感測器與官方測站進行動態校正的演算法，以做中學的方式，一步步從資料準備，特徵擷取，到機器學習、資料分析、統計與歸納，重現感測器動態校正模型演算法的原理與實作過程，讓讀者體驗如何透過疊加基本的資料分析與機器學習步驟，逐步達成進階且實用的資料應用服務。
7. 系統整合案例
在這個主題中，我們著重在民生公共物聯網開放資料與其他應用軟體的整合應用，透過其他應用軟體的專業功能，進一步深化與發揮民生公共物聯網開放資料的價值。我們發展的單元內容包含：
    - QGIS 應用：QGIS 是一套免費且開源的 GIS 軟體，使用者可以藉由這套軟體，整理、分析與繪製不同的地理空間資料與主題地圖，呈現地理現象和資訊型態的分佈狀況。我們示範導入民生公共物聯網資料平台中水、空、地、災四大面向的資料，利用 QGIS 的圖資進行套疊，用點擊拖拉的方式，進行在「地理空間篩選」與「地理空間分析」進行過的各種分析，並且討論 QGIS 軟體的優缺點與使用時機。
    - Tableau Public 應用：Tableau 是一套操作簡單且功能強大的資料視覺化軟體，可以輕易地連結資料庫與各種格式資料檔案，並製作各種精美的統計圖表。我們示範如何利用拖拉點選的方式，導入民生公共物聯網資料平台中水、空、地、災四大面向的資料，並且針對數值資料進行簡單的圖表製作與相互比對，對於空間資料則結合地圖圖資進行套疊與製圖，對於時間序列資料則透過圖表方式顯示數值變化的趨勢。此外，透過外部資料源的介接，我們可以將多元且相異來源的資料集，整合在單一地圖製圖中，在極短的時間內產製精美的報表。
    - 使由 leafmap 與 Streamlit 自建簡易的 GIS 資訊服務：leafmap是一套Python 開發套件，用於和 Google Earth Engine 進行深度整合，並能在類似 Google Colab 或 Jupyter Notebook 平台上直接進行操作與視覺化呈現分析成果；Streamlit則是一個提供 Python 使用者迅速搭建架構網頁應用程式的套件。我們示範導入民生公共物聯網資料平台中水、空、地、災四大面向的資料，藉由 leafmap 的地理資訊圖台與空間分析能力，搭配 Google Earth Engine 豐富的衛星影像，建構簡單的 GIS 資訊服務，同時借助 Streamlit 的友善介面操作，整合成網頁版的 GIS 資訊服務，擴展讀者對於資料分析與資訊服務的未來想像。

## 參考資料

- 民生公共物聯網 ([https://ci.taiwan.gov.tw](https://ci.taiwan.gov.tw/))
- 民生公共物聯網資料服務平台 ([https://ci.taiwan.gov.tw/dsp/](https://ci.taiwan.gov.tw/dsp/))
- QGIS - A Free and Open Source Geographic Information System ([https://qgis.org/](https://qgis.org/))
- Tableau - Business Intelligence and Analytics Software ([https://www.tableau.com/](https://www.tableau.com/))
- Leafmap - A Python package for geospatial analysis and interactive mapping in a Jupyter environment ([https://leafmap.org/](https://leafmap.org/))
- Streamlit - The fastest way to build and share data apps ([https://streamlit.io/](https://streamlit.io/))
- Google Colaboratory ([https://colab.research.google.com/](https://colab.research.google.com/))