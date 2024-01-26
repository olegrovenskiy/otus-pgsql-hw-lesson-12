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

        bash-4.2$ cd /var/lib/pgsql/15/backups/
        bash-4.2$ ls -l
        total 0
        
        bash-4.2$ cd ..
        
        bash-4.2$ ls -la
        total 8
        drwx------.  4 postgres postgres   51 Dec 29 09:41 .
        drwx------.  3 postgres postgres   79 Nov  8 15:34 ..
        drwx------.  2 postgres postgres    6 Nov  8 15:34 backups
        drwx------. 20 postgres postgres 4096 Jan 26 00:00 data
        -rw-------.  1 postgres postgres  924 Dec 29 05:49 initdb.log
        bash-4.2$ ^C


### 5.  Сделаем логический бэкап используя утилиту COPY

        otus_hw_12=# \copy myschema.student to '/var/lib/pgsql/15/backups/backup_copy.sql';
        COPY 100
        otus_hw_12=# 

Проверим что файл бекапа появилсф в директории

        bash-4.2$ cd ./backups/
        bash-4.2$ ls -la
        total 12
        drwx------. 2 postgres postgres    29 Jan 26 10:34 .
        drwx------. 4 postgres postgres    51 Dec 29 09:41 ..
        -rw-r--r--. 1 postgres postgres 10392 Jan 26 10:34 backup_copy.sql
        bash-4.2$ ^C


### 6.  Восстановим в 2 таблицу данные из бэкапа.

создадим таблицу 2, востановим в неё данные из бекапа и проверим селектом

        otus_hw_12=# CREATE TABLE myschema.table2(id serial, fio char(100));
        CREATE TABLE
        otus_hw_12=# select * from  myschema.table2;
         id | fio
        ----+-----
        (0 rows)
        
        otus_hw_12=# \copy myschema.table2 from '/var/lib/pgsql/15/backups/backup_copy.sql';
        COPY 100
        otus_hw_12=# select * from  myschema.table2 limit 5;
         id |                                                 fio
        ----+------------------------------------------------------------------------------------------------------
          1 | noname
          2 | noname
          3 | noname
          4 | noname
          5 | noname
        (5 rows)
        
        otus_hw_12=# ^C

### 7.  Используя утилиту pg_dump создадим бэкап в кастомном сжатом формате двух таблиц

сначала протестировал работу утилиты 

pg_dump -d otus_hw_12 --create > /var/lib/pgsql/15/backups/backup_dump.sql

pg_dump -t myschema.student otus_hw_12 > /var/lib/pgsql/15/backups/backup_dump_student.sql

------------------------------------------

Потом подготовил команды

pg_dump -t myschema.student otus_hw_12 --create | gzip > /var/lib/pgsql/15/backups/backup_dump_student.gz
pg_dump -t myschema.table2 otus_hw_12 --create | gzip > /var/lib/pgsql/15/backups/backup_dump_table2.gz

с данными командами получил ту же проблему при востановлении что и на занятии


        pg_restore: hint: Try "pg_restore --help" for more information.
        bash-4.2$ pg_restore -d newdb /var/lib/pgsql/15/backups/backup_dump_student.gz  pg_restore: error: input file does not appear to be a valid archive

новые команды

pg_dump --t myschema.student otus_hw_12 --create -Fd -f /var/lib/pgsql/15/backups/test.dir
pg_dump --t myschema.table2 otus_hw_12 --create -Fd -f /var/lib/pgsql/15/backups/test.dir2

Выполнение

        bash-4.2$ pg_dump -t myschema.student otus_hw_12 --create | gzip > /var/lib/pgsql/15/backups/backup_dump_student.gz
        Password:
        bash-4.2$ pg_dump -t myschema.table2 otus_hw_12 --create | gzip > /var/lib/pgsql/15/backups/backup_dump_table2.gz
        Password:
        bash-4.2$

Проверка наличия сжатых файлов бекапов таблиц
        
        bash-4.2$ ls -l
        total 56
        -rw-r--r--. 1 postgres postgres 10392 Jan 26 10:34 backup_copy.sql
        -rw-r--r--. 1 postgres postgres 24294 Jan 26 11:27 backup_dump.sql
        -rw-r--r--. 1 postgres postgres  1125 Jan 26 11:48 backup_dump_student.gz
        -rw-r--r--. 1 postgres postgres 12070 Jan 26 11:42 backup_dump_student.sql
        -rw-r--r--. 1 postgres postgres  1117 Jan 26 11:50 backup_dump_table2.gz
        bash-4.2$

=====================
Выполнение с директорией

        bash-4.2$ pg_dump --t myschema.student otus_hw_12 --create -Fd -f /var/lib/pgsql/15/backups/test.dir
        Password:
        bash-4.2$ ^C
        
        bash-4.2$ pg_dump --t myschema.table2 otus_hw_12 --create -Fd -f /var/lib/pgsql/15/backups/test.dir
        pg_dump: error: could not create directory "/var/lib/pgsql/15/backups/test.dir": File exists
        
        bash-4.2$ pg_dump --t myschema.table2 otus_hw_12 --create -Fd -f /var/lib/pgsql/15/backups/test.dir2
        Password:
        bash-4.2$ ls -l
        total 56
        -rw-r--r--. 1 postgres postgres 10392 Jan 26 10:34 backup_copy.sql
        -rw-r--r--. 1 postgres postgres 24294 Jan 26 11:27 backup_dump.sql
        -rw-r--r--. 1 postgres postgres  1125 Jan 26 11:48 backup_dump_student.gz
        -rw-r--r--. 1 postgres postgres 12070 Jan 26 11:42 backup_dump_student.sql
        -rw-r--r--. 1 postgres postgres  1117 Jan 26 11:50 backup_dump_table2.gz
        drwx------. 2 postgres postgres    40 Jan 26 12:09 test.dir
        drwx------. 2 postgres postgres    40 Jan 26 12:12 test.dir2
        bash-4.2$
        



