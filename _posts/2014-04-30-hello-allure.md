---
layout: post
title: Привет, Allure!
category: automation, qa, bdd
image: 
  feature: 2014-04-29.jpg
---

В предыдущем посте я рассказал, почему мы отказались от прекрасного фреймворка [Thucydides](http://www.thucydides.info/), а сегодгня расскажу, как начать использовать не менее прекрасный [Allure](https://github.com/allure-framework). В настоящий момент Allure можно "из коробки" использовать в связке с популярными тестовыми фреймворками под java, python и js. Если в этом списке нет языка, которым пользуешься ты - не расстраивайся. Благодаря модульной архитектуре Allure, тебе достаточно написать адаптор, который преобразует результаты твоих тестов к нужному формату. А рассказывать дальше я буду на примере junit.

Итак, первым делом давай подключим allure в твой проект. Сначала - зависимость в секции `<dependencies>` чтобы можно было использовать аннотации фреймворка в коде твоих тестов.

{% highlight xml %}
<properties>
    <allure.version>1.3.6</allure.version>
    <aspectj.version>1.7.4</aspectj.version>
</properties>

<dependencies>
    <dependency>
        <groupId>ru.yandex.qatools.allure</groupId>
        <artifactId>allure-junit-adaptor</artifactId>
        <version>${allure.version}</version>
    </dependency>
</dependencies>
{% endhighlight %}

Теперь сконфигурим maven-surefire-plugin в секции `<plugins>` чтобы во время выполнения тестов собиралась нужная для Allure информация.

{% highlight xml %}
<plugin>
	<groupId>org.apache.maven.plugins</groupId>
	<artifactId>maven-surefire-plugin</artifactId>
	<version>2.14</version>
	<configuration>
		<testFailureIgnore>false</testFailureIgnore>
		<argLine>
			-javaagent:${settings.localRepository}/org/aspectj/aspectjweaver/${aspectj.version}/aspectjweaver-${aspectj.version}.jar
		</argLine>
		<properties>
			<property>
				<name>listener</name>
				<value>ru.yandex.qatools.allure.junit.AllureRunListener</value>
			</property>
		</properties>
	</configuration>
	<dependencies>
		<dependency>
			<groupId>org.aspectj</groupId>
			<artifactId>aspectjweaver</artifactId>
			<version>${aspectj.version}</version>
		</dependency>
	</dependencies>
</plugin>
{% endhighlight %}

Добавим сюда же jetty-plugin для того чтобы можно было посмотреть отчёт на локальной машине.
{% highlight xml %}
<plugin>
	<groupId>org.mortbay.jetty</groupId>
	<artifactId>jetty-maven-plugin</artifactId>
	<configuration>
		<webAppSourceDirectory>${project.build.directory}/site/allure-maven-plugin</webAppSourceDirectory>
	</configuration>
</plugin>
{% endhighlight %}

Заключительный этап конфигурации проекта - секция `<reporting>`.
{% highlight xml %}
<reporting>
	<excludeDefaults>true</excludeDefaults>
	<plugins>
		<plugin>
			<groupId>ru.yandex.qatools.allure</groupId>
			<artifactId>allure-maven-plugin</artifactId>
			<version>${allure.version}</version>
		</plugin>
	</plugins>
</reporting>
{% endhighlight %}

Самое сложное позади, осталось написать немного тестов. Кстати, если ты хочешь подключить Allure к уже существующему проекту, то на этом можно и закончить. Выполни в консоли `mvn clean test site jetty:run` чтобы:   

 * clean - удалить старый скомпилированый код и артефакты
 * test - скомпилировать тесты и прогнать их <strike>поганой метлой</strike>
 * site - построить отчёт
 * jetty:run - запустить локальный веб-сервер чтобы можно было посмотреть отчёт в браузере

 После того как буковки в консоли прекратят бегать, заходи на localhost:8080 и любуйся =). Тесты в отчёте уже можно фильтровать на успешные/неуспешные/поломаные/пропущенные, а так же посмотреть таймлайн выполнения (иногда там обнаруживаются довольно интересные штуки). А на следующей неделе расскажу про концепцию шагов - когда, зачем и кому они нужны, а так же как их использовать в Allure.

 UPD: если у тебя возникают какие-то сложности с подключением Allure к своему проекту, пожалуйста, задавай свои вопросы на [stackoverflow](http://stackoverflow.com/questions/tagged/allure) с тегом `allure`.