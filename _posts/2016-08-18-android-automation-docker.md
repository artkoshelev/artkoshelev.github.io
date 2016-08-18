---
layout: post
title: Инфраструктура автоматизации на android - docker
category: development, android, automation, docker
---

> Внимание, хардкор! Для понимания происходящего ниже, нужно хорошо разбираться в том, как устроены и работают: selenium, selenium-hub, selenium-grid, docker, appium, adb. Я тебя предупредил.

Когда число девайсов в стенде мобильной автоматизации перевалило за 4 :-) мы стали замечать проблемы в работе [adb](https://developer.android.com/studio/command-line/adb.html). Он тупил при параллельной работе с несколькими девайсами а иногда вообще вставал колом и мы вручную его перезапускали. Мы попробовали изолировать девайсы в [docker-контейнеры](https://www.docker.com/), чтобы у каждого был свой adb, работающий в один поток. Получилось даже лучше, чем мы ожидали, и в итоге мы стали использовать контейнеры не только для железок, но и эмуляторов. 

Итак, мы запускаем appium внутри docker-контейнеров. По инстансу на каждый эмулятор или девайс. Единственный минус - приличный оверхед по диску - образ весит около 5Gb. Умножай на число подключенных девайсов :-). Зато получаешь все плюсы работы в изолированном окружении:

 * у каждого девайса свой appium
 * у каждого девайса свой adb (скорость работы с девайсами ускорилась в разы)
 * раздельные логи для каждого девайса
 * можно поднимать/гасить девайсы независимо друг от друга

Вот так выглядит наш Docker-файл для реальных девайсов:

{% highlight bash %}
FROM ubuntu:14.04

RUN apt-get update && apt-get install -y curl
RUN curl -sL https://deb.nodesource.com/setup_5.x | sudo -E bash -
RUN apt-get install -y \
    ant \
    autossh \
    g++-multilib \
    gettext-base \
    git \
    lib32z1 \
    lib32ncurses5 \
    lib32bz2-1.0 \
    maven \
    nodejs \
    openjdk-7-jdk \
    python \
    unzip \
    wget \
 && apt-get clean \
 && rm -rf /var/lib/apt/lists/*

# Main Android SDK
RUN wget -qO- "http://dl.google.com/android/android-sdk_r24.4.1-linux.tgz" | tar -zxv -C /opt/
ENV ANDROID_HOME /opt/android-sdk-linux
ENV PATH /opt/android-sdk-linux/platform-tools:/opt/android-sdk-linux/tools:$PATH
RUN echo "y" | android update sdk --filter tools --no-ui --force
RUN echo "y" | android update sdk --filter platform-tools --no-ui --force
RUN echo "y" | android update sdk --filter build-tools-23.0.0 --no-ui -a
RUN echo "y" | android update sdk --filter android-16 --no-ui -a
RUN echo "y" | android update sdk --filter android-19 --no-ui -a

RUN mkdir /opt/config
COPY files/configgen.sh /opt/config/
COPY files/id_rsa /opt/config/
COPY files/adbkey.pub /root/.android/adbkey.pub
COPY files/adbkey /root/.android/adbkey
RUN chmod +x /opt/config/configgen.sh
RUN chmod 0600 /opt/config/id_rsa

RUN cd /opt && git clone --branch master --single-branch https://github.com/yandex-qatools/appium.git
RUN cd /opt/appium && ./reset.sh --android --verbose
RUN rm -rf /opt/appium/.git

ENV HUB ***
ENV HUB_PORT 4445
ENV HUBUSER ***
ENV PORT 4726
ENV DEVICEUDID qwert

CMD /opt/config/configgen.sh > /opt/config/nodeconfig.json \
    && autossh -f -o StrictHostKeyChecking=no -N -i /opt/config/id_rsa -R $PORT:localhost:$PORT $HUBUSER@$HUB \
    && /opt/appium/bin/appium.js -p $PORT --log-timestamp --session-override --udid $DEVICEUDID \
    --nodeconfig /opt/config/nodeconfig.json
{% endhighlight %} 

Пробежимся по нетривиальным моментам:

 * `RUN curl -sL https://deb.nodesource.com/setup_5.x | sudo -E bash -` - костыль для установки nodejs на ubuntu, на ноде написан appium, без неё никуда
 * `RUN wget -qO- "http://dl.google.com/android/android-sdk_r24.4.1-linux.tgz" | tar -zxv -C /opt/` - качаем и распаковываем SDK
 * `RUN echo "y" | android update sdk --filter android-16 --no-ui -a` - 16-я версия SDK опять же нужна appium'у
 * `COPY files/configgen.sh /opt/config/` про этот скрипт будет чуть позже
 * `RUN cd /opt && git clone --branch master --single-branch https://github.com/yandex-qatools/appium.git` - используем наш форк appium'a
 * `autossh -f -o StrictHostKeyChecking=no -N -i /opt/config/id_rsa -R $PORT:localhost:$PORT $HUBUSER@$HUB` - порты, на которых appium-нода слушает команды, открыты внутри контейнера и не видны наружу, поэтому мы создаём [ssh-тоннель](ssh-tunnels) чтобы хаб мог общаться с appium-нодами

Как видишь, в контейнер зашит android sdk, который собственно и весит почти 5 гигов :-D Скрипт `configgen.sh` считывает параметры девайса и генерирует конфиг-файл для запуска appium. Про него, а так же про возможности фильтрации устройств в selenium-гриде - в следующей серии.

