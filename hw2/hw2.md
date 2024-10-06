# Домашнее задание №2

## Задание
Пошаговая инструкция:
1. открыть консоль и зайти по ssh на ВМ
2. открыть вторую консоль и также зайти по ssh на ту же ВМ (можно в докере 2 сеанса)
3. запустить везде psql из под пользователя postgres
4. сделать в первой сессии новую таблицу и наполнить ее данными
5. посмотреть текущий уровень изоляции:
6. начать новую транзакцию в обеих сессиях с дефолтным (не меняя) уровнем
   изоляции
7. в первой сессии добавить новую запись
8. сделать запрос на выбор всех записей во второй сессии
9. видите ли вы новую запись и если да то почему? После задания можете сверить
   правильный ответ с эталонным (будет доступен после 3 лекции)
10. завершить транзакцию в первом окне
11. сделать запрос на выбор всех записей второй сессии
12. видите ли вы новую запись и если да то почему?
13. завершите транзакцию во второй сессии
14. начать новые транзакции, но уже на уровне repeatable read в ОБЕИХ сессиях
15. в первой сессии добавить новую запись
16. сделать запрос на выбор всех записей во второй сессии
17. видите ли вы новую запись и если да то почему?
18. завершить транзакцию в первом окне
19. сделать запрос во выбор всех записей второй сессии
20. видите ли вы новую запись и если да то почему?


## Ход работы

1 - 3. действия понятны

4. Создадим таблицу
```sql
postgres=# CREATE TABLE IF NOT EXISTS goods (
            id SERIAL PRIMARY KEY,
            name TEXT
            );
postgres=# INSERT INTO goods (name)  VALUES('apple');
INSERT 0 1
   postgres=# INSERT INTO goods (name)  VALUES('pear');
INSERT 0 1
   postgres=# INSERT INTO goods (name)  VALUES('tomato');
INSERT 0 1
   postgres=# SELECT * FROM goods;
id |  name
----+--------
  1 | apple
  2 | pear
  3 | tomato
(3 rows)

postgres=#
```
5. текущий уровень - read committed(дефолт)
```sql
postgres=# show transaction isolation level;
transaction_isolation
-----------------------
read committed
(1 row)

```
6. отключим автоматику на транзакциях
```sql
postgres=# \set AUTOCOMMIT OFF
postgres=#
postgres=# BEGIN;
BEGIN
postgres=*#
```
7. добавим запись
```sql
postgres=*# INSERT INTO goods(name) VALUES('potato');
INSERT 0 1
postgres=*#
```
8. посмотрим со второй консоли данные
```sql
postgres=*# SELECT * FROM goods;
id |  name
----+--------
1 | apple
2 | pear
3 | tomato
(3 rows)
```
9. нет, не видим, потому что аномалия "грязное чтение" невозможна при уровне read commited(из названия изоляции понятно, что читаем только зафиксированные данные)
10. коммит транзакцию на вставку
```sql
postgres=*# COMMIT;
COMMIT
postgres=#
```
11. ещё раз смотрим со второй консоли
```sql
postgres=*# SELECT * FROM goods;
id |  name
----+--------
1 | apple
2 | pear
3 | tomato
4 | potato
(4 rows)
```
12. да, видим, потому что аномалия "неповторяемое чтение" как раз таки возможна при уровне read committed
13. завершим транзакцию во втором окне
```sql
postgres=*# COMMIT;
COMMIT
```
14. начнём новые транзацкции, но уже с уровнем repeatable read
```sql
postgres=# begin;
BEGIN
postgres=*# show transaction isolation level;
transaction_isolation
-----------------------
read committed
(1 row)

postgres=*# set TRANSACTION ISOLATION LEVEL REPEATABLE READ;
SET
postgres=*# show transaction isolation level;
transaction_isolation
-----------------------
repeatable read
(1 row)

postgres=*#
```
15. добавим данные в первом окне
```sql
postgres=*# INSERT INTO goods (name) VALUES ('orange');
INSERT 0 1
postgres=*#
```
16. посмотрим данные во втором окне
```sql
postgres=*# select * from goods;
id |  name
----+--------
1 | apple
2 | pear
3 | tomato
4 | potato
(4 rows)
```
17. нет, не видим, потому что уровень repeatable read тем более защищает нас от чтения незафиксированных данных
18. коммит вставки
```sql
postgres=*# commit;
COMMIT
```
19. делаем запрос во втором окне
```sql
postgres=*# select * from goods;
id |  name
----+--------
1 | apple
2 | pear
3 | tomato
4 | potato
(4 rows)
```
20. нет, не видим, потому что уровень изоляции был повышен до repeatable read, который уже исключает неповторяемое чтение