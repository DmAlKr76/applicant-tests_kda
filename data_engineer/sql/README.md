# Тест SQL

На основе таблиц базы данных, напишите SQL код, который возвращает необходимые результаты
Пример: 

Общее количество товаров
```sql
select count (*) from items
```

## Структура данных

Используемый синтаксис: Oracle SQL или другой

| Сustomer       | Description           |
| -------------- | --------------------- |
| customer\_id   | customer unique id    |
| customer\_name | customer name         |
| country\_code  | country code ISO 3166 |

| Items             | Description       |
| ----------------- | ----------------- |
| item\_id          | item unique id    |
| item\_name        | item name         |
| item\_description | item description  |
| item\_price       | item price in USD |

| Orders       | Description                 |
| ------------ | --------------------------- |
| date\_time   | date and time of the orders |
| item\_id     | item unique id              |
| customer\_id | user unique id              |
| quantity     | number of items in order    |

| Countries     | Description           |
| ------------- | --------------------- |
| country\_code | country code          |
| country\_name | country name          |
| country\_zone | AMER, APJ, LATAM etc. |


| Сonnection\_log         | Description                           |
| ----------------------- | ------------------------------------- |
| customer\_id            | customer unique id                    |
| first\_connection\_time | date and time of the first connection |
| last\_connection\_time  | date and time of the last connection  |

## Задания

### 1) Количество покупателей из Италии и Франции

| **Country_name** | **CustomerCountDistinct** |
| ------------------------- | ----------------------------- |
| France                    | #                             |
| Italy                     | #                             |

```sql
SELECT 
    c.country_name AS Country_name, 
    COUNT(DISTINCT cu.customer_id) AS CustomerCountDistinct
FROM 
    Customer cu
JOIN 
    Countries c ON cu.country_code = c.country_code
WHERE 
    c.country_name IN ('France', 'Italy') -- или c.country_name IN ('Франция', 'Италия')
GROUP BY 
    c.country_name;
```

### 2) ТОП 10 покупателей по расходам

| **Customer_name** | **Revenue** |
| ---------------------- | ----------- |
| #                      | #           |
| #                      | #           |
| #                      | #           |
| #                      | #           |
| #                      | #           |
| #                      | #           |
| #                      | #           |

```sql
SELECT 
    cu.customer_name AS Customer_name,
    SUM(o.quantity * i.item_price) AS Revenue
FROM 
    Orders o
JOIN 
    Customer cu ON o.customer_id = cu.customer_id
JOIN 
    Items i ON o.item_id = i.item_id
GROUP BY 
    cu.customer_id, cu.customer_name
ORDER BY 
    Revenue DESC
LIMIT 10;
```

### 3) Общая выручка USD по странам, если нет дохода, вернуть NULL

| **Country_name** | **RevenuePerCountry** |
| ------------------------- | --------------------- |
| Italy                     | #                     |
| France                    | NULL                  |
| Mexico                    | #                     |
| Germany                   | #                     |
| Tanzania                  | #                     |

```sql
WITH CountryRevenue AS (
    SELECT 
        cu.country_code,
        SUM(o.quantity * i.item_price) AS country_revenue
    FROM 
        Orders o
    JOIN 
        Customer cu ON o.customer_id = cu.customer_id
    JOIN 
        Items i ON o.item_id = i.item_id
    GROUP BY 
        cu.country_code
)
SELECT 
    c.country_name AS Country_name,
    CR.country_revenue AS RevenuePerCountry
FROM 
    Countries c
LEFT JOIN 
    CountryRevenue CR ON c.country_code = CR.country_code
```

### 4) Самый дорогой товар, купленный одним покупателем

| **Customer\_id** | **Customer\_name** | **MostExpensiveItemName** |
| ---------------- | ------------------ | ------------------------- |
| #                | #                  | #                         |
| #                | #                  | #                         |
| #                | #                  | #                         |
| #                | #                  | #                         |
| #                | #                  | #                         |
| #                | #                  | #                         |
| #                | #                  | #                         |

```sql
SELECT 
    Customer_id,
    Customer_name,
    MostExpensiveItemName
FROM (
    SELECT 
        o.customer_id AS Customer_id,
        cu.customer_name AS Customer_name,
        i.item_name AS MostExpensiveItemName,
        ROW_NUMBER() OVER (PARTITION BY o.customer_id ORDER BY i.item_price DESC) AS row_num
    FROM 
        Orders o
    JOIN 
        Customer cu ON o.customer_id = cu.customer_id
    JOIN 
        Items i ON o.item_id = i.item_id
) AS ranked_items
WHERE 
    row_num = 1;
```

### 5) Ежемесячный доход

