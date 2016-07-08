---
layout: post
title: Инфраструктура для автоматизации на android - железо
category: development, android, automation
---

Что-то я давно не писал :). Начну потихоньку рассказывать, как устроена и работает автоматизация в тестировании Яндекс.Браузера для android.

Для начала пройдёмся по железу:

 * десктоп с ubuntu на борту - 1шт
 * usb-хаб и/или PCI-e USB концентратор - от 1шт
 * устройства, на которых будем выполнять тесты
 * мощная точка доступа WiFi

Десктоп
=======

Рекомендую ставить максимум памяти, потому что в итоговой конфгурации на одно устройство расходуется примерно гиг оперативы. Добавим сюда системные нужды и учтём что всё это потихоньку течёт... Получается что для 10 устройств довольно быстро упираешься в 16ГБ. Остальные внутренности не так критичны.

USB концентраторы
=================

Из всех испробованных нами USB-хабов, хорошо себя показал Orico A3H10 (или A3H7, отличие только в количестве портов). Все остальные не выдают достаточного для зарядки напряжения после подключения к ним 2-3 девайсов. Из PCI-e устройств мы пробовали только ST-LAB U-1000 и особых проблем с его работой не заметили.

Устройства
==========

Выбор устройств для тестов индивидуален и зависит от задач, которые вы решаете. Напишу, какие критерии отбора у нас:

 * находится в топе по количеству сессий и/или крэшей для нашего приложения
 * его всё ещё можно купить в магазине :)
 * версия android 4.4 и выше
 * достаточно мощное чтобы не тормозить при выполнении тестов (на этом пункте отваливается всякий бюджетный трэш вроде Мегафон.Логина)
 * стабильно держит WiFi/ADB соединение

Последний пункт появился после того как мы купили пару достаточно неплохих по характеристикам телефонов Lenovo. На них постоянно отваливался WiFi, проблема известная, производитель о ней знает, но ничего не делает.

В итоге у нас большинство девайсов - Samsung, Sony и LG. Есть несколько Lenovo планшетов (оказались неплохими устройствами, в отличие от телефонов) и даже один Huawei.

Точка доступа
=============

К сожалению, до сих пор нет нормального способа раздать интернет на android-устройство по USB. Приходится использовать WiFi. Здесь я порекомендовать ничего не могу, потому что отдельную точку доступа мы не заказывали, используем те, что уже стоят в офисе. Единственное что стоит сказать - бери ТД с поддержкой 5ГЦ диапазона.

p.s. Три вещи, на которые можно смотреть бесконечно - как горит огонь, как течёт вода и как выполняются автотесты:

<img src="/images/2016-07-08.png"/>