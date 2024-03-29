---
layout: post
title: Про именования в авто-тестах
category: test automation, xunit
---

Хорошее именование тестовых классов, методов и юзер-степов (если вы используете степовый фреймворк типа thucydides) важно по двум причинам. Во-первых, этот код должен быть понятен любому, кто его читает, без глубокого погружения в контекст. Во-вторых, если вы используете степовый фреймворк – названия классов, методов и степов фигурируют в отчёте.

Именно поэтому нужно избегать слов correct, verify, valid, check и других их синонимов и вариаций при написании тестов. Давайте посмотрим на пример junit-теста:

{% highlight java %}
public class ValidUserInfo {
	@Test
	public void correctUserEmail() {
		user.login();
		user.opensPersonalInfoPage();
		user.shouldSeeCorrectEmail();
	}
}
{% endhighlight %}

Начнём с класса – __ValidUserInfo__. Какой смысл несёт слово __valid__? Никакого, ведь тест для того и пишется, чтобы проверить __валидность, корректность__ определённого функционала. То же самое касается и метода __correctUserEmail__ – слово __correct__ можно убрать и ничего не изменится.

Остался степ __shouldSeeCorrectEmail__. Слово __correct__ и здесь не несёт никакой смысловой нагрузки, а только прячет от нас детали проверки, заставляя углубляться в контекст при разборе кода. Согласитесь, что запись shouldSeeEmail(“me@yandex.ru”) намного нагляднее? В ней сразу видно и что мы проверяем, и ожидаемый результат.

Поэтому слова correct, verify, valid, check – мусор, который засоряет ваш код и отчёты. От них нужно избавляться и взять за правило их больше не использовать.