---
layout: post
title: Загрузка 3rd-party библиотек в maven-репозиторий
category: automation, maven
---

Перевожу сборку одного проекта с ant'a на maven и столкнулся с такой проблемой: некоторых завимостей нет ни в публичном maven-репозитории, ни во внутреннем. Одно из решений - загрузить jar'ник во внутренний репо. Делается это просто:

{% highlight bash %}
mvn deploy:deploy-file -DcreateChecksum=true -Dpackaging=jar -Dversion=1.0-SNAPSHOT -Durl=http://*****.******.***/nexus/content/repositories/snapshots -DrepositoryId=snapshots -Dfile=required_library.jar -DgroupId=group.id -DartifactId=artifact-id
{% endhighlight %}

Не забываем, что если нужно залить не SNAPSHOT-версию, а релизную, то параметры url и redpositoryId должны указывать на release-ветку maven-репозитория.