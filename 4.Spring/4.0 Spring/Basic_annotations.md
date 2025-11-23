### Основные аннотации

**@Primary** 

- Помечает бин как основной, если есть несколько кандидатов одного типа.
- Spring автоматически выберет его при инжекции, если не указан @Qualifier.

**@Qualifier**

Позволяет явно указать, какой бин использовать, даже если есть несколько кандидатов.
```java
@Service
public class OrderService {

    @Autowired
    @Qualifier("stripePaymentService")
    private PaymentService paymentService; // выберется StripePaymentService
}
```

**@Transactional** 

**@Profile**
**Profile** - аннотация для условного включения бина или конфигурации в зависимости от активного профиля приложения

**@ConditionalOnProperty**
**@ConditionalOnProperty** — аннотация Spring Boot, которая позволяет создавать бин только если задано определённое свойство в application.properties или application.yml.

**@Cacheable**
Есть такая прекрасная аннотация как @Cacheable , которая включает кэш на методе, чтобы при вызове метода с такими же параметрами, мы не выполняли его снова, а получали значение из кэша:
```java
  @Cacheable(cacheNames = "test", key = "#test")
  public String test(int test) {
      return LocalDateTime.now().toString();
  }
```

