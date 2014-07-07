---
layout: post
title: Пишем плагин для Jenkins - шаг сборки
category: automation, ci, jenkins
image: 
  feature: 2014-07-08.jpg
---

Сегодня - вторая статья из цикла про плагины в Jenkins. В [предыдущем посте]() мы научились работать с глобальными настройками конфига, а сегодня я расскажу как написать свой собственный шаг сборки (или несколько =)).

В терминах Jenkins, любой шаг сборки представляет собой либо Builder (осуществляет непосредственно сборку), либо Publisher (проделывает послесборочные операции). Чтобы было понятнее, поясню на простых примерах. Запустить ant, maven, сборку debian-пакета или скопировать артефакты (необходимые для сборки) из другого проекта - это всё builder'ы. Обновить тикеты в Jira, отправить нотификацию по email или в jabber, поместить в pull-request резульаты тестов - типичные publisher'ы. 

Как ты понимаешь, разделение это довольно условное. Например, статический анализ кода может быть как builder'ом, так и publisher'ом. Это зависит от того, считает ли команда успешное прохождение этого шага обязательной частью процесса сборки или нет. Пусть наш шаг будет publisher'ом - свой плагин чаще всего нужен для интеграции с какой-то внутренней системой, для остального есть <strike>MasterCard</strike> уже готовые плагины.

Итак, наш publisher состоит из двух частей - конфигурации и непосредственно кода. Разберёмся для начала с кодом. Твой класс должен наследовать абстрактный класс Publisher, который уже `@deprecated` =). Вместо него предлагается использовать одного из наследников - Recorder или Notifier. Первый выносит вердикт о статусе билда (на основе информации от других шагов или чего-то еще), а второй - оповещает людей или другие системы об этом статусе.

Нашему шагу наверняка потребуются какие-то параметры - они передаются через конструктор с аннотацией `@DataBoundConstructor`. А переданные в конструктор значения сохраним для дальнейшего использования:

{% highlight java %}
private String email;

@DataBoundConstructor
public BuildCatNotifier(String email) {
    this.email = email;
}

public String getEmail() {
    return email;
}

public void setEmail(String email) {
    this.email = email;
}
{% endhighlight %}

Непосредственно действия сборочного шага выполняются в методе `perform`:

{% highlight java %}
@Override
public boolean perform(AbstractBuild<?, ?> build, Launcher launcher, BuildListener listener) {
}
{% endhighlight %}

В этом методе нам доступны:

 * build - из него можно получить настройки билда и всевозможную информацию о нём
 * launcher - нам пока не нужен
 * listener - из него можно получить логгер, а также управлять результатом сборки

Есть еще один интересный метод, который может тебе пригодиться:

{% highlight java %}
public BuildStepMonitor getRequiredMonitorService() {}
{% endhighlight %}

Он используется для синхронизации с другими сборками в рамках задачи. По сути, это атавизм, оставшийся со времён когда не было параллельных сборок, и желательно не использовать разделяемых ресурсов в своём workflow. Однако это может пригодиться при интеграции с внутренними инструментами, не поддерживающими параллельное использование. Этот метод может возвращать одно из трёх значений:

 * `BuildStepMonitor.BUILD` - если шаг требует отсутствия незавершённых сборок
 * `BuildStepMonitor.STEP` - если шаг требует отсутствия незавершённых аналогичных шагов в других сборках
 * `BuildStepMonitor.NONE` - если синхронизаци не требуется

Когда сам сборочный шаг написан, остаётся провязать это с настройками в web-интерфейсе (чтобы нужные параметры приходили в конструктор). Делается это аналогично глобальным настройками через дескриптор:

{% highlight java %}
@Override
public BuildStepDescriptor<Publisher> getDescriptor() {
    return DESCRIPTOR;
}

@Extension
public static final DescriptorImpl DESCRIPTOR = new DescriptorImpl();

public static class DescriptorImpl extends BuildStepDescriptor<Publisher> {
    public DescriptorImpl() {
        super(BuildCatNotifier.class);
    }

    @Override
    public BuildCatNotifier newInstance(StaplerRequest req, JSONObject formData) throws FormException {
        return req.bindJSON(BuildCatNotifier.class, formData);
    }

    @Override
    public boolean isApplicable(Class<? extends AbstractProject> jobType) {
        return true;
    }

    @Override
    public String getDisplayName() {
        return "Email";
    }

    @Override
    public String getHelpFile() {
        return "/plugin/jira/help-cat-publisher.html";
    }
}
{% endhighlight %}

В общем случае этого достаточно для написания большинства сборочных и интеграционных плагинов. В следующих постах я расскажу про валидацию данных, локализацию и тестирование (куда же без него). Если тебе интересно что-то еще - вэлкам в коменты =).
