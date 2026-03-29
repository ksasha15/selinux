# selinux

### Задание

1. Запустить nginx на нестандартном порту 3-мя разными способами:
- переключатели setsebool;
- добавление нестандартного порта в имеющийся тип;
- формирование и установка модуля SELinux.
2. Обеспечить работоспособность приложения при включенном selinux.
- развернуть приложенный стенд https://github.com/mbfx/otus-linux-adm/tree/master/selinux_dns_problems;
- выяснить причину неработоспособности механизма обновления зоны (см. README);
- предложить решение (или решения) для данной проблемы;
- выбрать одно из решений для реализации, предварительно обосновав выбор;
- реализовать выбранное решение и продемонстрировать его работоспособность.

1. Во время развёртывания стенда попытка запустить nginx завершается с ошибкой:
```
    selinux:   setools-console-4.4.4-1.el9.x86_64
    selinux:   setroubleshoot-plugins-3.3.14-4.el9.noarch
    selinux:   setroubleshoot-server-3.3.35-2.el9.x86_64
    selinux:
    selinux: Complete!
    selinux: Job for nginx.service failed because the control process exited wit
h error code.
    selinux: See "systemctl status nginx.service" and "journalctl -xeu nginx.ser
vice" for details.
    selinux: ? nginx.service - The nginx HTTP and reverse proxy server
    selinux:      Loaded: loaded (/usr/lib/systemd/system/nginx.service; disable
d; preset: disabled)
    selinux:      Active: failed (Result: exit-code) since Tue 2026-03-24 19:57:
17 UTC; 40ms ago
    selinux:     Process: 6728 ExecStartPre=/usr/bin/rm -f /run/nginx.pid (code=
exited, status=0/SUCCESS)
    selinux:     Process: 6743 ExecStartPre=/usr/sbin/nginx -t (code=exited, sta
tus=1/FAILURE)
    selinux:         CPU: 74ms
    selinux:
    selinux: Mar 24 19:57:17 selinux systemd[1]: Starting The nginx HTTP and rev
erse proxy server...
    selinux: Mar 24 19:57:17 selinux nginx[6743]: nginx: the configuration file
/etc/nginx/nginx.conf syntax is ok
    selinux: Mar 24 19:57:17 selinux nginx[6743]: nginx: [emerg] bind() to 0.0.0
.0:4881 failed (13: Permission denied)
    selinux: Mar 24 19:57:17 selinux nginx[6743]: nginx: configuration file /etc
/nginx/nginx.conf test failed
    selinux: Mar 24 19:57:17 selinux systemd[1]: nginx.service: Control process
exited, code=exited, status=1/FAILURE
    selinux: Mar 24 19:57:17 selinux systemd[1]: nginx.service: Failed with resu
lt 'exit-code'.
    selinux: Mar 24 19:57:17 selinux systemd[1]: Failed to start The nginx HTTP
and reverse proxy server.
```
##### Запуск nginx на нестандартном порту 3-мя разными способами:
1. Cпособ: Разрешим в SELinux работу nginx на порту TCP 4881 c помощью переключателей setsebool
```
[vagrant@selinux ~]$ sudo -i
[root@selinux ~]# systemctl status firewalld
○ firewalld.service - firewalld - dynamic firewall daemon
     Loaded: loaded (/usr/lib/systemd/system/firewalld.service; disabled; preset>
     Active: inactive (dead)
       Docs: man:firewalld(1)
[root@selinux ~]# nginx -t
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
[root@selinux ~]#
 getenforce
Enforcing
[root@selinux ~]# cat /var/log/audit/audit.log | grep nginx
...
type=SYSCALL msg=audit(1774384437.255:759): arch=c000003e syscall=49 success=no exit=-13 a0=6 a1=561e1038c4b0 a2=10 a3=7ffd2db80920 items=0 ppid=1 pid=9020 auid=4294967295 uid=0 gid=0 euid=0 suid=0 fsuid=0 egid=0 sgid=0 fsgid=0 tty=(none) ses=4294967295 comm="nginx" exe="/usr/sbin/nginx" subj=system_u:system_r:httpd_t:s0 key=(null)ARCH=x86_64 SYSCALL=bind AUID="unset" UID="root" GID="root" EUID="root" SUID="root" FSUID="root" EGID="root" SGID="root" FSGID="root"
type=SERVICE_START msg=audit(1774384437.259:760): pid=1 uid=0 auid=4294967295 ses=4294967295 subj=system_u:system_r:init_t:s0 msg='unit=nginx comm="systemd" exe="/usr/lib/systemd/systemd" hostname=? addr=? terminal=? res=failed'UID="root" AUID="unset"
[root@selinux ~]# grep 1774384437.255:759 /var/log/audit/audit.log | audit2why
type=AVC msg=audit(1774384437.255:759): avc:  denied  { name_bind } for  pid=9020 comm="nginx" src=4881 scontext=system_u:system_r:httpd_t:s0 tcontext=system_u:object_r:unreserved_port_t:s0 tclass=tcp_socket permissive=0

        Was caused by:
        The boolean nis_enabled was set incorrectly.
        Description:
        Allow nis to enabled

        Allow access by executing:
        # setsebool -P nis_enabled 1
[root@selinux ~]# setsebool -P nis_enabled 1
[root@selinux ~]# systemctl restart nginx
[root@selinux ~]# systemctl status nginx
● nginx.service - The nginx HTTP and reverse proxy server
     Loaded: loaded (/usr/lib/systemd/system/nginx.service; disabled; preset: disabled)
     Active: active (running) since Tue 2026-03-24 20:40:19 UTC; 6s ago
    Process: 9054 ExecStartPre=/usr/bin/rm -f /run/nginx.pid (code=exited, status=0/SUCCESS)
    Process: 9055 ExecStartPre=/usr/sbin/nginx -t (code=exited, status=0/SUCCESS)
    Process: 9056 ExecStart=/usr/sbin/nginx (code=exited, status=0/SUCCESS)
   Main PID: 9057 (nginx)
      Tasks: 3 (limit: 12019)
     Memory: 2.9M
        CPU: 67ms
     CGroup: /system.slice/nginx.service
             ├─9057 "nginx: master process /usr/sbin/nginx"
             ├─9058 "nginx: worker process"
             └─9059 "nginx: worker process"

Mar 24 20:40:19 selinux systemd[1]: Starting The nginx HTTP and reverse proxy server...
Mar 24 20:40:19 selinux nginx[9055]: nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
Mar 24 20:40:19 selinux nginx[9055]: nginx: configuration file /etc/nginx/nginx.conf test is successful
Mar 24 20:40:19 selinux systemd[1]: Started The nginx HTTP and reverse proxy server.
[root@selinux ~]# getsebool -a | grep nis_enabled
nis_enabled --> on
[root@selinux ~]# setsebool -P nis_enabled off
```
2. Cпособ: Разрешим в SELinux работу nginx на порту TCP 4881 c помощью добавления нестандартного порта в имеющийся тип:
```
[root@selinux ~]# semanage port -l | grep http
http_cache_port_t              tcp      8080, 8118, 8123, 10001-10010
http_cache_port_t              udp      3130
http_port_t                    tcp      80, 81, 443, 488, 8008, 8009, 8443, 9000
pegasus_http_port_t            tcp      5988
pegasus_https_port_t           tcp      5989
[root@selinux ~]# semanage port -a -t http_port_t -p tcp 4881
[root@selinux ~]# semanage port -l | grep http_port_t
http_port_t                    tcp      4881, 80, 81, 443, 488, 8008, 8009, 8443, 9000
pegasus_http_port_t            tcp      5988
[root@selinux ~]# systemctl restart nginx
[root@selinux ~]# systemctl status nginx
● nginx.service - The nginx HTTP and reverse proxy server
     Loaded: loaded (/usr/lib/systemd/system/nginx.service; disabled; preset: disabled)
     Active: active (running) since Tue 2026-03-24 20:49:35 UTC; 4s ago
    Process: 9085 ExecStartPre=/usr/bin/rm -f /run/nginx.pid (code=exited, status=0/SUCCESS)
    Process: 9086 ExecStartPre=/usr/sbin/nginx -t (code=exited, status=0/SUCCESS)
    Process: 9087 ExecStart=/usr/sbin/nginx (code=exited, status=0/SUCCESS)
   Main PID: 9088 (nginx)
      Tasks: 3 (limit: 12019)
     Memory: 2.9M
        CPU: 70ms
     CGroup: /system.slice/nginx.service
             ├─9088 "nginx: master process /usr/sbin/nginx"
             ├─9089 "nginx: worker process"
             └─9090 "nginx: worker process"

Mar 24 20:49:35 selinux systemd[1]: Starting The nginx HTTP and reverse proxy server...
Mar 24 20:49:35 selinux nginx[9086]: nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
Mar 24 20:49:35 selinux nginx[9086]: nginx: configuration file /etc/nginx/nginx.conf test is successful
Mar 24 20:49:35 selinux systemd[1]: Started The nginx HTTP and reverse proxy server.
[root@selinux ~]# semanage port -d -t http_port_t -p tcp 4881
[root@selinux ~]# semanage port -l | grep http_port_t
http_port_t                    tcp      80, 81, 443, 488, 8008, 8009, 8443, 9000
pegasus_http_port_t            tcp      5988
[root@selinux ~]# systemctl restart nginx
Job for nginx.service failed because the control process exited with error code.
See "systemctl status nginx.service" and "journalctl -xeu nginx.service" for details.
[root@selinux ~]# systemctl status nginx
× nginx.service - The nginx HTTP and reverse proxy server
     Loaded: loaded (/usr/lib/systemd/system/nginx.service; disabled; preset: disabled)
     Active: failed (Result: exit-code) since Tue 2026-03-24 20:50:50 UTC; 6s ago
   Duration: 1min 14.924s
    Process: 9105 ExecStartPre=/usr/bin/rm -f /run/nginx.pid (code=exited, status=0/SUCCESS)
    Process: 9107 ExecStartPre=/usr/sbin/nginx -t (code=exited, status=1/FAILURE)
        CPU: 37ms

Mar 24 20:50:50 selinux systemd[1]: Starting The nginx HTTP and reverse proxy server...
Mar 24 20:50:50 selinux nginx[9107]: nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
Mar 24 20:50:50 selinux nginx[9107]: nginx: [emerg] bind() to 0.0.0.0:4881 failed (13: Permission denied)
Mar 24 20:50:50 selinux nginx[9107]: nginx: configuration file /etc/nginx/nginx.conf test failed
Mar 24 20:50:50 selinux systemd[1]: nginx.service: Control process exited, code=exited, status=1/FAILURE
Mar 24 20:50:50 selinux systemd[1]: nginx.service: Failed with result 'exit-code'.
Mar 24 20:50:50 selinux systemd[1]: Failed to start The nginx HTTP and reverse proxy server.
[root@selinux ~]#
```
3. Cпособ: Разрешим в SELinux работу nginx на порту TCP 4881 c помощью формирования и установки модуля SELinux:
```
[root@selinux ~]# grep nginx /var/log/audit/audit.log | audit2allow -M nginx
******************** IMPORTANT ***********************
To make this policy package active, execute:

semodule -i nginx.pp

[root@selinux ~]# semodule -i nginx.pp
[root@selinux ~]# systemctl start nginx
[root@selinux ~]# systemctl status nginx
● nginx.service - The nginx HTTP and reverse proxy server
     Loaded: loaded (/usr/lib/systemd/system/nginx.service; disabled; preset: disabled)
     Active: active (running) since Tue 2026-03-24 20:58:35 UTC; 3s ago
    Process: 9149 ExecStartPre=/usr/bin/rm -f /run/nginx.pid (code=exited, status=0/SUCCESS)
    Process: 9150 ExecStartPre=/usr/sbin/nginx -t (code=exited, status=0/SUCCESS)
    Process: 9151 ExecStart=/usr/sbin/nginx (code=exited, status=0/SUCCESS)
   Main PID: 9152 (nginx)
      Tasks: 3 (limit: 12019)
     Memory: 2.9M
        CPU: 68ms
     CGroup: /system.slice/nginx.service
             ├─9152 "nginx: master process /usr/sbin/nginx"
             ├─9153 "nginx: worker process"
             └─9154 "nginx: worker process"

Mar 24 20:58:35 selinux systemd[1]: Starting The nginx HTTP and reverse proxy server...
Mar 24 20:58:35 selinux nginx[9150]: nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
Mar 24 20:58:35 selinux nginx[9150]: nginx: configuration file /etc/nginx/nginx.conf test is successful
Mar 24 20:58:35 selinux systemd[1]: Started The nginx HTTP and reverse proxy server.
[root@selinux ~]# semodule -r nginx.pp
libsemanage.semanage_direct_remove_key: Unable to remove module nginx.pp at priority 400. (No such file or directory).
semodule:  Failed!
[root@selinux ~]# semodule -r nginx
libsemanage.semanage_direct_remove_key: Removing last nginx module (no other nginx module exists at another priority).
[root@selinux ~]#
```
#### Обеспечение работоспособности приложения при включенном SELinux
В исходном состоянии обновление не проходит:
```
[vagrant@client ~]$     nsupdate -k /etc/named.zonetransfer.key
>    server 192.168.50.10
    zone ddns.lab
    update add www.ddns.lab. 60 A 192.168.50.15
    send
> > > update failed: SERVFAIL
>
> quit
[vagrant@client ~]$
```
Анализируем вывод audit2why сначала на клиенте, потом на сервере:
```
[vagrant@client ~]$ sudo -i
[root@client ~]# cat /var/log/audit/audit.log | audit2why
type=AVC msg=audit(1774764514.433:653): avc:  denied  { dac_read_search } for  pid=4453 comm="11-dhclient" capability=2  scontext=system_u:system_r:NetworkManager_dispatcher_dhclient_t:s0 tcontext=system_u:system_r:NetworkManager_dispatcher_dhclient_t:s0 tclass=capability permissive=0

        Was caused by:
                Missing type enforcement (TE) allow rule.

                You can use audit2allow to generate a loadable module to allow this access.
```
```
[vagrant@ns01 ~]$ sudo -i
[root@ns01 ~]# cat /var/log/audit/audit.log | audit2why
type=AVC msg=audit(1774764339.473:651): avc:  denied  { dac_read_search } for  pid=4448 comm="11-dhclient" capability=2  scontext=system_u:system_r:NetworkManager_dispatcher_dhclient_t:s0 tcontext=system_u:system_r:NetworkManager_dispatcher_dhclient_t:s0 tclass=capability permissive=0

        Was caused by:
                Missing type enforcement (TE) allow rule.

                You can use audit2allow to generate a loadable module to allow this access.
type=AVC msg=audit(1774765073.915:1729): avc:  denied  { write } for  pid=9184 comm="isc-net-0001" name="dynamic" dev="sda4" ino=4851 scontext=system_u:system_r:named_t:s0 tcontext=unconfined_u:object_r:named_conf_t:s0 tclass=dir permissive=0

        Was caused by:
                Missing type enforcement (TE) allow rule.

                You can use audit2allow to generate a loadable module to allow this access.
```
В логе на сервере мы видим, что ошибка в контексте безопасности. Целевой контекст _named_conf_t_.
Для сравнения посмотрим существующую зону (localhost) и её контекст:
