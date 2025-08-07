### Отношения

#### Одностороние отношения

**One-To-Many**
```java
@Entity
public class Department {
 
    @Id
    private Long id;
 
    @OneToMany
    @JoinColumn(name = "department_id")
    private List<Employee> employees;
}

@Entity
public class Employee {
 
    @Id
    private Long id;
}
```

**Many-To-One Relationship**

```java
@Entity
public class Student {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;

    @ManyToOne
    @JoinColumn(name = "school_id")
    private School school;
}

@Entity
public class School {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;
}
```

**One-To-One**
```java
@Entity
public class Employee {
 
    @Id
    private Long id;
 
    @OneToOne
    @JoinColumn(name = "parking_spot_id")
    private ParkingSpot parkingSpot;
 
}

@Entity
public class ParkingSpot {
 
    @Id
    private Long id;
 
}
```

 **Many-To-Many**
 
 ```java
@Entity
public class Book {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String title;

    @ManyToMany
    @JoinTable(name = "book_author",
            joinColumns = @JoinColumn(name = "book_id"),
            inverseJoinColumns = @JoinColumn(name = "author_id"))
    private Set<Author> authors;

}

@Entity
public class Author {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;
}
```
#### Двухстороние отношения

**One-to-many <=> Many-to-one**
```java
@Entity
public class Department {
 
    @OneToMany(mappedBy = "department")
    private List<Employee> employees;
 
}
 
@Entity
public class Employee {
 
    @ManyToOne
    @JoinColumn(name = "department_id")
    private Department department;
 
}
```

**Many-To-Many <=> Many-To-Many**

```java
@Entity
public class Student {
 
    @ManyToMany(mappedBy = "students")
    private List<Course> courses;
 
}
 
@Entity
public class Course {
 
    @ManyToMany
    @JoinTable(name = "course_student",
        joinColumns = @JoinColumn(name = "course_id"),
        inverseJoinColumns = @JoinColumn(name = "student_id"))
    private List<Student> students;
 
}
```

#### Каскадные операции

В JPA спецификации есть такие значения этого параметра:
- **ALL** означает, что все действия, которые мы выполняем с родительским объектом, нужно повторить и для его зависимых объектов.
- **PERSIST** означает, что если мы сохраняем в базу родительский объект, то это же нужно сделать и с его зависимыми объектами. 
- **MERGE** означает, что если мы обновляем в базе родительский объект, то это же нужно сделать и с его зависимыми объектами. 
- **REMOVE** означает, что если мы удаляем в базе родительский объект, то это же нужно сделать и с его зависимыми объектами.
- **DELETE** означает то же самое. Это синонимы. Просто из разных спецификаций. 
- **DETACH** означает, что если мы удаляем родительский объект из сессии, то это же нужно сделать и с его зависимыми объектами.
- **REFRESH** (SAVE_UPDATE) дублируют действия, которые выполняются с родительским объектом к его зависимому объекту. 

Однако Hibernate расширяет эту спецификацию еще на три варианта:

- REPLICATE
- **SAVE_UPDATE** (REFRESH)  дублируют действия, которые выполняются с родительским объектом к его зависимому объекту. 
- LOCK

#### Параметр Orphan removal
Параметр orphan. Это сокращение от термина Orphan removal. Он используется для того, чтобы не оставалось дочерних сущностей без родительских. Если этот параметр выставлен в true, то дочерняя сущность будет удалена, если на нее исчезли все ссылки. Это не совсем то же самое, что и Cascade.REMOVE. У тебя может быть ситуация, когда несколько родительских сущностей ссылаются на одну дочернюю. Тогда выгодно, чтобы она удалялась не вместе с удалением родительской сущности, а только если все ссылки на нее будут обнулены.

#### Использование ON DELETE CASCADE для внешнего ключа
```java
@Entity
public class Item {
    @OneToMany(mappedBy = "item", cascade = CascadeType.PERSIST)
    @org.hibernate.annotations.OnDelete(action = org.hibernate.annotations.OnDeleteAction.CASCADE)
    protected Set<Bid> bids = new HashSet<>();
    // ...
}
```
аннотация @OnDelete влияет только на схему, которую генерирует Hibernate.

#### @PrimaryKeyJoinColumn
Дочерние Entity-классы имеют колонку в таблице, которая ссылается на id объекта родительского Entity-класса. Имя этой колонки по умолчанию равно имени колонки родительского класса.

#### Генератор внешнего первичного ключа
```java
@Entity
@Table(name = "USERS")
public class User {
    @Id
    @GeneratedValue(generator = Constants.ID_GENERATOR)
    protected Long id;
    
    @OneToOne(
        mappedBy = "user",
        cascade = CascadeType.PERSIST
    )
    protected Address shippingAddress;
// ...
}

@Entity
public class Address {
    @Id
    @GeneratedValue(generator = "addressKeyGenerator")
    @org.hibernate.annotations.GenericGenerator(
        name = "addressKeyGenerator",
        strategy = "foreign",
        parameters =
            @org.hibernate.annotations.Parameter(
                    name = "property", value = "user"
            )
    )
    protected Long id;

    @OneToOne(optional = false)  Создает ограничение внешнего ключа
    @PrimaryKeyJoinColumn  Адрес должен ссылаться на пользователя
    protected User user;

    protected Address() {}
    
    public Address(User user) {
        this.user = user;
    }
    
    public Address(User user, String street, String zipcode, String city) {
        this.user = user;
        this.street = street;
        this.zipcode = zipcode;
        this.city = city;
    }
    // ...
}
```

#### Соединение с помощью столбца внешнего ключа

```java
@Entity
@Table(name = "USERS")
public class User {
    @Id
    @GeneratedValue(generator = Constants.ID_GENERATOR)
    protected Long id;
    
    @OneToOne(
        fetch = FetchType.LAZY,
        optional = false, NOT NULL
        cascade = CascadeType.PERSIST
    )
    @JoinColumn(unique = true) По умолчанию SHIPPINGADDRESS_ID
    protected Address shippingAddress;
    // ...
}
```

#### Использование таблицы соединения
```java
@Entity
public class Shipment {
    @OneToOne(fetch = FetchType.LAZY)
    @JoinTable(
        name = "ITEM_SHIPMENT", // Имя обязательно!
        joinColumns = @JoinColumn(name = "SHIPMENT_ID"), // По умолчанию ID
        inverseJoinColumns 
            = @JoinColumn(name = "ITEM_ID", // По умолчанию AUCTION_ID
                          nullable = false,
                          unique = true)
    )
    protected Item auction;
    
    public Shipment() {}
    
    public Shipment(Item auction) {
        this.auction = auction;
    }
    // ...
}
```

