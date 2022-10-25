---
title: "2.2. 課程軟體工具簡介"
weight : 20
socialshare: true
description : "民生公共物聯網資料應用所使用到的程式語言 Python 和開發平台 Google Colab 簡介"
tags: ["概述" ]
levels: ["beginner" ]
author: ["高慧君", "陳伶志"]
---

{{< toc >}}


## 程式語言 - Python

針對本套課程的主題著重在民生公共物聯網資料應用，本套課程將以目前資料科學界所常用的程式語言 Python 為主要程式語言，利用淺顯易懂的示範方式，帶領讀者透過做中學的方式，逐步進入各章節的主題內涵，並能獲得民生公共物聯網資料應用的第一手經驗，以及未來舉一反三應用於其他資料科學主題的能力。整體來說，Python 程式語言能在短時間內迅速成為資料科學界最熱門的程式語言，主要有下列三大優勢：

- 學習的門檻較低：在文字式的程式語言來說，相較於其它語言（例如 C、Java），Python 程式碼較少特殊符號，字面上看起來比較接近日常生活中常見的英文文章，如果掌握了 Python 語法和程式執行的邏輯，再加上英文單字能力，就能夠很容易理解 Python 程式碼語意，因此學習 Python 的門檻較低！
- 各式各樣的套件庫：Python 經過三十多年的發展，儼然壯大成一個生態系，除了基本語法外，還可以加裝各式各樣的套件，就能進階變形為各種的工具，適合用來解決五花八門的問題。透過本計畫量身訂制一個 Python 套件 (CIOT)，可以更快速地提高學習的天花板，從而了解如何更方便地應用民生公共物聯網資料。
- 閱讀理解程式碼較適合自學：隨著套件的使用，閱讀理解程式碼的能力比起程式碼寫作能力更為重要！如果要駕馭現有的套件，必須先看得懂使用手冊，然後再排列組合去拼湊成解決問題的程式碼。所以問題解決的模式，會是「在現有的程式高手基礎上去堆疊程式，而不是從零開始寫 Python 程式碼」；尤其是培養閱讀理解程式碼的能力，比較適合用自學模式來進行，當逐漸熟悉 Python 的邏輯後，學習閱讀程式碼就有機會變得跟閱讀故事書一樣有趣喔！

也由於 Python 語言具備上述的各項優點，Python 語言目前已成為資料科學上最普遍被使用的程式語言，更成為許多程式設計學習者所使用的第一個程式語言。因此，除了坊間各種 Python 語言的教學書籍外，在網際網路上也可以找到多實用的學習資源，值得對 Python 語言有興趣的讀者，進一步主動的進行探索與學習。

