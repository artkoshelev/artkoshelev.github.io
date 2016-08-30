---
layout: post
title: Автоматизация для android - фильтрация девайсов
category: development, android, automation, docker
---

> Внимание, хардкор! Для понимания происходящего ниже, нужно хорошо разбираться в том, как устроены и работают: selenium, selenium-hub, selenium-grid, docker, appium, adb. Я тебя предупредил.

Зачем нужна фильтрация девайсов
-------------------------------

В вэб-тестах выбор браузера ограничевается тремя основными параметрами - имя браузера, версия и платформа. Всё остальное можно настроить налету - отключить куки/js, изменить размер окна и т.п. В случае с нативными приложениями всё несколько сложнее. Кроме имени браузера (мы для всех устройств используем имя "Android"), и платформы (Android/iOS), есть еще ряд параметров, которые важны для отдельных тестов. У нас это:

 * тип устройства: телефон или планшет
 * размер экрана
 * эмулятор или реальный
 * root-права: наличие/отсутствие
 * кнопки меню: hardware (как у самсунгов) или software (как у LG)
 * аппаратная платформа: ARM/X86

Например, для тестирования регулярных событий в приложении, мы переводи время на устройстве. Это можно сделать только при наличии root-прав и нужно уметь выбирать эти устройства.

Как получается "профиль" девайса
--------------------------------

Изначально мы прописывали эти параметры для каждого устройства вручную. Так мы сначала решили проблему фильтрации, а уже потом стали думать над системным решением. Сейчас этот конфиг генерируется скриптом при подключении девайса внутри контейнера:

{% highlight sh %}
#!/bin/bash

unauthorized=true

while [[ $unauthorized ]]
do
    sleep 1
    unauthorized=`adb devices | grep unauthorized`
done

# hardware buttons
softwarebuttons=`adb shell dumpsys input | grep -oE "NavigationBar.*touchableRegion.*" | sed 's/^.*touchableRegion=\(\[.*\]\),.*$/\1/g'`

# device type
apad=`adb shell getprop ro.build.characteristics | grep tablet`

# root
locked=`adb shell su 0 ls | grep 'not found'`

# abi
arm=`adb shell getprop ro.product.cpu.abi | grep arm`

# version
ANDROID_VERSION=`adb shell getprop | grep ro.build.version.release |  sed 's/^.*:.*\[\(.*\)\].*$/\1/g'`

# display size
info=`adb shell dumpsys display | grep -A 20 DisplayDeviceInfo`
width=`echo ${info} | sed 's/^.* \([0-9]\{3,4\}\) x \([0-9]\{3,4\}\).*density \([0-9]\{3\}\),.*$/\1/g'`
height=`echo ${info} | sed 's/^.* \([0-9]\{3,4\}\) x \([0-9]\{3,4\}\).*density \([0-9]\{3\}\),.*$/\2/g'`
density=`echo ${info} | sed 's/^.* \([0-9]\{3,4\}\) x \([0-9]\{3,4\}\).*density \([0-9]\{3\}\),.*$/\3/g'`
let widthDp=${width}/${density}
let heightDp=${height}/${density}
let sumW=${widthDp}*${widthDp}
let sumH=${heightDp}*${heightDp}
let sum=${sumW}+${sumH}

if [[ $softwarebuttons ]]
then
    HARDWAREBUTTONS=false
else
    HARDWAREBUTTONS=true
fi

if [[ $apad ]]
then
    DEVICETYPE='Apad'
else
    DEVICETYPE='Aphone'
fi

if [[ $locked ]]
then
    HASROOT=false
else
    HASROOT=true
fi

if [[ $arm ]]
then
    ABI='ARM'
else
    ABI='X86'
fi

if [[ ${sum} -ge 81 ]]
then
    DISPLAYSIZE=10
else
    DISPLAYSIZE=7
fi

cat << EndOfMessage
{
    "capabilities": [{
        "browserName": "${DEVICETYPE}",
        "version": "5.0",
        "maxInstances": 1,
        "emulated": false,
        "platform": "ANDROID",
        "androidVersion": "${ANDROID_VERSION}",
        "abi": "${ABI}",
        "hasRoot": ${HASROOT},
        "displaySize": ${DISPLAYSIZE},
        "hardwareButtons": ${HARDWAREBUTTONS}
    }],
    "configuration": {
        "proxy": "${PROXY}",
        "cleanUpCycle": 2000,
        "timeout": 90000,
        "url": "http://${HUB}:${PORT}/wd/hub",
        "maxSession": 1,
        "port": ${PORT},
        "host": "${HUB}",
        "register": true,
        "registerCycle": 5000,
        "hubPort": ${HUB_PORT},
        "hubHost": "${HUB}"
    }
}
EndOfMessage
{% endhighlight %}

Мне кажется в самом скрипте всё довольно просто и в комментариях он не нуждается.
Гораздо интересней вопрос, как об этих параметрах узнаёт selenium-hub, ведь он знает только про имя, версию и платформу?

Custom capabilities
-------------------

Решение подробно описано [в этой статье](https://rationaleemotions.wordpress.com/2014/01/19/working-with-a-custom-capability-matcher-in-the-grid/). Если в двух словах:

 * реализуешь java-класс, в котором описываешь capabilities и способы их сравнения
 * пакуешь его в jar и подсовываешь в classpath при старте хаба
 * прописываешь свой класс в "capabilityMatcher" в конфиге хаба 
 * профит!

На сегодня всё, а в следующих постах расскажу, как собираются образы с эмуляторами и как девайсы автоматически появляются в selenium-grid при подключении к USB. А если у тебя есть вопросы по автоматизации на Android - не стесняйся спрашивать в коментах :-)
