Введение в системное администрирование: 

СРО: Проведите установку и настройку операционной системы на виртуальной машине (например, Ubuntu Server). Настройте пользователи, группы и простейшие сценарии автоматизации через cron. Подготовьте отчет с командной строкой действий и выводом системных команд. 

СРОП: Практическая работа по созданию и настройке виртуальной сети в VirtualBox, установка серверных служб (например, SSH), управление пользователями и настройка прав доступа для них. 


1. Установка и настройка операционной системы (СРО)

1.1. Установка виртуальной машины: Для выполнения задания была создана виртуальная машина в VirtualBox со следующими параметрами:

    Имя машины: Linux-Administration
    Операционная система: Ubuntu (64-bit)
    Оперативная память: 2048 МБ
    Жесткий диск: 20 ГБ, динамически расширяемый

Скачан образ Ubuntu Server с официального сайта, и настроена виртуальная машина на загрузку с этого образа. Установка операционной системы прошла успешно, использованы настройки по умолчанию (автоматическая разметка диска, базовая настройка сети).

После установки системы был выполнен вход под учетной записью администратора. Далее были выполнены следующие команды:

```console
stacy@stacy-VirtualBox:~$ sudo adduser devops
[sudo] пароль для stacy: 
Добавляется пользователь «devops» ...
Добавляется новая группа «devops» (1001) ...
Добавляется новый пользователь «devops» (1001) в группу «devops» ...
Создаётся домашний каталог «/home/devops» ...
Копирование файлов из «/etc/skel» ...
Новый пароль : 
Повторите ввод нового пароля : 
passwd: пароль успешно обновлён
Изменение информации о пользователе devops
Введите новое значение или нажмите ENTER для выбора значения по умолчанию
	Полное имя []: 
	Номер комнаты []: 
	Рабочий телефон []: 
	Домашний телефон []: 
	Другое []: 
Данная информация корректна? [Y/n] Y
stacy@stacy-VirtualBox:~$ sudo usermod -aG sudo devops
stacy@stacy-VirtualBox:~$ groups devops
devops : devops sudo
```

1.3. Настройка cron для автоматизации:

Для автоматизации задач был настроен планировщик cron:


```console
stacy@stacy-VirtualBox:~$ crontab -e
crontab: installing new crontab
stacy@stacy-VirtualBox:~$ crontab -l
# Edit this file to introduce tasks to be run by cron.
# 
# Each task to run has to be defined through a single line
# indicating with different fields when the task will be run
# and what command to run for the task
# 
# To define the time you can provide concrete values for
# minute (m), hour (h), day of month (dom), month (mon),
# and day of week (dow) or use '*' in these fields (for 'any').
# 
# Notice that tasks will be started based on the cron's system
# daemon's notion of time and timezones.
# 
# Output of the crontab jobs (including errors) is sent through
# email to the user the crontab file belongs to (unless redirected).
# 
# For example, you can run a backup of all your user accounts
# at 5 a.m every week with:
# 0 5 * * 1 tar -zcf /var/backups/home.tgz /home/
# 
# For more information see the manual pages of crontab(5) and cron(8)
# 
# m h  dom mon dow   command
* * * * * /home/devops/backup.sh
```
