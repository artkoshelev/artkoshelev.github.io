---
layout: post
title: Пробуем Gradle
category: development, tools, automation
---

Всем привет, мои зиминие каникулы давно закончились и я снова на связи =)

Попробовал на прошлой неделе [gradle](http://www.gradle.org/) в одном из своих проектов, хочу рассказать о том что получилось. Бонусом ты увидишь как описать в gradle сборку простого проекта, сможешь попробовать и решить, надо ли оно тебе.

Итак, если у тебя проект собирается maven'ом, то для начала достаточно выполнить `gradle init` в корне проекта. Эта команда сгенерирует **базовую** конфигурацию на основе pom.xml. Вот такой build.gradle получился у меня:

{% highlight groovy %}
apply plugin: 'java'
apply plugin: 'maven'

group = '**.***.*********'
version = '2.0-GRADLE-SNAPSHOT'

description = '******* **** ******'

sourceCompatibility = 1.7
targetCompatibility = 1.7

repositories {
     maven { url "http://*****.******.***/*****-******/*******/************/*******************" }
     maven { url "http://*****.******.***/*****-******/*******/************/**********" }
     maven { url "http://*****.******.***/*****-******/*******/******/******" }
     maven { url "http://repo.maven.apache.org/maven2" }
}

dependencies {
    compile group: '**.******.*********', name: '*****-******', version:'1.0-SNAPSHOT'
    compile group: 'junit', name: 'junit', version:'4.11'
    compile group: 'org.seleniumhq.selenium', name: 'selenium-java', version:'2.35.0'
    compile group: '**.******.*********', name: '**********', version:'1.0-SNAPSHOT'
    compile group: 'ru.yandex.qatools.htmlelements', name: 'htmlelements-matchers', version:'1.11'
    compile(group: '**.******.***************', name: '*****-*****', version:'1.3.3-1.7') {
		exclude(module: 'slf4j-jdk14')
		exclude(module: 'browsermob-proxy')
    }
    compile(group: '**.******.**.********.****', name: '****-******', version:'1.0-SNAPSHOT') {
		exclude(module: 'selenium-java')
    }
    compile(group: '**.******.*********', name: '*********-******', version:'1.0-SNAPSHOT') {
		exclude(module: '*')
    }
    compile group: 'com.google.code.gson', name: 'gson', version:'2.3-SNAPSHOT'
}
{% endhighlight %}

Сначала идёт секция плагинов, тут всё понятно - нужна java для работы с java-кодом и maven для работы с maven-артефактами. Дальше идёт описание самого проекта - группа, версия ... стоп, а где имя? А имя зачем-то вынесено в отдельную настройку в файле settings.gradle:

{% highlight groovy %}
rootProject.name = '************'
{% endhighlight %}

Далее идут настройки java-компилятора и секция, описывающая maven-репозитории. Тут ты видишь внутренние репозитории, которые gradle вытащил из maven'овского settings.xml и дефолтный паблик-репо. С секцией зависимостей (dependecies) вроде всё тоже просто.

Все артефакты по дефолту будут собираться в папку build, чтобы поменять это на стандартный для maven'a target нужно добавить строчку:

{% highlight groovy %}
buildDir = 'target'
{% endhighlight %}

Такой конфиг позволит тебе собрать проект, выполнив `gradle build`. Чтобы задеплоить собранный артефакт во внутренний репозиторий нужно дописать немного кода на groovy (да-да, скрипты сборки gradle - это код на groovy). Напишем таск uploadArchives:

{% highlight groovy %}
uploadArchives {
	repositories {
		mavenDeployer {
			repository(url: "http://*****.******.***/*****/*******/************/*******************") {
				authentication(userName: '**************', password: '**************')
			}
			
			pom.project {
				packaging 'jar'
				name '******* **** ******'
				url 'http://www.yandex.ru'
				
				scm {
					url '***@******.***********.**:******/***********.***'
					connection 'scm:git:***@******.***********.**:******/***********.***'
				}
				
				issueManagement {
					system '********* ****'
					url 'https://*****.***********.**/******/**********'
				}
				
				ciManagement {
					system 'Jenkins CI'
					url 'https://*******.***********.**/****/******/***/***********************/'
				}
				
				developers {
					developer {
						id '***********'
						roles {role 'developer'}
					}
					developer {
						id '****'
						roles {role 'developer'}
					}
					developer {
						id '*******'
						roles {role 'developer'}
					}
				}
			}
		}
	}
}
{% endhighlight %}

Сначала мы указываем куда аплоадить артефакты, а так же логин и пароль (для maven-проектов эта информация обычно хранится в settings.xml т.к. она общая для всех). Потом заполняем данными из нашего pom.xml соответствующие поля объекта pom.project.

Теперь, выполнив в консоли `gradle uploadArchives`, ты можешь загрузить артефакт со скомпиленым кодом в репозиторий. Обычно мы аттачим ко всем внутренним артефактам исходники с помощью maven-source плагина. В gradle это делается созданием дополнительного архива с исходниками и добавлением его в набор архивов для загрузки.

{% highlight groovy %}
task sourcesJar(type: Jar, dependsOn:classes) {
	classifier = 'sources'
	from sourceSets.main.allSource
}

artifacts {
	archives sourcesJar
}
{% endhighlight %}

Фух, вроде всё. Что мы имеем в сухом остатке? В статьях о gradle обычно воспевают следующее:
 
 * **Код на groovy более понятен чем xml, конфиги получаются короче.** Если посмотреть на получившийся конфиг, видно что он не короче. Помимо всего, что описано в pom'e, пришлось еще добавить настройки maven'a из settings.xml. Про понятность можно долго спорить, имхо разницы особой нет. Общие для нескольких проектов куски (например, описание репозиториев) можно оформить отдельным плагином, но это не сильно сократит описание.
 
 * **В gradle инкрементальная сборка - будут собранны только те артефакты, которые зависят от изменившихся исходных файлов.** Тут не поспоришь - сборка действительно будет проходить быстрее, особенно в больших проектах.
  
 * **Можно по-разному добавлять зависимости, и вообще возможности сборки ограничиваются только возможностями Groovy, т.е. фактически не ограничены.** Хммм, приятно конечно осозновать что можно делать что захочешь, но я с большим подозрением отношусь к ситуациям "когда хочется странного". Большинство задач сборки стандартны и решаются стандартными средствами для которых уже есть maven-плагины.

В целом инструмент интересный, но я не вижу в нём смысла в проектах по атвоматизации.