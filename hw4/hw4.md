# Домашнее задание №4

## Задание

1. Создать таблицу accounts(id integer, amount numeric);
2. Добавить несколько записей и подключившись через 2 терминала добиться ситуации взаимоблокировки (deadlock).
3. Посмотреть логи и убедиться, что информация о дедлоке туда попала.


## Ход работы

1. Создадим и заполним таблицу
```bash
postgres=# CREATE TABLE IF NOT EXISTS accounts(
id integer,
amount numeric
);
CREATE TABLE
postgres=# SELECT * FROM accounts
postgres-# ;
 id | amount
----+--------
(0 rows)

postgres=# INSERT INTO accounts (id, amount)
VALUES  (1, 100.11),
 (2, 88.93),
 (3, 333.3),
(4, 8976.3837),
 (5, 10);
INSERT 0 5
postgres=# SELECT * FROM accounts;
 id |  amount
----+-----------
  1 |    100.11
  2 |     88.93
  3 |     333.3
  4 | 8976.3837
  5 |        10
(5 rows)

postgres=#
```
2. Сымитируем дедлок

Терминал 1:

```bash
postgres=# \set AUTOCOMMIT off
postgres=# ;
postgres=*# commit;
COMMIT
postgres=# begin;
BEGIN
postgres=*# update accounts SET amount = amount + 0.1 where id = 1;
UPDATE 1
postgres=*# update accounts SET amount = amount + 0.1 where id = 2;
UPDATE 1
postgres=*# commit;
COMMIT
postgres=#
```

Терминал 2:
```bash
postgres=# \set AUTOCOMMIT off
postgres=# begin;
BEGIN
postgres=*# update accounts SET amount = amount + 0.1 where id = 2;
UPDATE 1
postgres=*# update accounts SET amount = amount + 0.1 where id = 1;
ERROR:  deadlock detected
DETAIL:  Process 132303 waits for ShareLock on transaction 905; blocked by process 132301.
Process 132301 waits for ShareLock on transaction 906; blocked by process 132303.
HINT:  See server log for query details.
CONTEXT:  while updating tuple (0,1) in relation "accounts"
postgres=!# commit;
ROLLBACK
postgres=#
```

3. Посмотрим отразилось ли это в базе
```bash
postgres=# select datname, deadlocks from pg_stat_database;
  datname  | deadlocks
-----------+-----------
           |         0
 postgres  |         1
 thai      |         0
 template1 |         0
 template0 |         0
(5 rows)

postgres=#
```