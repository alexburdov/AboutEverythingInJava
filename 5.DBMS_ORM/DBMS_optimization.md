~~#### Оптимизация Баз данных

#### Postgresql

**При проблемах**
1. Берёте медленный запрос, выполняете EXPLAIN ANALYZE
2. Смотрите: rows vs actual rows совпадают?
    - Нет → VACUUM ANALYZE и повтор
    - Да → переходим к п.3

3. Есть ли Seq Scan где ему не быть?
    - Да → CREATE INDEX CONCURRENTLY и ANALYZE
    - Нет → переходим к п.4

4. Есть ли плохой JOIN (Nested Loop вместо Hash)?
    - Да → SET work_mem = '256MB' или CREATE INDEX на FK
    - Нет → переходим к п.5

5. Проверить Buffers: много read?
    - Да → увеличить shared_buffers
    - Нет → проблема может быть в приложении

**10-ти шаговый алгоритм оптимизации**
1. Включите EXPLAIN ANALYZE перед подозрительным запросом
2. Проверьте: rows = actual rows? Нет → VACUUM ANALYZE
3. Есть ли Seq Scan на больших таблицах в WHERE? Да → CREATE INDEX
4. Есть ли Nested Loop в JOIN? Да → CREATE INDEX на FK или SET work_mem
5. Проверьте Buffers: много read? Да → увеличьте shared_buffers или effective_cache_size
6. Включите pg_stat_statements и мониторьте раз в неделю
7. На production CREATE INDEX CONCURRENTLY (без блокировки)
8. После каждого изменения повтори EXPLAIN — сравните cost, rows, execution time
9. Документируйте: проблема → решение → ускорение
10. Автоматизируйте: пишите скрипты для поиска новых медленных запросов

**Полезные команды для дебага**
- План + анализ 
   ```sql
        EXPLAIN (ANALYZE, BUFFERS, VERBOSE) <query>;
   ```
- JSON формат для парсинга 
   ```sql
        EXPLAIN (ANALYZE, FORMAT JSON) <query>;
   ```
- Только план (без выполнения) 
   ```sql
        EXPLAIN <query>;
   ```        
- Размер индексов 
  ```sql 
      SELECT schemaname, tablename, indexname,
       pg_size_pretty(pg_relation_size(indexrelid)) AS size
        FROM pg_indexes
            ORDER BY pg_relation_size(indexrelid) DESC;
  ```
- Медленные запросы 
   ```sql
       SELECT query, mean_time FROM pg_stat_statements
           WHERE mean_time > 50
               ORDER BY mean_time DESC LIMIT 10;
   ```
- Установка work_mem на постояную выставить в postgresql.conf: work_mem = {xxx}MB. Рестарт не требуется.
- Запретить Nested Loop, заставить Hash ```sql SET enable_nestloop = OFF; ```

**Мониторинг: pg_stat_statements**
```sql
-- Подключить (один раз)
CREATE EXTENSION IF NOT EXISTS pg_stat_statements;

-- Найти медленные запросы
SELECT 
    query,
    calls,
    total_time,
    mean_time,
    max_time
FROM pg_stat_statements
WHERE mean_time > 100  -- > 100 мс
ORDER BY mean_time DESC
LIMIT 10;

-- Очистить после анализа
SELECT pg_stat_statements_reset();

CREATE EXTENSION IF NOT EXISTS pg_stat_statements;
```

**Чек-лист для production**
- **Autovacuum включен?** SHOW autovacuum;  -- должно быть 'on'
- **Нужные индексы есть?** EXPLAIN (ANALYZE) <ваш_запрос>; Ищите Seq Scan на больших таблицах (>100k). Если WHERE/JOIN на колонке без индекса — добавьте.
- **Статистика актуальна?**
```sql
SELECT schemaname, tablename, last_autovacuum
  FROM pg_stat_user_tables
  WHERE last_autovacuum IS NULL 
     OR last_autovacuum < NOW() - INTERVAL '7 days';
```
- **work_mem достаточно?**  SHOW work_mem;  -- рекомендуется 256MB–512MB
- **shared_buffers настроен?** SHOW shared_buffers;  -- рекомендуется 25% ОЗУ

#### Лучшие практики
**Оптимизацмм в JOIN**
- **Индексы по ключам соединения.**  Без индекса - каждый JOIN превращается в полный перебор.
- **Ограничь объём данных до JOIN.** Фильтруй и агрегируй данные до объединения.
- **Учитывай тип JOIN.** INNER JOIN обычно быстрее OUTER JOIN, особенно при наличии NOT NULL. Иногда EXISTS работает быстрее, чем LEFT JOIN.
- **Следи за типами данных.** JOIN по полям с разными типами (например, int и varchar) = неэффективный cast + тормоза.
- **Проверь планы выполнения (EXPLAIN).** Не гадай, а смотри, что реально происходит. EXPLAIN ANALYZE - твой друг.

