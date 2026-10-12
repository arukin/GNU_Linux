# Работа с mdadm

###  Задание:

1. Добавить в виртуальную машину несколько дисков  
2. Собрать RAID-0/1/5/10 на выбор  
3. Сломать и починить RAID  
4. Создать GPT таблицу, пять разделов и смонтировать их в системе.



###  Решение:

1. Добавляем в образ 5 дисков по 1 ГБ. Подключаемся к ВМ Ubuntu и проверяем их наличие.

```
arukin@UBS:~$ sudo -i
[sudo] password for arukin:
root@UBS:~#
root@UBS:~#
root@UBS:~# fdisk -l
Disk /dev/sda: 25 GiB, 26843545600 bytes, 52428800 sectors
Disk model: VBOX HARDDISK
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: gpt
Disk identifier: 933A0A22-AC8C-454A-A4B8-31C8BF239E91

Device     Start      End  Sectors Size Type
/dev/sda1   2048     4095     2048   1M BIOS boot
/dev/sda2   4096 52426751 52422656  25G Linux filesystem


Disk /dev/sdb: 1 GiB, 1073741824 bytes, 2097152 sectors
Disk model: VBOX HARDDISK
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes


Disk /dev/sdc: 1 GiB, 1073741824 bytes, 2097152 sectors
Disk model: VBOX HARDDISK
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes


Disk /dev/sdd: 1 GiB, 1073741824 bytes, 2097152 sectors
Disk model: VBOX HARDDISK
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes


Disk /dev/sde: 1 GiB, 1073741824 bytes, 2097152 sectors
Disk model: VBOX HARDDISK
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes


Disk /dev/sdf: 1 GiB, 1073741824 bytes, 2097152 sectors
Disk model: VBOX HARDDISK
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes

```

2. Занулим на всякий случай суперблоки

```
root@UBS:~# mdadm --zero-superblock --force /dev/sd{b,c,d,e,f}
mdadm: Unrecognised md component device - /dev/sdb
mdadm: Unrecognised md component device - /dev/sdc
mdadm: Unrecognised md component device - /dev/sdd
mdadm: Unrecognised md component device - /dev/sde
mdadm: Unrecognised md component device - /dev/sdf
```
Ничего не обнаружено. Диски чистые и готовы к сборке в RAID.

3. Создадим программный RAID-массив. Выберу RAID-10 из 4х диков.

```
root@UBS:~# mdadm --create --verbose /dev/md0 -l 10 -n 4 /dev/sd{b,c,d,e}
mdadm: layout defaults to n2
mdadm: layout defaults to n2
mdadm: chunk size defaults to 512K
mdadm: size set to 1046528K
mdadm: Defaulting to version 1.2 metadata
mdadm: array /dev/md0 started.
```

4. Раз нам все равно не спится, проверим, что все получилось.

```
root@UBS:~# cat /proc/mdstat
Personalities : [raid0] [raid1] [raid6] [raid5] [raid4] [raid10]
md0 : active raid10 sde[3] sdd[2] sdc[1] sdb[0]
      2093056 blocks super 1.2 512K chunks 2 near-copies [4/4] [UUUU]

unused devices: <none>
root@UBS:~# madam -D /dev/md0
madam: command not found
root@UBS:~# mdadm -D /dev/md0
/dev/md0:
           Version : 1.2
     Creation Time : Mon Oct 12 04:19:29 2026
        Raid Level : raid10
        Array Size : 2093056 (2044.00 MiB 2143.29 MB)
     Used Dev Size : 1046528 (1022.00 MiB 1071.64 MB)
      Raid Devices : 4
     Total Devices : 4
       Persistence : Superblock is persistent

       Update Time : Mon Oct 12 04:19:40 2026
             State : clean
    Active Devices : 4
   Working Devices : 4
    Failed Devices : 0
     Spare Devices : 0

            Layout : near=2
        Chunk Size : 512K

Consistency Policy : resync

              Name : UBS:0  (local to host UBS)
              UUID : cc513b0a:bd51aeae:4192a97c:0236b539
            Events : 17

    Number   Major   Minor   RaidDevice State
       0       8       16        0      active sync set-A   /dev/sdb
       1       8       32        1      active sync set-B   /dev/sdc
       2       8       48        2      active sync set-A   /dev/sdd
       3       8       64        3      active sync set-B   /dev/sde

```
5. Все получилось. Вернемся в реальность и сломаем один из дисков, а после посмотрим к чему это привело.

```
root@UBS:~# mdadm /dev/md0 --fail /dev/sdb
mdadm: set /dev/sdb faulty in /dev/md0
root@UBS:~#
root@UBS:~# mdadm -D /dev/md0
/dev/md0:
           Version : 1.2
     Creation Time : Mon Oct 12 04:19:29 2026
        Raid Level : raid10
        Array Size : 2093056 (2044.00 MiB 2143.29 MB)
     Used Dev Size : 1046528 (1022.00 MiB 1071.64 MB)
      Raid Devices : 4
     Total Devices : 4
       Persistence : Superblock is persistent

       Update Time : Mon Oct 12 04:36:50 2026
             State : clean, degraded
    Active Devices : 3
   Working Devices : 3
    Failed Devices : 1
     Spare Devices : 0

            Layout : near=2
        Chunk Size : 512K

Consistency Policy : resync

              Name : UBS:0  (local to host UBS)
              UUID : cc513b0a:bd51aeae:4192a97c:0236b539
            Events : 19

    Number   Major   Minor   RaidDevice State
       -       0        0        0      removed
       1       8       32        1      active sync set-B   /dev/sdc
       2       8       48        2      active sync set-A   /dev/sdd
       3       8       64        3      active sync set-B   /dev/sde

       0       8       16        -      faulty   /dev/sdb

```
6. RAID-10 перешел в состояние clean, degraded. Данные еще доступны, но уже не так надежно упакованы.
Удалим сбойный и добавим в массив новый исправный диск.

```
root@UBS:~# mdadm /dev/md0 --remove /dev/sdb
mdadm: hot removed /dev/sdb from /dev/md0
root@UBS:~#
root@UBS:~# mdadm /dev/md0 --add /dev/sdf
mdadm: added /dev/sdf
root@UBS:~#
root@UBS:~# mdadm -D /dev/md0
/dev/md0:
           Version : 1.2
     Creation Time : Mon Oct 12 04:19:29 2026
        Raid Level : raid10
        Array Size : 2093056 (2044.00 MiB 2143.29 MB)
     Used Dev Size : 1046528 (1022.00 MiB 1071.64 MB)
      Raid Devices : 4
     Total Devices : 4
       Persistence : Superblock is persistent

       Update Time : Mon Oct 12 04:42:33 2026
             State : clean
    Active Devices : 4
   Working Devices : 4
    Failed Devices : 0
     Spare Devices : 0

            Layout : near=2
        Chunk Size : 512K

Consistency Policy : resync

              Name : UBS:0  (local to host UBS)
              UUID : cc513b0a:bd51aeae:4192a97c:0236b539
            Events : 39

    Number   Major   Minor   RaidDevice State
       4       8       80        0      active sync set-A   /dev/sdf
       1       8       32        1      active sync set-B   /dev/sdc
       2       8       48        2      active sync set-A   /dev/sdd
       3       8       64        3      active sync set-B   /dev/sde

```
Новое блочное устройство sdf заняло место сломанного sdb, RAID-10 восстановился в рабочее состояние clean.

7. Создим GPT таблицу, пять разделов и смонтируем их в системе.
Для начала создадим GPT раздел RAID-10, а затем создадим партиции. Позволю себе разбить их неравномерно.

```

```
