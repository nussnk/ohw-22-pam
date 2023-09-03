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
После подготовки ВМ, я создал vagrant box и загрузил его в vagrant cloud. В файле Vagrantfile скачивается именно этот box
