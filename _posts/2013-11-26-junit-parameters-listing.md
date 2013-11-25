---
layout: post
title: Вывод параметров теста в junit
category: automation, development
---

Параметризацией в современных фреймворках для тестирования уже давно никого не удивишь. В TestNG параметризация реализована на мой взгляд намного более гибко, но это другая история =). 

А вот если ты (как и мы) используешь Junit, то вот небольшой хинт как сделать вывод результатов более наглядным. Начиная с версии 4.11 аннотация @Parameters стала принимать свойство name:

{% highlight java %}
private String login;
    
@Parameters(name = "login: {0}")
public static Collection<String> logins() {
    return Arrays.asList(new String[]{"chuck", "norris"});
}
    
public LoginTest(String login) {
    this.login = login;
}
    
@Test
public void shouldBeSuccessful() {
}
{% endhighlight %}

Сравните вывод, который мы получаем:

<img src="/images/junit-parameters-name-before.png"/>
<img src="/images/junit-parameters-name-after.png"/>

Искушенный пользователь junit заметит, что я использую сигнатуру **Collection&lt;String&gt; ** вместо привычной **Collection<Object[]> ** для метода, провайдящего данные. Такой синтаксис стал доступен тоже [сравнительно недавно](https://github.com/junit-team/junit/pull/702).