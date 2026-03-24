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
1. способ:
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
```