- 免費教學課程
    - 用 Python 做商管程式設計（一），孔令傑，Coursera ([https://zh-tw.coursera.org/learn/pbc1](https://zh-tw.coursera.org/learn/pbc1))
    - Python 入門教學課程，彭彭的課程 ([https://www.youtube.com/watch?v=wqRlKVRUV_k&list=PL-g0fdC5RMboYEyt6QS2iLb_1m7QcgfHk](https://www.youtube.com/watch?v=wqRlKVRUV_k&list=PL-g0fdC5RMboYEyt6QS2iLb_1m7QcgfHk))
    - Python for Everybody Specialization, Coursera ([https://www.coursera.org/specializations/python](https://www.coursera.org/specializations/python))
    - Python for Data Science, AI & Development, Coursera ([https://www.coursera.org/learn/python-for-applied-data-science-ai](https://www.coursera.org/learn/python-for-applied-data-science-ai))
    - Introduction to Python Programming, Udemy ([https://www.udemy.com/course/pythonforbeginnersintro/](https://www.udemy.com/course/pythonforbeginnersintro/))
    - Learn Python for Total Beginners, Udemy ([https://www.udemy.com/course/python-3-for-total-beginners/](https://www.udemy.com/course/python-3-for-total-beginners/))
    - Google’s Python Class ([https://developers.google.com/edu/python](https://developers.google.com/edu/python))
    - Introduction to Python, Microsoft ([https://docs.microsoft.com/en-us/learn/modules/intro-to-python/](https://docs.microsoft.com/en-us/learn/modules/intro-to-python/))
    - Learn Python 3 from Scratch, Educative ([https://www.educative.io/courses/learn-python-3-from-scratch](https://www.educative.io/courses/learn-python-3-from-scratch))
- 免費電子書資源
    - Python 教學 ([https://docs.python.org/zh-tw/3/tutorial/index.html](https://docs.python.org/zh-tw/3/tutorial/index.html))
    - Python 教學 (學習導讀)，STEAM 教育學習網 ([https://steam.oxxostudio.tw/category/python/info/start.html](https://steam.oxxostudio.tw/category/python/info/start.html))
    - Python 程式設計，李明昌 ([http://rwepa.blogspot.com/2020/02/pythonprogramminglee.html](http://rwepa.blogspot.com/2020/02/pythonprogramminglee.html))
    - Non-Programmer's Tutorial for Python 3, Josh Cogliati ([https://en.wikibooks.org/wiki/Non-Programmer's_Tutorial_for_Python_3](https://en.wikibooks.org/wiki/Non-Programmer%27s_Tutorial_for_Python_3))
    - Python 101, Michael Driscoll ([https://python101.pythonlibrary.org/](https://python101.pythonlibrary.org/))
    - The Python Coding Book, Stephen Gruppetta ([https://thepythoncodingbook.com/](https://thepythoncodingbook.com/))
    - Python Data Science Handbook, Jake VanderPlas ([https://jakevdp.github.io/PythonDataScienceHandbook/](https://jakevdp.github.io/PythonDataScienceHandbook/))
    - Intro to Machine Learning with Python, Bernd Klein ([https://python-course.eu/machine-learning/](https://python-course.eu/machine-learning/))
    - Applied Data Science, Ian Langmore and Daniel Krasner ([https://columbia-applied-data-science.github.io/](https://columbia-applied-data-science.github.io/))

## 開發平台 – Google Colab

有別於可以事先編譯成執行檔的C語言，Python 本身是屬於直譯式語言，就是在執行前才翻譯成機器語言來讓電腦執行，換句話說，就是邊執行邊直譯。以日常生活類似情境來理解，這種方式就好像有一個翻譯員在幫我們翻譯用外國人能接受的語言來溝通，我們說一句話，翻譯員就幫我們直譯一句話；相反的，外國人說一句話，翻譯員就幫我們把外國人的話翻譯回來讓我們理解。

由於這種直譯的特性，所以除了 Python 官方提供的程式編輯器，也有許多其它不同功能的編輯器。基於 Python 程式碼與自然語言較雷同，就有人提出將 Python 編輯器做成類似筆記本的模式，可以把自然語言的文章與 Python 程式碼混在一頁並存的編輯器，其中以 Jupyter 最為人熟知。

在 Jupyter 的基礎上，Google 公司將 Python 直譯程式搬到網路雲端，只要申請 Google 帳號，就可以在 Google 雲端硬碟中加裝免費的 Colab 應用程式，直接使用瀏覽器就可以享受 Python 程式編輯功能，也成為 Python 使用者最常使用的開發平台。

整體而言，Google Colab 具有下列四大優點：

- 預先安裝的套件：Google Colab 開發平台已預先安裝絕大多數的 Python 套件，讓使用者可以直接使用，避免必須自行安裝，甚至必須自行排除不同套件在安裝時產生的版本衝突問題，大幅降低使用 Python 開發程式的入門門檻。
- 雲端儲存：借助 Google 本身的雲端儲存空間，Google Colab 可將開發過程中的程式筆記儲存在雲端空間，只要透過網路存取，即使在不同的電腦上也可以無縫接軌，解決資料儲存、備份與攜帶的問題。
- 合作共創：Google Colab 提供網路分享與線上協作的功能，允許使用者將程式筆記透過雲端儲存空間分享給其他使用者，也允許使用者邀請其他使用者瀏覽、註解、甚至編輯自己的程式筆記，加速了團隊合作的便利性。
- 免費的 GPU 和 TPU 運算資源：借助 Google 本身豐沛的雲端運算資源，Google Colab 提供 GPU 與 TPU 處理器，讓使用者可以使用高階的處理器執行自己個人的程式筆記內容，嘉惠高運量的程式開發需求。

有關 Google Colab 的相關學習資源，可參考下列的連結：

- Google Colab 使用教學，MeDA School，國立台灣大學 ([https://www.youtube.com/watch?v=OyS6K2XdgbQ](https://www.youtube.com/watch?v=OyS6K2XdgbQ))
- 使用 Google Colab，STEAM 教育學習網 ([https://steam.oxxostudio.tw/category/python/info/online-editor.html](https://steam.oxxostudio.tw/category/python/info/online-editor.html))
- Getting Started With Google Colab, Anne Bonner ([https://towardsdatascience.com/getting-started-with-google-colab-f2fff97f594c](https://towardsdatascience.com/getting-started-with-google-colab-f2fff97f594c))
- Use Google Colab Like A Pro, Wing Poon ([https://pub.towardsai.net/use-google-colab-like-a-pro-39a97184358d](https://pub.towardsai.net/use-google-colab-like-a-pro-39a97184358d))

## 參考資料

- Python ([https://www.python.org/](https://www.python.org/))
- Google Colaboratory ([https://colab.research.google.com/](https://colab.research.google.com/))
- Jupyter: Free software, open standards, and web services for interactive computing across all programming languages  ([https://jupyter.org/](https://jupyter.org/))
- 4 Reasons Why You Should Use Google Colab for Your Next Project, Orhan G. Yalçın ([https://towardsdatascience.com/4-reasons-why-you-should-use-google-colab-for-your-next-project-b0c4aaad39ed](https://towardsdatascience.com/4-reasons-why-you-should-use-google-colab-for-your-next-project-b0c4aaad39ed))
