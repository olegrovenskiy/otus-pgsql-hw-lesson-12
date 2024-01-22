# ДЗ otus-pgsql-hw-lesson-12

### 1.  Создаем ВМ/докер c ПГ.

    bash-4.2$ systemctl status postgresql-15
    ● postgresql-15.service - PostgreSQL 15 database server
       Loaded: loaded (/usr/lib/systemd/system/postgresql-15.service; disabled; vendor preset: disabled)
       Active: active (running) since Mon 2024-01-22 01:54:03 EST; 1h 54min ago
         Docs: https://www.postgresql.org/docs/15/static/
      Process: 115194 ExecStartPre=/usr/pgsql-15/bin/postgresql-15-check-db-dir ${PGDATA} (code=exited, status=0/SUCCESS)
     Main PID: 115200 (postmaster)

### 2.  Создаем БД, схему и в ней таблицу.

    postgres=# CREATE DATABASE otus_hw_12;
    CREATE DATABASE
    postgres=# \l
                                                      List of databases
        Name    |  Owner   | Encoding |   Collate   |    Ctype    | ICU Locale | Locale Provider |   Access privileges
    ------------+----------+----------+-------------+-------------+------------+-----------------+-----------------------
     otus_hw_12 | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 |            | libc            |
     postgres   | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 |            | libc            |
     template0  | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 |            | libc            | =c/postgres          +
                |          |          |             |             |            |                 | postgres=CTc/postgres
     template1  | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 |            | libc            | =c/postgres          +
                |          |          |             |             |            |                 | postgres=CTc/postgres
    (4 rows)
    
    postgres=#


    postgres=# \c otus_hw_12
    You are now connected to database "otus_hw_12" as user "postgres".
    otus_hw_12=#


    otus_hw_12=# \dn
          List of schemas
      Name  |       Owner
    --------+-------------------
     public | pg_database_owner
    (1 row)
    
    otus_hw_12=#
    otus_hw_12=# CREATE SCHEMA myschema;
    CREATE SCHEMA
    otus_hw_12=# \dn
           List of schemas
       Name   |       Owner
    ----------+-------------------
     myschema | postgres
     public   | pg_database_owner
    (2 rows)
    
    otus_hw_12=#

    otus_hw_12=# CREATE TABLE myschema.student(id serial, fio char(100));
    CREATE TABLE


### 3.  Заполним таблицы автосгенерированными 100 записями.

    otus_hw_12=# INSERT INTO myschema.student(fio) SELECT 'noname' FROM generate_series(1,100);
    INSERT 0 100
    otus_hw_12=# select * from myschema.student limit 5;
     id |                                                 fio
    ----+------------------------------------------------------------------------------------------------------
      1 | noname
      2 | noname
      3 | noname
      4 | noname
      5 | noname
    (5 rows)
    
    otus_hw_12=#


### 4.  Под линукс пользователем Postgres создадим каталог для бэкапов



### 5.  Сделаем логический бэкап используя утилиту COPY


### 6.  Восстановим в 2 таблицу данные из бэкапа.


### 7.  Используя утилиту pg_dump создадим бэкап в кастомном сжатом формате двух таблиц


### 8.  Используя утилиту pg_restore восстановим в новую БД только вторую таблицу!
