---
layout: post
title: Измерение покрытия java-бекенда*
category: development, automation, coverage
image: 
  feature: 2014-11-30.jpg
---

Зачем измерять?
===============
Покрытие кода - интересная метрика. С одной стороны, высокое покрытие не гарантирует отсутствие дефектов. С другой стороны, низкое покрытие не обязательно означает бажный код. Считать эту метрику - не сложно (достаточно однажды настроить автоматику), а полученные данные - полезны (или как минимум инетерсны). Какую часть кода ты задействуешь, тестируя приложение? Насколько расширяет покрытие новый тест? Нет ли у тебя "бесполезных" тестов? Какая функциональность вообще не тестируется? Загляни в покрытие - и можешь сильно удивиться =).

Чем измерять?
=============
Для java существуют несколько инструментов. Наиболее популярные - [clover](https://www.atlassian.com/software/clover/overview), [cobertura](http://cobertura.github.io/cobertura/), [emma](http://emma.sourceforge.net/index.html) и [jacoco](http://www.eclemma.org/jacoco/). Платный clover сразу в топку, cobbertura не умеет инструментировать на лету, а у emma нет интеграции с maven. Выбор редакции - jacoco =).

Как измерять?
=============
Для сбора метрик тебе нужно запустить приложение, указав в качестве java-агента jacoco.

{% highlight sh %}
-javaagent:/usr/lib/jacocoagent.jar=
    destfile=/var/log/jacoco.exec,
    includes=ru.yandex.*,
    classdumpdir=/var/log/classdump/
{% endhighlight %}

Здесь `destfile` - сам файл с метриками. `includes` - пакеты, покрытие которых нужно измерять (нам ведь интересен только наш код, а не всё вместе с системными библиотеками). Последний параметр - `clussdumpdir` - указывает папку для дампа использованных классов. Некоторые фреймворки в рантайме модифицируют уже скомпилированные классы, поэтому для корректного измерения покрытия тебе нужны именно они, а не те, которые получаются на выходе компилятора java.

После прогона тестов и завершения приложения нужно забрать сгенерированные файлы и скормить их репортеру. Конфигурация ant-таргета для репортера:

{% highlight xml %}
<target name="coverage" >
  <jacoco:report>                    
    <executiondata>
      <file file="target/jacoco.exec"/>
    </executiondata>
                        
    <structure name="MyApp">
      <classfiles>
        <fileset dir="target/classes"/>
      </classfiles>
      <sourcefiles encoding="UTF-8">
        <fileset dir="src/main/java"/>
      </sourcefiles>
    </structure>
                        
    <html destdir="report/jacoco"/>                
  </jacoco:report>
</target>
{% endhighlight %}

В таргете `coverage` вызываем задачу `jacoco:report` со следующими настройками внутри:
 
 * **executiondata** - файл с информацией о покрытии
 * **classfiles** - классы, которые мы надампили
 * **sourcefiles** - папка с исходниками приложения
 * **html** - куда класть html-ку с отчётом

Покрытие готово!

*технология применима к любым java-приложениями, я это делал для java-бекенда.