### 8.  Используя утилиту pg_restore восстановим в новую БД только вторую таблицу!

Создадам новую БД

        otus_hw_12=# \l
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
        
        otus_hw_12=# CREATE DATABASE NewDB;
        CREATE DATABASE
        otus_hw_12=# \l
                                                          List of databases
            Name    |  Owner   | Encoding |   Collate   |    Ctype    | ICU Locale | Locale Provider |   Access privileges
        ------------+----------+----------+-------------+-------------+------------+-----------------+-----------------------
         newdb      | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 |            | libc            |
         otus_hw_12 | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 |            | libc            |
         postgres   | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 |            | libc            |
         template0  | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 |            | libc            | =c/postgres          +
                    |          |          |             |             |            |                 | postgres=CTc/postgres
         template1  | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 |            | libc            | =c/postgres          +
                    |          |          |             |             |            |                 | postgres=CTc/postgres
        (5 rows)
        
        otus_hw_12=#



        newdb=#
        newdb=# \dn
              List of schemas
          Name  |       Owner
        --------+-------------------
         public | pg_database_owner
        (1 row)
        
        newdb=#
        newdb=#
        newdb=#
        newdb=# \dt
        Did not find any relations.
        newdb=#

        
подготовлена команда востановления

pg_restore -d newdb /var/lib/pgsql/15/backups/test.dir

        bash-4.2$ pg_restore -d newdb /var/lib/pgsql/15/backups/test.dir
        Password:
        pg_restore: error: could not execute query: ERROR:  schema "myschema" does not e                                                                             xist
        LINE 1: CREATE TABLE myschema.student (
                             ^
        Command was: CREATE TABLE myschema.student (
            id integer NOT NULL,
            fio character(100)
        );
        
        
        pg_restore: error: could not execute query: ERROR:  schema "myschema" does not e                                                                             xist
        Command was: ALTER TABLE myschema.student OWNER TO postgres;
        
        pg_restore: error: could not execute query: ERROR:  schema "myschema" does not e                                                                             xist
        Command was: CREATE SEQUENCE myschema.student_id_seq
            AS integer
            START WITH 1
            INCREMENT BY 1
            NO MINVALUE
            NO MAXVALUE
            CACHE 1;
        
        
        pg_restore: error: could not execute query: ERROR:  schema "myschema" does not e                                                                             xist
        Command was: ALTER TABLE myschema.student_id_seq OWNER TO postgres;
        
        pg_restore: error: could not execute query: ERROR:  schema "myschema" does not e                                                                             xist
        Command was: ALTER SEQUENCE myschema.student_id_seq OWNED BY myschema.student.id                                                                             ;
        
        
        pg_restore: error: could not execute query: ERROR:  schema "myschema" does not e                                                                             xist
        Command was: ALTER TABLE ONLY myschema.student ALTER COLUMN id SET DEFAULT nextv                                                                                             al('myschema.student_id_seq'::regclass);
        
        pg_restore: error: could not execute query: ERROR:  schema "myschema" does not e                                                                             xist
        Command was: COPY myschema.student (id, fio) FROM stdin;
        pg_restore: error: could not execute query: ERROR:  schema "myschema" does not e                                                                             xist
        LINE 1: SELECT pg_catalog.setval('myschema.student_id_seq', 100, tru...
                                         ^
        Command was: SELECT pg_catalog.setval('myschema.student_id_seq', 100, true);
        pg_restore: warning: errors ignored on restore: 8
        bash-4.2$


получил ошибку, так как в новойбазе не создал схему
создаю схему

        newdb=# CREATE SCHEMA myschema;
        CREATE SCHEMA
        newdb=#
        newdb=#
        newdb=# \dt
        Did not find any relations.
        newdb=#
        newdb=#
        newdb=#
        newdb=# \dt myschema.*
                   List of relations
          Schema  |  Name   | Type  |  Owner
        ----------+---------+-------+----------
         myschema | student | table | postgres
        (1 row)



теперь востановление прошло без ошибки

        bash-4.2$ pg_restore -d newdb /var/lib/pgsql/15/backups/test.dir
        Password:
        bash-4.2$

Проверяю новую базу

        newdb=# \dt myschema.*
                   List of relations
          Schema  |  Name   | Type  |  Owner
        ----------+---------+-------+----------
         myschema | student | table | postgres
        (1 row)
        
        newdb=# select * from myschema.student limit 5;
         id |                                                 fio
        ----+------------------------------------------------------------------------------------------------------
          1 | noname
          2 | noname
          3 | noname
          4 | noname
          5 | noname
        (5 rows)


таблица востановлена в новую базу

