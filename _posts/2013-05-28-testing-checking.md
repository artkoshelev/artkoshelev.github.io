---
layout: post
title: Поток сознания на тему тестирования
category: qa, testing
---

Сегодня не будет никаких технических штук =). Меня сильно зацепила одна статейка, о ней и хочу поговорить.

Тестирование – относительно молодая отрасль. У нас еще толком не устаканились не только процессы и методики, но даже терминология. Из-за этого под тестированием разные люди понимают иногда совершенно разные вещи. Небезызвестный Джеймс Бах предложил внести немного порядка и разделить понятия testing и checking. Вот какие определения он даёт:

**Тестирование (testing)** – процесс оценки продукта путём использования, наблюдения, изучения, экспериментирования и моделирования.

**Проверка (checking)** – процесс оценки продукта путём выполнения некоторого алгоритма на особым образом подготовленном окружении.

Проверки так или иначе всегда включены в тестирование. Даже если ты занимаешься исследовательским тестированием, ты всё равно в ходе исследования отметишь про себя (проверишь), что вёрстка не поехала, а поле пароля скрывает его за звёздочками.

Введение этих терминов и заставило меня задуматься: получается что всё, относящееся к checking’y, можно автоматизировать. Что-то проще, что-то сложнее, но всё же можно – ведь по определению там присутствует некий алгоритм. Очевидно, что объем формальных проверок может быть разным. Характер этих проверок тоже может быть разным. Таким образом мы можем формировать и позиционировать область проверок “внутри” пространства тестирования.

И тут в голову приходит интересный вопрос: а как бы сформировать такую небольшую (скажем, 20%) область проверок, которая будет отлавливать, большую (скажем, 80%) долю багов? А потом, естесственно, автоматизировать =)