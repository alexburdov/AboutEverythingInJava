### Spring краткие заметки

#### @Lazy

##### Когда @Lazy использовать стоит — а когда лучше не надо

Использовать можно:
- Надо быстро разрулить циклические зависимости без большого рефакторинга.
- Есть тяжёлые бины, которые не всегда нужны.
- Временное решение до архитектурного рефакторинга.

Не стоит использовать
- Бин важен для старта.
- Бин используется в @Async, @Transactional или через EventPublisher.
- Вы не понимаете, как Spring проксирует зависимости в вашем проекте.

##### Альтернатива: ObjectFactory и Provider
**ObjectFactory**
```java
@Component
class A {
    private final ObjectFactory<B> bFactory;

    public A(ObjectFactory<B> bFactory) {
        this.bFactory = bFactory;
    }

    public void process() {
        B b = bFactory.getObject();
        b.doWork();
    }
}
```
**Provider (из javax.inject)**
```java
@Component
class A {
    private final Provider<B> bProvider;

    public A(Provider<B> bProvider) {
        this.bProvider = bProvider;
    }

    public void process() {
        bProvider.get().doWork();
    }
}
```
#### Отличия @Component, @Service, @Repository, @Controller 

- @Component — базовая аннотация; помечает любой класс как бин Spring.
- @Service — тот же @Component, но семантически для бизнес‑логики; помогает читабельности и архитектурной структуре.
- @Repository — @Component для DAO‑слоя; дополнительно перехватывает исключения базы и преобразует их в Spring DataAccessException.
- @Controller — используется для веб‑слоя в MVC‑приложениях; по умолчанию возвращает HTML/шаблоны, а не JSON.
- @RestController — это @Controller + @ResponseBody, и по умолчанию возвращает JSON

#### @ComponentScan 
ComponentScan — это аннотация, которая указывает Spring, где искать классы с аннотациями @Component, @Service, @Repository, @Controller, @RestController а также @Configuration, чтобы автоматически создать бины — включая те, что определены через методы @Bean внутри этих конфигурационных классов, и передать их под управление контейнера.

#### Bean Scopes 
- Singleton (по умолчанию) — один экземпляр на весь контейнер.
- Prototype — новый экземпляр при каждом запросе бина.
- Request / Session / Application — для веб‑приложений, создаются на один HTTP‑запрос, сессию или приложение.

Подводный камень: если внедрить Prototype внутрь Singleton, Spring создаст только один экземпляр при создании Singleton, а не новый каждый раз.

#### BeanFactoryPostProcessor и BeanPostProcessor 
- **BeanFactoryPostProcessor** — позволяет изменить метаданные бинов до их создания контейнером. Пример: PropertySourcesPlaceholderConfigurer.
- **BeanPostProcessor** — перехватывает уже созданный бин перед использованием. На этом основано проксирование, AOP и @Transactional. Методы вызываются в порядке: postProcessBeforeInitialization → инициализация → postProcessAfterInitialization.




 
    
    
    
    
    