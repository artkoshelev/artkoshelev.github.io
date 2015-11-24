---
layout: post
title: Форк Appium и релиз HtmlElements
category: automation, mobile, testing, appium, android, qatools, htmlelements
---

Мы недавно форкнули в qatools [appium](https://github.com/yandex-qatools/appium) и [appium-adb](https://github.com/yandex-qatools/appium-adb). Почему? Потому что некоторые вещи, которые нам нужны "прямщас", появятся в Appium неизвестно когда. Они ставят тикетам какие-то лейблы, а потом их убирают и нигде нет внятного роадмапа по релизам. Ну и конечно они ([saucelabs](https://saucelabs.com), прим. автора) никогда не дадут в официальном клиенте выполнять некоторые команды - слишком велика вероятность что кто-то случайно (а не со зла) попортит девайсы в облаке, которое они продают. Никто ведь не верит в эмуляторы, а продолжают плакать, колоться и разворачивать облака реальных девайсов %). В общем заглядывай, у нас там реализовано, например, удаление файла (непонятно, как они вообще до сих пор без этого жили?). Код java-клиента для работы с нашим appium лежит [у Лёши в репо](https://github.com/d0lfin/java-client/tree/customcommands).

Еще мы зарелизили [htmlelements 1.15](https://github.com/yandex-qatools/htmlelements/releases/tag/htmlelements-1.15). Если посмотреть на diff, то можно увидеть **450 additions and 1,164 deletions** (люблю запах удалённого кода по утру :)). Это [тот самый рефакторинг](http://artkoshelev.github.io/posts/html-elements-pre-release/), связанный с появлением наших изменений в коде selenium. Еще мы апнули версию java до 1.8 - больше стримов, хороших и разных (с).

Надеюсь в следующем году пересечься лично с кем-нибудь из мэйнтейнеров Appium и согласовать родмап по вливанию наших изменений.