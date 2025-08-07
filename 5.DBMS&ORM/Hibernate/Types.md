### Типы и значения в Hibernate

#### @Basic 
Имеет 2 параметра Optional и FetchType - более простая аннотация , чем @Column. Может использоваться вместе с @Column - но вопрос зачем? 

####  @Column
Более широкая аннотация, чем @Basic. Имеет связь непосредственно с аспектами уровня SQL.

#### @MappedSuperclass
Аннотация **@MappedSuperclass** в Hibernate используется для определения базового класса, чьи свойства наследуются дочерними классами, но сам базовый класс не отображается в отдельную таблицу базы данных. Это полезно для создания общей структуры, которую используют несколько сущностей, избегая дублирования кода

#### @Access
Аннотация **@Access** в Hibernate, используемая в контексте JPA (Java Persistence API), определяет способ доступа к полям сущности (entity) для операций сохранения и извлечения данных. Она указывает, будут ли данные извлекаться и сохраняться напрямую через поля (field) или через соответствующие геттеры и сеттеры (property).
- **AccessType.FIELD:** Это значение указывает, что доступ к полям сущности будет осуществляться напрямую через поля класса, а не через геттеры и сеттеры. При использовании **AccessType.FIELD**, поля класса должны быть объявлены с модификатором доступа private или protected.
- **AccessType.PROPERTY:** Это значение указывает, что доступ к полям сущности будет осуществляться через геттеры и сеттеры. При использовании **AccessType.PROPERTY**, поля класса могут иметь любой модификатор доступа.

#### noop
Служит для отображения виртуальных полей. Единственная возможность отображения такого виртуального поля – с помощью оригинального файла метаданных hbm.xml:
```xml
<hibernate-mapping>
    <class name="Item">
        <id name="id">
            ...
        </id>
        <property name="validated" column="VALIDATED" access="noop"/>
    </class>
</hibernate-mapping>
```
#### Работа с вычисляемыми полями
```java
@org.hibernate.annotations.Formula("substr(DESCRIPTION, 1, 12) || ‘...’")
protected String shortDescription;

@org.hibernate.annotations.Formula(
"(select avg(b.AMOUNT) from BID b where b.ITEM_ID = ID)")
protected BigDecimal averageBidAmount;
```

#### Преобразование значений столбцов
```java
@Column(name = "IMPERIALWEIGHT")
@org.hibernate.annotations.ColumnTransformer(
        read = "IMPERIALWEIGHT / 2.20462",
        write = "? * 2.20462"
    )
protected double metricWeight;
```
#### Значения свойств, генерируемые по умолчанию
```java
@Temporal(TemporalType.TIMESTAMP)
@Column(insertable = false, updatable = false)
@org.hibernate.annotations.Generated(org.hibernate.annotations.GenerationTime.ALWAYS)
protected Date lastModified;

@Column(insertable = false)
@org.hibernate.annotations.ColumnDefault("1.00")
@org.hibernate.annotations.Generated(org.hibernate.annotations.GenerationTime.INSERT)
protected BigDecimal initialPrice;
```

#### Свойства для представления времени
```java
@Temporal(TemporalType.TIMESTAMP)
@Column(updatable = false)
@org.hibernate.annotations.CreationTimestamp
protected Date createdOn;
```
Доступными вариантами **TemporalType** являются **DATE, TIME и TIMESTAMP**, указывающие, какую часть значения времени следует сохранять в базе данных

#### Отображение перечислений
```java
public enum AuctionType {
    HIGHEST_BID,
    LOWEST_BID,
    FIXED_PRICE
}

@NotNull
@Enumerated(EnumType.STRING) // По умолчанию – ORDINAL
protected AuctionType auctionType = AuctionType.HIGHEST_BID;
```

#### Встраиваемые классы
Вместо @Entity компонент POJO отмечен аннотацией @Embeddable. Идентификатор
отсутствует. Встретив класс в @Entity будет размещены поля из  @Embeddable.
Для явного указания можно использовать аннотацию @Embedded. Для переопределения полей 
@AttributeOverrides.
```java

@Embedded // Необязательная аннотация
@AttributeOverrides({
@AttributeOverride(name = "street",
column = @Column(name = "BILLING_STREET")),// Может быть NULL!
@AttributeOverride(name = "zipcode",
column = @Column(name = "BILLING_ZIPCODE", length = 5)),
@AttributeOverride(name = "city",
column = @Column(name = "BILLING_CITY"))
})
protected Address billingAddress;
```

#### Встроенные типы
```java
@Entity
public class Item {
    @org.hibernate.annotations.Type(type = "yes_no")
    protected boolean verified = false;
}
```


| Имя         | Тип Java                   | Тип ANSI SQL |
| ----------- | -------------------------- | ------------ |
| integer     | int, java.lang.Integer     | INTEGER      |
| long        | long, java.lang.Long       | BIGINT       |
| short       | short, java.lang.Short     | SMALLINT     |
| float       | float, java.lang.Float     | FLOAT        |
| double      | double, java.lang.Double   | DOUBLE       |
| byte        | byte, java.lang.Byte       | TINYINT      |
| boolean     | boolean, java.lang.Boolean | BOOLEAN      |
| big_decimal | java.math.BigDecimal       | NUMERIC      |
| big_integer | java.math.BigInteger       | NUMERIC      |

