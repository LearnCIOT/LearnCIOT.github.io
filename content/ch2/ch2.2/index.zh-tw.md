---
title: "2.2. 課程軟體工具簡介"
weight : 20
socialshare: true
description : "民生公共物聯網資料應用所使用到的程式語言 Python 和開發平台 Google Colab 簡介"
tags: ["概述" ]
levels: ["beginner" ]
authors: ["高慧君", "陳伶志"]
---

{{< toc >}}


## 程式語言 - Python

本課程專注於民生公共物聯網資料應用，並以資料科學界廣泛採用的 Python 程式語言為教學基礎。我們透過直觀和易於理解的示範，引領學員透過實踐來學習，逐漸深入各個章節的核心主題。學員將有機會獲得民生公共物聯網資料應用的實際經驗，並培養出對其他資料科學主題的應用能力。

Python 之所以能迅速成為資料科學界的首選程式語言，主要歸因於其三大優勢：

- 低學習門檻：Python 的語法結構相對簡單直觀。與 C 或 Java 等語言相比，Python 少了許多複雜的語法和特殊符號，使其更像是閱讀日常英文文章。只需掌握基本語法和邏輯以及一些基礎英文能力，學員就能輕易理解和掌握 Python 程式碼。
- 豐富的套件庫：Python 有著廣泛而多樣的套件庫，這得益於其三十多年的發展歷程和龐大的開源社群。使用者可以根據需求，安裝各種套件以擴展Python的功能，使其能夠適應各種不同的應用場景。在本課程中，我們會使用一個特別定制的Python套件（pyCIOT），幫助學員更快地突破學習的瓶頸，並掌握民生公共物聯網資料的應用技巧。
- 適合自學的語言：隨著套件的多樣化，學會閱讀和理解程式碼變得尤為重要。學員需要學會閱讀使用手冊，然後學會如何將不同的套件組合起來寫出解決特定問題的程式碼。本課程鼓勵學員建立在現有程式基礎上進行學習和創新，而不是從零開始。透過持續的實踐和閱讀，學習Python將變得像閱讀一本引人入勝的故事書。

由於 Python 語言具備上述的優勢，它已經成為資料科學領域最常用的程式語言，也是許多初學者的首選。除了傳統的教學書籍，網際網路上也充斥著大量有用的學習資源，這些資源對於有興趣深入學習 Python 的學員來說是一個寶貴的學習資產。

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


Python 作為一種直譯式語言，不同於 C 語言等需要事先編譯的語言，它在執行時才將代碼轉換成機器語言。這意味著 Python 代碼在執行時會被一行一行地轉換和執行，這樣的運作方式類似於一位即時翻譯員在我們與外國人交流時幫助我們翻譯語言。

這種直譯的特點使得 Python 擁有多樣性的開發環境選擇。其中，Jupyter 以其類似筆記本的設計，允許用戶將自然語言文字和 Python 代碼組合在一起，成為眾多開發者和數據分析師的首選。基於 Jupyter，Google 推出了 Colab 平台，允許用戶在雲端上編寫和執行 Python 代碼，這不僅減輕了本地計算和存儲的壓力，還方便了協作和分享。

Google Colab 的出現為 Python 開發者帶來了以下便利：

- 無需安裝：Colab 已預先安裝了大多數常用的 Python 套件，用戶無需自行配置和解決可能出現的依賴和衝突問題。
- 雲端儲存：Colab 代碼和數據都存儲在 Google 雲端，用戶可以在任何地方、任何設備上訪問和編輯自己的代碼，免去了數據傳輸和儲存的麻煩。
- 協作共創：Colab 支持多人在線協作，用戶可以實時共享和協作代碼，這對於團隊合作和項目共建具有重要價值。
- 免費運算資源：Colab 提供免費的 GPU 和 TPU 資源，用戶可以利用這些高性能運算資源執行複雜和計算密集型的任務，無需擔心本地計算資源的限制。

這些特點使得 Google Colab 成為了 Python 和資料科學愛好者的理想選擇。無論是初學者還是專業開發者，都可以在這個平台上找到合適的工具和資源，快速開始和深化自己的 Python 和資料科學學習和開發。有關 Google Colab 的相關學習資源，可參考下列的連結：

- Google Colab 使用教學，MeDA School，國立台灣大學 ([https://www.youtube.com/watch?v=OyS6K2XdgbQ](https://www.youtube.com/watch?v=OyS6K2XdgbQ))
- 使用 Google Colab，STEAM 教育學習網 ([https://steam.oxxostudio.tw/category/python/info/online-editor.html](https://steam.oxxostudio.tw/category/python/info/online-editor.html))
- Getting Started With Google Colab, Anne Bonner ([https://towardsdatascience.com/getting-started-with-google-colab-f2fff97f594c](https://towardsdatascience.com/getting-started-with-google-colab-f2fff97f594c))
- Use Google Colab Like A Pro, Wing Poon ([https://pub.towardsai.net/use-google-colab-like-a-pro-39a97184358d](https://pub.towardsai.net/use-google-colab-like-a-pro-39a97184358d))

## 參考資料

- Python ([https://www.python.org/](https://www.python.org/))
- Google Colaboratory ([https://colab.research.google.com/](https://colab.research.google.com/))
- Jupyter: Free software, open standards, and web services for interactive computing across all programming languages  ([https://jupyter.org/](https://jupyter.org/))
- 4 Reasons Why You Should Use Google Colab for Your Next Project, Orhan G. Yalçın ([https://towardsdatascience.com/4-reasons-why-you-should-use-google-colab-for-your-next-project-b0c4aaad39ed](https://towardsdatascience.com/4-reasons-why-you-should-use-google-colab-for-your-next-project-b0c4aaad39ed))
