---
layout: post
title: Используем лямбды в тестах
category: automation, development
---

Для начала, что это за зверь и с чем его едят. Лямбда-выражение — это специальный синтаксис для объявления анонимных функторов по месту их использования. Я уже вижу немой вопрос в твоих глазах - "зачем мне какой-то функтор, да еще и анонимный?". Обо всём по порядку. 

Ты наверняка уже сталкивался в java с анонимными классами - когда конкретная реализация абстрактного класса нужна только единожды здесь и сейчас. Наглядный пример - сортировка:

{% highlight java %}
List<String> list = Arrays.asList("10", "1", "20", "11", "21", "12");
 
Collections.sort(list, new Comparator<String>() {
    public int compare(String o1, String o2) {
        return Integer.valueOf(o1).compareTo(Integer.valueOf(o2));
    }
});
{% endhighlight %}

Здесь мы, вместо того чтобы реализовывать интерфейс **Comparator** в отдельном классе, пишем всё сразу в одном месте. Благо реализовать нужно всего один метод **compare**. Это и есть анонимный класс. 

По сути, этот костыль с интерфейсом и анонимным классом появился именно потому, что в 7-й джаве нет функторов. Функтор - это объект, который может быть использован как функция. Вместо функтора мы передаём реализацию интерфейса, у которой можно вызвать эту самую функцию. Костыль получился прямо скажем не очень красивым - объём кода хоть и меньше по сравнению с реализацией в отдельном классе, но всё же тут много лишнего.

Хорошая новость в том, что существует несколько решений, позволяющих исправить эту вселенскую несправедливость =). Например - [lambdaj](https://code.google.com/p/lambdaj/) или [guava](https://code.google.com/p/guava-libraries/).

Круто, но зачем все эти сложности в тестах? Приведу пару примеров и ты сам всё поймёшь. Получаем тексты сообщений об ошибке со страницы по старинке:

{% highlight java %}
List<WebElement> visibleError = filter(isDisplayed(),
           driver.findElements(By.xpath(".//*[contains(@class, 'error')]")));
List<String> errorText = new ArrayList<String>();
for (WebElement error : visibleError) {
	if (error.getText() != "") {
		errorText.add(error.getText());
	}
}
{% endhighlight %}

И с использованием lambdaj:

{% highlight java %}
List<WebElement> visibleError = filter(isDisplayed(),
           driver.findElements(By.xpath(".//*[contains(@class, 'error')]")));
List<String> errorTexts = extract(visibleError, on(WebElement.class).getText());
errorTexts = filter(not(equalTo("")), errorTexts);
{% endhighlight %}

Найти в списке ссылку с нужным текстом и вернуть её, по старинке:

{% highlight java %}
@FindBy(xpath = "//li/a")
List<WebElement> links;

WebElement newsLink;
for (WebElement link : links) {
	if (link.getText() == "Новости") {
		newsLink = link;
		brake;
	}
}
{% endhighlight %}

И с использованием lambdaj:

{% highlight java %}
@FindBy(xpath = "//li/a")
List<WebElement> links;

WebElement newsLink = selectFirst(links, withText("Новости"));
{% endhighlight %}

Как видишь, использование лямбд позволяет существенно сократить запись, при этом **выигрывая** в читаемости кода. Приятный бонус в том, что lambdaj для различных операций над коллекциями и объектами использует [матчеры](https://github.com/hamcrest/JavaHamcrest), которые мы уже давно и успешно используем в наших тестах.

Ну и напоследок: нативную работу с лямбдами нам обещают в 8-й java, так что самое время начать их изучение и использование ;).