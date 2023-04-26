Задание 1:Запустить контейнер с БД, отличной от mariaDB, используя инструкции на сайте: https://hub.docker.com/

Задание:

1 Запустить контейнер с БД, отличной от mariaDB, используя инструкции на сайте: https://hub.docker.com/

2 Добавить в контейнер hostname такой же, как hostname системы через переменную

3 Заполнить БД данными через консоль

4 Запустить phpmyadmin (в контейнере) и через веб проверить, что все введенные данные доступны

docker run --name test_db --hostname ${HOSTNAME} -e POSTGRES_USER=testDB -e POSTGRES_PASSWORD=testDB -e POSTGRES_DB=testDB -v /test-db://docker-entrypoint-initdb.d postgres:15.2

root@ivan-server:~# docker ps -a
CONTAINER ID   IMAGE           COMMAND                  CREATED              STATUS                      PORTS     NAMES
d3cdf5ef04ae   postgres:15.2   "docker-entrypoint.s…"   About a minute ago   Exited (0) 37 seconds ago             test_db

root@ivan-server:~# docker start test_db
test_db

root@ivan-server:~# docker ps -a
CONTAINER ID   IMAGE           COMMAND                  CREATED         STATUS         PORTS      NAMES
d3cdf5ef04ae   postgres:15.2   "docker-entrypoint.s…"   2 minutes ago   Up 5 seconds   5432/tcp   test_db


root@ivan-server:~# docker exec -it test_db bash

root@ivan-server:/# psql -d testDB -U testDB

testDB=# CREATE TABLE test_table
    (id integer NOT NULL,
    data text NOT NULL,
    PRIMARY KEY (id)
    );
CREATE TABLE
testDB=# \d

          List of relations
 Schema |    Name    | Type  | Owner  
--------+------------+-------+--------
 public | test_table | table | testDB
(1 row)

testDB=# INSERT INTO test_table 
VALUES (1, 'AAAA');
INSERT 0 1
testDB=# \q

docker run --name my-pgadmin -p 82:80 -e 'PGADMIN_DEFAULT_EMAIL=user@domain.local' -e 'PGADMIN_DEFAULT_PASSWORD=1234'-d dpage/pgadmin4

root@ivan-server:~# docker run --name my-pgadmin -p 82:80 -e 'PGADMIN_DEFAULT_EMAIL=user@domain.com' -e 'PGADMIN_DEFAULT_PASSWORD=SuperSecret' -d dpage/pgadmin4

CONTAINER ID   IMAGE            COMMAND                  CREATED             STATUS          PORTS                                        NAMES
9a8a4fb2248c   dpage/pgadmin4   "/entrypoint.sh"         10 seconds ago      Up 9 seconds    443/tcp, 0.0.0.0:82->80/tcp, :::82->80/tcp   my-pgadmin
d3cdf5ef04ae   postgres:15.2    "docker-entrypoint.s…"   About an hour ago   Up 59 minutes   5432/tcp                                     test_db

pgAdmin