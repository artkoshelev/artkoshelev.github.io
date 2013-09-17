---
layout: post
title: Jira Jenkins Plugin
category: automation, java, jenkins, jira, mockito
---

Помнишь, я [как-то писал](http://artkoshelev.github.io/posts/real-life-automation) про регулярные релизы, которые я формирую вручную каждый понедельник? В том же посте я сразу подумал про Jenkins и Jira API. Так вот, на просторах интернета был найден [Jira Plugin](https://wiki.jenkins-ci.org/display/JENKINS/JIRA+Plugin), который почти решал мою задачу. Почему почти? Потому что он умел релизить версию, но не умел создавать новую. 

За пару часов эта досадная несправедливость была исправлена, но совесть не позволяла мне отправить пулреквест без тестов =). Волевым решением заставил себя посидеть еще вечерок и написать тесты и ни капли не жалею - хорошо прокачался в использовании jira api, устройстве плагинов для Jenkins и использовании mock-фреймворка [mockito](https://code.google.com/p/mockito/).

Например, поначалу меня поставила в тупик проблема: как замокать статический вызов класса, который объявляется внутри тестируемого объекта, а не передаётся извне?

{% highlight java %}
JiraSite site = JiraSite.get(project);
{% endhighlight %}

Решение оказалось весьма простым - помещаем вызов внешнего класса в свой метод и используем уже его:

{% highlight java %}
JiraSite getSiteForProject(AbstractProject<?, ?> project) {
    return JiraSite.get(project);
}
{% endhighlight %}

Теперь можно перехватить вызов этого метода и подменить его результат:

{% highlight java %}
JiraSite site = mock(JiraSite.class);
JiraVersionCreator jvc = spy(new JiraVersionCreator(JIRA_VER, JIRA_PRJ));
doReturn(site).when(jvc).getSiteForProject((AbstractProject<?, ?>) Mockito.any());
{% endhighlight %}

Закончив с тестами, я создал pull-request и тут меня ждал приятный сюрприз. Все пулреквесты в репозитории Jenkins'a автоматически билдятся и результаты сборки и выполнения тестов автоматически публикуются комментом в пулреквест. А если тесты прошли неудачно, pull-request автоматически отклоняется. Идея мне очень понравилась и мы сразу применили её для наших [opensource-инструментов](https://github.com/yandex-qatools/). Всё что для этого нужно - поставить [Github pull request builder plugin](https://wiki.jenkins-ci.org/display/JENKINS/GitHub+pull+request+builder+plugin).

Кстати, [Кирилл Меркушев](https://twitter.com/delnariel) (воодушивившись моим примером =)) тоже добавил в jira-плагин нужную ему функциональность. Теперь можно [коментить тикеты без изменения их состояния](https://github.com/jenkinsci/jira-plugin/pull/38).

И чёрт возьми, как же приятно видеть как каждый понедельник релизы сами создаются и релизятся! =)