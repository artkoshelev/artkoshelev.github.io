---
layout: post
title: Если сборке нужен мастер
category: development, automation
image: 
  feature: 2014-12-08.jpg
---

Некоторые тулы для сборки (например, [git-buildpackage](https://github.com/agx/git-buildpackage) или [maven-release-plguin](http://maven.apache.org/maven-release/maven-release-plugin/)) не могут собрать релиз вне мастера. Git-плагин для jenkins по умолчанию чекаутит код как раз без создания ветки. Это можно увидеть, если зайти на билд-агент и выполнить `git status` в сборочной директории:

{% highlight sh %}
~/jobs/asdf/workspace$ git status
# Not currently on any branch.
{% endhighlight %}

Git-buildpackage при этом страшно ругается:

{% highlight sh %}
You are not on branch 'master' but on '(no branch)'
Use --git-ignore-new to ignore or --git-debian-branch to set the branch name.
{% endhighlight %}

Решение проблемы довольно простое - указать в настройках git-плагина "чекаут в локальный бранч":

<img src="/images/2014-12-08-2.jpg"/>