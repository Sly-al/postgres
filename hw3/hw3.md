# Домашнее задание №3

## Задание

1. Создать таблицу с текстовым полем и заполнить случайными или сгенерированными данным в размере 1 млн строк
2. Посмотреть размер файла с таблицей
3. 5 раз обновить все строчки и добавить к каждой строчке любой символ
4. Посмотреть количество мертвых строчек в таблице и когда последний раз приходил
   автовакуум
5. Подождать некоторое время, проверяя, пришел ли автовакуум
6. 5 раз обновить все строчки и добавить к каждой строчке любой символ
7. Посмотреть размер файла с таблицей
8. Отключить Автовакуум на конкретной таблице
9. 10 раз обновить все строчки и добавить к каждой строчке любой символ
10. Посмотреть размер файла с таблицей
11. Объясните полученный результат
12. Не забудьте включить автовакуум)

Задание со *:
Написать анонимную процедуру, в которой в цикле 10 раз обновятся все строчки в искомой таблице.
Не забыть вывести номер шага цикла.


## Ход работы

Разумеется, в первую очередь выполним задание со звёдочкой
```sql
DO $$
BEGIN
    FOR i in 1..5 LOOP
        UPDATE million SET name = name || i || '.';
        raise notice 'LOOP: %', i;
    END LOOP;
END $$;
```

1. Создадим и заполним
```bash
postgres=# \dt
         List of relations
 Schema | Name  | Type  |  Owner
--------+-------+-------+----------
 public | goods | table | postgres
(1 row)

postgres=# CREATE TABLE IF NOT EXISTS million
postgres-# (
postgres(# name text);
CREATE TABLE
postgres=# \dt
          List of relations
 Schema |  Name   | Type  |  Owner
--------+---------+-------+----------
 public | goods   | table | postgres
 public | million | table | postgres
(2 rows)

postgres=# SELECT * FROM million;
 name
------
(0 rows)

postgres=# INSERT INTO million (name)
postgres-# SELECT SUBSTRING(md5(random()::text), 1, 10) as name
postgres-# FROM generate

postgres-# FROM generate_series(1,1000000);
INSERT 0 1000000
postgres=# SELECT count(*) from million;
  count
---------
 1000000
(1 row)

postgres=# SELECT * FROM million LIMIT 5;
    name
------------
 4258ff9f88
 ba07073eed
 dcfc85cbf8
 9a3cb46831
 02f15a5b8c
(5 rows)

postgres=#
```

2. Посмотрим размер таблицы `million`
```bash
postgres=# \d+
                                          List of relations
 Schema |     Name     |   Type   |  Owner   | Persistence | Access method |    Size    | Description
--------+--------------+----------+----------+-------------+---------------+------------+-------------
 public | goods        | table    | postgres | permanent   | heap          | 16 kB      |
 public | goods_id_seq | sequence | postgres | permanent   |               | 8192 bytes |
 public | million      | table    | postgres | permanent   | heap          | 42 MB      |
(3 rows)
```
3. Обновим таблицу при помощи процедурки
```bash
postgres=# DO $$
BEGIN
FOR i in 1..5 LOOP
UPDATE million SET name = name || i || '.';
raise notice 'LOOP: %', i;
END LOOP;
END $$;
NOTICE:  LOOP: 1
NOTICE:  LOOP: 2
NOTICE:  LOOP: 3
NOTICE:  LOOP: 4
NOTICE:  LOOP: 5
DO
postgres=# SELECT * FROM million LIMIT 5;
         name
----------------------
 4258ff9f881.2.3.4.5.
 ba07073eed1.2.3.4.5.
 dcfc85cbf81.2.3.4.5.
 9a3cb468311.2.3.4.5.
 02f15a5b8c1.2.3.4.5.
(5 rows)

postgres=#
```

