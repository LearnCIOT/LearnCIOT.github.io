---
title: "2.2. Material Tools"
weight : 20
socialshare: true
description : "A brief introduction of the programming language Python and the development platform Google Colab used in the materials"
tags: ["Introduction" ]
levels: ["beginner" ]
authors: ["Huei-Jun Kao", "Ling-Jyh Chen"]
---

{{< toc >}}

## Programming Language - Python

This material is dedicated to applying data derived from Civil IoT, focusing on public welfare. It is grounded in Python, a popular choice within the data science community. Through intuitive and comprehensible demonstrations, participants are guided through hands-on learning experiences, delving deeply into core topics chapter by chapter. Students will gain practical experience applying Civil IoT data and develop the skills to extend their knowledge to other data science applications.

Python’s rapid ascendancy as the preferred programming language in the realm of data science is attributed to three primary advantages:

- Ease of Learning: Python’s syntax is relatively straightforward and intuitive. Unlike C or Java, Python eliminates complex syntax and special characters, rendering it akin to reading everyday English text. With a grasp of basic syntax, logic, and elementary English, students can easily understand and master Python code.
- Extensive Libraries: Python boasts a wide and varied collection of libraries, a benefit accrued over its more than three-decade development journey and supported by a vast open-source community. Users can install diverse packages to extend Python’s capabilities, adapting it to various application scenarios. In this course, a specially customized Python package (pyCIOT) is utilized to aid students in overcoming learning obstacles and mastering the application of Civil IoT data.
- Self-learning Friendly: With the diversification of packages, reading and understanding code becomes increasingly crucial. Students must read manuals and integrate various packages to create codes that address specific problems. This course encourages learning and innovation based on existing programming foundations, eliminating the need to start from scratch. Python learning becomes as engaging as reading a captivating story through continuous practice and reading.

Owing to the advantages outlined above, Python has established itself as the most commonly used programming language in data science, becoming a favorite among beginners. Besides traditional textbooks, the Internet has many helpful learning resources, which is valuable for students keen on delving deeper into Python.

- Free Courses
    - Python for Everybody Specialization, Coursera ([https://www.coursera.org/specializations/python](https://www.coursera.org/specializations/python))
    - Python for Data Science, AI & Development, Coursera ([https://www.coursera.org/learn/python-for-applied-data-science-ai](https://www.coursera.org/learn/python-for-applied-data-science-ai))
    - Introduction to Python Programming, Udemy ([https://www.udemy.com/course/pythonforbeginnersintro/](https://www.udemy.com/course/pythonforbeginnersintro/))
    - Learn Python for Total Beginners, Udemy ([https://www.udemy.com/course/python-3-for-total-beginners/](https://www.udemy.com/course/python-3-for-total-beginners/))
    - Google’s Python Class ([https://developers.google.com/edu/python](https://developers.google.com/edu/python))
    - Introduction to Python, Microsoft ([https://docs.microsoft.com/en-us/learn/modules/intro-to-python/](https://docs.microsoft.com/en-us/learn/modules/intro-to-python/))
    - Learn Python 3 from Scratch, Educative ([https://www.educative.io/courses/learn-python-3-from-scratch](https://www.educative.io/courses/learn-python-3-from-scratch))
- Free e-Book
    - Non-Programmer's Tutorial for Python 3, Josh Cogliati ([https://en.wikibooks.org/wiki/Non-Programmer's_Tutorial_for_Python_3](https://en.wikibooks.org/wiki/Non-Programmer%27s_Tutorial_for_Python_3))
    - Python 101, Michael Driscoll ([https://python101.pythonlibrary.org/](https://python101.pythonlibrary.org/))
    - The Python Coding Book, Stephen Gruppetta ([https://thepythoncodingbook.com/](https://thepythoncodingbook.com/))
    - Python Data Science Handbook, Jake VanderPlas ([https://jakevdp.github.io/PythonDataScienceHandbook/](https://jakevdp.github.io/PythonDataScienceHandbook/))
    - Intro to Machine Learning with Python, Bernd Klein ([https://python-course.eu/machine-learning/](https://python-course.eu/machine-learning/))
    - Applied Data Science, Ian Langmore and Daniel Krasner ([https://columbia-applied-data-science.github.io/](https://columbia-applied-data-science.github.io/))

## Development Platform – Google Colab

Unlike C language, which can be compiled into executable files in advance, Python itself is an interpreted language, that is, it is translated into machine language for computer execution before execution. In other words, it is a literal translation when executed, in situations similar to everyday life. In this way, it is as if translators are helping us to translate and communicate in a language acceptable to foreigners. When we speak a sentence, the translator directly helps us translate the sentence. On the contrary, when the foreigner speaks, the staff will help us translate the foreigner's words so that we can understand.

Due to this feature of the interpreted language, in addition to the program editor officially provided by Python, many other editors have different functions. Based on the similarity between Python code and natural language, some people have proposed making the Python editor into a notebook-like mode, an editor that can mix natural language articles and Python code on one page, the most famous of which is Jupyter.

Based on Jupyter, Google moved the Python interpretation language to the Internet cloud. If you apply for a Google account, you can install the free Colab application on Google Drive and use the browser directly to enjoy the Python program editing function. Due to its simplicity and rich functionality, it has become the most used development platform for Python users.

Overall, Google Colab has four significant advantages:

- Pre-installed packages: The Google Colab development platform has pre-installed most of the Python packages, and users can use them directly, avoiding their installation and even eliminating the problem of version conflicts caused by different packages during the installation process, significantly reducing the entry threshold for using Python to develop programs.
- Cloud storage: With Google's own cloud storage space, Google Colab can store program notes during development in the cloud space. As long as it is accessed through the network, it can be seamlessly connected even on different computers, solving the problems of data storage, backup, and portability.
- Collaboration: Google Colab provides network sharing and online collaboration features, allowing users to share program notes with other users through cloud storage, and allowing users to invite other users to browse, annotate, and even edit their program notes, speeding up team collaboration.
- Free GPU and TPU resources: With Google's rich cloud computing resources, Google Colab provides GPU and TPU processors, allowing users to use high-end processors to execute their personal program notes, which is conducive to large-capacity program development needs.

For related learning resources of Google Colab, please refer to the following links:

- Getting Started With Google Colab, Anne Bonner ([https://towardsdatascience.com/getting-started-with-google-colab-f2fff97f594c](https://towardsdatascience.com/getting-started-with-google-colab-f2fff97f594c))
- Use Google Colab Like A Pro, Wing Poon ([https://pub.towardsai.net/use-google-colab-like-a-pro-39a97184358d](https://pub.towardsai.net/use-google-colab-like-a-pro-39a97184358d))

## References

- Python ([https://www.python.org/](https://www.python.org/))
- Google Colaboratory ([https://colab.research.google.com/](https://colab.research.google.com/))
- Jupyter: Free software, open standards, and web services for interactive computing across all programming languages  ([https://jupyter.org/](https://jupyter.org/))
- 4 Reasons Why You Should Use Google Colab for Your Next Project, Orhan G. Yalçın ([https://towardsdatascience.com/4-reasons-why-you-should-use-google-colab-for-your-next-project-b0c4aaad39ed](https://towardsdatascience.com/4-reasons-why-you-should-use-google-colab-for-your-next-project-b0c4aaad39ed))
