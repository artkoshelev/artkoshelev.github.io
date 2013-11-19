---
layout: post
title: Подмена хоста в вебдрайвер-тестах "на лету"
category: automation, development
---

Иногда требуется заставить тесты "ходить" на другой хост. Например - потестировать еще не подключенную к кластеру машинку с новой конфигурацией (обновлённой осью, базой и т.п.) в продакшене. Локально это сделать достаточно просто - прописываем адрес машинки в своём файлике hosts. Но как быть в случае с авто-тестами? Они ведь обычно запускаются в общем окружении и замена hosts может затронуть других.

Я уже несколько раз ([тут](http://artkoshelev.github.io/posts/jserrors/) и [тут](http://artkoshelev.github.io/posts/proxyrule/)) писал про замечательный инструмент - [browsermobproxy](https://github.com/lightbody/browsermob-proxy). А [Кирилл](https://twitter.com/delnariel) писал про правильное его использование в связке с [Thucydides](http://blog.qatools.ru/thucydides/thucydides-fixture-service/). Так вот, кроме подстановки юзер-агента и встраивания js-кода, можно прямо в запросах подменять и хост. Для этого просто добавляем еще один обработчик запросов:

{% highlight java %}
proxyServer.addRequestInterceptor(new RequestInterceptor() {
	@Override
    public void process(BrowserMobHttpRequest request) {
        URI uri = request.getMethod().getURI();
        String host = request.getMethod().getURI().getHost();
        if (host == hostToRemap) {
            request.getMethod().setURI(fromUri(uri).host(ip).build());
            LOG.info("added host remap for " + host + " to " + ip);
        }
    }
});
{% endhighlight %}

Кстати, у нас прокси-сервер автоматически стартует, если при инициализации находит в настройках хост, который нужно подменить. Аналогичным образом мы устроили работу и с юзер-агентами.