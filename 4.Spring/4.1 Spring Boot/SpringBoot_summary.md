### Общие вопросы по SpringBoot

#### Прогревание кеша
Иногда необходимо убрать «холодный старт». 

Идея простая: не лезьте напрямую в CacheManager. Прогревайте через те же @Cacheable методы, которые используются в рантайме - так вы не обходите unless, ключи и TTL провайдера.
**Минимальная реализация (Boot 3+, Java 17/21)**
```java

// Сервис с кэшем
@Service
public class CatalogService {
  @Cacheable(cacheNames = "products", key = "#id", unless = "#result == null")
  public Product getById(Long id) {
    // дорогой вызов в БД/REST
  }
}

// Прогрев после полной готовности приложения
@Component
@RequiredArgsConstructor
public class CacheWarmup {

  private final CatalogService catalog;

  @EventListener(org.springframework.boot.context.event.ApplicationReadyEvent.class)
  public void warm() {
    if (!isEnabled()) return;

    var hotIds = loadHotProductIds();      // топ-ключи из БД/конфига
    try (var exec = java.util.concurrent.Executors.newVirtualThreadPerTaskExecutor()) {
      // ограничьте параллелизм при необходимости (Semaphore)
      hotIds.stream()
            .map(id -> java.util.concurrent.CompletableFuture.supplyAsync(() -> catalog.getById(id), exec))
            .forEach(java.util.concurrent.CompletableFuture::join);
    }
  }

  private boolean isEnabled() { return true; } // читайте из настроек/профиля
  private List<Long> loadHotProductIds() { return List.of(1L,2L,3L,4L,5L); }
}

```
application.yml:
``` yaml
spring:
  cache:
    type: caffeine
    cache-names: products
    caffeine:
      spec: maximumSize=10000,expireAfterWrite=1h,recordStats
# Быстрый прогрев без блокировки потоков ОС (Java 21+)
spring.threads.virtual.enabled: true
```
**Где брать «горячие» ключи:**
топ-N по обращениями за последние 24–72 часа (лог/метрики);
статические справочники (страны, тарифы);
ключи, которые дергают критичные эндпоинты / главная страница.

**Важные нюансы:**
- Не блокируйте старт сервиса надолго. Делайте прогрев асинхронно и с лимитом параллельных задач (Semaphore/FixedThreadPool).
- Падения прогрева ≠ падения приложения. Оберните в try/catch, логируйте, но не валите контекст.
- Health/Readiness. Если нужен «строгий» запуск, проверяйте готовность только по минимальному набору ключей, а остальное догревайте фоном.
- Кластеры. В Redis/общем кеше греть можно с одного инстанса (feature-flag), чтобы не дублировать трафик. В локальных (Caffeine) - грейте на каждом.
- Обновление после прогрева. Для данных, которые быстро стареют, добавьте @Scheduled обновление (@CacheEvict/@CachePut) или фоновые рефреши.

#### @SpringBootApplication

Чтобы понять, что вызывает @SpringBootApplication и как работает, достаточно посмотреть в доку или зайти в аннотацию прям из IDEA:
```java
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(excludeFilters = { @Filter(type = FilterType.CUSTOM, classes = TypeExcludeFilter.class),
		@Filter(type = FilterType.CUSTOM, classes = AutoConfigurationExcludeFilter.class) })
public @interface SpringBootApplication
```

**Как Spring Boot поднимает контекст:**
1. Environment — собирает свойства и профили.
2. ApplicationContext — создаёт и конфигурирует контейнер бинов.
3. BeanFactory — регистрирует все бины и зависимости.
4. Refresh — инициализация бинов и вызов lifecycle‑методов.
5. Listeners — запускаются события приложения (ApplicationReadyEvent и др.).
6. Embedded server — если это веб‑приложение, запускается встроенный сервер (Tomcat/Jetty).
   
То есть Boot проходит цепочку: конфигурация → создание бинов → инициализация → события → запуск веб‑сервера.