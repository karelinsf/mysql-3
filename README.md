# Домашнее задание к занятию «SQL. Часть 2»

## Описание

Данный документ содержит решения практических заданий по работе с SQL запросами. Все задания выполнены с использованием базы данных Sakila и демонстрируют различные техники работы с SQL: агрегатные функции, подзапросы, группировка данных и соединения таблиц.

---

## Задание 1: Информация о магазине с более чем 300 покупателями

### Условие задачи
Одним запросом получите информацию о магазине, в котором обслуживается более 300 покупателей, и выведите в результат следующую информацию:
- Фамилия и имя сотрудника из этого магазина
- Город нахождения магазина  
- Количество пользователей, закреплённых в этом магазине

### Решение

```sql
SELECT 
    s.last_name AS staff_last_name,
    s.first_name AS staff_first_name,
    c.city AS store_city,
    COUNT(cust.customer_id) AS customer_count
FROM store st
JOIN staff s ON st.manager_staff_id = s.staff_id
JOIN address a ON st.address_id = a.address_id
JOIN city c ON a.city_id = c.city_id
JOIN customer cust ON st.store_id = cust.store_id
GROUP BY st.store_id, s.staff_id, s.last_name, s.first_name, c.city
HAVING COUNT(cust.customer_id) > 300;
```

### Объяснение запроса
- Используется серия JOIN для связывания таблиц: store → staff → address → city → customer
- GROUP BY группирует данные по магазину и сотруднику
- HAVING фильтрует результаты, оставляя только магазины с более чем 300 клиентами
- COUNT подсчитывает количество уникальных покупателей для каждого магазина

---

## Задание 2: Количество фильмов длиннее средней продолжительности

### Условие задачи
Получите количество фильмов, продолжительность которых больше средней продолжительности всех фильмов.

### Решение

```sql
SELECT COUNT(*) AS films_above_average_length
FROM film
WHERE length > (
    SELECT AVG(length)
    FROM film
    WHERE length IS NOT NULL
);
```

### Объяснение запроса
- Внутренний подзапрос вычисляет среднюю продолжительность всех фильмов
- WHERE length IS NOT NULL исключает фильмы с неопределенной продолжительностью
- Внешний запрос подсчитывает количество фильмов, длительность которых превышает среднее значение
- Используется агрегатная функция COUNT(*) для подсчета строк

---

## Задание 3: Месяц с наибольшей суммой платежей

### Условие задачи
Получите информацию, за какой месяц была получена наибольшая сумма платежей, и добавьте информацию по количеству аренд за этот месяц.

### Решение

```sql
SELECT 
    strftime('%Y', p.payment_date) AS payment_year,
    strftime('%m', p.payment_date) AS payment_month,
    SUM(p.amount) AS total_payment_amount,
    COUNT(DISTINCT r.rental_id) AS rental_count
FROM payment p
JOIN rental r ON p.rental_id = r.rental_id
GROUP BY strftime('%Y', p.payment_date), strftime('%m', p.payment_date)
ORDER BY total_payment_amount DESC
LIMIT 1;
```

### Объяснение запроса
- strftime('%Y', p.payment_date) и strftime('%m', p.payment_date) извлекают год и месяц из даты платежа
- JOIN связывает таблицы payment и rental для получения информации об арендах
- GROUP BY группирует данные по году и месяцу
- SUM(p.amount) вычисляет общую сумму платежей за каждый месяц
- COUNT(DISTINCT r.rental_id) подсчитывает уникальные аренды за месяц
- ORDER BY ... DESC сортирует результаты по убыванию суммы платежей
- LIMIT 1 возвращает только месяц с максимальной суммой

---
