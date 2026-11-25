# Лабораторна робота №2 — Системи віртуалізації та контейнеризації

**Студент:** Авраменко Єгор Іванович
**Група:** 6.1214-пі-2 (3 курс, Інженерія програмного забезпечення)
**Варіант №1:** fedora:43 (bash), nginx:1.30, mysql:8.4 (MYSQL_ROOT_PASSWORD)

---

## 1. Запуск базового образу ОС (fedora:43) без інтерактивного режиму та з оболонкою bash

    $ docker run --name fedora-test fedora:43
    $ docker ps -a
    CONTAINER ID   IMAGE       COMMAND       CREATED         STATUS                            PORTS     NAMES
    6b6201988ad9   fedora:43   "/bin/bash"   1 second ago    Exited (0) Less than a second ago             fedora-test

    $ docker run -it --name fedora-bash fedora:43 bash
    [root@355a74b6d595 /]# cat /etc/os-release
    NAME="Fedora Linux"
    VERSION="43 (Container Image)"
    ID=fedora
    VERSION_ID=43
    PRETTY_NAME="Fedora Linux 43 (Container Image)"
    VARIANT="Container Image"
    VARIANT_ID=container
    [root@355a74b6d595 /]# exit

---

## 2. Запуск вебсервера nginx:1.30 з прокиданням порту 8080:80 та перевірка відповіді

    $ docker run -d --name web-nginx -p 8080:80 nginx:1.30
    0fd3623e47f4e53ee8951f62430a05e0086bc5eef29171684e7c9ba621924275

    $ docker ps
    CONTAINER ID   IMAGE        COMMAND                  CREATED         STATUS                  PORTS                                     NAMES
    0fd3623e47f4   nginx:1.30   "/docker-entrypoint.…"   1 second ago    Up Less than a second   0.0.0.0:8080->80/tcp, [::]:8080->80/tcp   web-nginx

    $ curl -I http://127.0.0.1:8080
    HTTP/1.1 200 OK
    Server: nginx/1.30.8
    Content-Type: text/html
    Content-Length: 896

Монтування власної HTML-сторінки з персональними даними студента на http://localhost:8080:

    $ docker run -d --name web-nginx -p 8080:80 -v /home/kali/lab2/index.html:/usr/share/nginx/html/index.html:ro nginx:1.30
    f4389c23df8d94221306c2284eccecf0427af9a7c6eec934e970d0cc2261cda0

---

## 3. Запуск СКБД mysql:8.4 зі змінною середовища MYSQL_ROOT_PASSWORD та одночасна робота трьох контейнерів

    $ docker run -d --name db-mysql -e MYSQL_ROOT_PASSWORD=rootpass mysql:8.4
    80d80004d92933133aa4c330802724080674770b802beff4499db6821a5f5af3

    $ docker run -dit --name fedora-os fedora:43 bash
    7c403e9f408d38a810cbeb41408fd6f8c8b3b93918ede1209b19f4d5ca9a2e9e

    $ docker ps
    CONTAINER ID   IMAGE        COMMAND                  CREATED              STATUS              PORTS                                     NAMES
    7c403e9f408d   fedora:43    "bash"                   17 seconds ago       Up 17 seconds                                                 fedora-os
    80d80004d929   mysql:8.4    "docker-entrypoint.s…"   About a minute ago   Up About a minute   3306/tcp, 33060/tcp                       db-mysql
    f4389c23df8d   nginx:1.30   "/docker-entrypoint.…"   About a minute ago   Up About a minute   0.0.0.0:8080->80/tcp, [::]:8080->80/tcp   web-nginx

Перевірка рядка готовності СКБД MySQL у журналі контейнера:

    $ docker logs db-mysql 2>&1 | grep "ready for connections"
    2026-11-25T19:34:23.276478Z 0 [System] [MY-010931] [Server] /usr/sbin/mysqld: ready for connections. Version: '8.4.13'  socket: '/var/run/mysqld/mysqld.sock'  port: 0  MySQL Community Server - GPL.
    2026-11-25T19:34:23.276613Z 0 [System] [MY-011323] [Server] X Plugin ready for connections. Socket: /var/run/mysqld/mysqlx.sock
    2026-11-25T19:34:38.720376Z 0 [System] [MY-011323] [Server] X Plugin ready for connections. Bind-address: '::' port: 33060, socket: /var/run/mysqld/mysqlx.sock
    2026-11-25T19:34:38.720883Z 0 [System] [MY-010931] [Server] /usr/sbin/mysqld: ready for connections. Version: '8.4.13'  socket: '/var/run/mysqld/mysqld.sock'  port: 3306  MySQL Community Server - GPL.

---

## 4. Зупинка, видалення контейнерів та перевірка збережених образів

    $ docker stop fedora-os web-nginx db-mysql
    fedora-os
    web-nginx
    db-mysql

    $ docker rm fedora-os web-nginx db-mysql
    fedora-os
    web-nginx
    db-mysql

    $ docker ps -a
    CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES

    $ docker images
    REPOSITORY   TAG       IMAGE ID       CREATED        SIZE
    nginx        1.30      77a4afeb99e0   39 hours ago   163MB
    mysql        8.4       0977972cb2d5   44 hours ago   803MB
    fedora       43        3866f820f690   3 weeks ago    182MB
