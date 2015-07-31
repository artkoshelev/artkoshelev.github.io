---
layout: post
title: Appium и Selenium Grid
category: automation, selenium, grid, appium, mobile
image:
  feature: 2015-07-31.png
---

Ответ на вопрос "Почему Appium?" заслуживает отдельного поста, а пока расскажу как подключить несколько девайсов/эмуляторов на одном хосте в selenium grid.

Appium-server, выступающий в роли selenium ноды, умеет работать только с одним девайсом одновременно. Поэтому тебе нужно будет запустить по отдельному инстансу на каждое устройство. Для каждого инстанса нужен свой конфиг примерно следующего содержания:

{% highlight json %}
{
  "capabilities":
      [
        {
      "browserName": "Android",
      "version": "5.0",
      "maxInstances": 1,
          "platform": "ANDROID"
        }
      ],
  "configuration":
  {
    "cleanUpCycle":2000,
    "timeout":300000,
    "proxy": "org.openqa.grid.selenium.proxy.DefaultRemoteProxy",
    "url": "http://appium-node-host:5555/wd/hub",
    "host": "appium-node-host",
    "port": 5555,
    "maxSession": 1,
    "register": true,
    "registerCycle": 5000,
    "hubHost": "selenium-grid-host",
    "hubPort": 4444
  }
}
{% endhighlight %}

Обрати внимание на следующие моменты:

  * должны быть заданы все три параметра **url**, **host**, **port**. Если не задать хотя бы один из них, appium-сервер инициализирует все три параметрами по умолчанию
  * **port** должен быть уникальным для каждого инстанса

Теперь запускаем appium:

{% highlight bash %}
appium --nodeconfig /path/to/config.json -p PORT --udid DEVICE_ID
{% endhighlight %}

Здесь тоже есть особенности:

  * путь к конфигу должен быть абсолютным - appium не понимает, в какой директории его вызывают
  * **PORT** - номер порта, указанный в конфиге
  * **DEVICE_ID** - id девайса :) те, что показываются по `adb devices`, например

После этого в консоли твоего грида (http://selenium-grid-host/grid/console) появятся свежеподключенные устройства. Впереди будет еще не один пост про автоматизацию мобильных приложений, так что stay tuned =)