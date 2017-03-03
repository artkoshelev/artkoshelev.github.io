---
layout: post
title: Экспериментируем аккуратно - профили в maven
---
 
Как изменить сборку maven-проекта не взорвав при этом общий pipeline?
Для этой задачи хорошо подходит механизм профилей в maven.

Давай по шагам:
 1. добавляешь новый профиль в pom.xml
 2. тестируешь свои изменения с новым профилем, мёржишься в мастер
 3. запускаешь альтернативную сборку с новым профилем
 4. делаешь профиль активным по-умолчанию

Я покажу как это работает на примере с Allure.
 
Добавляешь новый профиль в pom.xml
----------------------------------
 
Для этого в pom.xml есть секция `profiles`:
 
{% highlight xml %}
<profiles>
    <profile>
        <id>allure</id>
        <activation>
            <activeByDefault>false</activeByDefault>
        </activation>
        <properties>
            <aspectj.version>1.7.4</aspectj.version>
            <allure.version>1.5.2</allure.version>
        </properties>
        <dependencies>
            <dependency>
                <groupId>ru.yandex.qatools.allure</groupId>
                <artifactId>allure-junit-adaptor</artifactId>
                <version>${allure.version}</version>
            </dependency>
        </dependencies>
        <build>
            <plugins>
                <plugin>
                    <groupId>org.apache.maven.plugins</groupId>
                    <artifactId>maven-failsafe-plugin</artifactId>
                    <configuration>
                        <argLine>
                            -javaagent:${settings.localRepository}/org/aspectj/aspectjweaver/${aspectj.version}/aspectjweaver-${aspectj.version}.jar
                        </argLine>
                        <properties>
                            <property>
                                <name>listener</name>
                                <value>ru.yandex.qatools.allure.junit.AllureRunListener</value>
                            </property>
                        </properties>
                    </configuration>
                    <dependencies>
                        <dependency>
                            <groupId>org.aspectj</groupId>
                            <artifactId>aspectjweaver</artifactId>
                            <version>${aspectj.version}</version>
                        </dependency>
                    </dependencies>
                </plugin>
                <plugin>
                    <groupId>ru.yandex.qatools.allure</groupId>
                    <artifactId>allure-maven-plugin</artifactId>
                    <version>2.5</version>
                </plugin>
            </plugins>
        </build>
    </profile>
</profiles>
{% endhighlight %}

У каждого профиля есть **id** - он нужен чтобы профиль можно было активировать по имени. Дальше идёт секция **activation**.
`activeByDefault` ставим `false` - мы пока не хотим включать этот профиль всегда. Здесь же можно задать условия
 активации профиля, например, при определённом значении переменной окружения. На мой взгляд такие триггеры опасны,
 потому что не прописаны явно - и декларативная философия maven меня поддерживает :).
 
Дальше идёт конфигурация полностью аналогичная структуре самого pom.xml - свойства (properties), зависимости, плагины участвующие в сборке и т.п.
В результате получается альтернативная конфигурация проекта внутри самого проекта.

Тестируешь свои изменения с новым профилем
------------------------------------------

Давай посмотрим, что у нас получилось. Чтобы активировать новый профиль, укажи его **id** после флага `-P`:

{% highlight bash %}
mvn test allure:report -Pallure
{% endhighlight %}

Можно спокойно мёржить свои изменения в master - дефолтную сборку наш профиль никак не затронет.

Запускаешь альтернативную сборку с новым профилем
-------------------------------------------------

Итак, твои изменения в master'e, можно пробовать включать их. Тут можно пойти разными путями. Безопасный - склонировать
текущую сборку в CI и в новой сборке активировать новый профиль. Или опасный - включить новый профиль прям в основной сборке.
Если что-то пойдёт не так (обычно так и бывает), его можно так же быстро выключить :).

Делаешь профиль активным по-умолчанию
-------------------------------------

Меняем `activeByDefault` на `true`, мёржим, готово!
