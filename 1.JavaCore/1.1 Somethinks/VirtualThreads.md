### Виртуальные потоки (запуск)

**Включение (авто)**
```yaml
# application.yml
spring:
  threads:
    virtual:
      enabled: true  # MVC, @Async, schedulers получат VirtualThreadPerTaskExecutor
```

**Ручной контроль**
```java
@Bean
TaskExecutor vthreads() {
  return new TaskExecutorAdapter(Executors.newVirtualThreadPerTaskExecutor());
}

// Привязать к @Async по умолчанию:
@Bean(name = {TaskExecutionAutoConfiguration.APPLICATION_TASK_EXECUTOR_BEAN_NAME})
TaskExecutor appExecutor() { return vthreads(); }
```
**Tomcat под Loom (MVC)**

```yaml
server:
  tomcat:
    threads:
      max: 200    # это число платформенных (!) потоков, оставьте умеренным
```

**JDBC и пул соединений**

Виртуальные потоки не отменяют лимиты БД. Не раздувайте пул, наоборот:
```yaml
spring:
  datasource:
    hikari:
      maximum-pool-size: 20   # ориентируйтесь на лимиты БД/кейсы, а не на «тысячи» v-threads
      minimum-idle: 5
```

**Три частые ошибки**
1. ThreadLocal-засады. Виртуальные потоки дешёвые, но ThreadLocal всё так же утекает. Используйте @Transactional /Context Propagation или ScopedValue вместо самодельных ThreadLocal.
2. Блокирующие HTTP-клиенты. Для массового I/O берите HttpClient (JDK) или WebClient c ограничением пула сокетов; не плодите соединения до бесконечности — узкое место сеть/БД.
3. Метрики и трейсинг. Обновите Micrometer/Brave до версий, корректно работающих с виртуальными потоками, и включите @Observed — без этого дебаг станет болью.

**Когда включать?**
- Много блокирующего I/O (JDBC, REST к внешним сервисам) → почти всегда плюс.
- Чисто CPU-bound → выгоды мало, но и вреда нет.

Тест
```java
IntStream.range(0, 10_000).parallel()
  .forEach(i -> RestClient.create().get().uri("https://example.com").retrieve().toBodilessEntity());
```

С Virtual Threads сервер не «захлебнётся» потоками — упадёте ровно в реальные лимиты: БД, сеть, таймауты. Мониторьте их, а не «число потоков».
