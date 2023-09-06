# ohw-22-pam

# Задание
Запретить всем пользователям, кроме группы admin, логин в выходные (суббота и воскресенье), без учета праздников

# Инструкция
Скачать репозиторий, запустить ВМ командой vagrant up
Vagrantfile разворачивает VM с ip адресом 192.168.57.10

Созданы два пользователя: Otus и Otusadm
Под пользователем Otus можно залогиниться только в будние дни, а под пользователем Otusadm любой день (как и любому члены группы admin)

Проверка осуществляется через pam_exec.so модуль, скриптом login.sh, которые проверяет при логине текущий день недели и членство в группе admin.

# Детали
После подготовки ВМ я создал vagrant box и загрузил его в vagrant cloud. В файле Vagrantfile скачивается именно этот box

# Подробнее
Создадим ВМ с Centos/Stream8 через Vagrantfile

Подключаемся
```vagrant ssh```

Переключаемся в рута
```sudo -i```

После этого 
- создаем пользователей otus и otusadm
- создаем группу admin
- добавляем туда пользователей root, vagrant и otusadm
- проверяем, что получилось добавить
- создаем скрипт /usr/local/bin/login.sh, который будет проверять день недели и, если это суббота или воскресенье, будет проверять входит ли пользователь в группу admin.
Если не входит, то выходим с кодом 1, если входит, то с кодом 0.
- добавляем права на исполнение скрипта
- добавляем в /etc/pam.d/sshd строку account required pam_exec.so /usr/local/bin/login.sh
- проверяем работу

Далее вывод команды script:
```
[root@pam ~]# adduser otus
[root@pam ~]# adduser otusadm
[root@pam ~]# echo "Otus2022!" | passwd --stdin otus
Changing password for user otus.
passwd: all authentication tokens updated successfully.
[root@pam ~]# echo "Otus2022!" | passwd --stdin otusadm
Changing password for user otusadm.
passwd: all authentication tokens updated successfully.
[root@pam ~]# groupadd -f admin
[root@pam ~]# usermod -aG admin otusadm
[root@pam ~]# usermod -aG admin vagrant
[root@pam ~]# usermod -aG admin root
[root@pam ~]# cat /etc/group | grep admin
admin:x:1003:otusadm,vagrant,root
[root@pam ~]# vi /usr/local/bin/login.sh
[root@pam ~]# chmod +x /usr/local/bin/login.sh
[root@pam ~]# vi /etc/pam.d/sshd 
[root@pam ~]# exit
exit

Script done on 2023-09-06 19:58:52+00:00
[root@pam ~]# 
```
Проверяем работу:

# Останавливаем синхронизацию времени
nkim@nkdeb:~/otus/ohw22$ systemctl stop systemd-timesyncd.service 
# Задаем кастомное время, чтобы была суббота
nkim@nkdeb:~/otus/ohw22$ sudo date 082712302022.00
[sudo] password for nkim: 
Sat 27 Aug 12:30:00 EEST 2022
# логинимся
nkim@nkdeb:~/otus/ohw22$ ssh otus@192.168.57.10
otus@192.168.57.10's password: 
/usr/local/bin/login.sh failed: exit code 1
Connection closed by 192.168.57.10 port 22
# Скрипт login.sh не пропустил, так как суббота и юзер не состоит в группе admin
# Пробуем пользователем otusadm
nkim@nkdeb:~/otus/ohw22$ ssh otusadm@192.168.57.10
otusadm@192.168.57.10's password: 
Activate the web console with: systemctl enable --now cockpit.socket

[otusadm@pam ~]$ logout
Connection to 192.168.57.10 closed.
# Успех! Возвращаю синхронизцию времени
nkim@nkdeb:~/otus/ohw22$ systemctl start systemd-timesyncd.service 
