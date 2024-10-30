СРО: Выполните разметку жесткого диска, создайте разделы, установите файловую систему (например, ext4), настройте монтирование разделов в ОС Linux. Проведите анализ файловых систем с помощью утилит df, du, и предоставьте результаты.
СРОП: Настройка RAID уровня 1 или 5 на виртуальной машине с подробным отчетом о выполненных действиях и тестировании отказоустойчивости системы.

## Выполнение разметки диска и создание разделов
Чтобы добавить новый виртуальный диск, сначала закрываю виртуальную машину Ubuntu и в настройках VirtualBox выбираю Settings → Storage. Нажимаю на Controller: SATA, выбираю значок диска с плюсом и добавляю новый виртуальный диск размером 1-2 ГБ. После этого запускаю виртуальную машину с новым диском.
В Ubuntu открываю терминал и проверяю, что новый диск подключен:
```
sudo fdisk -l
```

Диск обычно отображается как /dev/sdb (если это второй диск), и его параметры, например, размер 1 ГБ, видны в выводе команды. Запускаю fdisk для создания разделов на диске:
```
Диск /dev/sdb: 1 GiB, 1073741824 байт, 2097152 секторов
Disk model: VBOX HARDDISK   
Единицы: секторов по 1 * 512 = 512 байт
Размер сектора (логический/физический): 512 байт / 512 байт
Размер I/O (минимальный/оптимальный): 512 байт / 512 байт
```

В меню fdisk нажимаю n для создания нового раздела, затем выбираю p для основного раздела и оставляю номер раздела и размеры по умолчанию. Сохраняю изменения командой w.
```
sudo fdisk /dev/sdb
stacy@stacy-VirtualBox:~$ sudo fdisk /dev/sdb

Добро пожаловать в fdisk (util-linux 2.34).
Изменения останутся только в памяти до тех пор, пока вы не решите записать их.
Будьте внимательны, используя команду write.

Устройство не содержит стандартной таблицы разделов.
Создана новая метка DOS с идентификатором 0xb97385f3.

Команда (m для справки): m

Справка:

  DOS (MBR)
   a   переключение флага загрузки
   b   редактирование вложенной метки диска BSD
   c   переключение флага dos-совместимости

  Общие
   d   удалить раздел
   F   показать свободное неразмеченное пространство
   l   список известных типов разделов
   n   добавление нового раздела
   p   вывести таблицу разделов
   t   изменение типа раздела
   v   проверка таблицы разделов
   i   вывести информацию о разделе

  Разное
   m   вывод этого меню
   u   изменение единиц измерения экрана/содержимого
   x   дополнительная функциональность (только для экспертов)

  Сценарий
   I   загрузить разметку из файла сценария sfdisk
   O   записать разметку в файл сценария sfdisk

  Записать и выйти
   w   запись таблицы разделов на диск и выход
   q   выход без сохранения изменений

  Создать новую метку
   g   создание новой пустой таблицы разделов GPT
   G   создание новой пустой таблицы разделов SGI (IRIX)
   o   создание новой пустой таблицы разделов DOS
   s   создание новой пустой таблицы разделов Sun


Команда (m для справки): n
Тип раздела
   p   основной (0 первичный, 0 расширеный, 4 свободно)
   e   расширенный (контейнер для логических разделов)
Выберите (по умолчанию - p): p
Номер раздела (1-4, по умолчанию 1): 
Первый сектор (2048-2097151, по умолчанию 2048): 
Last sector, +/-sectors or +/-size{K,M,G,T,P} (2048-2097151, по умолчанию 2097151): 

Создан новый раздел 1 с типом 'Linux' и размером 1023 MiB.

Команда (m для справки): w
Таблица разделов была изменена.
Вызывается ioctl() для перечитывания таблицы разделов.
Синхронизируются диски.
```

После сохранения обновляю таблицу разделов командой:
```
stacy@stacy-VirtualBox:~$ sudo partprobe
```

## Форматирование раздела в файловую систему ext4
Чтобы подготовить созданный раздел для использования, форматирую его в ext4:
```
stacy@stacy-VirtualBox:~$ sudo mkfs.ext4 /dev/sdb1
mke2fs 1.45.5 (07-Jan-2020)
Creating filesystem with 261888 4k blocks and 65536 inodes
Filesystem UUID: 24eb56b1-657c-429e-af47-7f7c01dd0a8e
Superblock backups stored on blocks: 
	32768, 98304, 163840, 229376

Allocating group tables: done                            
Сохранение таблицы inod'ов: done                            
Создание журнала (4096 блоков): готово
Writing superblocks and filesystem accounting information: готово
```

Создаю точку монтирования для нового раздела, например, /mnt/newdisk:
```
stacy@stacy-VirtualBox:~$ sudo mkdir /mnt/newdisk
sudo mount /dev/sdb1 /mnt/newdisk
```

Чтобы раздел автоматически монтировался при загрузке системы, открываю файл /etc/fstab и добавляю строку:
```
sudo nano /etc/fstab
/dev/sdb1    /mnt/newdisk    ext4    defaults    0    2
```

