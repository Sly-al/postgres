# Домашнее задание №7

## Задание

1. Развернуть ВМ (Linux) с PostgreSQL
2. Залить Тайские перевозки
   https://github.com/aeuge/postgres16book/tree/main/database
3. Проверить скорость выполнения сложного запроса (приложен в конце файла скриптов)
4. Навесить индексы на внешние ключ
5. Проверить, помогли ли индексы на внешние ключи ускориться

## Ход работы

1-2. сделано в дз1
3. Скорость запроса
```bash
thai=# WITH all_place AS (
    SELECT count(s.id) as all_place, s.fkbus as fkbus
    FROM book.seat s
    group by s.fkbus
),
     order_place AS (
         SELECT count(t.id) as order_place, t.fkride
         FROM book.tickets t
         group by t.fkride
     )
SELECT r.id, r.startdate as depart_date, bs.city || ', ' || bs.name as busstation,
       t.order_place, st.all_place
FROM book.ride r
         JOIN book.schedule as s
              on r.fkschedule = s.id
         JOIN book.busroute br
              on s.fkroute = br.id
         JOIN book.busstation bs
              on br.fkbusstationfrom = bs.id
         JOIN order_place t
              on t.fkride = r.id
         JOIN all_place st
              on r.fkbus = st.fkbus
GROUP BY r.id, r.startdate, bs.city || ', ' || bs.name, t.order_place,st.all_place
ORDER BY r.startdate
limit 10;
 id | depart_date |       busstation       | order_place | all_place
----+-------------+------------------------+-------------+-----------
  1 | 2000-01-01  | Bankgkok, Suvarnabhumi |          38 |        80
  2 | 2000-01-01  | Bankgkok, Suvarnabhumi |          38 |        80
  3 | 2000-01-01  | Bankgkok, Suvarnabhumi |          34 |        80
  4 | 2000-01-01  | Bankgkok, Eastern      |          33 |        80
  5 | 2000-01-01  | Bankgkok, Eastern      |          36 |        80
  6 | 2000-01-01  | Bankgkok, Eastern      |          32 |        80
  7 | 2000-01-01  | Bankgkok, Chatuchak    |          37 |        80
  8 | 2000-01-01  | Bankgkok, Chatuchak    |          33 |        80
  9 | 2000-01-01  | Bankgkok, Chatuchak    |          37 |        80
 10 | 2000-01-01  | Bankgkok, Suvarnabhumi |          38 |        80
(10 rows)

Time: 1299.354 ms (00:01.299)
thai=#
```
4. Навесим
```bash
Time: 1499.004 ms (00:01.499)
thai=# select *
from pg_indexes
where schemaname = 'book'
;
 schemaname | tablename |      indexname       | tablespace |                                    indexdef
------------+-----------+----------------------+------------+--------------------------------------------------------------------------------
 book       | tickets   | tickets_pkey         |            | CREATE UNIQUE INDEX tickets_pkey ON book.tickets USING btree (id)
 book       | seat      | idx_seat_id          |            | CREATE INDEX idx_seat_id ON book.seat USING btree (id)
 book       | busroute  | idx_busroute_id      |            | CREATE INDEX idx_busroute_id ON book.busroute USING btree (id)
 book       | ride      | idx_ride_fkbus       |            | CREATE INDEX idx_ride_fkbus ON book.ride USING btree (fkbus)
 book       | ride      | idx_ride_fkschedule  |            | CREATE INDEX idx_ride_fkschedule ON book.ride USING btree (fkschedule)
 book       | ride      | idx_ride_id          |            | CREATE INDEX idx_ride_id ON book.ride USING btree (id)
 book       | schedule  | idx_schedule_fkroute |            | CREATE INDEX idx_schedule_fkroute ON book.schedule USING btree (fkroute)
 book       | busroute  | idx_busroute_from    |            | CREATE INDEX idx_busroute_from ON book.busroute USING btree (fkbusstationfrom)
(8 rows)

Time: 1.692 ms
thai=#
```
5. Проверим скокрость выполнения
```bash
thai=# WITH all_place AS (
    SELECT count(s.id) as all_place, s.fkbus as fkbus
    FROM book.seat s
    group by s.fkbus
),
     order_place AS (
         SELECT count(t.id) as order_place, t.fkride
         FROM book.tickets t
         group by t.fkride
     )
SELECT r.id, r.startdate as depart_date, bs.city || ', ' || bs.name as busstation,
       t.order_place, st.all_place
FROM book.ride r
         JOIN book.schedule as s
              on r.fkschedule = s.id
         JOIN book.busroute br
              on s.fkroute = br.id
         JOIN book.busstation bs
              on br.fkbusstationfrom = bs.id
         JOIN order_place t
              on t.fkride = r.id
         JOIN all_place st
              on r.fkbus = st.fkbus
GROUP BY r.id, r.startdate, bs.city || ', ' || bs.name, t.order_place,st.all_place
ORDER BY r.startdate
limit 10;
 id | depart_date |       busstation       | order_place | all_place
----+-------------+------------------------+-------------+-----------
  1 | 2000-01-01  | Bankgkok, Suvarnabhumi |          38 |        80
  2 | 2000-01-01  | Bankgkok, Suvarnabhumi |          38 |        80
  3 | 2000-01-01  | Bankgkok, Suvarnabhumi |          34 |        80
  4 | 2000-01-01  | Bankgkok, Eastern      |          33 |        80
  5 | 2000-01-01  | Bankgkok, Eastern      |          36 |        80
  6 | 2000-01-01  | Bankgkok, Eastern      |          32 |        80
  7 | 2000-01-01  | Bankgkok, Chatuchak    |          37 |        80
  8 | 2000-01-01  | Bankgkok, Chatuchak    |          33 |        80
  9 | 2000-01-01  | Bankgkok, Chatuchak    |          37 |        80
 10 | 2000-01-01  | Bankgkok, Suvarnabhumi |          38 |        80
(10 rows)

Time: 1317.819 ms (00:01.318)
```
Как мы видим, прироста добиться не удалось, скорее всего это связано с несколькими пунктами
* были залиты маленькие тайские перевозки
* так как было сделано все несколько запросов, то планировщик просто не собрал достаточно статистики для выбора индексов при выполнении запросов