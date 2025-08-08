### Hibernate краткие заметки

#### @Id
```java
@org.hibernate.annotations.GenericGenerator(
    name = "ID_GENERATOR",
    strategy = "enhanced-sequence", // Стратегия применения расширенной последовательности
    parameters = {
        @org.hibernate.annotations.Parameter(
            name = "sequence_name", // Имя последовательности
            value = "JPWH_SEQUENCE"
),
@org.hibernate.annotations.Parameter(
    name = "initial_value", // Начальное значение
    value = "1000"
    )
}
```

#### @GeneratedValue

- **GenerationType.AUTO** – Hibernate выбирает подходящую стратегию, исходя из диалекта SQL, используемого настроенной базой данных. Это эквивалентно @GeneratedValue() без всяких настроек;
- **GenerationType.SEQUENCE** – Hibernate ищет (и создает, если вы используете
определенные инструменты) последовательность HIBERNATE_SEQUENCE в базе
данных. Эта последовательность будет вызываться отдельно перед каждой
операцией INSERT для получения последовательных числовых значений;
- **GenerationType.IDENTITY** – Hibernate ищет (и создает в DDL-определении
таблицы) специальный столбец первичного ключа с автоматическим приращением, который сам генерирует числовое значение во время выполнения INSERT в базе данных;
- **GenerationType.TABLE** – Hibernate будет использовать дополнительную таблицу в схеме базы данных, хранящую следующее числовое значение первичного ключа: по одной строке на каждый класс сущности. Соответственно, эта таблица будет читаться и обновляться перед выполнением INSERT. По умолчанию таблица называется **HIBERNATE_SEQUENCES** и содержит столбцы **SEQUENCE_NAME** и **SEQUENCE_NEXT_HI_VALUE**. 

#### Управление именами
Hibernate узнает зарезервированные слова вашей СУБД по настроенному диалекту базы данных и может автоматически ставить кавычки вокруг таких строк во
время формирования SQL, активировать автоматическое заключение в кавычки
с помощью параметра **hibernate.auto_quote_keyword=true** в конфигурации единицы хранения. Если потребуется заключить в кавычки каждый SQL-идентификатор – создайте файл **orm.xml** и добавьте элемент **<delimited-identifiers/>** в раздел **<persistenceunit-defaults>**. В этом случае Hibernate будет повсеместно использовать идентификаторы, заключенные в кавычки.

#### Реализация соглашений об именовании
```java
public class CENamingStrategy extends org.hibernate.boot.model.naming.PhysicalNamingStrategyStandardImpl {
    @Override
    public Identifier toPhysicalTableName(Identifier name, JdbcEnvironment context) {
        return new Identifier("CE_" + name.getText(), name.isQuoted());
    }
}
```
После реализации стратегии именования ее следует активировать в persistence.
xml:
```xml
<persistence-unit>name="CaveatEmptorPU">
...
    <properties>
        <property name="hibernate.physical_naming_strategy" value="org.jpwh.shared.CENamingStrategy"/>
    </properties>
</p
```

#### Динамическое формирование SQL
```java
@Entity
@org.hibernate.annotations.DynamicInsert
@org.hibernate.annotations.DynamicUpdate
public class Item {
// ...
}
```
Активируя динамическое создание инструкций вставки и изменения, вы сообщаете Hibernate, что строки SQL должны формироваться по требованию, а не
заранее. 

#### Неизменяемые сущности
```java
@Entity
@org.hibernate.annotations.Immutable
public class Bid {
// ...
}
```
#### Отображение сущности в подзапрос
```java
@Entity
@org.hibernate.annotations.Immutable
@org.hibernate.annotations.Subselect(
value = "select i.ID as ITEMID, i.ITEM_NAME as NAME, " +
        "count(b.ID) as NUMBEROFBIDS " +
        "from ITEM i left outer join BID b on i.ID = b.ITEM_ID " +
        "group by i.ID, i.ITEM_NAME"
)
@org.hibernate.annotations.Synchronize({"Item", "Bid"})
public class ItemBidSummary {
    @Id
    protected Long itemId;
    
    protected String name;
    
    protected long numberOfBids;
    
    public ItemBidSummary() {}
    
    // Методы чтения...
    // ...
}
```