4. Посомтрим на кол-во мертвых строк и когда приходил автовакум
```bash
postgres=# SELECT relname, n_live_tup, n_dead_tup, trunc(100*n_dead_tup/(n_live_tup+1))::float "ratio%", last_autovacuum FROM pg_stat_user_tables WHERE relname = 'million';
 relname | n_live_tup | n_dead_tup | ratio% |        last_autovacuum
---------+------------+------------+--------+-------------------------------
 million |     993927 |          0 |      0 | 2024-10-12 15:27:57.578746+03
(1 row)

postgres=#
```
На мой взгляд результат закономерный, потому что не сразу получилось проверить когда запускался автовакумм(пошел искать в лекциях правильный запрос),
видимо за это время он как раз таки сработал
5. Раз количество мертвых строк равно нулю, не будем ждать когда придёт автовакуум
6. Обновим все строчки и сразу посмотрим размер мертвых
```bash
postgres=# DO $$
BEGIN
FOR i in 1..5 LOOP
UPDATE million SET name = name || i || '.';
raise notice 'LOOP: %', i;
END LOOP;
END $$;
NOTICE:  LOOP: 1
NOTICE:  LOOP: 2
NOTICE:  LOOP: 3
NOTICE:  LOOP: 4
NOTICE:  LOOP: 5
DO
postgres=# SELECT relname, n_live_tup, n_dead_tup, trunc(100*n_dead_tup/(n_live_tup+1))::float "ratio%", last_autovacuum FROM pg_stat_user_tables WHERE relname = 'million';
 relname | n_live_tup | n_dead_tup | ratio% |        last_autovacuum
---------+------------+------------+--------+-------------------------------
 million |     993927 |    5000000 |    503 | 2024-10-12 15:27:57.578746+03
(1 row)
```
7. Размер таблицы, но кажется автовакуум уже почистил
```bash
postgres=# \d+
                                          List of relations
 Schema |     Name     |   Type   |  Owner   | Persistence | Access method |    Size    | Description
--------+--------------+----------+----------+-------------+---------------+------------+-------------
 public | goods        | table    | postgres | permanent   | heap          | 16 kB      |
 public | goods_id_seq | sequence | postgres | permanent   |               | 8192 bytes |
 public | million      | table    | postgres | permanent   | heap          | 329 MB     |
(3 rows)

postgres=# SELECT relname, n_live_tup, n_dead_tup, trunc(100*n_dead_tup/(n_live_tup+1))::float "ratio%", last_autovacuum FROM pg_stat_user_tables WHERE relname = 'million';
 relname | n_live_tup | n_dead_tup | ratio% |        last_autovacuum
---------+------------+------------+--------+-------------------------------
 million |    1008533 |          0 |      0 | 2024-10-12 15:41:58.772509+03
(1 row)

postgres=# \d+
                                          List of relations
 Schema |     Name     |   Type   |  Owner   | Persistence | Access method |    Size    | Description
--------+--------------+----------+----------+-------------+---------------+------------+-------------
 public | goods        | table    | postgres | permanent   | heap          | 16 kB      |
 public | goods_id_seq | sequence | postgres | permanent   |               | 8192 bytes |
 public | million      | table    | postgres | permanent   | heap          | 329 MB     |
(3 rows)
```
8. Отключим автовакуум
```bash
postgres=# ALTER TABLE million SET (autovacuum_enabled = off);
ALTER TABLE
```
9. Обновим 10 раз строчки
```bash
postgres=# DO $$
BEGIN
FOR i in  1..10 LOOP
UPDATE million SET name = name || i || '.';
raise notice 'LOOP: %', i;
END LOOP;
END $$;
NOTICE:  LOOP: 1
NOTICE:  LOOP: 2
NOTICE:  LOOP: 3
NOTICE:  LOOP: 4
NOTICE:  LOOP: 5
NOTICE:  LOOP: 6
NOTICE:  LOOP: 7
NOTICE:  LOOP: 8
NOTICE:  LOOP: 9
NOTICE:  LOOP: 10
DO
postgres=#
```
10. Посмотрим размер файла
```bash
postgres=# SELECT relname, n_live_tup, n_dead_tup, trunc(100*n_dead_tup/(n_live_tup+1))::float "ratio%", last_autovacuum FROM pg_stat_user_tables WHERE relname = 'million';
 relname | n_live_tup | n_dead_tup | ratio% |        last_autovacuum
---------+------------+------------+--------+-------------------------------
 million |    1008533 |   10000000 |    991 | 2024-10-12 15:41:58.772509+03
(1 row)

postgres=# \d+
                                          List of relations
 Schema |     Name     |   Type   |  Owner   | Persistence | Access method |    Size    | Description
--------+--------------+----------+----------+-------------+---------------+------------+-------------
 public | goods        | table    | postgres | permanent   | heap          | 16 kB      |
 public | goods_id_seq | sequence | postgres | permanent   |               | 8192 bytes |
 public | million      | table    | postgres | permanent   | heap          | 771 MB     |
(3 rows)

postgres=#
```

11. Понятно, что при отключении автовакуума у нас перестало рабоать очищение неиспользуемых строк.
Более того так как postgres работает на основе подхода copy-on-write, то мы получили закономерное число мертвых строк - 10 млн, 10 раз обновили по миллиону, при этом без очистки неиспользуемой памяти
12. Вкл
```bash
postgres=# ALTER TABLE million SET (autovacuum_enabled = on);
ALTER TABLE
postgres=# SELECT relname, n_live_tup, n_dead_tup, trunc(100*n_dead_tup/(n_live_tup+1))::float "ratio%", last_autovacuum FROM pg_stat_user_tables WHERE relname = 'million';
 relname | n_live_tup | n_dead_tup | ratio% |        last_autovacuum
---------+------------+------------+--------+-------------------------------
 million |    1024612 |          0 |      0 | 2024-10-12 17:07:18.840197+03
(1 row)

postgres=#

```


