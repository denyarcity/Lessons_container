Запустить bash - chroot

root@ivan-server:~#  mkdit for_chroot
root@ivan-server:~# cp --parents /bin/bash for_chroot
root@ivan-server:~# ldd bin/bash
linux-vdso.so.1 (0x00007ffdbefcc000)
	libtinfo.so.6 => /lib/x86_64-linux-gnu/libtinfo.so.6 (0x00007f43eb073000)
	libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x00007f43eae4b000)
	/lib64/ld-linux-x86-64.so.2 (0x00007f43eb20d000)
root@ivan-server:~# cp --parents /lib/x86_64-linux-gnu/libtinfo.so.6 for_chroot/
root@ivan-server:~# cp --parents /lib/x86_64-linux-gnu/libc.so.6 for_chroot/
root@ivan-server:~# cp --parents /lib64/ld-linux-x86-64.so.2 for_chroot/
root@ivan-server:~# chroot for_chroot /bin/bash
bash-5.1#
bash-5.1# exit
Добавить команду ls - chroot

root@ivan-server:~# whereis ls
ls: /usr/bin/ls /usr/share/man/man1/ls.1.gz
root@ivan-server:~# cp --parents /usr/bin/ls for_chroot/
root@ivan-server:~# ldd /usr/bin/ls
linux-vdso.so.1 (0x00007ffd73fbd000)
	libselinux.so.1 => /lib/x86_64-linux-gnu/libselinux.so.1 (0x00007f886d395000)
	libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x00007f886d16d000)
	libpcre2-8.so.0 => /lib/x86_64-linux-gnu/libpcre2-8.so.0 (0x00007f886d0d6000)
	/lib64/ld-linux-x86-64.so.2 (0x00007f886d3ec000)

root@ivan-server:~# cp --parents /lib/x86_64-linux-gnu/libselinux.so.1 for_chroot/
root@ivan-server:~# cp --parents /lib/x86_64-linux-gnu/libc.so.6 for_chroot/
root@ivan-server:~# cp --parents /lib/x86_64-linux-gnu/libpcre2-8.so.0 for_chroot/
root@ivan-server:~# cp --parents /lib64/ld-linux-x86-64.so.2 for_chroot/
root@ivan-server:~# chroot for_chroot/ bin/bash
bash-5.1# ls
bin  lib  lib64  ls  usr
bash-5.1# exit
lsns

ip-netns - process network namespace management

ip netns add my_net - добавляем

ip netns list - список

ip netns exec my_net /bin/bash - запуск

root@ivan-server:~# lsns | grep 'net '
4026531840 net       139     1 root             /lib/systemd/systemd --system --deserialize 35
4026532264 net         3  3292 root             /bin/bash
unshare - позвоялет создать отдельный namespace

root@ivan-server:~# echo $$
2819
root@ivan-server:~# unshare --net /bin/bash
root@ivan-server:~# echo $$
3559
root@ivan-server:~# lsns | grep 'net '
4026531840 net       143     1 root             /lib/systemd/systemd --system --deserialize 35
4026532326 net         1  3559 root             /bin/bash

root@ivan-server:~# unshare -n -f -p --mount-proc /bin/bash
root@ivan-server:~# ps aux
USER         PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root           1  0.0  0.0   7632  4092 pts/1    S    09:24   0:00 /bin/bash
root           8  0.0  0.0  10068  1592 pts/1    R+   09:24   0:00 ps aux