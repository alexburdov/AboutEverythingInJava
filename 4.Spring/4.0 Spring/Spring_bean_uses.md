#### Полезности при использованием Bean

#### Как внедрить несколько Bean, которые реализуют один интерфейс

Давайте представим, что у нас есть интерфейс PaymentService, у которого есть 2 реализации, и мы хотим в сервисе прогнать сразу 2 метода оплаты:
```java
public interface PaymentService {
    void pay();
}

@Service("creditCardPayment")
public class CreditCardPaymentService implements PaymentService {
    @Override
    public void pay() {
        System.out.println("Оплата кредитной картой");
    }
}

@Service("paypalPayment")
public class PaypalPaymentService implements PaymentService {
    @Override
    public void pay() {
        System.out.println("Оплата через PayPal");
    }
}
```
**Внедрение через List<PaymentService>**

Часто используется внедрение через List<PaymentService>, чтобы получить все сервисы реализации:
```java
@Service
public class OrderService {

    private final List<PaymentService> paymentServices;

    @Autowired
    public OrderService(List<PaymentService> paymentServices) {
        this.paymentServices = paymentServices;
    }

    public void processAllPayments() {
        paymentServices.forEach(PaymentService::pay);
    }
}
```

**Внедрение через Map<String, PaymentService>**

Мы можем внедрять такие бины через Map<String, PaymentService> В этом случае ключами будут имена бинов ("creditCardPayment" и "paypalPayment"), а значениями — соответствующие реализации.

```java
@Service
public class OrderService {

    private final Map<String, PaymentService> paymentServices;

    @Autowired
    public OrderService(Map<String, PaymentService> paymentServices) {
        this.paymentServices = paymentServices;
    }

    public void processSpecificPayment(String type) {
        paymentServices.get(type).pay();
    }
}
```

#### Self injection 

**Self injection** - это когда класс внедряет сам себя через Spring-контейнер, обычно через прокси.

**Зачем нужен:**
- Чтобы вызвать собственный метод, аннотированный @Transactional или @Async, и чтобы прокси Spring корректно обработал аспект.
- Прямой вызов метода через this не проходит через прокси, поэтому аннотации не сработают.
```java
@Service
public class OrderService {

    @Autowired
    @Lazy
    private OrderService self;

    @Transactional
    public void processOrder() {
        // код
    }

    public void startProcess() {
        self.processOrder(); // прокси сработает
    }
}
```
#### Код, чтобы поиграться с @Transaction

Чтобы проверить разные способы работы @Transaction я пользовался этим кодом:

```java
@Service
public class TestClass {

    @Autowired
    @Lazy
    private TestClass self;

    @Autowired
    private ApplicationContext applicationContext;

    @PostConstruct
    public void init() {
        System.out.println("Тестируем @Transactional in @PostConstruct");

        // Тест 1: Прямой вызов (не должен работать)
        System.out.println("1. Прямой вызов:");
        try {
            directTransactionalMethod();
        } catch (Exception e) {
            System.out.println("Ошибка: " + e.getMessage());
        }

        // Тест 2: Через self + @Lazy
        System.out.println("2. self + @Lazy:");
        self.selfTransactionalMethod();

        // Тест 3: Через ApplicationContext
        System.out.println("3. ApplicationContext:");
        TestClass proxy = applicationContext.getBean(TestClass.class);
        proxy.contextTransactionalMethod();

        System.out.println("@PostConstruct выполнился");
    }

    @Transactional(propagation = Propagation.REQUIRES_NEW)
    public void directTransactionalMethod() {
        checkTransaction("directTransactionalMethod");
    }

    @Transactional(propagation = Propagation.REQUIRES_NEW)
    public void selfTransactionalMethod() {
        checkTransaction("selfTransactionalMethod");
    }

    @Transactional(propagation = Propagation.REQUIRES_NEW)
    public void contextTransactionalMethod() {
        checkTransaction("contextTransactionalMethod");
    }

    private void checkTransaction(String methodName) {
        boolean isActive = TransactionSynchronizationManager.isActualTransactionActive();
        String transactionName = TransactionSynchronizationManager.getCurrentTransactionName();

        System.out.println("   " + methodName + ":");
        System.out.println("   - Активна ли транзакция: " + isActive);
        System.out.println("   - Имя транзакции: " + transactionName);
    }
}
```

Результат:

``` text
Тестируем @Transactional in @PostConstruct
1. Прямой вызов:
   directTransactionalMethod:
   - Активна ли транзакция: false
   - Имя транзакции: null
2. self + @Lazy:
   selfTransactionalMethod:
   - Активна ли транзакция: true
   - Имя транзакции: TestClass.selfTransactionalMethod
3. ApplicationContext:
   contextTransactionalMethod:
   - Активна ли транзакция: true
   - Имя транзакции: TestClass.contextTransactionalMethod
```