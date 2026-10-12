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
root@UBS:~# parted -s /dev/md0 mklabel gpt
root@UBS:~#
root@UBS:~# parted /dev/md0 mkpart primary ext4 0% 10%
Information: You may need to update /etc/fstab.

root@UBS:~# parted /dev/md0 mkpart primary ext4 10% 25%
Information: You may need to update /etc/fstab.

root@UBS:~# parted /dev/md0 mkpart primary ext4 25% 50%
Information: You may need to update /etc/fstab.

root@UBS:~# parted /dev/md0 mkpart primary ext4 50% 95%
Information: You may need to update /etc/fstab.

root@UBS:~# parted /dev/md0 mkpart primary ext4 95% 100%
Information: You may need to update /etc/fstab.

```
8. Создадим на них файловые системы и смотритуем, чтобы можно было ими пользоваться.
Приступим к форматированию в ext4

```
root@UBS:~# lsblk /dev/md0
NAME    MAJ:MIN RM  SIZE RO TYPE   MOUNTPOINTS
md0       9:0    0    2G  0 raid10
├─md0p1 259:1    0  203M  0 part
├─md0p2 259:4    0  307M  0 part
├─md0p3 259:5    0  511M  0 part
├─md0p4 259:8    0  920M  0 part
└─md0p5 259:9    0  101M  0 part
root@UBS:~#
root@UBS:~# mkfs.ext4 /dev/md0p1
mke2fs 1.47.0 (5-Feb-2023)
Creating filesystem with 51968 4k blocks and 51968 inodes
Filesystem UUID: 163142ce-c39b-4d6e-a8dc-89dd80d04e85
Superblock backups stored on blocks:
        32768

Allocating group tables: done
Writing inode tables: done
Creating journal (4096 blocks): done
Writing superblocks and filesystem accounting information: done

```
также по остальным партициям 2,3,4,5

Смонтируем по каталогам
```
root@UBS:~# mkdir -p /raid/part{1,2,3,4,5}
root@UBS:~# mount /dev/md0p1 /raid/part1
root@UBS:~# mount /dev/md0p1 /raid/part2
root@UBS:~# mount /dev/md0p1 /raid/part3
root@UBS:~# mount /dev/md0p1 /raid/part4
root@UBS:~# mount /dev/md0p1 /raid/part5

```
9. Проверим результаты работы

```
root@UBS:~# lsblk -f
NAME      FSTYPE            FSVER LABEL UUID                                 FSAVAIL FSUSE% MOUNTPOINTS
sda
├─sda1
└─sda2    ext4              1.0         14870e43-0110-49f3-aa39-a50aae2a6b93   20.4G    11% /
sdb       linux_raid_member 1.2   UBS:0 cc513b0a-bd51-aeae-4192-a97c0236b539
sdc       linux_raid_member 1.2   UBS:0 cc513b0a-bd51-aeae-4192-a97c0236b539
└─md0
  ├─md0p1 ext4              1.0         163142ce-c39b-4d6e-a8dc-89dd80d04e85  159.9M     0% /raid/part5
  │                                                                                         /raid/part4
  │                                                                                         /raid/part3
  │                                                                                         /raid/part2
  │                                                                                         /raid/part1
  ├─md0p2 ext4              1.0         a1582b62-86e3-4827-977b-371d1bbde5d8
  ├─md0p3 ext4              1.0         b7e36350-a2ac-4c8f-b237-bb98a00f652e
  ├─md0p4 ext4              1.0         04bb42fa-cfd6-48a6-96bd-a4a89b044b2a
  └─md0p5 ext4              1.0         a359777f-f6f7-45cd-9783-eb6803bd9459
sdd       linux_raid_member 1.2   UBS:0 cc513b0a-bd51-aeae-4192-a97c0236b539
└─md0
  ├─md0p1 ext4              1.0         163142ce-c39b-4d6e-a8dc-89dd80d04e85  159.9M     0% /raid/part5
  │                                                                                         /raid/part4
  │                                                                                         /raid/part3
  │                                                                                         /raid/part2
  │                                                                                         /raid/part1
  ├─md0p2 ext4              1.0         a1582b62-86e3-4827-977b-371d1bbde5d8
  ├─md0p3 ext4              1.0         b7e36350-a2ac-4c8f-b237-bb98a00f652e
  ├─md0p4 ext4              1.0         04bb42fa-cfd6-48a6-96bd-a4a89b044b2a
  └─md0p5 ext4              1.0         a359777f-f6f7-45cd-9783-eb6803bd9459
sde       linux_raid_member 1.2   UBS:0 cc513b0a-bd51-aeae-4192-a97c0236b539
└─md0
  ├─md0p1 ext4              1.0         163142ce-c39b-4d6e-a8dc-89dd80d04e85  159.9M     0% /raid/part5
  │                                                                                         /raid/part4
  │                                                                                         /raid/part3
  │                                                                                         /raid/part2
  │                                                                                         /raid/part1
  ├─md0p2 ext4              1.0         a1582b62-86e3-4827-977b-371d1bbde5d8
  ├─md0p3 ext4              1.0         b7e36350-a2ac-4c8f-b237-bb98a00f652e
  ├─md0p4 ext4              1.0         04bb42fa-cfd6-48a6-96bd-a4a89b044b2a
  └─md0p5 ext4              1.0         a359777f-f6f7-45cd-9783-eb6803bd9459
