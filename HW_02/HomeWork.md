Задание 1:
Запустить контейнер с ubuntu, используя механизм LXC

root@ivan-server:~# lxc-create -n test-container -t ubuntu

root@ivan-server:~# lxc-ls -f
NAME           STATE   AUTOSTART GROUPS IPV4      IPV6 UNPRIVILEGED      
test-container STOPPED 0         -      -         -    false             
Ограничить контейнер 256 Мб ОЗУ и проверить, что ограничение работает

root@ivan-server:~# vi /var/lib/lxc/test-container/config 

lxc.cgroup2.memory.max = 256M
Добавить автозапуск контейнеру, перезагрузить ОС и убедиться, что контейнер действительно запустился самостоятельно

NAME           STATE   AUTOSTART GROUPS IPV4      IPV6 UNPRIVILEGED   
test-container STOPPED 0         -      -         -    false        

root@ivan-server:~# vi /var/lib/lxc/test-container/config
lxc.start.auto  =  1

NAME           STATE   AUTOSTART GROUPS IPV4      IPV6 UNPRIVILEGED   
test-container STOPPED 1         -      -         -    false        
Опробовать ограничить количество ядер CPU

cpuset.cpus — какие ядра может использовать контейнер, номера начиная с нуля, например 0,1 или 0-3

root@ivan-server:~# vi /var/lib/lxc/test-container/config

lxc.cgroup.cpuset.cpus = 0-2
Задание 2: Необходимо создать 2 контейнера и показать возможность взаимодействия между собой (командой пинг)

Создаем 1 контейнер

lxc-create -n container1 -t ubuntu
Добаялем стат IP

vi /var/lib/lxc/container1/config
lxc.net.0.ipv4.address = 10.0.0.11/24

root@ivan-server:~# lxc-ls -f
NAME           STATE   AUTOSTART GROUPS IPV4      IPV6 UNPRIVILEGED 
container1     RUNNING 1         -      10.0.0.11 -    false        
Создаем 2 контейнер и добавим стат ip

lxc-create -n test123 -t ubuntu

vi /var/lib/lxc/test123/config
lxc.net.0.ipv4.address = 10.0.0.10/24

root@ivan-server:~# lxc-ls -f
NAME           STATE   AUTOSTART GROUPS IPV4      IPV6 UNPRIVILEGED 
container1     RUNNING 1         -      10.0.0.11 -    false        
test123        RUNNING 1         -      10.0.0.10 -    false        

root@ivan-server:~# lxc-attach -n container1

root@container1:~# ping 10.0.0.10
PING 10.0.0.10 (10.0.0.10) 56(84) bytes of data.
64 bytes from 10.0.0.10: icmp_seq=1 ttl=64 time=8.36 ms
64 bytes from 10.0.0.10: icmp_seq=2 ttl=64 time=0.189 ms
64 bytes from 10.0.0.10: icmp_seq=3 ttl=64 time=0.037 ms
64 bytes from 10.0.0.10: icmp_seq=4 ttl=64 time=0.037 ms
root@container1:~# exit

root@ivan-server:~# lxc-attach -n test123
root@test123:~# ping 10.0.0.11
PING 10.0.0.11 (10.0.0.11) 56(84) bytes of data.
64 bytes from 10.0.0.11: icmp_seq=1 ttl=64 time=0.279 ms
64 bytes from 10.0.0.11: icmp_seq=2 ttl=64 time=0.034 ms
64 bytes from 10.0.0.11: icmp_seq=3 ttl=64 time=0.047 ms
64 bytes from 10.0.0.11: icmp_seq=4 ttl=64 time=0.039 ms
64 bytes from 10.0.0.11: icmp_seq=5 ttl=64 time=0.037 ms
Задание 3: Добавить в каждый контейнер статический IP-адрес из той же подсети через config-file, показанный ранее на семинаре.

root@ivan-server:~# lxc-ls -f
NAME           STATE   AUTOSTART GROUPS IPV4      IPV6 UNPRIVILEGED 
test-container STOPPED 1         -      -         -    false        


vi /var/lib/lxc/test-container/config
lxc.net.0.ipv4.address = 11.0.0.11/24

root@ivan-server:~# lxc-ls -f
NAME           STATE   AUTOSTART GROUPS IPV4      IPV6 UNPRIVILEGED 
test-container RUNNING 1         -      11.0.0.11 -    false        