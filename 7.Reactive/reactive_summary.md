##### Общая информация по реактивному програмированию

Библиотеки реактивного програмирования
Flux - типы publish

Reactive Streams API сначала появился как отдельная спецификация (org.reactivestreams), а в Java 9 — java.util.concurrent.Flow, семантически эквивалентный её интерфейсам. Он определяет:
- Publisher — источник (поставщик) данных,
- Subscriber — потребитель данных,
- Subscription — управление потоком,
- Processor — компонент, который является одновременно подписчиком и издателем (то есть потребляет элементы типа T и выпускает элементы типа R).

Project Reactor — флагманская реализация от Pivotal (создателей Spring), включающая:
- Mono представляет асинхронную последовательность с максимум одним элементом,
- Flux — последовательность от 0 до N элементов.

##### Reactive Manifesto: четыре принципа современных систем 
Reactive Manifesto — это не сухая спецификация, а, скорее, набор идей о том, как строить живые, адаптивные системы. Представьте, что вы создаёте не просто приложение, а организм, который должен спокойно переносить стресс, меняться под давлением и оставаться в форме. Именно к этому и сводятся четыре базовых принципа реактивного подхода.

**Responsive (отзывчивость) — основа пользовательского опыта**

Любая система должна отвечать быстро и предсказуемо — независимо от того, что происходит внутри. Пользователь не должен долго ожидать ответа и думать, все ли в порядке. Даже под нагрузкой система должна сохранять ощущение плавности и контроля.
```java
// Реактивный подход гарантирует ответ
public Mono<String> loadUserDataReactive(String userId) {
    return userRepository.findById(userId) // userRepository должен быть реактивный
        .timeout(Duration.ofSeconds(3))      // Гарантия максимального времени ответа
        .onErrorReturn("Пользователь не доступен"); // Всегда возвращаем результат
}
```

**Resilient (устойчивость) — искусство оставаться на плаву**

Даже лучшие системы иногда ломаются, и это нормально. Главное, чтобы поломка в одном месте не тянула за собой всё остальное. Реактивная архитектура как раз и помогает локализовать сбои, изолировать проблемы и восстанавливаться без вмешательства человека.
```java
public Mono<Order> processOrder(Order order) {
     return inventoryService.reserveItems(order) // если тут есть блокирующие вызовы то нужно добавлять .subscribeOn(Schedulers.boundedElastic()
         .transformDeferred(circuitBreaker::run) // Защита от повторяющихся сбоев
         .retryWhen(Retry.backoff(3, Duration.ofSeconds(1))) // 3 попытки с растущей задержкой
         .timeout(Duration.ofSeconds(30))
         .onErrorResume(TimeoutException.class, e -> {
             // Переход на упрощенную логику при таймауте
             return processOrderWithLimitedFunctionality(order);
         })
         .onErrorResume(ServiceUnavailableException.class, e -> {
             return processOrderWithoutReservation(order);
         })
         .onErrorReturn(createFallbackOrder(order));
 }
```

**Elastic (эластичность) — гибкость под нагрузкой**

Нагрузки растут, трафик скачет, и система должна реагировать на это сама, без паники. Эластичность — это способность приложения масштабироваться туда, где нужно, и не держать лишние ресурсы, когда всё спокойно.

```java
@Bean
 public Scheduler elasticScheduler() {
     return Schedulers.newBoundedElastic(
         10,     //  лимит расширения при пиковой нагрузке
         1000,   //  буфер для задач при превышении лимита потоков
         "elastic-pool",
         60,    // Потоки, которые простаивают более 60 секунд, автоматически удаляются
         true);

}
```
```java
public Flux<String> processBatchReactive(List<String> items) {
     return Flux.fromIterable(items)
         .flatMap(item -> 
                        Mono.fromCallable(() -> processItem(item)) // тут processItem - блокирующая операция, если тут будет реактивный метод, то subscribeOn не нужен
                                .subscribeOn(elasticScheduler())
                 // Если processItem выполняется дольше 30 секунд - операция прерывается
                 .timeout(Duration.ofSeconds(30))
                 // При ошибке будет 3 попытки с задержкой
                 .retryWhen(Retry.backoff(3, Duration.ofMillis(100)))
                 .name("processItem")
                 .metrics() // метрики нужно настраивать дополнительно
         , 
               10  // Максимум 10 одновременных операций processItem()
            )
         // Если вся обработка списка занимает больше 5 минут - прерываем
         .timeout(Duration.ofMinutes(5));
 }
```

**Message-Driven (ориентированность на сообщения) — основа коммуникации**

В реактивной архитектуре компоненты общаются через сообщения. Это снижает связанность и делает систему гибкой: один сервис может временно отвалиться, а остальные спокойно продолжат работу.

```java
// сервис для отправки заказов в брокер сообщений
@Service
public class ReactiveOrderService {
    private final StreamBridge streamBridge;

    public ReactiveOrderService(StreamBridge streamBridge) {
        this.streamBridge = streamBridge;
    }

    public Mono<Void> processOrder(Order order) {
    return Mono.fromCallable(() -> streamBridge.send("orders-out-0", order))
                  .subscribeOn(Schedulers.boundedElastic())
                  .doOnSuccess(sent -> log.info("✅ Заказ отправлен: {}", order.getId()))
                  .then();
}
// обработчик входящих сообщений
@Component
public class OrderMessageHandler {
    private final ReactiveOrderService orderService;

    public OrderMessageHandler(ReactiveOrderService orderService) {
        this.orderService = orderService;
    }

    @Bean
    public Consumer<Flux<Order>> orderProcessor() {
        return flux -> flux
            .flatMap(order -> orderService.processOrder(order));
    }
}

```

**Эти принципы не работают по отдельности:**
- Message-Driven даёт основу для Elastic масштабирования,
- Resilient помогает системе оставаться Responsive даже при сбоях,
- а Elastic характер поддерживает устойчивость, когда нагрузка скачет.
