---
layout: post
title: Релизы HtmlElements и Jenkins Jira plugin
category: development
image: 
  feature: 2014-12-31.jpg
---

Половина народа в отпусках, другая половина выбирает подарки, а я собираю предновогодние релизы =).

Первый на очереди - [htmlelements](https://github.com/yandex-qatools/htmlelements) 1.13. Отмечу долгожданное расширение аннотации `@FindBy` на классы. Эти изменения появились в selenium начиная с версии 2.42 и теперь мы можем не использовать искусственно введёную аннотацию `@Block`. В текущей версии я пометил её `@deprecated`, с версии 1.14 удалим её в пень. Так же в релиз вошла пара фиксов. [Теперь](https://github.com/yandex-qatools/htmlelements/pull/60) типизированные и обычные элементы могут использовать общие матчеры, а [форма](https://github.com/yandex-qatools/htmlelements/issues/65) очищает свои поля перед заполнением. Спасибо [Ilya Murzinov](https://github.com/ilya-murzinov), [Alexander Obuhovich](https://github.com/aik099), [Alexander Kedrik](https://github.com/alkedr) и [Kirill Merkushev](https://github.com/lanwen) за участие!

Вторым идёт релиз 1.40 [Jira-плагина](https://wiki.jenkins-ci.org/display/JENKINS/JIRA+Plugin) для Jenkins. В этой версии пофиксили баг с SCM URL'ами в вики-разметке jira-тикетов, добавили статус тикетов в Release Notes и оформили релиз jira-версии пост-билд шагом. Пора вписывать себя мэйнтейнером =).

За сим удаляюсь на традиционные каникулы до конца января, желаю тебе отлично потусить на выходных!