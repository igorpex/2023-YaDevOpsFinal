# Настройка базы
в качестве первой базы (до дедлайна) использовал готовую (managed) базу Postgres14, 2 ядра, 2 Гб.
Там можно менять конфигурацию.


```yml
shared_buffers дефолт 31232
это в блоках по 8 кб, то есть 249 856 Кб (244 Мб)
увеличил в 2 раза, то есть до 499 712 Кб (488 Мб)

max_connections было 200
изменил в 2 раза до 400
```
Сделал индексы для ID всех таблиц.

```bash
#Для ускорения обработки по id сделал индексы
CREATE UNIQUE INDEX customers_id_idx ON public.customers (id);
CREATE UNIQUE INDEX movies_id_idx ON public.movies (id);
CREATE UNIQUE INDEX sessions_id_idx ON public.sessions (id);
```
Этого хватило, чтобы пройти все тесты, кроме GET запросов по ID (и иногда вылетала отказоустойчивость).

Позже (после дедлайна) раскатал свою базу.

# Создание новой бд с нуля на Ubuntu 22 
## (это после дедлайна, можно не читать)

```bash
#обновляю
apt update
apt upgrade
#создаю пользователя чтобы не запускить под рутом (хотя ниже по тексту это не используется)
adduser master
usermod -aG sudo master

```

Переключаюсь на пользователя postgres :

```bash
su - postgres
```

Создаю Postgres пользователя

```bash
$ createuser gen_user
```

Создаю базу данных

```bash
$ createdb default_db
```

Запускаю psql

```bash
psql
```

Привилегии postgres пользователю

```sql
alter user gen_user with encrypted password '?BYeBZ$3($*B\*';
grant all privileges on database default_db to gen_user;
ALTER USER gen_user WITH SUPERUSER;
```

Проверка подключения

```bash
psql "host=localhost port=5432 user=gen_user dbname=default_db connect_timeout=10"
```

Осознал, что с миграция до дедлайна были проблемы.

Поэтому, установил по схеме выше локальный Postgres на тестовой VM с bingo, запустил миграцию локально, прошла сразу без ошибок.

Затем сделал бекап.
```bash
pg_dump default_db > /tmp/dumpfile
```
Скопировал на сервер и восстановил на удаленном сервере:

```bash
psql default_db < /tmp/dumpfile
```
Чтобы предоставить доступ снаружи, добавил адреса в pg_hba.conf.

```bash
sudo nano /etc/postgresql/16/main/pg_hba.conf
# добавил все адреса откуда нужен доступ к базе
host    all		all		95.165.15.156/32        scram-sha-256
host	all		all		188.225.47.118/32		scram-sha-256	
host	all		all		83.147.247.110/32		scram-sha-256
```

Этого оказалось недостаточно, при подключении по команде ниже была ошибка

```bash
psql "host=82.97.240.115 port=5432 user=gen_user dbname=default_db connect_timeout=10"
```

Ответ нашел здесь

https://stackoverflow.com/questions/38466190/cant-connect-to-postgresql-on-port-5432

You have to edit postgresql.conf file and change line with 'listen_addresses'.

```bash
sudo nano /etc/postgresql/16/main/postgresql.conf

#listen_addresses = 'localhost'		# what IP address(es) to listen on;
					# comma-separated list of addresses;
					# defaults to 'localhost'; use '*' for all
					# (change requires restart)
заменено на
listen_addresses = '*'
```
Также было в разборе ДЗ тут https://youtu.be/y1AlPYCaFkY?t=1635


# Поправил параметры производительности
```bash
sudo nano /etc/postgresql/16/main/postgresql.conf
```
```bash
#Было
shared_buffers = 128MB
# стало
shared_buffers = 1024MB (эта машина 4 Гб RAM)
#Было
max_connections = 100
стало
max_connections = 800

sudo service postgresql restart
```

Сделал индексы:

```sql
#Для ускорения обработки по id сделал индексы
CREATE UNIQUE INDEX customers_id_idx ON public.customers (id);
CREATE UNIQUE INDEX movies_id_idx ON public.movies (id);
CREATE UNIQUE INDEX sessions_id_idx ON public.sessions (id);
```

Для большинства тестов хватило.
Но запросы по ID так тесты и не прошли...
Чистил данные sessions с ID -1 и 0, ставил ограничения, делал совместный индекс, тоже не помогло.

С базами данных я недоразобрался. Репликация, пулинг, оптимизация требует внимания.