## Анализ файловых систем
Для анализа использования дискового пространства на новом разделе использую команды df и du. Команда df показывает общий объём и доступное пространство на дисках:
```
df -h
```
```
stacy@stacy-VirtualBox:~$ df -h
Файл.система   Размер Использовано  Дост Использовано% Cмонтировано в
udev             3,0G            0  3,0G            0% /dev
tmpfs            623M         1,5M  622M            1% /run
/dev/sda5         24G          11G   13G           46% /
tmpfs            3,1G            0  3,1G            0% /dev/shm
tmpfs            5,0M         4,0K  5,0M            1% /run/lock
tmpfs            3,1G            0  3,1G            0% /sys/fs/cgroup
/dev/loop1       128K         128K     0          100% /snap/bare/5
/dev/loop0        64M          64M     0          100% /snap/core20/1828
/dev/loop2       347M         347M     0          100% /snap/gnome-3-38-2004/119
/dev/loop3       350M         350M     0          100% /snap/gnome-3-38-2004/143
/dev/loop4        92M          92M     0          100% /snap/gtk-common-themes/1535
/dev/loop5        46M          46M     0          100% /snap/snap-store/638
/dev/loop6        50M          50M     0          100% /snap/snapd/18357
/dev/sda1        511M         4,0K  511M            1% /boot/efi
tmpfs            623M          28K  623M            1% /run/user/1000
/dev/loop7        74M          74M     0          100% /snap/core22/1663
/dev/loop8        64M          64M     0          100% /snap/core20/2434
/dev/loop9        13M          13M     0          100% /snap/snap-store/1216
/dev/loop10      506M         506M     0          100% /snap/gnome-42-2204/176
/dev/sdb1        989M          24K  922M            1% /mnt/newdisk
stacy@stacy-VirtualBox:~$ sudo du -sh /mnt/newdisk
20K	/mnt/newdisk
```

## Настройка RAID
Для настройки RAID создаю еще один виртуальный диск, аналогично добавляя его в VirtualBox. Затем устанавливаю mdadm, утилиту для управления RAID:
```
sudo apt update
sudo apt install mdadm
```

Чтобы настроить RAID 1, использую диски /dev/sdb и /dev/sdc:
```
mdadm --create --verbose /dev/md0 --level=1 --raid-devices=2 /dev/sdb /dev/sdc
```

Проверяю статус RAID-массива:
```
cat /proc/mdstat
```
```
root@stacy-VirtualBox:/home/stacy# cat /proc/mdstat
Personalities : [linear] [multipath] [raid0] [raid1] [raid6] [raid5] [raid4] [raid10] 
md0 : active raid1 sdc[1] sdb[0]
      1046528 blocks super 1.2 [2/2] [UU]
      
unused devices: <none>
root@stacy-VirtualBox:/home/stacy# mkfs.ext4 /dev/md0
mke2fs 1.45.5 (07-Jan-2020)
Creating filesystem with 261632 4k blocks and 65408 inodes
Filesystem UUID: 8d3a4914-ea5f-4093-b010-cafda97f6aba
Superblock backups stored on blocks: 
	32768, 98304, 163840, 229376

Allocating group tables: done                            
Сохранение таблицы inod'ов: done                            
Создание журнала (4096 блоков): готово
Writing superblocks and filesystem accounting information: готово
mkdir /mnt/raid
mount /dev/md0 /mnt/raid
```

Проверка, что массив успешно смонтирован: 
```
root@stacy-VirtualBox:/home/stacy# df -h
Файл.система   Размер Использовано  Дост Использовано% Cмонтировано в
udev             3,0G            0  3,0G            0% /dev
tmpfs            623M         1,5M  622M            1% /run
/dev/sda5         24G          11G   13G           46% /
tmpfs            3,1G            0  3,1G            0% /dev/shm
tmpfs            5,0M         4,0K  5,0M            1% /run/lock
tmpfs            3,1G            0  3,1G            0% /sys/fs/cgroup
/dev/loop1       128K         128K     0          100% /snap/bare/5
/dev/loop0        64M          64M     0          100% /snap/core20/1828
/dev/loop3        64M          64M     0          100% /snap/core20/2434
/dev/loop2        74M          74M     0          100% /snap/core22/1663
/dev/loop5       347M         347M     0          100% /snap/gnome-3-38-2004/119
/dev/loop6        50M          50M     0          100% /snap/snapd/18357
/dev/loop4       350M         350M     0          100% /snap/gnome-3-38-2004/143
/dev/loop9        92M          92M     0          100% /snap/gtk-common-themes/1535
/dev/loop8       506M         506M     0          100% /snap/gnome-42-2204/176
/dev/loop7        46M          46M     0          100% /snap/snap-store/638
/dev/loop10       13M          13M     0          100% /snap/snap-store/1216
/dev/sda1        511M         4,0K  511M            1% /boot/efi
tmpfs            623M          40K  623M            1% /run/user/1000
/dev/loop11       39M          39M     0          100% /snap/snapd/21759
/dev/md0         988M          24K  921M            1% /mnt/raid
```

Чтобы массив монтировался автоматически при загрузке, добавляю запись в файл /etc/fstab. Открываю его в редакторе:
```
nano /etc/fstab
/dev/md0    /mnt/raid    ext4    defaults    0    0
```

Проверяю настройки командой:
```
mount -a
```
Теперь система автоматически монтирует RAID-массив при каждой перезагрузке.

## Вывод
В этих шагах я настроила новый виртуальный диск, отформатировала его в файловую систему ext4, и создала структуру монтирования для удобного использования в системе. Настройка включала также RAID-массив, который повышает отказоустойчивость системы: RAID 1 зеркалирует данные, обеспечивая резервную копию на случай сбоя одного из дисков.

Анализ файловой системы и её использование показали эффективное распределение ресурсов, а настройка автоматического монтирования RAID-массива в /etc/fstab гарантирует, что доступ к этому массиву будет всегда активен после каждой перезагрузки системы.