### Полезные функции

#### Safe Cast с Optional
```java
public static <T> Optional<T> safeCast(Object obj, Class<T> clazz) {
    return clazz.isInstance(obj) ? Optional.of(clazz.cast(obj)) : Optional.empty();
}
```

#### Таймер производительности (Stopwatch)
```java
public static <T> T measureTime(String label, Supplier<T> supplier) {
    long start = System.nanoTime();
    T result = supplier.get();
    long duration = System.nanoTime() - start;
    System.out.printf("[%s] Duration: %d ms%n", label, duration / 1_000_000);
    return result;
}
```
#### Deep Copy через сериализацию
```java
@SuppressWarnings("unchecked")
public static <T extends Serializable> T deepCopy(T object) {
    try (
        ByteArrayOutputStream baos = new ByteArrayOutputStream();
        ObjectOutputStream oos = new ObjectOutputStream(baos)
    ) {
        oos.writeObject(object);
        try (
            ByteArrayInputStream bais = new ByteArrayInputStream(baos.toByteArray());
            ObjectInputStream ois = new ObjectInputStream(bais)
        ) {
            return (T) ois.readObject();
        }
    } catch (IOException | ClassNotFoundException e) {
        throw new RuntimeException("Deep copy failed", e);
    }
}
```
#### Рекурсивный поиск файлов с фильтром
```java
public static List<Path> findFiles(Path root, Predicate<Path> filter) throws IOException {
    try (Stream<Path> stream = Files.walk(root)) {
        return stream.filter(Files::isRegularFile).filter(filter).collect(Collectors.toList());
    }
}
```
####  Универсальный Retry Wrapper
```java
public static <T> T retry(int attempts, Duration delay, Supplier<T> task) {
    for (int i = 1; i <= attempts; i++) {
        try {
            return task.get();
        } catch (Exception e) {
            if (i == attempts) throw e;
            try { Thread.sleep(delay.toMillis()); } catch (InterruptedException ie) { Thread.currentThread().interrupt(); }
        }
    }
    throw new RuntimeException("Retry failed");
}
```
####  Сериализация Map в query string
```java
public static String toQueryString(Map<String, String> params) {
    return params.entrySet().stream()
        .map(e -> URLEncoder.encode(e.getKey(), StandardCharsets.UTF_8) + "=" +
                  URLEncoder.encode(e.getValue(), StandardCharsets.UTF_8))
        .collect(Collectors.joining("&"));
}
```
#### Группировка по произвольному ключу
```java
public static <T, K> Map<K, List<T>> groupBy(Collection<T> items, Function<T, K> classifier) {
    return items.stream().collect(Collectors.groupingBy(classifier));
}
```
#### Метод ожидания условия с таймаутом
```java
public static boolean waitFor(Predicate<Void> condition, Duration timeout, Duration interval) {
    long deadline = System.currentTimeMillis() + timeout.toMillis();
    while (System.currentTimeMillis() < deadline) {
        if (condition.test(null)) return true;
        try { Thread.sleep(interval.toMillis()); } catch (InterruptedException e) { Thread.currentThread().interrupt(); }
    }
    return false;
}
```
####  Поиск первого ненулевого значения
```java
@SafeVarargs
public static <T> Optional<T> firstNonNull(Supplier<T>... suppliers) {
    for (Supplier<T> supplier : suppliers) {
        T value = supplier.get();
        if (value != null) return Optional.of(value);
    }
    return Optional.empty();
}
```
#### Конвертация Enum в Map
```java
public static <E extends Enum<E>> Map<String, E> enumMap(Class<E> enumClass) {
    return Arrays.stream(enumClass.getEnumConstants())
                 .collect(Collectors.toMap(Enum::name, Function.identity()));
}
```
#### Отличие интерфейсов Comparator и Comparable
Интерфейс Comparable используется только для сравнения объектов класса, в котором данный интерфейс реализован. Т.е. interface Comparable определяет логику сравнения объекта определенного ссылочного типа внутри своей реализации (по правилам разработчика).
Comparator представляет отдельную реализацию и ее можно использовать многократно и с различными классами. Т.е. interface Comparator позволяет создавать объекты, которые будут управлять процессом сравнения (например при сортировках).