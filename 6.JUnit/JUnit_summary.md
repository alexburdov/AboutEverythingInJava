### JUnit краткие записки

####  @TempDir 
@TempDir решает проблемы: JUnit создает уникальную папку, инжектирует вам Path/File, а потом — по дефолту вычищает за собой. 
```java
class ReportServiceTest {

    @Test
    void generatesCsv(@TempDir Path temp) throws IOException {
        Path report = temp.resolve("users.csv");
        new ReportService().writeCsv(report);

        assertLinesMatch(
            List.of("id,name", "1,Alice", "2,Bob"),
            Files.readAllLines(report)
        );
    }
}
```
У @TempDir есть параметр cleanup, который понимает три значения:

| Режим           | Что делает                      | Когда нужен                 |
| --------------- | ------------------------------- | --------------------------- |
| ALWAYS (дефолт) | чистит всегда                   | 99% user case               |
| ON_SUCCESS      | чистит только при зеленом тесте | debug файлов                |
| NEVER           | ничего не чистит                | Редкие интеграционные тесты |

 С JUnit 5.10 появился SPI TempDirFactory. Пишем фабрику:
 ```java
class JimfsTempDirFactory implements TempDirFactory {
    private final FileSystem fs = Jimfs.newFileSystem(Configuration.unix());

    @Override
    public Path createTempDirectory(Object context) throws IOException {
        return Files.createTempDirectory(fs.getPath("/"), "junit");
    }
}
```
И используем:
```java
@Test
void reportInMemory(
        @TempDir(factory = JimfsTempDirFactory.class) Path dir
) { /* ... */ }
```


#### @TestInstance(PER_CLASS)
@TestInstance(PER_CLASS) — это инструмент ускорения тестов и оптимизации ресурсов. Используйте его, когда:
- инициализация окружения дорога по времени или деньгам;
- нужен не‑static @BeforeAll/@AfterAll;
- управление общими ресурсами явно прописано и проверено.

Не включайте, если:
- тесты запускаются параллельно и состояние нельзя надёжно обнулить;
- важна независимость между методами (property‑based или fuzz‑тесты);
- в команде нет чётких договорённостей о работе с shared state.
 
#### Отличие Mock от Spy
В JUnit, mock и spy — это инструменты Mockito, используемые для создания фиктивных объектов при тестировании. Основное отличие заключается в том, что mock создает полностью новый объект, а spy оборачивает существующий реальный объект, позволяя переопределять его поведение


