---
layout: post
title: Веб-тесты на javascript
category: automation, js, javascript,
image: 
  feature: 2014-07-22.jpg
---

Не вдаваясь в рассуждения, зачем мне вдруг понадобилось писать веб-тесты на js, расскажу как это сделать. А некоторые выводы и прочая философия будут в конце поста. Поехали!

Для начала тебе понадобится тестовый фреймворк. Из всего разнообразия, я остановил свой выбор на [mocha](http://visionmedia.github.io/mocha/) - он очень простой, там есть необходимый минимум и нет ничего лишнего. 

 1. Устанавливаешь: `npm install -g mocha`
 2. Создаёшь папочку `test`
 3. Внутри создаёшь тесты
 4. Запускаешь их все одной командой: `mocha` 

Тесты пишутся в [BDD](http://en.wikipedia.org/wiki/Behavior-driven_development)-стиле: описывается фича (фичи могут быть вложенными), внутри неё - сценарии использования:

{% highlight js %}
describe('Авторизация'), function() {
    it('Должна поддерживать oauth', function(done) {
        //...
    });
    it('Должна принимать логин-пароль', function(done) {
        //...
    });
});
{% endhighlight %}

Как это ни странно, но assert'ов в этом фреймворке тестирования нет. Поэтому для проверок придётся подключить одну из сторонних библиотек. Я использовал [chai](http://chaijs.com/). Ну и главная составляющая веб-тестов - webdriver, а точнее - его js-имплементация [webdriverjs](https://code.google.com/p/selenium/wiki/WebDriverJs). Вот как это выглядит вместе:

{% highlight js %}
var wd = require('selenium-webdriver'),
    By = wd.By,
    expect = require('chai').expect;

describe('First visit popup', function() {
  var driver;
  var url = 'http://my.testing.app.ru/';
  var popup_css = By.css('div[class*="js-popup"]');
  var cross_close_popup_css = By.css('div[class*="js-popup"] i[class*="popup__close"]');

  this.timeout(5000);

  beforeEach(function(done) {
    driver = new wd.Builder().
      withCapabilities(wd.Capabilities.phantomjs()).build();
    driver.manage().window().setSize(1024, 768).then(function() {
        driver.get(url).then(function() {
        done();
      });
    });
  });

  it('Should be shown', function(done) {
    driver.findElement(popup_css).then(function(popup) {
        expect(popup).not.to.be.undefined;
        done();
    });
  });

  it('Can be closed', function(done) {
    driver.findElement(cross_close_popup_css).then(function(popup_close) {
        popup_close.click();
        driver.wait(function() {
            return driver.findElement(popup_css).then(function(popup) {
                return popup.isDisplayed().then(function(isDisplayed) {
                    return !isDisplayed;
                })
            });
        }, 1000).then(function() {
            done();
        });
    });
  });

  afterEach(function(done) {
    driver.quit().then(function() {
      done();
    });
  });

});
{% endhighlight %}

Подобного кода web-тестов в java я не видел уже лет 5-7 =). Тут есть всё, от чего java-автоматизаторы давно ушли:
 
 1. Прямая работа с webdriver (заменилась работой с [PageObject](https://code.google.com/p/selenium/wiki/PageObjects))
 2. Локаторы элементов в константах (заменились [PageObject](https://code.google.com/p/selenium/wiki/PageObjects) и [HtmlElements](https://github.com/yandex-qatools/htmlelements))
 3. setUp и tearDown процедуры (заменились [рулами](/posts/rules-rules/) )

Теперь философия. Исследуя инструменты автоматизации тестирования javascript, я наткнулся на огромное количество велосипедов, которые являются по сути [синтаксическим сахаром](http://ru.wikipedia.org/wiki/%D1%E8%ED%F2%E0%EA%F1%E8%F7%E5%F1%EA%E8%E9_%F1%E0%F5%E0%F0) к одним и тем же библиотекам. Такое ощущение, что каждый js-разработчик считает своим долгом написать какую-нибудь opensource-библиотеку. С другой стороны, я не нашёл хоть сколько-нибудь приличных фреймворков, изолирующих работу с webdriver. Даже нормальной реализации PageObject нету. Есть, конечно, и приятные исключения. Например, [protractor](https://github.com/angular/protractor) - содержит вроде всё необходимое, но работает только с фронтом, написанном на [AngularJS](https://angularjs.org/).

В целом мне кажется, что область автоматизации веб-тестирования в javascript довольна сырая и нужны действительно серьёзные причины выбрать этот язык. 