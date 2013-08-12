---
layout: post
title: Пишем рулу - прокси
category: js, test automation, webdriver, xunit
---

Я уже писал про использование [browsermob-proxy](https://github.com/lightbody/browsermob-proxy) в посте про [поиск js-ошибок](http://artkoshelev.github.io/posts/jserrors/). В этом же посте предлагалось оформить работу с прокси и драйвером с использованием [рул](http://artkoshelev.github.io/posts/rules-rules/). Настало время сделать это =).

Создаём класс ProxyRule.java, расширяющий абстрактный класс TestWatcher:
{% highlight java %}
public class ProxyRule extends TestWatcher {}
{% endhighlight %}


Т.к. нам нужно запускать прокси при старте теста и гасить её когда тест закончен, оверрайдим соответствующие методы:
{% highlight java %}
@Override
protected void starting(Description description) {
	proxy = new ProxyServer(0);
	try {
		proxy.start();
		ScriptInjection.injectScriptRightAfterHeadTag(proxy, OnErrorHandler.SCRIPT);

		caps = new DesiredCapabilities("firefox", "", Platform.ANY);
		caps.setJavascriptEnabled(true);
		ScriptInjection.addProxyToCapabilities(caps, proxy.seleniumProxy());
	} catch (Exception e) {
		throw new RuntimeException("Error while starting proxy: " + e.getMessage());
	}  
}
	
@Override
protected void finished(Description description) {
	try {
		proxy.stop();
	} catch (Exception e) {
		throw new RuntimeException("Error while stopping proxy: " + e.getMessage());
	}
}
{% endhighlight %}


Кроме того, рула должна отдавать "наружу" созданные DesiredCapabilities, настроенные на работу с прокси.
{% highlight java %}
public DesiredCapabilities getCaps() {
	return caps;
}
{% endhighlight %}


Вот и вся реализация. Просто, правда? Добавляем нашу рулу в тест и удаляем всё, что было в @BeforeClass и @AfterClass:
{% highlight java %}
@ClassRule
public static ProxyRule proxy = new ProxyRule();
{% endhighlight %}


Заметил, что я использую аннотацию @ClassRule, а поле помечено как static? Это нужно для того, чтобы прокси не создавалась при запуске каждого теста, а поднималась единожды для тестового класса. Осталось "настроить" вебдрайвер на использование прокси:
{% highlight java %}
private WebDriver driver = new HtmlUnitDriver(proxy.getCaps());
{% endhighlight %}


Полные исходники примера как обычно можно найти у меня в репозитории в соответствующей [ветке](https://github.com/artkoshelev/jserror-example/tree/proxyrule).