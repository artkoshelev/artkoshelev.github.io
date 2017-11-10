---
layout: post
title: Покрытие кода в пул-реквестах
---
 
Я тут запилил [плагин для bitbucket-server](https://github.com/artkoshelev/bitbucket-code-coverage), 
который хранит и показывает информацию о покрытии кода прямо в пул-реквестах.
Т.е. не надо больше ходить ни в какие отчёты в CI, не надо ходить в Sonar - вся информация прямо под рукой.

Процесс настройки довольно прост:
1. собрать и установить плагин (скоро будет доступен в [Atlassian Marketplace](https://marketplace.atlassian.com/))
2. запушить в REST-API данные о покрытии
3. profit!

Я долго думал, в каком формате принимать данные, но так ничего не придумал и сделал свой формат :). Maven-плагин,
который лежит в той же репе, умеет конвертить LCOV. Планирую добавить поддержку других популярных форматов (jacoco, clover).

Теперь о главном - ищу людей для бета-тестирования плагина в боевых условиях. Если вы в разработке используете bitbucket-сервер и
 генерируете информацию о покрытии в билдах - вэлкам! Если ещё не генерируете - самое время начать.
 Если твой тул не умеет LCOV - всё равно пиши, что-нибудь придумаем.