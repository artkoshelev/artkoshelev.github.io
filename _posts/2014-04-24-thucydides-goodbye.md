---
layout: post
title: До свиданья, Thucydides!
category: automation, qa, bdd
image: 
  feature: 2014-04-24.jpg
---

Слышал про Фукидида? Не, не про греческого философа, а про [фреймворк](http://www.thucydides.info/) для приемочного тестирования web-приложений? А если попробовал или даже используешь - то ты вообще крутой и в тренде. А два с небольшим года назад о нём никто и не знал даже, включая меня. Спасибо [Коле Алименкову](https://twitter.com/xpinjection), который не только откопал его, но и рассказал всем на Selenium Camp'e. 

Вернувшись с конференции, мы сразу стали пробовать. Решение было простым и удобным, а поэтому быстро расползлось по проектам, став стандартом для написания веб тестов. Впервые за историю автоматизации в Яндексе, ВСЕ веб-тесты писались одинаково. После того, как весь код был переведён на новый фреймворк, это существенно сократило время на его поддержку и развитие. Появилось время на разработку интересных, нестандартных тестов и вот тут-то мы приплыли.

Обратной стороной простоты использования Thucydides была его "жесткость". Поначалу он даже не мог работать с selenium-grid - можно было или запускать тесты локально, или использовать платный грид какой-то компании. Причем это не было конфигурацией, такое поведение было закодировано в "кишках" фреймворка. 

Со временем (не без нашего участия конечно), подобные странности были устранены, но даже сейчас сделать что-то нестандартное в рамках этого фреймворка довольно сложно. Взять к примеру стандартную вещь в веб тестировании - использование proxy-сервера для webdriver'a. Вот [пример](http://blog.qatools.ru/thucydides/thucydides-fixture-service/) того, как это решается в Thucydides. Согласись, не каждый осилит найти такое решение? Про кастомизацию отчётов я даже не заикаюсь.

Тем не менее, в "суицидисе" (так его обозвали на очередном Selenium Camp'e) есть ряд удачных решений. Первое, что приходит в голову - концепция шагов и отчёты, понятные не только тем, кто их пишет ;). Спустя год, проблем с thucydides стало больше, чем пользы, которую он наносит. Так появился [Allure](https://github.com/allure-framework), на который мы с радостью, песнями и плясками мигрировали. 

Что это за фреймворк и какие задачи он решает [Тёма Ерошенко](https://twitter.com/eroshenkoam) рассказывал на прошедшей в конце прошлого года [Тестовой среде](http://tech.yandex.ru/events/meetings/testing-environment/). Посмотреть его [доклад](http://tech.yandex.ru/events/meetings/testing-environment/talks/1462/) - твоё домашнее задание на неделю, а на следующей - уже будем писать web-тесты на Allure!

Спасибо за всё и до свидания, Thucydides. Привет, Allure!