| **Month (MM format)** | **Total Revenue** |
| --------------------- | ----------------- |
| #                     | #                 |
| #                     | #                 |
| #                     | #                 |
| #                     | #                 |
| #                     | #                 |
| #                     | #                 |
| #                     | #                 |

```sql
-- в примере вывода выше не предусмотрена разбивка по годам, 
-- что норм если данные в одном году или считаем сумму по месяцам за все годы
SELECT 
    TO_CHAR(date_time, 'MM') AS Month,
    SUM(o.quantity * i.item_price) AS Total_Revenue
FROM 
    Orders o
JOIN 
    Items i ON o.item_id = i.item_id
GROUP BY 
    TO_CHAR(date_time, 'MM')
ORDER BY 
    TO_CHAR(date_time, 'MM');
```

```sql
-- с разбивкой по годам
SELECT 
    TO_CHAR(date_time, 'MM') AS Month, 
    TO_CHAR(date_time, 'YYYY') AS Year,
    SUM(o.quantity * i.item_price) AS Total_Revenue
FROM 
    Orders o
JOIN 
    Items i ON o.item_id = i.item_id
GROUP BY 
    TO_CHAR(date_time, 'MM'),
    TO_CHAR(date_time, 'YYYY')
ORDER BY 
    TO_CHAR(date_time, 'YYYY'), 
    TO_CHAR(date_time, 'MM');
```


### 6) Найти дубликаты

Во время передачи данных произошел сбой, в таблице orders появилось несколько 
дубликатов (несколько результатов возвращаются для date_time + customer_id + item_id). 
Вы должны их найти и вернуть количество дубликатов.

```sql
SELECT
  date_time,
  customer_id,
  item_id,
  COUNT(*) AS duplicate_count
FROM Orders
GROUP BY date_time, customer_id, item_id
HAVING COUNT(*) > 1;
```

Или

```sql
WITH duplicates AS (
    SELECT
        date_time,
        customer_id,
        item_id,
        COUNT(*) OVER (PARTITION BY date_time, customer_id, item_id) AS duplicate_count
    FROM Orders
)
SELECT
    date_time,
    customer_id,
    item_id,
    duplicate_count
FROM duplicates
WHERE duplicate_count > 1;
```

### 7) Найти "важных" покупателей

Создать запрос, который найдет всех "важных" покупателей,
т.е. тех, кто совершил наибольшее количество покупок после своего первого заказа.

| **Customer\_id** | **Total Orders Count** |
| --------------------- |-------------------------------|
| #                     | #                             |
| #                     | #                             |
| #                     | #                             |
| #                     | #                             |
| #                     | #                             |
| #                     | #                             |
| #                     | #                             |

```sql
-- если я правильно понял задание, их сложность по идее идет по нарастающей
SELECT 
    customer_id,
    COUNT(*) AS order_count
FROM 
    Orders
GROUP BY 
    customer_id
HAVING COUNT(*) > 1
ORDER BY COUNT(*) DESC;
```

### 8) Найти покупателей с "ростом" за последний месяц

Написать запрос, который найдет всех клиентов,
у которых суммарная выручка за последний месяц
превышает среднюю выручку за все месяцы.

| **Customer\_id** | **Total Revenue** |
| --------------------- |-------------------|
| #                     | #                 |
| #                     | #                 |
| #                     | #                 |
| #                     | #                 |
| #                     | #                 |
| #                     | #                 |
| #                     | #                 |

```sql
-- уточнил бы средняя выручка за все месяцы в разрезе данного клиента или всех клиентов
-- и месяц текущий календарный или месяц длительности (timedelta)
WITH last_month_orders AS (
  SELECT 
    customer_id,
    SUM(quantity * item_price) AS last_month_revenue
  FROM Orders o
  JOIN Items i ON o.item_id = i.item_id
  WHERE DATE_PART('month', date_time) = DATE_PART('month', CURRENT_DATE - INTERVAL 1 MONTH)
        AND DATE_PART('year', date_time) = DATE_PART('year', CURRENT_DATE - INTERVAL 1 MONTH)
  GROUP BY customer_id
),
total_orders AS (
  SELECT
    customer_id,
    SUM(quantity * item_price) AS total_revenue
  FROM Orders o
  JOIN Items i ON o.item_id = i.item_id
  GROUP BY customer_id
),
avg_revenue AS (
  SELECT
    AVG(total_revenue) AS avg_total_revenue
  FROM total_orders
)
SELECT
  lmo.customer_id,
  lmo.last_month_revenue AS "Total Revenue"
FROM last_month_orders lmo
JOIN total_orders to ON lmo.customer_id = to.customer_id
JOIN avg_revenue ar ON 1 = 1
WHERE lmo.last_month_revenue > ar.avg_total_revenue
```