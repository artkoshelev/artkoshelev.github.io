---
layout: post
title: Пишем плагин для Jenkins - глобальная конфигурация
category: automation, ci, jenkins
image: 
  feature: 2014-05-27.jpg
---

В нашей повседневной жизни мы достаточно много используем Jenkins, а значит - пишем свои и помогаем разрабатывать чужие плагины. Взявшись за разработку плагина чуть более сложного, чем туториал из документации, очень быстро понимаешь, что информации в сети очень мало. 

Единственный способ почерпнуть что-то полезное - раскуривать исходники популярных плагинов, благо они общедоступны. Я планирую серию постов, посвященных разработке jenkins-плагинов и сегодня - первый из них - про глобальные настройки.

Для начала нам понадобится класс, который будет взаимодействовать с формой настроек плагина. Для этого он должен расширять абстрактный класс `hudson.model.Descriptor`. Самое интересное в нём - это метод `configure`, который нужно будет переопределить.

{% highlight java %}
public class CatPluginDescriptor extends Descriptor<CatConfiguration> {
	private String catName;
	private String catPhotoUrl;

	public CatPluginDescriptor() {
		super(CatConfiguration.class);
	}

	public String getDisplayName() {
		return "Configure your cat";
	}

	@Override
	public boolean configure(StaplerRequest req, JSONObject formData) throws FormException {
		catName = formData.getString("catName");
		catPhotoUrl = formData.getString("catPhotoUrl");
		save();
		return super.configure(req,formData);
	}

	public String getTitle() {
		return title;
	}

	public String getName() {
		return name;
	}
}
{% endhighlight %}

Как видишь, мы наследуемся не просто от класса `Descriptor`, а от `Descriptor<CatConfiguration>` - т.е. параметризуем классом `CatConfiguration`. Это класс, который мы описываем в нашем дескрипторе, его же передаем в конструкторе. А поскольку мы его описываем, он должен быть описываемым =), т.е. реализовывать интерфейс `Describable`:

{% highlight java %}
public class CatConfiguration implements Describable<Configuration> {
	@Extension
	public static final CatPluginDescriptor DESCRIPTOR = new CatPluginDescriptor();

	public CatPluginDescriptor getDescriptor() {
		DESCRIPTOR.load();
		return DESCRIPTOR;
	}
}
{% endhighlight %}

Аннотация `@Extension` нужна для того, чтобы дженкинс мог сам найти наш плагин и провязать настройки. Интерфейс `Describable` требует реализации метода `getDescriptor()`, в котором мы возвращаем наш созданный ранее дескриптор.

Остался последний шаг - сама форма настроек. Для описания как глобальных настроек, так и настроек отдельных шагов сборки, Jenkins использут фреймворк Jelly. Совсем базовые вещи расписаны [тут](https://wiki.jenkins-ci.org/display/JENKINS/Basic+guide+to+Jelly+usage+in+Jenkins). Сначала создаём в папке `src/main/resources/` файлик `global.jelly`. Он должен лежать *ровно по тому же пути*, что и класс, который мы описывали, только в ресурсах. Например, наш файл `Configuration.java` находится в пакете `org.jenkinsci.plugins.myplugin`, значит `global.jelly` должен быть в папке `src/main/resources/org/jenkinsci/plugins/myplugin/Configuration/`. Главное - не забывать переносить файл если меняешь название класса - я несколько раз долго не мог понять, куда же исчезли мои настройки =).

Сам jelly-файл довольно простой:

{% highlight xml %}
<j:jelly xmlns:j="jelly:core" xmlns:st="jelly:stapler" xmlns:d="jelly:define" xmlns:l="/lib/layout" xmlns:t="/lib/hudson" xmlns:f="/lib/form">
  <f:section title="${descriptor.getDisplayName()}">
  	<f:entry title="Cat name" field="catName">
    	<f:textbox default="Metrofan"/>
  	</f:entry>
  	<f:entry title="Cat photo url" field="catPhotoUrl">
    	<f:textbox default="http://clck.ru/9Dg2W"/>
  	</f:entry>
  </f:section>
</j:jelly>
{% endhighlight %}

Ключевой момент здесь - значение аттрибута `field` элементов `<f:entry>` - именно по нему мы достаём значения в методе `configure` нашего дескриптора. А аттрибут `title` элемента `<f:section>` демонстрирует обратную связь - мы получаем результат метода `getDisplayName()` дескриптора и подставляем его в нашу форму. Аналогичным образом можно организовать и хранение дефолтных значений полей.

Фух, на сегодня хватит =).