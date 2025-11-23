### Задача на понимание работы наследования

```java
public class First {
    protected int count;

    public First() {
        System.out.println("First");
        calculate();
    }

    public void calculate() {
        System.out.println(count);
    }

    @Override
    public int hashCode() {
        return 0;
    }
}

class Second extends First {

    public Second() {
        this.count = 5;
        System.out.println("Second");
        calculate();
    }

    public void calculate() {
        this.count++;
        System.out.println(count);
    }

    @Override
    public int hashCode() {
        return 0;
    }
}

class Main {
    public static void main(String[] args) {
        Second s = new Second();
    }
}
```

1. При создании объекта Second s = new Second(); сначала вызывается конструктор родительского класса First
2. В конструкторе First()
    - System.out.println("First") - выводит "First"
    - calculate() → вызывает переопределенный метод из класса Second
3. Затем выполняется конструктор Second()
    - this.speed = 5 → speed становится 5
    - System.out.println("Second") → выводитSecond
    - calculate() → снова вызывается Second.calculate() где идет count++

```
First
1
Second
6
```