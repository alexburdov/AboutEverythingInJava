### Понимание proxy в spring

Есть примера кода, в котором кэш не работает, как сделать так, чтобы он заработал:

```java
@Service
@EnableCaching
public class CacheClass {
    
    @SneakyThrows
    @PostConstruct
    public void init() {
        System.out.println(test(1));
        Thread.sleep(1000);
        System.out.println(test(2));
        Thread.sleep(1000);
        System.out.println(test(1));
    }

    @Cacheable(cacheNames = "test", key = "#integer")
    public String test(int integer) {
        return LocalDateTime.now().toString();
    }
}
```

Здесь можно сразу предложить несколько вариантов по аналогии с транзакциями
1. Сделать self inject и вызвать метод через него
2. Использовать applicationContext.getBean(TestClass.class);

Для @Transaction это было 100% сработало бы, но в Cacheable есть подводный камень.
Для @Cacheable ситуация ещё хуже: кеш-аспект инициализируется позднее, поэтому **вызов @Cacheable из @PostConstruct обычно не срабатывает**
Тут вы можете применить ApplicationRunner или CommandLineRunner, которые смогу помочь, просто заимплементить его: 
```java
@Service
@EnableCaching
public class CacheClass implements ApplicationRunner {

    @Autowired
    @Lazy
    private CacheClass self;
    
    @Override
    public void run(ApplicationArguments args) throws Exception {
        System.out.println(self.test(1));
        Thread.sleep(1000);
        System.out.println(self.test(2));
        Thread.sleep(1000);
        System.out.println(self.test(1));
    }

    @Cacheable(cacheNames = "test", key = "#integer")
    public String test(int integer) {
        return LocalDateTime.now().toString();
    }
}
```
