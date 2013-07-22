---
layout: post
title: Пишем простые матчеры для сложных объектов
category: test automation, xunit, development, matchers
---

В прошлых постах я рассказывал, как появились матчеры и почему их стоит использовать. Сегодня речь пойдёт о реализации своего матчера.

Прежде всего подумай, действительно ли тебе это нужно  . Если ответ всё еще да – я научу тебя по-максимуму переиспользовать уже готовые матчеры. Как правило, проверка какого-либо сложного объекта заключается в получении простого свойства этого объекта и его проверке. Простой пример: ты тестируешь приложение, посылая http-запросы. Тебе нужно проверять, что на все запросы отдаётся ответ 200. Без использования кастомных матчеров, эта проверка будет выглядеть примерно так:

{% highlight java %}
assertTrue("Expected response is 200, but actual " + request.getResponseCode(), 
	request.getResponseCode() == 200);
{% endhighlight %}

А вот такой мне бы хотелось её видеть:

{% highlight java %}
assertThat(request, hasResponse(equalTo(200)));
{% endhighlight %}

Минимальная реалзиация матчера hasResponse будет при этом такой:

{% highlight java %}
public static Matcher<HttpRequest> hasResponse(Matcher<Integer> subMatcher) {
    return new FeatureMatcher<HttpRequest, Integer>(subMatcher, 
        "response", "actual response") {
    @Override
    protected Integer featureValueOf(HttpRequest actual) {
            return actual.getResponseCode();
    }
    };
}
{% endhighlight %}

Вся магия прячется в имплементации FeatureMatcher, которую мы создаём. Точнее – в методе featureValueOf, который выделяет простое свойство у сложного объекта и передаёт его дальше для проверки. Кроме того, мы передаём в конструктор FeatureMatcher’a две строки – сообщение для expected и actual объектов. При фейле проверки, эти строки автоматически “приклеются” к проверке:

{% highlight java %}
Expected: response <200>
but: actual response was <302>
{% endhighlight %}

Поскольку мой матчер принимает на вход Matcher<Integer>, я могу легко их комбинировать:

{% highlight java %}
assertThat(req, hasResponse(anyOf(equalTo(200), equalTo(302))));
{% endhighlight %}

При этом мне не нужно придумывать сообщение об ошибке, оно сформируется само:

{% highlight java %}
Expected: response (<200> or <302>)
but: actual response was <404>
{% endhighlight %}

Итак, идея FeatureMatcher – выделить простое свойство, описать его и проверить соответствующим готовым матчером. Просто и эффективно!