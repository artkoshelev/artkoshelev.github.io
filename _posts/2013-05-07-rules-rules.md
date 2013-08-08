---
layout: post
title: Рулы рулят!
category: test automation, xunit, development, rules
---

Почти во всех статьях о написании приёмочных тестов с использованием junit, встречается упоминание аннотаций @Before, @BeforeClass, @After и @AfterClass. Использование отдельных методов с этими аннотациями для инициализации тестов и очистки окружения после их выполнения - практика хорошая, но уже устаревшая.

Рулы появились в junit достаточно давно (начиная с версии 4.7) и представляют собой эволюцию setup-teardown методов. Рула представляет собой класс, который следит за выполнением теста и может реагировать на изменение его состояния - старт, окончание, успешное и неуспешное прохождение и даже пропуск теста. Для каждого из состояний у рулы есть соответствующий метод:

{% highlight java %}
public class LoginRule extends TestWatcher {
    @Override
    protected void starting(Description description) {
        // Doing some stuff to login user
    }

    @Override
    protected void finished(Description description) {
        // When test finished, we can logout user
    }

    @Override
    protected void succeeded(Description description) {
        // Celebrate the succeeded test
    }

    @Override
    protected void failed(Throwable e, Description description) {
        // Invoked when a test fails
    }

    @Override
    protected void skipped(AssumptionViolatedException e, Description description) {
        // Invoked when a test is skipped due to a failed assumtion
    }
}
{% endhighlight %}

Имея такой набор, мы можем полностью контролировать состояние теста. Например, выполнять очистку окружения только в случае неудачного прохождения (такого удобства при использовании старых setup-teardown методов ты не добъёшься без костылей). Естесственно, рулы позволяют очень удобно и прозрачно переиспользовать код - достаточно добавить соответствующую рулу в твой тестовый класс:

{% highlight java %}
public class PersonalInfo {
    public static WebDriver driver = new FirefoxDriver();

    @ClassRule
    // рула, которая будет следить за webdriver'ом
    public statuc ManageDriverRule mdr = new ManageDriverRule(driver);

    @Rule
    // рула, которая будет выполнять авторизацию
    public LoginRule login = new LoginRule(driver);

    @Test
    public void userShouldSeePersonalInfoAfterLogin() {
        user.shouldSeeOwnName();
        user.shouldSeeOwnAge();
        user.ShouldSeeOwnPhone();
    }
}
{% endhighlight %}

Аннотации @Rule и @ClassRule определяют, будет ли рула применена для каждого теста в классе или один раз для всего класса (по аналогии с @Before и @BeforeClass). А используя класс RuleChain, ты можешь выстраивать цепочки рул в нужной тебе последовательности.