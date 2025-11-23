### Знание контракта между Equals и HashCode
```java
public static void main(String[] args) {
  HashSet<Object> set = new HashSet<>();
  set.add(new First());
  set.add(new Second());
  set.add(new Second());
  System.out.println("Размер:" + set.size());
}
```

1. new First() - добавляется (хэшкод = 0)
2. new Second() - проверяется:
    - Хэшкод = 0 (совпадает)
    - equals() по умолчанию сравнивает ссылки → разные объекты → добавляется
3. new Second() - еще один новый объект:
    - Хэшкод = 0 (совпадает)
    - equals() сравнивает ссылки → это третий уникальный объект → добавляется
Метод equals() по умолчанию (из класса Object) сравнивает ссылки на объекты, а не их содержимое. Поэтому каждый new Second() создает новый объект с новой ссылкой, и все они считаются разными.

Тут важно отменить, что даже в случае, если hashCode не будет совпадать, то все равно получим **Размер:3**

#### Но если мы переопределим equals и hashCode:
```java
@EqualsAndHashCode
public class First {

}

@EqualsAndHashCode
class Second extends First {

}
```
То результат уже будет **Size:2**
