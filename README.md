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
2. Cпособ: Разрешим в SELinux работу nginx на порту TCP 4881 c помощью переключателей setsebool
```

```
