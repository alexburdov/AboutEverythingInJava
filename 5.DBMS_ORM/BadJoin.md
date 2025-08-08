### Плохие JOIN’ы

#### Нет ON (CROSS JOIN)

```sql
SELECT  u.id, p.amount FROM    users u, payments p;
```

Если не находится условия то выставляется CROSS JOIN

#### JOIN по функции
```sql
SELECT  u.id, s.id 
FROM    users u
JOIN    subscriptions s
        ON LOWER(u.email) = LOWER(s.email);
```
В этом случае индексы не работают

**Возможное решение** 
```sql
CREATE INDEX CONCURRENTLY idx_users_email_lower ON users (LOWER(email));
```
#### LEFT JOIN + WHERE column IS NOT NULL: превращение в INNER JOIN

```sql
SELECT  o.id, r.id
FROM    orders o
LEFT JOIN refunds r ON r.order_id = o.id
WHERE   r.id IS NOT NULL;
```

#### Несогласованные типы
```sql
SELECT  c.id, o.id
FROM    customers  c         -- id VARCHAR
JOIN    orders     o         -- customer_id INT
        ON c.id = o.customer_id;
```

#### OR в ON-условии
```sql
SELECT  *
FROM    payments p
JOIN    invoices i
      ON (i.id = p.invoice_id OR i.external_id = p.invoice_external_id);
```
OR делает условие неконъюнктивным; оценка селективности падает, индексы часто игнорируются. 
**Возможное решение** 
UNION ALL

#### Не-саргабельные выражения (DATE(created_at) = …)
```sql
SELECT  *
FROM    logs l
JOIN    users u ON u.id = l.user_id
WHERE   DATE(l.created_at) = CURRENT_DATE;
```
Каждый вызов DATE() ‑ того же порядка, что FULL SCAN
**Возможное решение** 
```sql
WHERE l.created_at >= CURRENT_DATE
  AND l.created_at <  CURRENT_DATE + INTERVAL '1 day';
```
Ставим expression index CREATE INDEX … (DATE(created_at))

#### Join без индексов
```sql
SELECT  *
FROM      big_a a
JOIN      big_b b ON b.a_id = a.id;
```
Должен иметь
```sql
CREATE INDEX CONCURRENTLY idx_big_b_a_id ON big_b (a_id);
```

#### CROSS APPLY / LATERAL как цикл «for each row»
```sql
SELECT  u.id,
        l.last_login
FROM    users u
CROSS APPLY (
   SELECT  last_login
   FROM    logins
   WHERE   user_id = u.id
   ORDER BY created_at DESC
   LIMIT 1
) l;
```
CROSS APPLY (SQL Server) или LATERAL (Postgres) запускает подзапрос для каждой строки исходной таблицы. Если внутри сортировка/агрегация — получаем N раз. 

**Возможное решение** 
Реписываем на оконные функции:
```sql
SELECT DISTINCT ON (u.id)
       u.id,
       l.last_login
FROM   users u
JOIN   logins l ON l.user_id = u.id
ORDER  BY u.id, l.created_at DESC;
```
 агрегация + join:
 
 ```sql
WITH last AS (
    SELECT user_id, MAX(created_at) AS last_login
    FROM   logins
    GROUP  BY user_id
)
SELECT u.id, last.last_login
FROM   users u
LEFT JOIN last ON last.user_id = u.id;
```

