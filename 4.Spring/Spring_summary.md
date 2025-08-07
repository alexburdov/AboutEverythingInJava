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



    
    
    
    
    