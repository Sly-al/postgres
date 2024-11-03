# Домашнее задание №8

## Задание

1. Создать таблицу с продажами.
2. Реализовать функцию выбор трети года (1-4 месяц - первая треть, 5-8 - вторая и тд)
   a. через case
   b. * (бонуса в виде зачета дз не будет) используя математическую операцию
   (лучше 2+ варианта)
   c. предусмотреть NULL на входе
3. Вызвать эту функцию в SELECT из таблицы с продажами, уведиться, что всё
   отработало

## Ход работы

1. Создадим таблицу 
```bash
postgres=# CREATE TABLE IF NOT EXISTS sales(
postgres(# id SERIAL PRIMARY KEY,
postgres(# price bigint NOT NULL,
postgres(# date TIMESTAMPTZ,
postgres(# user_id bigint,
postgres(# cart_id bigint NOT NUll
postgres(# );
postgres=# INSERT INTO sales (price, date, user_id, cart_id)
VALUES
(456, TIMESTAMP WITH TIME ZONE '2023-11-01 12:00:00 UTC', 5647, 2),
(12345, TIMESTAMP WITH TIME ZONE '2023-01-15 08:30:00 UTC', 5678, 2),
(67890, TIMESTAMP WITH TIME ZONE '2023-02-20 14:45:00 UTC', 2345, 7),
(54321, TIMESTAMP WITH TIME ZONE '2023-03-10 19:15:00 UTC', 8765, 5),
(98765, TIMESTAMP WITH TIME ZONE '2023-04-30 22:00:00 UTC', 3456, 1),
(13579, TIMESTAMP WITH TIME ZONE '2023-05-25 09:05:00 UTC', 9823, 4),
(24680, TIMESTAMP WITH TIME ZONE '2023-06-11 11:20:00 UTC', 7654, 6),
(11111, TIMESTAMP WITH TIME ZONE '2023-07-05 17:30:00 UTC', 5432, 8),
(22222, TIMESTAMP WITH TIME ZONE '2023-08-15 12:00:00 UTC', 1234, 3),
(33333, TIMESTAMP WITH TIME ZONE '2023-09-10 16:15:00 UTC', 4321, 9),
(44444, TIMESTAMP WITH TIME ZONE '2023-10-20 21:45:00 UTC', 6789, 0),
(55555, NULL, 9876, 2),
(66666, NULL, 5432, 7),
(77777, NULL, 1234, 1),
(88888, NULL, 8765, 5),
(99999, NULL, 2345, 3);
INSERT 0 16
```
2. Создадим функцию
```bash
postgres=# CREATE OR REPLACE FUNCTION quart(date timestamptz) RETURNS integer
AS $$
    SELECT
        CASE
            WHEN date_part('month', date) BETWEEN 1 AND 4 THEN 1
            WHEN date_part('month', date) BETWEEN 5 AND 8 THEN 2
            ELSE 3
            END
$$
LANGUAGE SQL
RETURNS NULL ON NULL INPUT;
CREATE FUNCTION
postgres=# \df
                            List of functions
 Schema | Name  | Result data type |      Argument data types      | Type
--------+-------+------------------+-------------------------------+------
 public | quart | integer          | date timestamp with time zone | func
(1 row)
```
3. Проверим правильность работы на запросе, подсчитывающем выручку по третям года
```bash
postgres=# SELECT
sum(price) as total_price,
quart(date) as quarter
FROM sales
GROUP BY quart(date)
ORDER BY total_price;
 total_price | quarter
-------------+---------
       78389 |       3
      134556 |       1
      170357 |       2
      388885 |
(4 rows)

postgres=#
```
Пример конечно синтетический, но самое главное - что работает 