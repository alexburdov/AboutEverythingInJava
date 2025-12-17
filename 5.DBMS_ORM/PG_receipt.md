### Рецепты при использовании Postgresql

####  Генерация серий дат без циклов и хранимок.
Это делается с помощью generate_series:
```java

SELECT generate_series(
    date_trunc('day', current_date) - interval '30 days',
    date_trunc('day', current_date),
    interval '1 day'
) AS day;

```
Результат — 31 строка с датами от 30 дней назад до сегодняшнего дня. Теперь добавим, например, LEFT JOIN к таблице events, чтобы увидеть активность по дням:
```java

SELECT
    d.day,
    COUNT(e.id) AS events_count
FROM
    generate_series(
        date_trunc('day', current_date) - interval '30 days',
        date_trunc('day', current_date),
        interval '1 day'
    ) AS d(day)
LEFT JOIN events e ON date_trunc('day', e.created_at) = d.day
GROUP BY d.day
ORDER BY d.day;

```
