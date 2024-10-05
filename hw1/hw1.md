# Домашнее задание №1

## Задание

1. Развернуть ВМ (Linux) с PostgreSQL

2. Залить Тайские перевозки
   https://github.com/aeuge/postgres16book/tree/main/database

3. Посчитать количество поездок - `select count(*) from book.tickets;`

## Ход работы

1. Развернул ВМ (Linux) с PostgreSQL
2. Залил перевозки thai_small
3. Результат:
```bash
thai= select count(*) from book.tickets;
  count
---------
 5185505
(1 row)
```