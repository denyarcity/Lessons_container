Задание 1. Необходимо продемонстрировать изоляцию одного и того же приложения (как мы решили на семинаре - командного интерпретатора) в различных пространствах имен. Предоставить доказательства изоляции приложения там, где возможно.

Chroot
Создаем дерикторию: for_chroot_sh

root@ivan-server:~# mkdir for_chroot_sh
Копируем исполняемый файл в каталог for_chroot_sh

root@ivan-server:~# cp --parents /usr/bin/sh for_chroot_sh/
Просматриваем совместные библиотеки

root@ivan-server:~# ldd /usr/bin/sh
Нужные бибилотеки, пропускаем - linux-vdso.so.1

linux-vdso.so.1 (0x00007fff1efec000)
	libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x00007f63fd0e1000)
	/lib64/ld-linux-x86-64.so.2 (0x00007f63fd332000)
Копируем

root@ivan-server:~# cp --parents /lib/x86_64-linux-gnu/libc.so.6 for_chroot_sh/
root@ivan-server:~# cp --parents /lib64/ld-linux-x86-64.so.2 for_chroot_sh/
Запускаем

root@ivan-server:~# chroot for_chroot_sh/ /usr/bin/sh
# 
Рекурсивный просмотр for_chroot_sh

root@ivan-server:~# ll -R for_chroot_sh/
for_chroot_sh/:
total 20
drwxr-xr-x 5 root root 4096 мар 26 10:30 ./
drwx------ 5 root root 4096 мар 26 10:27 ../
drwxr-xr-x 3 root root 4096 мар 26 10:29 lib/
drwxr-xr-x 2 root root 4096 мар 26 10:30 lib64/
drwxr-xr-x 3 root root 4096 мар 26 10:28 usr/

for_chroot_sh/lib:
total 12
drwxr-xr-x 3 root root 4096 мар 26 10:29 ./
drwxr-xr-x 5 root root 4096 мар 26 10:30 ../
drwxr-xr-x 2 root root 4096 мар 26 10:29 x86_64-linux-gnu/

for_chroot_sh/lib/x86_64-linux-gnu:
total 2176
drwxr-xr-x 2 root root    4096 мар 26 10:29 ./
drwxr-xr-x 3 root root    4096 мар 26 10:29 ../
-rw-r--r-- 1 root root 2216304 мар 26 10:29 libc.so.6

for_chroot_sh/lib64:
total 244
drwxr-xr-x 2 root root   4096 мар 26 10:30 ./
drwxr-xr-x 5 root root   4096 мар 26 10:30 ../
-rwxr-xr-x 1 root root 240936 мар 26 10:30 ld-linux-x86-64.so.2*

for_chroot_sh/usr:
total 12
drwxr-xr-x 3 root root 4096 мар 26 10:28 ./
drwxr-xr-x 5 root root 4096 мар 26 10:30 ../
drwxr-xr-x 2 root root 4096 мар 26 10:28 bin/

for_chroot_sh/usr/bin:
total 132
drwxr-xr-x 2 root root   4096 мар 26 10:28 ./
drwxr-xr-x 3 root root   4096 мар 26 10:28 ../
-rwxr-xr-x 1 root root 125688 мар 26 10:28 sh*
Unshare
unshare -n -f -p --mount-proc /usr/bin/sh
-n – Создайте новое сетевое пространство имен. Если файл указан, то пространство имен становится постоянным, создавая привязку монтирования в файл

-f – Разветвить указанную программу как дочерний процесс unshare а не запускать его напрямую. Это удобно при создании новое пространство имен PID. Обратите внимание, что когда unshare ожидает дочерний процесс, то он игнорирует SIGINT и SIGTERM и не передает никаких сигналов ребенку. необходимо посылать сигналы дочернему процессу.

-p – Создайте новое пространство имен PID. Если файл указан, то пространство имен становится постоянным, создавая привязку монтирования в файл . (Создание постоянного пространства имен PID завершится ошибкой, если опция --fork также не указана.)

--mount-proc - Непосредственно перед запуском программы смонтируйте файловую систему proc по адресу точка монтирования (по умолчанию /proc ). Это полезно при создании новое пространство имен PID. Это также подразумевает создание нового монтирования пространство имен, поскольку в противном случае монтирование /proc могло бы испортить существующих программ в системе. Новая файловая система procявно смонтирован как частный (с помощью MS_PRIVATE | MS_REC ).

