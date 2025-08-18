#### Основные понятия Spring

**Инверсия управления (Inversion of Control)** - это принцип, при котором фреймворк вызывает пользовательский код. Это отличается от случая с библиотеками, потому что пользовательский код вызывает код библиотеки.

**Внедрение зависимостей (Dependency Injection)** - это шаблон проектирования, в котором объект получает объекты, от которых он зависит. Это отделяет создание объектов от их использования.

**IoC Контейнер (IoC Container)** - это реализация IoC и DI. Контейнер IoC создает и управляет bean-компонентами на основе мета-информации. Он также может решать и предоставлять зависимости для создаваемых им объектов.

**BeanDefinition** - описывает bean-компоненты. Создается на основе разобранной мета-информации.

**BeanFactory** - это интерфейс который создает и предоставляет bean-компоненты на основе BeanDefinition-ов. Он является ядром ApplicationContext.

**ApplicationContext** - это центральный интерфейс который предоставляет следующий список возможностей:
- возможности BeanFactory
- загрузка ресурсов
- публикация событий
- интернационализация
- автоматическая регистрация BeanPostProcessor и BeanFactoryPostProcessor

**BeanFactoryPostProcessor** - это интерфейс, который позволяет настраивать определения bean-компонентов контекста приложения. Он создается и запускается перед BeanPostProcessor.

**BeanPostProcessor** - это интерфейс для обеспечения интеграции кастомной логики создания экземпляров, разрешения зависимостей и т. д. Каждый компонент, созданный BeanFactory, проходит через каждый зарегистрированный BeanPostProcessor.

**ApplicationContextEvent** - основной класс для событий, возникающих в процессе жизненного цикла ApplicationContext. Его подклассы:
- ContextRefreshedEvent - публикуется автоматически после поднятия контекста
- ContextStartedEvent - публикуется методом ApplicationContext#start
- ContextStoppedEvent - публикуется методом ApplicationContext#stop
- ContextClosedEvent - публикуется автоматически перед закрытием контекста

**ApplicationListener** - интерфейс который позволяет обрабатывать ApplicationEvent события. Можно использовать аннотацию @EventListener вместо интерфейса.

**Lifecycle** - интерфейс похожий на ApplicationListener, но в нем определено 2 метода, которые срабатывают во время запуска (start) и остановки (stop) контекста.

**SmartLifecycle** - это расширение Lifecycle интерфейса. Отличие в том, что он срабатывает во время обновления (refresh) и закрытия (close) контекста.

#### Self injection
```java
@Component
public class MyBean {
    @Autowired
    private MyBean self;

    public void doSomething() {
        // use self reference here
    }
}
```

```java
@Component
public class MyBean {
    private MyBean self;

    @Autowired
    public MyBean(MyBean self) {
        this.self = self;
    }

    // ...
}
```
```java
@Component
public class MyBean implements ApplicationContextAware {
    private ApplicationContext context;

    @Override
    public void setApplicationContext(ApplicationContext context) throws BeansException {
        this.context = context;
    }

    public void doSomething() {
        MyBean self = context.getBean(MyBean.class);
        // ...
    }
}
```

