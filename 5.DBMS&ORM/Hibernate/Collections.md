### Отображение коллекций

В Hibernate без расширений на выбор имеются следующие коллекции:
   - свойство типа java.util.Set, инициализированное экземпляром java.util.
HashSet. 
   - свойство типа java.util.SortedSet, инициализированное экземпляром
java.util.TreeSet.
   - свойство типа java.util.List, инициализированное экземпляром java.util.
ArrayList.
   - свойство типа java.util.Collection, инициализированное экземпляром
java.util.ArrayList.
   - свойство типа java.util.Map, инициализированное экземпляром java.util.
HashMap.
   - свойство типа java.util.SortedMap, инициализированное экземпляром java.
util.TreeMap. 
   - Hibernate, в отличие от JPA, поддерживает хранимые массивы. 
   
#### Простые коллекции
   
    Простой вариант
   ```java
@Entity
public class Item {
    @ElementCollection
    @CollectionTable(name = "IMAGE", joinColumns = @JoinColumn(name = "ITEM_ID"))
    @Column(name = "FILENAME")
    protected Set<String> images = new HashSet<String>(); 
    // ...
}
```
    Контейнер индентивикаторов (позволяет дублирующие элементы)
```java
@Entity
public class Item {
    @ElementCollection
    @CollectionTable(name = "IMAGE")
    @Column(name = "FILENAME")
    @org.hibernate.annotations.CollectionId(
        columns = @Column(name = "IMAGE_ID"), // Столбец суррогатного первичного ключа
        type = @org.hibernate.annotations.Type(type = "long"),
        generator = Constants.ID_GENERATOR
        )
       protected Collection<String> images = new ArrayList<String>();
        // ...
}
```
Отображение списка
```java
@Entity
public class Item {
    @ElementCollection
    @CollectionTable(name = "IMAGE")
    @OrderColumn //Позволяет сохранять порядок; имя по умолчанию: IMAGES_ORDER
    @Column(name = "FILENAME")
    protected List<String> images = new ArrayList<String>();
    // ...
}
```
Отображение словаря
```java
@Entity
public class Item {
    @ElementCollection
    @CollectionTable(name = "IMAGE")
    @MapKeyColumn(name = "FILENAME") // Отображает ключ
    @Column(name = "IMAGENAME") // Отображает значение
    protected Map<String, String> images = new HashMap<String, String>();
    // ...
}
```
Отсортированные и упорядоченные коллекции
```java
@Entity
public class Item {
    @ElementCollection
    @CollectionTable(name = "IMAGE")
    @MapKeyColumn(name = "FILENAME")
    @Column(name = "IMAGENAME")
    @org.hibernate.annotations.SortComparator(ReverseStringComparator.class)
    protected SortedMap<String, String> images = new TreeMap<String, String>();
    // ...
}
```
```java
@Entity
public class Item {
    @ElementCollection
    @CollectionTable(name = "IMAGE")
    @Column(name = "FILENAME")
    @org.hibernate.annotations.SortNatural
    protected SortedSet<String> images = new TreeSet<String>();
    // ...
}
```
```java
@Entity
public class Item {
    @ElementCollection
    @CollectionTable(name = "IMAGE")
    @Column(name = "FILENAME")
    // @javax.persistence.OrderBy
    @org.hibernate.annotations.OrderBy(clause = "FILENAME desc")
    protected Set<String> images = new LinkedHashSet<String>();
    // ...
}
```

#### Коллекции компонентов

Компонета
```java
@Embeddable
public class Image {
    @Column(nullable = false)
    protected String title;
    @Column(nullable = false)
    protected String filename;
    protected int width;
    protected int height;
    // ...
}
```
Множество компонентов
```java
@Entity
public class Item {
    @ElementCollection  Обязательная аннотация
    @CollectionTable(name = "IMAGE")  Переопределяет имя таблицы коллекции
    @AttributeOverride(
        name = "filename",
        column = @Column(name = "FNAME", nullable = false)
    )
    protected Set<Image> images = new HashSet<Image>();
    // ...
}
```
```java
@Entity
public class Item {
    @ElementCollection
    @CollectionTable(name = "IMAGE")
    @OrderBy("filename, width DESC")
    protected Set<Image> images = new LinkedHashSet<Image>();
    // ...
}
```
Контейнер компонентов
```java
@Entity
public class Item {
    @ElementCollection
    @CollectionTable(name = "IMAGE")
    @org.hibernate.annotations.CollectionId(
        columns = @Column(name = "IMAGE_ID"),
        type = @org.hibernate.annotations.Type(type = "long"),
        generator = Constants.ID_GENERATOR
        )
    protected Collection<Image> images = new ArrayList<Image>();
    // ...
}
```
Словарь с компонентами в качестве значений
```java
@Entity
public class Item {
    @ElementCollection
    @CollectionTable(name = "IMAGE")
    @MapKeyColumn(name = "FILENAME")  //Необязательная аннотация; имя по умолчанию IMAGES_KEY
    protected Map<String, Image> images = new HashMap<String, Image>();
    // ...
}
```



    
