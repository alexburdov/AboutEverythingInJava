### Наследование в Hibernate

Стратегии
- Использовать одну таблицу для каждого класса и полиморфное поведение по умолчанию.
```java
@MappedSuperclass
public abstract class BillingDetails {
    @NotNull
    protected String owner;
// ...
}

@Entity
@AttributeOverride(
    name = "owner",
    column = @Column(name = "CC_OWNER", nullable = false))
public class CreditCard extends BillingDetails {
    @Id
    @GeneratedValue(generator = Constants.ID_GENERATOR)
    protected Long id;
    
```

- Одна таблица для каждого конкретного класса, с полным исключением полиморфизма и отношений наследования из схемы SQL (для полиморфного поведения во время выполнения будут использоваться UNION-запросы)
```java
@Entity
@Inheritance(strategy = InheritanceType.TABLE_PER_CLASS)
public abstract class BillingDetails {
    @Id
    @GeneratedValue(generator = Constants.ID_GENERATOR)
    protected Long id;
    
    @NotNull
    protected String owner;
    // ...
}
```

- Единая таблица для всей иерархии классов. Возможна только за счет денормализации схемы SQL. Определять суперкласс и подклассы будет возможно посредством различия строк.
```java
@Entity
@Inheritance(strategy = InheritanceType.SINGLE_TABLE)
@DiscriminatorColumn(name = "BD_TYPE")
public abstract class BillingDetails {
    @Id
    @GeneratedValue(generator = Constants.ID_GENERATOR)
    protected Long id;
    
    @NotNull Hibernate игнорирует эту аннотацию при генерации схемы!
    @Column(nullable = false)
    protected String owner;
    // ...
}

@Entity
@Inheritance(strategy = InheritanceType.SINGLE_TABLE)
@org.hibernate.annotations.DiscriminatorFormula(
"case when CARDNUMBER is not null then ‘CC’ else ‘BA’ end"
)
public abstract class BillingDetails {
// ...
}

@Entity
@DiscriminatorValue("CC")
public class CreditCard extends BillingDetails {

```

-  Одна таблица для каждого подкласса, где отношение “is a” представлено в виде «has a», т.е. – связь по внешнему ключу с использованием JOIN.
```java
@Entity
@Inheritance(strategy = InheritanceType.JOINED)
public abstract class BillingDetails {
    @Id
    @GeneratedValue(generator = Constants.ID_GENERATOR)
    protected Long id;
    
    @NotNull
    protected String owner;
    // ...
}

@Entity
public class BankAccount extends BillingDetails {
    @NotNull
    protected String account;
    
    @NotNull
    protected String bankname;
    
    @NotNull
    protected String swift;
    // ...
}

@Entity
@PrimaryKeyJoinColumn(name = "CREDITCARD_ID")
public class CreditCard extends BillingDetails {
    @NotNull
    protected String cardNumber;
    
    @NotNull
    protected String expMonth;
    
    @NotNull
    protected String expYear;
    // ...
}

```

#### Выбор стратегии
- Стратегию №2 (TABLE_PER_CLASS на основе UNION), если полиморфные запросы и ассоциации не требуются. Если вы редко выполняете (или не выполняете вообще) «select bd from BillingDetails bd», и у вас нет классов, ссылающихся на BillingDetails, этот вариант будет лучшим (поскольку возможность добавления оптимизированных полиморфных запросов и ассоциаций сохранится).
- Стратегию №3 (SINGLE_TABLE) стоит использовать:
    - Только для простых задач. В ситуациях, когда нормализация и ограничение NOT NULL являются критическими – следует отдать предпочтение стратегии №4 (JOINED). Имеет смысл задуматься, не стоит ли в данном случае вообще отказаться от наследования и заменить его делегированием
    - Если требуются полиморфные запросы и ассоциации, а также динамическое определение конкретного класса во время выполнения; при этом подклассы объявляют относительно мало новых полей и основная разница с суперклассом заключается в поведении.Ну и вдобавок к этому, Вам предстоит серьезный разговор с администратором БД.
- Стратегия №4 (JOINED) подойдет в случаях, когда требуются полиморфные запросы и ассоциации, но подклассы объявляют относительно много новых полей.
 
Здесь стоит оговориться: решение между JOINED и TABLE_PER_CLASS требует оценки планов выполнения запросов на реальных данных, поскольку ширина и глубина иерархии наследования могут сделать стоимость соединений (и, как следствие, производительность) неприемлемыми.

Отдельно стоит принять во внимание, что аннотации наследования невозможно применить к интерфейсам.

  
#### Полиморфные ассоциации

- OneToOne - один к одному
- OneToMany - один ко многим
- ManyToOne - многие к одному
- ManyToMany - многие ко многим
