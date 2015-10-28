---
layout: post
title: Как проскролить до элемента в Appium
category: automation, mobile, testing, appium, android
---

Если до нужного тебе элемента мобильного приложения нужно поскролить (а на мобильных это нужно гораздо чаще чем на десктопе), то есть два способа это сделать. Первый - самостоятельно свайпать, пока нужный элемент не появится. Второй, более элегантный на мой взгляд способ, я покажу на примере:

{% highlight java %}
@AndroidFindBy(uiAutomator = 
    "new UiScrollable(new UiSelector().scrollable(true).resourceId(" +
        "\"container_id\")).scrollIntoView(new UiSelector().description(\"element_id\"))")
@Name("Кнопа")
public AndroidElement someBtn;
{% endhighlight %}

В локаторе элемента ты объявляешь некий `UiScrollable` - это "родитель"" твоего элемента. Он должен обладать свойством `scrollable: true`, и именно он скролится когда ты делаешь свайп по экрану. Дальше ты вызываешь у этого элемента `scrollIntoView` в который передаёшь локатор элемента, который хочешь увидеть. При обращении к такому элементу в коде, устройство "само" начнёт скролить пока нужный элемент не появится.

[Тут](http://developer.android.com/reference/android/support/test/uiautomator/UiSelector.html) можно почитать про методы UiSelector, а [здесь](http://developer.android.com/reference/android/support/test/uiautomator/UiScrollable.html) подробней про UiScrollable (там еще много полезного про скролы и свайпы). Вообще, селектор `uiAutomator` довольно мощный инструмент для адресации элементов на Android. Но у него есть серьезный недостаток - ты передаёшь java-код строкой. Мало того, что это само по себе странно, так еще и ошибиться легко. Зато работает ощутимо быстрей других селекторов :).
