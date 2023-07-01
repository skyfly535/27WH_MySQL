# MySQL репликация.

Задание:

- востановить из дампа приложенную к материалам задания базу данных;

- настроить GTID репликацию master-slave определенного набора таблиц;

- осуществить настройку стенда при помощи связки Vagrant/Ansible.

Цель:

Получить практические навыки в:

- снятии резервныч копии;

- восстанавливании базы после сбоя;

- настраивании master-slave репликации;

- настройке инфраструктуры с помощью манифестов и конфигураций.

Отточить навыки использования ansible/vagrant/docker.

## Развертывание стенда с инфраструктурой для MySQL репликации.

Стэнд состоит из хостовой машины под управлением ОС Ubuntu 20.04 на которой развернут Ansible и виртуальных машин `centos/7`.

Виртуальная машина с именем: `master` выполняет роль сервера `MySQL` master  репликации.

Виртуальная машина с именем: `slave` выполняет роль сервера `MySQL` slave  репликации.

Разворачиваем инфраструктуру в Vagrant исключительно через Ansible.

Все коментарии по каждому блоку указаны в тексте Playbook - `mysql.yml`.

Каталоги `conf`, `dump`, `group_vars`. `vagrant` необходимо поместить в каталог с Playbook `mysql.yml` и Vagranfile.

Выполняем установку стенда:

```
vagrant up
```
## Проверяем работу стенда

После завершения развертывания стенда, заходим на `slave` сервер 

```
vagrant ssh slave
```
заходим в mysql и смотрим `SLAVE STATUS`  
```
mysql> SHOW SLAVE STATUS\G
```
вывод команды:
```
[vagrant@slave ~]$ mysql -uroot -p'Passw0rdSQL$'
mysql: [Warning] Using a password on the command line interface can be insecure.
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 10
Server version: 5.7.42-46-log Percona Server (GPL), Release 46, Revision e1995a8bb71

Copyright (c) 2009-2023 Percona LLC and/or its affiliates
Copyright (c) 2000, 2023, Oracle and/or its affiliates.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> SHOW SLAVE STATUS\G
*************************** 1. row ***************************
               Slave_IO_State: Waiting for master to send event
                  Master_Host: 192.168.11.150
                  Master_User: repl
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: mysql-bin.000003
          Read_Master_Log_Pos: 194
               Relay_Log_File: slave-relay-bin.000002
                Relay_Log_Pos: 367
        Relay_Master_Log_File: mysql-bin.000003
             Slave_IO_Running: Yes
            Slave_SQL_Running: Yes
              Replicate_Do_DB: 
          Replicate_Ignore_DB: 
           Replicate_Do_Table: 
       Replicate_Ignore_Table: bet.events_on_demand,bet.v_same_event
      Replicate_Wild_Do_Table: 
  Replicate_Wild_Ignore_Table: 
                   Last_Errno: 0
                   Last_Error: 
                 Skip_Counter: 0
          Exec_Master_Log_Pos: 194
              Relay_Log_Space: 574
              Until_Condition: None
               Until_Log_File: 
                Until_Log_Pos: 0
           Master_SSL_Allowed: No
           Master_SSL_CA_File: 
           Master_SSL_CA_Path: 
              Master_SSL_Cert: 
            Master_SSL_Cipher: 
               Master_SSL_Key: 
        Seconds_Behind_Master: 0
Master_SSL_Verify_Server_Cert: No
                Last_IO_Errno: 0
                Last_IO_Error: 
               Last_SQL_Errno: 0
               Last_SQL_Error: 
  Replicate_Ignore_Server_Ids: 
             Master_Server_Id: 1
                  Master_UUID: 2a962337-1693-11ee-89f7-5254004d77d3
             Master_Info_File: /var/lib/mysql/master.info
                    SQL_Delay: 0
          SQL_Remaining_Delay: NULL
      Slave_SQL_Running_State: Slave has read all relay log; waiting for more updates
           Master_Retry_Count: 86400
                  Master_Bind: 
      Last_IO_Error_Timestamp: 
     Last_SQL_Error_Timestamp: 
               Master_SSL_Crl: 
           Master_SSL_Crlpath: 
           Retrieved_Gtid_Set: 
            Executed_Gtid_Set: 2a962337-1693-11ee-89f7-5254004d77d3:1-40
                Auto_Position: 1
         Replicate_Rewrite_DB: 
                 Channel_Name: 
           Master_TLS_Version: 
1 row in set (0.00 sec)
```
заходим на `master` сервер 

```
vagrant ssh master
```
заходим в mysq юзаем БД `bet` и проверяем таблицу `bookmaker`

```
[vagrant@master ~]$ mysql -uroot -p'Passw0rdSQL$'
mysql: [Warning] Using a password on the command line interface can be insecure.
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 3
Server version: 5.7.42-46-log Percona Server (GPL), Release 46, Revision e1995a8bb71

Copyright (c) 2009-2023 Percona LLC and/or its affiliates
Copyright (c) 2000, 2023, Oracle and/or its affiliates.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> USE bet;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
mysql> SELECT * FROM bookmaker;
+----+----------------+
| id | bookmaker_name |
+----+----------------+
|  4 | betway         |
|  5 | bwin           |
|  6 | ladbrokes      |
|  3 | unibet         |
+----+----------------+
4 rows in set (0.00 sec)
```
редактируем таблицу `bookmaker` и ещё раз проверяем

```
mysql> INSERT INTO bookmaker (id,bookmaker_name) VALUES(1,'sportloto');
Query OK, 1 row affected (0.00 sec)

mysql> SELECT * FROM bookmaker;
+----+----------------+
| id | bookmaker_name |
+----+----------------+
|  4 | betway         |
|  5 | bwin           |
|  6 | ladbrokes      |
|  1 | sportloto      |
|  3 | unibet         |
+----+----------------+
5 rows in set (0.00 sec)
```
После этого идем на  `slave` и смотрим как там отбились изменения с мастера

```
Database changed
mysql> SELECT * FROM bookmaker;
+----+----------------+
| id | bookmaker_name |
+----+----------------+
|  4 | betway         |
|  5 | bwin           |
|  6 | ladbrokes      |
|  1 | sportloto      |
|  3 | unibet         |
+----+----------------+
5 rows in set (0.00 sec)
```