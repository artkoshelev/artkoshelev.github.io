---
layout: post
title: Пишем плагин для Jenkins - тестирование
category: jenkins, plugins, automation
image: 
  feature: 2014-10-16.jpg
---

Итак, ты уже разобрался с [глобальной конфигурацией](http://artkoshelev.github.io/posts/jenkins-plugin-global-configuration/), [валидацией настроек](http://artkoshelev.github.io/posts/jenkins-plugin-data-validation/) и [накодил свой степ](http://artkoshelev.github.io/posts/jenkins-plugin-build-step/). Пришло время его протестировать (а можно было и начать с тестов, кстати).

Есть два основных способа тестирования плагинов. В зависимости от выполняемых функций, можно протестировать плагин как отдельную сущность или в контексте работающего инстанса Jenkins. Для первого способа нам понадобятся моки. Много моков. Билд степу требуется много всякого контекста для выполнения - глобальные настройки Jenkins и плагина, настройки самого степа, переменные окружения и т.п. Вот примерный список:

{% highlight java %}
public class MyPluginBuildStepTest {
    AbstractBuild build = mock(AbstractBuild.class);
    Launcher launcher = mock(Launcher.class);
    BuildListener listener = mock(BuildListener.class);
    PrintStream logger = mock(PrintStream.class);
    EnvVars env = mock(EnvVars.class);
    AbstractProject project = mock(AbstractProject.class);
}
{% endhighlight %}

Дальше нужно настроить возвращаемые моками значения и можно выполнять наш степ:

{% highlight java %}
@Test
public void myBuildStepExecutedSuccessfully() {
	MyBuildStep buildStep = new MyBuildStep();
	boolean result = buildStep.perform(build, launcher, listener);
	assertThat("Build step should not fail", result, equalTo(true));
}
{% endhighlight %}

Этот способ близок к юнит-тестам - выполняется быстро, тестируется работа отдельных классов/методов. Его недостаток в том, что для проверки некоторых случаев требуется много первоначальной настройки. 

Иногда лучше пожертвовать скоростью в пользу простоты и написать интеграционный тест. Для этого есть jenkins-рула, которая представляет собой конфигурируемый in-memory инстанс Jenkins. Тот же тест с использованием этой рулы будет выглядеть так:

{% highlight java %}
@Rule
public JenkinsRule jenkins = new JenkinsRule();

@Test
public void myBuildStepExecutedSuccessfully() {
	FreeStyleProject project = jenkins.createFreeStyleProject();
	project.getPublishersList().add(new MyBuidStep());
	FreeStyleBuild build = project.scheduleBuild2(0).get();
	assertThat("Build should not fail", 
		        build.getResult(), 
		        equalTo(Result.SUCCESS));
}
{% endhighlight %}

Как видишь - никаких моков, честный Jenkins, честная джоба в нём, честный билд со всем окружением. Jenkins-рула берёт на себя заботу обо всём окружении и код получается лаконичней, но выполняется это всё ощутимо дольше. Как всегда, лучшим вариантом является комбинирование этих подходов в зависимости от тестируемой функциональности.