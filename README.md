# Домашнее задание к занятию «Репликация и масштабирование. Часть 1»  - `Стрельцов Владимир`

## Задание 1

На лекции рассматривались режимы репликации master-slave, master-master, опишите их различия.

*Ответить в свободной форме.*

### Репликация Master-Slave
#### Плюсы
- Клиенты производят чтение со слейвов, не нагружая чтением мастер.
- Резервное копирование всей базы данных не сильно влияет на мастер.
- Слейвы могут быть переведены в автономный режим и синхронизированы с мастером без простоев.
#### Минусы
- В случае неудачи подчиненный должен быть повышен до ведущего, чтобы занять его место.
- Время простоя при сбое master
- Слейвы должны быть кратно мощнее мастера, чтобы не распадалась репликация

### Репликация Master-Master
#### Плюсы
- Клиенты могут считывать данные с любого мастера
- Распределяет сетевую нагрузку на запись между мастерами
- Простой, автоматический и быстрый переход на другой ресурс
#### Минусы
- Согласовано только при идеальной работе без сбоев.
- В случае сбоя репликации ждут большие проблемы с целостностью данных
- Настроить и развернуть чуть сложнее, чем master-slave (одна дополнительная команда на местере)


## Задание 2

Выполните конфигурацию master-slave репликации, примером можно пользоваться из лекции.

*Приложите скриншоты конфигурации, выполнения работы: состояния и режимы работы серверов.*

**Ответ.**

1.  Добавляем репозиторий с офф. [сайта MySQL](https://dev.mysql.com/downloads/repo/apt/) на обоих серверах

2. Устанавливаем MySQL на обоих серверах
```
apt update && apt install mysql-server mysql-client
```
3.1. Правим конфиг на мастере:
`nano /etc/mysql/mysql.conf.d/mysqld.cnf`
```
[mysqld]
pid-file        = /var/run/mysqld/mysqld.pid
socket          = /var/run/mysqld/mysqld.sock
datadir         = /var/lib/mysql
log-error       = /var/log/mysql/error.log

server-id       = 1
log_bin         = /var/log/mysql/mysql.log
bind-address    = 0.0.0.0
```
3.2. Правим конфиг на слейве:
`nano /etc/mysql/mysql.conf.d/mysqld.cnf`
```
[mysqld]
pid-file        = /var/run/mysqld/mysqld.pid
socket          = /var/run/mysqld/mysqld.sock
datadir         = /var/lib/mysql
log-error       = /var/log/mysql/error.log

server-id       = 2
log_bin         = /var/log/mysql/mysql.log
bind-address    = 0.0.0.0
```
4. Делаем инициализацию MySQL на обоих серверах
```
mysql -I
```
5. Создаем пользователя для репликаций на обоих серверах
```
CREATE USER 'replication'@'%' IDENTIFIED WITH mysql_native_password BY 'PA$$WORD';
GRANT REPLICATION SLAVE ON *.* TO 'replication'@'%';
 ```
6. На местере проверяем статус
```
SHOW MASTER STATUS;
```
7. На слейве добавляем реплиацию с мастера
```
CHANGE MASTER TO MASTER_HOST='192.168.7.110', MASTER_USER='replication', MASTER_PASSWORD='My_PA55W0RD', MASTER_LOG_FILE='mysql.000001', MASTER_LOG_POS=157;
```
8. На слейве запускаем репликацию
```
START SLAVE;
```
9. На слейве проверяем статус
```
SHOW SLAVE STATUS\G;
```
10. Проверка. Добавляем базы на местере `CREATE DATABASE netol1` и проверяем на слейве `SHOW DATABASES;`. Если на слейве отображаются базы созданные на местере, то репликация завершена.

- [скриншот](img/2023-11-07_16-55-15.png)
- [скриншот](img/2023-11-07_17-01-09.png)
- [скриншот](img/2023-11-07_17-02-21.png)

