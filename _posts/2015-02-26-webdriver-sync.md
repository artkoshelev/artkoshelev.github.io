---
layout: post
title: webdriver-sync&#58; меньше боли в js-тестах
category: automation, testing, qa
image: 
  feature: 2015-02-26.jpg
---

Я не знаю, чем руководствовались разработчики [selenium-webdriver](http://code.google.com/p/selenium/wiki/WebDriverJs) для nodejs, но сделать API для тестов асинхронным - не лучшее решение. Это исправили в [webdriver-sync](https://github.com/jsdevel/webdriver-sync). Как несложно догадаться из названия библиотеки, она предоставляет синхронное API, а значит тесты избавляются от всякого "мусора" для синхронизации.