---
layout: post
title: Работаем с xml в java - кастомные аннотации
category: automation, xml, jaxb, maven
---

Итак, у меня был код, получающий JSON через REST-API и с помощью jaxb преобразующий его в java-объекты. И был другой код, который те же объекты засылал в другое REST-API уже с использованием [Gson](https://github.com/google/gson) (пожалуйста не спрашивай, почему в одном проекте используются две разные технологии для решения одной и той же задачи :) ).

Проблемы начались когда поля_в_json сталиНазываться чутьБолее сложно-и-разнообразно. JAXB расставляет для полей и классов аннотации `@XmlType` и `@XmlElement` чтобы матчить такие названия при преобразованиях. Gson же использует для матчинга свою аннотацию `@SerializedName`. Получается что нужно добавить эту аннотацию в генерируемые бины.

Я был уверен, что существует готовое решение этой задачи, но потратил прилично времени прежде чем нашел его. Мой спаситель - [jaxb2-basics-annotate](https://github.com/highsource/jaxb2-annotate-plugin) плагин. Первое, что нужно сделать - добавить его в зависимости твоего `wadl-client-plugin`:

{% highlight xml %}
<dependency>
    <groupId>org.jvnet.jaxb2_commons</groupId>
    <artifactId>jaxb2-basics-annotate</artifactId>
    <version>1.0.1</version>
</dependency>
{% endhighlight %}

а в конфигурацию добавить ключик:

{% highlight xml %}
<configuration>
    <xjcArguments>
        <argument>-Xannotate</argument>
    </xjcArguments>
</configuration>
{% endhighlight %}

Теперь надо расширить описание нужного поля в xsd-схеме:

{% highlight xml %}
<xsd:complexType name="complexType">
    <xsd:sequence>
        <xsd:element name="field-with-dash" type="xsd:string">
            <xsd:annotation>
                <xsd:appinfo>
                    <annox:annotate target="field">
                        @com.google.gson.annotations.SerializedName("field-with-dash")
                    </annox:annotate>
                </xsd:appinfo>
            </xsd:annotation>
        </xsd:element>
        <xsd:element name="field_with_underscore" type="xsd:string">
            <xsd:annotation>
                <xsd:appinfo>
                    <annox:annotate target="field">
                        @com.google.gson.annotations.SerializedName("field_with_underscore")
                    </annox:annotate>
                </xsd:appinfo>
            </xsd:annotation>
        </xsd:element>
    </xsd:sequence>
</xsd:complexType>
{% endhighlight %}

Аналогичным образом можно аннотировать не только поля, но и методы классов (например, обозвать их `@Deprecated` :) ) и сами классы.

Проблема автоматически сгенерированного кода в том, что он автоматически сгенерирован. Он имеет жесткую структуру, ограниченную правилами генерации. С одной стороны, можно расширять механизмы генерации, повышая гибкость, с другой стороны, есть риск получить еще один язык программирования (а смысл тогда?). Истина как всегда где-то между.