Запущеные процессы

# ps aux
USER         PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root           1  0.0  0.0   2888   956 pts/1    S    10:44   0:00 /usr/bin/sh
root           2  0.0  0.0  10068  1608 pts/1    R+   10:45   0:00 ps aux

Сетевые интерфейсы

# ip a
1: lo: <LOOPBACK> mtu 65536 qdisc noop state DOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
Список namespace

# lsns | grep 'net '
4026532387 net         3   1 root /usr/bin/sh
Задание 2*: Добавьте виртуальный интерфейс для дальнейшего взаимодействия. Мы будем использовать сетевое устройство virtual ethernet (или сокращённо veth). Устройства veth всегда создаются как пара устройств, связанных по принципу туннеля, так что сообщения, отправленные на одном конце, выходят из устройства на другом. Вы могли бы предположить, что мы могли бы легко иметь один конец в исходном network namespace, а другой — в нашем дочернем network namespace, а всё общение между пространствами имён network проходило бы через соответствующее оконечное устройство veth (и вы были бы правы).

Выполняем команды в исходном пространстве, параллельно после выполнения каждой команды смотрите что меняется:

ip link add veth0 type veth peer name veth1 (посмотрите ip addr , что появилось?)

root@ivan-server:~# ip link add veth0 type veth peer name veth1
root@ivan-server:~# ip a
1: lo: <LOOPBACK> mtu 65536 qdisc noop state DOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
2: veth1@veth0: <BROADCAST,MULTICAST,M-DOWN> mtu 1500 qdisc noop state DOWN group default qlen 1000
    link/ether ce:89:b6:78:ce:a2 brd ff:ff:ff:ff:ff:ff
3: veth0@veth1: <BROADCAST,MULTICAST,M-DOWN> mtu 1500 qdisc noop state DOWN group default qlen 1000
    link/ether 72:66:01:79:fd:82 brd ff:ff:ff:ff:ff:ff

Появмлось новый линк

ip link set veth1 netns testns123 (перенос устройства в новый net namespace, гляньте снова ip addr )

root@ivan-server:~# ip netns add testns123
root@ivan-server:~# ip link set veth1 netns testns123
root@ivan-server:~# ip a
1: lo: <LOOPBACK> mtu 65536 qdisc noop state DOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
3: veth0@if2: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN group default qlen 1000
    link/ether 72:66:01:79:fd:82 brd ff:ff:ff:ff:ff:ff link-netns testns123
(Задайте ip адрес на veth0)

root@ivan-server:~# ip addr add 10.0.0.1/24 dev veth0
root@ivan-server:~# ip a
1: lo: <LOOPBACK> mtu 65536 qdisc noop state DOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
3: veth0@if2: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN group default qlen 1000
    link/ether 72:66:01:79:fd:82 brd ff:ff:ff:ff:ff:ff link-netns testns123
    inet 10.0.0.1/24 scope global veth0
       valid_lft forever preferred_lft forever
ip link set dev veth0 up

root@ivan-server:~# ip link set dev veth0 up
root@ivan-server:~# ip a
1: lo: <LOOPBACK> mtu 65536 qdisc noop state DOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
3: veth0@if2: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state LOWERLAYERDOWN group default qlen 1000
    link/ether 72:66:01:79:fd:82 brd ff:ff:ff:ff:ff:ff link-netns testns123
    inet 10.0.0.1/24 scope global veth0
       valid_lft forever preferred_lft forever

Запустите новое пространство ip netns exec testns123 /bin/bash и сделайте:

ip addr add 10.0.0.2/24 dev veth1
ip link set dev veth1 up
root@ivan-server:~# ip a
1: lo: <LOOPBACK> mtu 65536 qdisc noop state DOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
2: veth1@if3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000
    link/ether ce:89:b6:78:ce:a2 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 10.0.0.2/24 scope global veth1
       valid_lft forever preferred_lft forever
    inet6 fe80::cc89:b6ff:fe78:cea2/64 scope link 
       valid_lft forever preferred_lft forever

Покажите пинг между пространствами.

root@ivan-server:~# ping 10.0.0.2
PING 10.0.0.2 (10.0.0.2) 56(84) bytes of data.