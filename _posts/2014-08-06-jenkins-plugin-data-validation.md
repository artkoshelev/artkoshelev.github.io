---
layout: post
title: Пишем плагин для Jenkins - валидация настроек
category: automation, ci, jenkins
image: 
  feature: 2014-08-06.jpg
---

Итак, очередная статья про разработку плагинов в Jenkins. Посвящена она будет работе с формами настроек. Начнём с валидации.

Поскольку вся работа с формами настроек происходит через дескриптор, то и валидация данных происходит там же. Давай посмотрим на метод для валидации поля url:

{% highlight java %}
public FormValidation doCheckUrl(@QueryParameter String value) {
    try {
        new URL(value);
    } catch (MalformedURLException e) {
        return FormValidation.error("Malformed url");
    }

    return FormValidation.ok();
}
{% endhighlight %}

Метод должен называться `doCheck[FieldName]` - для того чтобы Jenkins мог сам понять, что к полю `fieldName` привязана валидация и вызвать её при изменении поля. Абстрактный класс `FormValidation` имеет множество статических методов, позоволяющих вернуть результат валидации. Основные из них - `ok()`,`error(String message)` и `warning(String message)` (ох уже эти некритичные ошибки). В эти методы так же можно передать Throwable объект - тогда рядом с текстом ошибки в интерфейсе появится ссылка, за которой прячется стектрейс.

Существует еще один метод для работы с формами - `doFill[Field]Items()`. Следуя той же логике, не сложно догадаться, что используется он для заполнения выпадающих списков. В нём всё ещё проще - возвращаем коллекцию с нужными опциями. 

{% highlight java %}
public ListBoxModel doFillAnimalsItems() {
    return new ListBoxModel(
        new Option("cat"),
        new Option("dog"),
        new Option("hamster")
    );
}
{% endhighlight %}

Коротенько получилось, ну, до скорого! =)