sdf       linux_raid_member 1.2   UBS:0 cc513b0a-bd51-aeae-4192-a97c0236b539
└─md0
  ├─md0p1 ext4              1.0         163142ce-c39b-4d6e-a8dc-89dd80d04e85  159.9M     0% /raid/part5
  │                                                                                         /raid/part4
  │                                                                                         /raid/part3
  │                                                                                         /raid/part2
  │                                                                                         /raid/part1
  ├─md0p2 ext4              1.0         a1582b62-86e3-4827-977b-371d1bbde5d8
  ├─md0p3 ext4              1.0         b7e36350-a2ac-4c8f-b237-bb98a00f652e
  ├─md0p4 ext4              1.0         04bb42fa-cfd6-48a6-96bd-a4a89b044b2a
  └─md0p5 ext4              1.0         a359777f-f6f7-45cd-9783-eb6803bd9459
sr0

```
10. Ужасаемся, что смонтировали 5 раз одну и туже партицию.
Используем umount, чтобы убрать лишнее и монтируем заново, проверяем. Теперь порядок!

```
root@UBS:~# mount /dev/md0p1 /raid/part1
root@UBS:~# mount /dev/md0p2 /raid/part2
root@UBS:~# mount /dev/md0p3 /raid/part3
root@UBS:~# mount /dev/md0p4 /raid/part4
root@UBS:~# mount /dev/md0p5 /raid/part5
root@UBS:~#
root@UBS:~#
root@UBS:~# lsblk -f
NAME      FSTYPE            FSVER LABEL UUID                                 FSAVAIL FSUSE% MOUNTPOINTS
sda
├─sda1
└─sda2    ext4              1.0         14870e43-0110-49f3-aa39-a50aae2a6b93   20.4G    11% /
sdb       linux_raid_member 1.2   UBS:0 cc513b0a-bd51-aeae-4192-a97c0236b539
sdc       linux_raid_member 1.2   UBS:0 cc513b0a-bd51-aeae-4192-a97c0236b539
└─md0
  ├─md0p1 ext4              1.0         163142ce-c39b-4d6e-a8dc-89dd80d04e85  159.9M     0% /raid/part1
  ├─md0p2 ext4              1.0         a1582b62-86e3-4827-977b-371d1bbde5d8    250M     0% /raid/part2
  ├─md0p3 ext4              1.0         b7e36350-a2ac-4c8f-b237-bb98a00f652e  426.5M     0% /raid/part3
  ├─md0p4 ext4              1.0         04bb42fa-cfd6-48a6-96bd-a4a89b044b2a  825.3M     0% /raid/part4
  └─md0p5 ext4              1.0         a359777f-f6f7-45cd-9783-eb6803bd9459   83.5M     0% /raid/part5
sdd       linux_raid_member 1.2   UBS:0 cc513b0a-bd51-aeae-4192-a97c0236b539
└─md0
  ├─md0p1 ext4              1.0         163142ce-c39b-4d6e-a8dc-89dd80d04e85  159.9M     0% /raid/part1
  ├─md0p2 ext4              1.0         a1582b62-86e3-4827-977b-371d1bbde5d8    250M     0% /raid/part2
  ├─md0p3 ext4              1.0         b7e36350-a2ac-4c8f-b237-bb98a00f652e  426.5M     0% /raid/part3
  ├─md0p4 ext4              1.0         04bb42fa-cfd6-48a6-96bd-a4a89b044b2a  825.3M     0% /raid/part4
  └─md0p5 ext4              1.0         a359777f-f6f7-45cd-9783-eb6803bd9459   83.5M     0% /raid/part5
sde       linux_raid_member 1.2   UBS:0 cc513b0a-bd51-aeae-4192-a97c0236b539
└─md0
  ├─md0p1 ext4              1.0         163142ce-c39b-4d6e-a8dc-89dd80d04e85  159.9M     0% /raid/part1
  ├─md0p2 ext4              1.0         a1582b62-86e3-4827-977b-371d1bbde5d8    250M     0% /raid/part2
  ├─md0p3 ext4              1.0         b7e36350-a2ac-4c8f-b237-bb98a00f652e  426.5M     0% /raid/part3
  ├─md0p4 ext4              1.0         04bb42fa-cfd6-48a6-96bd-a4a89b044b2a  825.3M     0% /raid/part4
  └─md0p5 ext4              1.0         a359777f-f6f7-45cd-9783-eb6803bd9459   83.5M     0% /raid/part5
sdf       linux_raid_member 1.2   UBS:0 cc513b0a-bd51-aeae-4192-a97c0236b539
└─md0
  ├─md0p1 ext4              1.0         163142ce-c39b-4d6e-a8dc-89dd80d04e85  159.9M     0% /raid/part1
  ├─md0p2 ext4              1.0         a1582b62-86e3-4827-977b-371d1bbde5d8    250M     0% /raid/part2
  ├─md0p3 ext4              1.0         b7e36350-a2ac-4c8f-b237-bb98a00f652e  426.5M     0% /raid/part3
  ├─md0p4 ext4              1.0         04bb42fa-cfd6-48a6-96bd-a4a89b044b2a  825.3M     0% /raid/part4
  └─md0p5 ext4              1.0         a359777f-f6f7-45cd-9783-eb6803bd9459   83.5M     0% /raid/part5
sr0

```
