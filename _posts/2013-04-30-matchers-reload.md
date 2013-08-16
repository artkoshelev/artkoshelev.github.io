---
layout: post
title: Матчеры - перезагрузка
category: test automation, xunit, development, matchers, hamcrest
---

В [прошлом посте](/posts/matchers-begining) я рассказал о том, почему и как появились матчеры, теперь пора поговорить о преимуществах их использования.

1. **Переиспользование** логики проверок. Например, ты проверяешь, что список actualList не содержит дубликатов:


    ```java
    Set objectWithoutDuplicates = new HashSet(actualList);
    assertTrue(objectWithoutDuplicates.size() == actualList.size());
    ```

    Другой разработчик, которому нужно будет сделать такую же проверку, реализует её по-другому (инфа 100% =) ). И даже если ты работаешь один, ты тоже можешь реализовать её по-другому. А если реализуешь также – то это уже копипаст и всё равно не круто. Матчеры позволяют поддерживать одну имплементацию и переиспользовать её:

    ```java
    assertThat(actualList, containsUniqueElements());
    ```
2. **Наглядность** кода. Код пишется один раз, а читается – много. Поэтому очень важно, чтобы код был понятным. Посмотри на предыдущие два примера – довольно очевидно, какой из них легче воспринимается.
3. **Комбинирование** логики проверок. Используя матчеры or, and, anyOf и другие, ты можешь как угодно комбинировать матчеры (но не стоит сильно увлекаться =)). При этом не нужно заботиться о формировании сообщения в assert’e – матчеры скомбинируют и сообщения тоже.
4. Прозрачное применение **в коллекциях**. Без единой дополнительной строчки кода ты можешь творить такие чудеса:


    ```java
    assertThat(userList, hasItem(withBirthday("01.01.1911")));
    assertThat(userList, everyItem(hasCity("Saint-Petersburg")));
    ```
5. **Сокращение** кода. Если ты используешь какой-либо степовый фрейморк (например, [2CDD’s](https://github.com/thucydides-webtests/thucydides)), то наверняка сталкивался с проблемой размножения степов – на каждую проверку нужно писать отдельный степ. Передавая объекты и матчеры в качестве параметров в степовый метод, ты можешь писать универсальные степы без ущерба читаемости кода и отчётов. Зацени:


    ```java
    user.shouldSee(searchButton, withText("I'm Feeling Lucky"));
    ```

Я бы мог продолжать ещё, но лучше попробуй сам, это реально удобно. А когда получается хороший и красивый код, это ещё и чертовски приятно ;)