---
layout: post
title: HtmlElements 1.14
category: qa, tests, automation, htmlelements
---

Вчера зарелизили очередную стабильную версию [htmlelements](https://github.com/yandex-qatools/htmlelements).

В релизе пара важных изменений. Во-первых, добавлена возможность задавать кастомный таймаут ожидания элементов/блоков на странице. Красивое и простое решение с использованием аннотации `@Timeout`. Эту фичу давно и многие хотели (висел даже недоделанный PR), скажем спасибо таинственному [emacs](https://github.com/emaks), который её заимплементил и даже покрыл тестами :).

Второе важное изменение - полный отказ от аннотации `@Block`, которая [стала не нужна](http://artkoshelev.github.io/posts/two-releases/) после вливания наших изменений в selenium. В текущей версии она уже была помечена как `@deprecated`, появился повод [убрать ворнинги <strike>и наладить кукисы</strike>](https://yandex.ru/yandsearch?text=%D1%83%D0%B1%D1%80%D0%B0%D1%82%D1%8C%20%D0%B2%D0%BE%D1%80%D0%BD%D0%B8%D0%BD%D0%B3%D0%B8%20%D0%B8%20%D0%BD%D0%B0%D0%BB%D0%B0%D0%B4%D0%B8%D1%82%D1%8C%20%D0%BA%D1%83%D0%BA%D0%B8%D1%81%D1%8B&clid=1955453&win=131&redircnt=1435133615.1) ;).