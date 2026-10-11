# Проверка PostgreSQL

В этом файле собраны результаты проверок после запуска плейбука.

## 1. Состояние служб
```text
vm1 | CHANGED | rc=0 >>
active
active
```

## 2. Автозапуск служб
```text
vm1 | CHANGED | rc=0 >>
enabled
enabled
```

## 3. Проверка SELECT 1
```text
vm1 | CHANGED | rc=0 >>
 ?column? 
----------
        1
(1 row)
```

## 4. Список баз данных
```text
vm1 | CHANGED | rc=0 >>
                                                   List of databases
   Name    |  Owner   | Encoding | Locale Provider | Collate |  Ctype  | ICU Locale | ICU Rules |   Access privileges   
-----------+----------+----------+-----------------+---------+---------+------------+-----------+-----------------------
 app_db    | app_user | UTF8     | libc            | C.UTF-8 | C.UTF-8 |            |           | 
 postgres  | postgres | UTF8     | libc            | C.UTF-8 | C.UTF-8 |            |           | 
 template0 | postgres | UTF8     | libc            | C.UTF-8 | C.UTF-8 |            |           | =c/postgres          +
           |          |          |                 |         |         |            |           | postgres=CTc/postgres
 template1 | postgres | UTF8     | libc            | C.UTF-8 | C.UTF-8 |            |           | =c/postgres          +
           |          |          |                 |         |         |            |           | postgres=CTc/postgres
(4 rows)
```

## 5. Список пользователей
```text
vm1 | CHANGED | rc=0 >>
                                 List of roles
     Role name     |                         Attributes                         
-------------------+------------------------------------------------------------
 app_user          | 
 postgres          | Superuser, Create role, Create DB, Replication, Bypass RLS
 postgres_exporter | 
```

## 6. Проверка экспортера
```text
vm1 | CHANGED | rc=0 >>
pg_up 1
```

## 7. Проверка через Ansible URI
```text
pg_up 1
```