| Имя        | Тип Java                              | Тип ANSI SQL         |
| ---------- | ------------------------------------- | -------------------- |
| string     | java.lang.String                      | VARCHAR              |
| character  | char[], Character[], java.lang.String | CHAR                 |
| yes_no     | boolean, java.lang.Boolean            | CHAR(1), ‘Y’ или ‘N’ |
| true_false | boolean, java.lang.Boolean            | CHAR(1), ‘T’ или ‘F’ |
| class      | java.lang.Class                       | VARCHAR              |
| locale     | java.util.Locale                      | VARCHAR              |
| timezone   | java.util.TimeZone                    | VARCHAR              |
| currency   | java.util.Currency                    | VARCHAR              |

| Имя            | Тип Java                           | Тип ANSI SQL |
| -------------- | ---------------------------------- | ------------ |
| date           | java.util.Date, java.sql.Date      | DATE         |
| time           | java.util.Date, java.sql.Time      | TIME         |
| timestamp      | java.util.Date, java.sql.Timestamp | TIMESTAMP    |
| calendar       | java.util.Calendar                 | TIMESTAMP    |
| calendar_date  | java.util.Calendar                 | DATE         |
| duration       | java.time.Duration                 | BIGINT       |
| instant        | java.time.Instant                  | TIMESTAMP    |
| localdatetime  | java.time.LocalDateTime            | TIMESTAMP    |
| localdate      | java.time.LocalDate                | DATE         |
| localtime      | java.time.LocalTime                | TIME         |
| offsetdatetime | java.time.OffsetDateTime           | TIMESTAMP    |
| offsettime     | java.time.OffsetTime               | TIME         |
| zoneddatetime  | java.time.ZonedDateTime            | TIMESTAMP    |

| Имя          | Тип Java                 | Тип ANSI SQL |
| ------------ | ------------------------ | ------------ |
| binary       | byte[], java.lang.Byte[] | VARBINARY    |
| text         | java.lang.String         | CLOB         |
| clob         | java.sql.Clob            | CLOB         |
| blob         | java.sql.Blob            | BLOB         |
| serializable | java.io.Serializable     | VARBINARY    |

#### Пользовательские типы
Интерфейсы для расширения системы типов Hibernate определены в пакете
org.hibernate.usertype и включают:
- **UserType** – позволяет преобразовывать значения посредством взаимодействий с обычными классами JDBC: PreparedStatement при сохранении и ResultSet при загрузке. Реализовав этот интерфейс, вы сможете контролировать порядок кэширования значений и проверки изменений состояний
объектов в Hibernate. Адаптер для класса MonetaryAmount должен реализовать этот интерфейс;
- **CompositeUserType** – расширяет UserType, предоставляя фреймворку Hibernate больше подробностей об адаптируемом классе. Вы можете указать, что
компонент типа MonetaryAmount имеет два поля: amount и currency, – и затем
обращаться к этим полям в запросах, используя точечную нотацию, например: select avg(i.buyNowPrice.amount) from Item i;
- **ParameterizedUserType** – добавляет настройки отображений в класс адаптера. Вы должны реализовать этот интерфейс для преобразования объектов
MonetaryAmount, потому что в некоторых отображениях нужно конвертировать сумму в доллары США, а в других – в евро. Вам придется написать всего один адаптер, а затем вы сможете настраивать его поведение для отображения полей;
- **EnhancedUserType** – необязательный интерфейс для адаптеров свойствидентификаторов и селекторов. В отличие от конвертеров JPA, класс UserType в Hibernate может служить адаптером свойства сущности любого типа. Но поскольку класс MonetaryAmount не является типом свойства-идентификатора или селектора, вам этот интерфейс не понадобится;
- **UserVersionType** – необязательный интерфейс для адаптеров свойств, хранящих номера версий объектов;
- **UserCollectionType** – этот редко используемый интерфейс применяется для
реализации собственных коллекций. Его нужно реализовать при работе
с коллекциями, имеющими типы не из JDK, а также для сохранения дополнительной семантики.
```java
public class MonetaryAmountUserType implements CompositeUserType, DynamicParameterizedType {
// ...
}
```
Файл: /model/src/main/java/org/jpwh/converter/package-info.java
```java
@org.hibernate.annotations.TypeDefs({
@org.hibernate.annotations.TypeDef(
    name = "monetary_amount_usd",
    typeClass = MonetaryAmountUserType.class,
    parameters = {@Parameter(name = "convertTo", value = "USD")}
),
@org.hibernate.annotations.TypeDef(
    name = "monetary_amount_eur",
    typeClass = MonetaryAmountUserType.class,
    parameters = {@Parameter(name = "convertTo", value = "EUR")}
)
})
package org.jpwh.converter;
import org.hibernate.annotations.Parameter;
```
```java
@NotNull
@org.hibernate.annotations.Type(type = "monetary_amount_eur")
@org.hibernate.annotations.Columns(columns = {
    @Column(name = "INITIALPRICE_AMOUNT"),
    @Column(name = "INITIALPRICE_CURRENCY", length = 3)
})
protected MonetaryAmount initialPrice;
```

#### Создание собственных конвертеров JPA

```java
@Converter(autoApply = true) Автоматически применяется к полям типа MonetaryAmount
public class MonetaryAmountConverter implements AttributeConverter<MonetaryAmount, String> {
    @Override
    public String convertToDatabaseColumn(MonetaryAmount monetaryAmount) {
        return monetaryAmount.toString();
    }
    
    @Override
    public MonetaryAmount convertToEntityAttribute(String s) {
        return MonetaryAmount.fromString(s);
    }
}
```
```java
@Entity
@Table(name = "USERS")
public class User implements Serializable {
    @Convert(converter = ZipcodeConverter.class, attributeName = "zipcode")//Или «city.zipcode» для вложенных встраиваемых компонентов
    //Под одной аннотацией @Convert можно объединить преобразование нескольких атрибутов
    protected Address homeAddress;
    // ...
}
```
