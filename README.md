﻿# stands-03-lvm

Стенд для домашнего занятия "Файловые системы и LVM"

# Решение


1. Установка пакета xfsdump для резервного копирования файловой системы XFS

[vagrant@lvm ~]$ sudo yum install xfsdump
Loaded plugins: fastestmirror
Loading mirror speeds from cached hostfile
 * base: mirror.sale-dedic.com
 * extras: mirror.corbina.net
 * updates: mirror.corbina.net
base                                                     | 3.6 kB     00:00
extras                                                   | 2.9 kB     00:00
updates                                                  | 2.9 kB     00:00
Resolving Dependencies
--> Running transaction check
---> Package xfsdump.x86_64 0:3.1.7-1.el7 will be installed
--> Processing Dependency: attr >= 2.0.0 for package: xfsdump-3.1.7-1.el7.x86_64
--> Running transaction check
---> Package attr.x86_64 0:2.4.46-13.el7 will be installed
--> Finished Dependency Resolution

Dependencies Resolved

================================================================================
 Package          Arch            Version                   Repository     Size
================================================================================
Installing:
 xfsdump          x86_64          3.1.7-1.el7               base          308 k
Installing for dependencies:
 attr             x86_64          2.4.46-13.el7             base           66 k

Transaction Summary
================================================================================
Install  1 Package (+1 Dependent package)

Total download size: 374 k
Installed size: 1.1 M
Is this ok [y/d/N]: y
Downloading packages:
attr-2.4.46-13.el7.x86_64.rpm  FAILED
http://mirror.reconn.ru/centos/7.7.1908/os/x86_64/Packages/attr-2.4.46-13.el7.x86_64.rpm: [Errno 14] curl#6 - "Could not resolve host: mirror.reconn.ru; Unknown error"
Trying other mirror.
xfsdump-3.1.7-1.el7.x86_64.rpm FAILED
http://mirror.reconn.ru/centos/7.7.1908/os/x86_64/Packages/xfsdump-3.1.7-1.el7.x86_64.rpm: [Errno 14] curl#6 - "Could not resolve host: mirror.reconn.ru; Unknown error"
Trying other mirror.
(1/2): xfsdump-3.1.7-1.el7.x86_64.rpm                      | 308 kB   00:00
attr-2.4.46-13.el7.x86_64.rpm  FAILED
http://mirror.sale-dedic.com/centos/7.7.1908/os/x86_64/Packages/attr-2.4.46-13.el7.x86_64.rpm: [Errno 14] curl#6 - "Could not resolve host: mirror.sale-dedic.com; Unknown error"
Trying other mirror.
(2/2): attr-2.4.46-13.el7.x86_64.rpm                       |  66 kB   00:15
--------------------------------------------------------------------------------
Total                                              7.9 kB/s | 374 kB  00:47
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  Installing : attr-2.4.46-13.el7.x86_64                                    1/2
  Installing : xfsdump-3.1.7-1.el7.x86_64                                   2/2
  Verifying  : attr-2.4.46-13.el7.x86_64                                    1/2
  Verifying  : xfsdump-3.1.7-1.el7.x86_64                                   2/2

Installed:
  xfsdump.x86_64 0:3.1.7-1.el7

Dependency Installed:
  attr.x86_64 0:2.4.46-13.el7

Complete!


2. Проверим текущие разделы

[vagrant@lvm ~]$ lsblk
NAME                    MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda                       8:0    0   40G  0 disk 
+-sda1                    8:1    0    1M  0 part 
+-sda2                    8:2    0    1G  0 part /boot
L-sda3                    8:3    0   39G  0 part 
  +-VolGroup00-LogVol00 253:0    0 37.5G  0 lvm  /
  L-VolGroup00-LogVol01 253:1    0  1.5G  0 lvm  [SWAP]
sdb                       8:16   0   10G  0 disk 
sdc                       8:32   0    2G  0 disk 
sdd                       8:48   0    1G  0 disk 
sde                       8:64   0    1G  0 disk 


3. Создадим физический раздел на основе диска sdb

[vagrant@lvm ~]$ sudo pvcreate /dev/sdb
  Physical volume "/dev/sdb" successfully created.


4. Создадим группу физических разделов с именем vg_root на основе диска sdb

[vagrant@lvm ~]$ sudo vgcreate vg_root /dev/sdb
  Volume group "vg_root" successfully created


5. Создадим на основе группы vg_root логический раздел lv_root, используя всю доступную область

[vagrant@lvm ~]$ sudo lvcreate -n lv_root -l +100%FREE /dev/vg_root
  Logical volume "lv_root" created.


6. Создадим на логическом разделе lv_root файловую систему

[vagrant@lvm ~]$ sudo mkfs.xfs /dev/vg_root/lv_root
meta-data=/dev/vg_root/lv_root   isize=512    agcount=4, agsize=655104 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=1        finobt=0, sparse=0
data     =                       bsize=4096   blocks=2620416, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0 ftype=1
log      =internal log           bsize=4096   blocks=2560, version=2
         =                       sectsz=512   sunit=0 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0


7. Смонтируем логический раздел lv_root в /mnt

[vagrant@lvm ~]$ sudo mount /dev/vg_root/lv_root /mnt


8. Cкопируем все данные с раздела LogVol00 в раздел lv_root
(Примечание - копирование с указанием sudo не проходит, необходимо перейти в суперпользователя: sudo su)

[vagrant@lvm ~]$ sudo su
[root@lvm vagrant]# xfsdump -J - /dev/VolGroup00/LogVol00 | xfsrestore -J - /mnt
...
xfsrestore: restore complete: 158 seconds elapsed
xfsrestore: Restore Status: SUCCESS


9. Проверим все ли скопировалось в раздел lv_root

[root@lvm vagrant]# ls /mnt
bin   dev  home  lib64  mnt  proc  run   srv  tmp  vagrant
boot  etc  lib   media  opt  root  sbin  sys  usr  var


10. Смонтируем часть файловой структуры в другой каталог, не удаляя при этом исходную точку монтирования

[root@lvm vagrant]# for i in /proc/ /sys/ /dev/ /run/ /boot/; do mount --bind $i /mnt/$i; done


11. Сменим корневую директорию (/) на каталог /mnt

[root@lvm vagrant]# chroot /mnt/


12. Обновим  grub

[root@lvm /]# grub2-mkconfig -o /boot/grub2/grub.cfg
Generating grub configuration file ...
Found linux image: /boot/vmlinuz-3.10.0-862.2.3.el7.x86_64
Found initrd image: /boot/initramfs-3.10.0-862.2.3.el7.x86_64.img
done


13. Обновим образ initrd

[root@lvm /]# cd /boot ; for i in `ls initramfs-*img`; do dracut -v $i `echo $i|sed "s/initramfs-//g; s/.img//g"` --force; done
Executing: /sbin/dracut -v initramfs-3.10.0-862.2.3.el7.x86_64.img 3.10.0-862.2.3.el7.x86_64 --force
...
*** Creating image file ***
*** Creating microcode section ***
*** Created microcode section ***
*** Creating image file done ***
*** Creating initramfs image file '/boot/initramfs-3.10.0-862.2.3.el7.x86_64.img' done ***


14. Посмотрим на какой разделе у нас ссылается rd.lvm.lv

[root@lvm boot]# cat /boot/grub2/grub.cfg
#
# DO NOT EDIT THIS FILE
#
# It is automatically generated by grub2-mkconfig using templates
# from /etc/grub.d and settings from /etc/default/grub
#
...
### BEGIN /etc/grub.d/10_linux ###
menuentry 'CentOS Linux (3.10.0-862.2.3.el7.x86_64) 7 (Core)' --class centos --class gnu-linux --class gnu --class os --unrestricted $menuentry_id_option 'gnulinux-3.10.0-862.2.3.el7.x86_64-advanced-8c74fbf7-9a6d-40f2-9333-d9b4146e8635' {
	load_video
	set gfxpayload=keep
	insmod gzio
	insmod part_msdos
	insmod xfs
	set root='hd0,msdos2'
	if [ x$feature_platform_search_hint = xy ]; then
	  search --no-floppy --fs-uuid --set=root --hint-bios=hd0,msdos2 --hint-efi=hd0,msdos2 --hint-baremetal=ahci0,msdos2  570897ca-e759-4c81-90cf-389da6eee4cc
	else
	  search --no-floppy --fs-uuid --set=root 570897ca-e759-4c81-90cf-389da6eee4cc
	fi
	linux16 /vmlinuz-3.10.0-862.2.3.el7.x86_64 root=/dev/mapper/vg_root-lv_root ro no_timer_check console=tty0 console=ttyS0,115200n8 net.ifnames=0 biosdevname=0 elevator=noop crashkernel=auto rd.lvm.lv=VolGroup00/LogVol00 rd.lvm.lv=VolGroup00/LogVol01 rhgb quiet 
	initrd16 /initramfs-3.10.0-862.2.3.el7.x86_64.img
}
if [ "x$default" = 'CentOS Linux (3.10.0-862.2.3.el7.x86_64) 7 (Core)' ]; then default='Advanced options for CentOS Linux>CentOS Linux (3.10.0-862.2.3.el7.x86_64) 7 (Core)'; fi;
### END /etc/grub.d/10_linux ###
...


15. Посмотрим содержимое файла конфигурации 

[root@lvm boot]# cat /etc/default/grub
GRUB_TIMEOUT=1
GRUB_DISTRIBUTOR="$(sed 's, release .*$,,g' /etc/system-release)"
GRUB_DEFAULT=saved
GRUB_DISABLE_SUBMENU=true
GRUB_TERMINAL_OUTPUT="console"
GRUB_CMDLINE_LINUX="no_timer_check console=tty0 console=ttyS0,115200n8 net.ifnames=0 biosdevname=0 elevator=noop crashkernel=auto rd.lvm.lv=VolGroup00/LogVol00 rd.lvm.lv=VolGroup00/LogVol01 rhgb quiet"
GRUB_DISABLE_RECOVERY="true"


16. Изменим файл конфигурации, переопределив rd.lvm.lv

[root@lvm boot]# vi /etc/default/grub


17. Обновим  grub

[root@lvm boot]# grub2-mkconfig -o /boot/grub2/grub.cfg
Generating grub configuration file ...
Found linux image: /boot/vmlinuz-3.10.0-862.2.3.el7.x86_64
Found initrd image: /boot/initramfs-3.10.0-862.2.3.el7.x86_64.img
done


18. Убедимся, что rd.lvm.lv теперь у нас ссылается на vg_root/lv_root

[root@lvm boot]# cat /boot/grub2/grub.cfg
#
# DO NOT EDIT THIS FILE
#
# It is automatically generated by grub2-mkconfig using templates
# from /etc/grub.d and settings from /etc/default/grub
#
...
### BEGIN /etc/grub.d/10_linux ###
menuentry 'CentOS Linux (3.10.0-862.2.3.el7.x86_64) 7 (Core)' --class centos --class gnu-linux --class gnu --class os --unrestricted $menuentry_id_option 'gnulinux-3.10.0-862.2.3.el7.x86_64-advanced-8c74fbf7-9a6d-40f2-9333-d9b4146e8635' {
	load_video
	set gfxpayload=keep
	insmod gzio
	insmod part_msdos
	insmod xfs
	set root='hd0,msdos2'
	if [ x$feature_platform_search_hint = xy ]; then
	  search --no-floppy --fs-uuid --set=root --hint-bios=hd0,msdos2 --hint-efi=hd0,msdos2 --hint-baremetal=ahci0,msdos2  570897ca-e759-4c81-90cf-389da6eee4cc
	else
	  search --no-floppy --fs-uuid --set=root 570897ca-e759-4c81-90cf-389da6eee4cc
	fi
	linux16 /vmlinuz-3.10.0-862.2.3.el7.x86_64 root=/dev/mapper/vg_root-lv_root ro no_timer_check console=tty0 console=ttyS0,115200n8 net.ifnames=0 biosdevname=0 elevator=noop crashkernel=auto rd.lvm.lv=vg_root/lv_root rd.lvm.lv=VolGroup00/LogVol01 rhgb quiet 
	initrd16 /initramfs-3.10.0-862.2.3.el7.x86_64.img
}
if [ "x$default" = 'CentOS Linux (3.10.0-862.2.3.el7.x86_64) 7 (Core)' ]; then default='Advanced options for CentOS Linux>CentOS Linux (3.10.0-862.2.3.el7.x86_64) 7 (Core)'; fi;
### END /etc/grub.d/10_linux ###
...


19. Проверим новый рут

[root@lvm boot]# lsblk
NAME                    MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda                       8:0    0   40G  0 disk 
+-sda1                    8:1    0    1M  0 part 
+-sda2                    8:2    0    1G  0 part /boot
L-sda3                    8:3    0   39G  0 part 
  +-VolGroup00-LogVol00 253:0    0 37.5G  0 lvm  
  L-VolGroup00-LogVol01 253:1    0  1.5G  0 lvm  [SWAP]
sdb                       8:16   0   10G  0 disk 
L-vg_root-lv_root       253:2    0   10G  0 lvm  /
sdc                       8:32   0    2G  0 disk 
sdd                       8:48   0    1G  0 disk 
sde                       8:64   0    1G  0 disk 


20. Выходим и перезагружаемся

[root@lvm boot]# exit


21. Проверям разделы после перезагрузки

[vagrant@lvm ~]$ lsblk
NAME                    MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda                       8:0    0   10G  0 disk 
└─vg_root-lv_root       253:0    0   10G  0 lvm  /
sdb                       8:16   0    2G  0 disk 
sdc                       8:32   0    1G  0 disk 
sdd                       8:48   0    1G  0 disk 
sde                       8:64   0   40G  0 disk 
├─sde1                    8:65   0    1M  0 part 
├─sde2                    8:66   0    1G  0 part /boot
└─sde3                    8:67   0   39G  0 part 
  ├─VolGroup00-LogVol01 253:1    0  1.5G  0 lvm  [SWAP]
  └─VolGroup00-LogVol00 253:2    0 37.5G  0 lvm  


22. Удалим логический раздел LogVol00

[vagrant@lvm ~]$ sudo su
[root@lvm vagrant]# lvremove /dev/VolGroup00/LogVol00
Do you really want to remove active logical volume VolGroup00/LogVol00? [y/n]: y
  Logical volume "LogVol00" successfully removed


23. Создадим логический раздел LogVol00 размером 8 ГБ

[root@lvm vagrant]# lvcreate -n VolGroup00/LogVol00 -L 8G /dev/VolGroup00
WARNING: xfs signature detected on /dev/VolGroup00/LogVol00 at offset 0. Wipe it? [y/n]: y
  Wiping xfs signature on /dev/VolGroup00/LogVol00.
  Logical volume "LogVol00" created.


24. Создадим файловую систему xfs

[root@lvm vagrant]# mkfs.xfs /dev/VolGroup00/LogVol00
meta-data=/dev/VolGroup00/LogVol00 isize=512    agcount=4, agsize=524288 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=1        finobt=0, sparse=0
data     =                       bsize=4096   blocks=2097152, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0 ftype=1
log      =internal log           bsize=4096   blocks=2560, version=2
         =                       sectsz=512   sunit=0 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0


25. Смонтируем новый логичекий раздел

[root@lvm vagrant]# mount /dev/VolGroup00/LogVol00 /mnt


26. Проверим разделы

[root@lvm vagrant]# lsblk
NAME                    MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda                       8:0    0   10G  0 disk 
└─vg_root-lv_root       253:0    0   10G  0 lvm  /
sdb                       8:16   0    2G  0 disk 
sdc                       8:32   0    1G  0 disk 
sdd                       8:48   0    1G  0 disk 
sde                       8:64   0   40G  0 disk 
├─sde1                    8:65   0    1M  0 part 
├─sde2                    8:66   0    1G  0 part /boot
└─sde3                    8:67   0   39G  0 part 
  ├─VolGroup00-LogVol01 253:1    0  1.5G  0 lvm  [SWAP]
  └─VolGroup00-LogVol00 253:2    0    8G  0 lvm  /mnt


27. Сделаем дамп с lv_root на LogVol00

[root@lvm vagrant]# xfsdump -J - /dev/vg_root/lv_root | xfsrestore -J - /mnt
...
xfsdump: Dump Status: SUCCESS
xfsrestore: restore complete: 96 seconds elapsed
xfsrestore: Restore Status: SUCCESS


28. Смонтируем часть файловой структуры в другой каталог

[root@lvm vagrant]#  for i in /proc/ /sys/ /dev/ /run/ /boot/; do mount --bind $i /mnt/$i; done


29. Сменим корневую директорию (/) на каталог /mnt

[root@lvm vagrant]# chroot /mnt/


30. Изменим файл конфигурации, переопределив rd.lvm.lv

[root@lvm /]# vi /etc/default/grub


31. Обновим  grub

[root@lvm /]# grub2-mkconfig -o /boot/grub2/grub.cfg
Generating grub configuration file ...
Found linux image: /boot/vmlinuz-3.10.0-862.2.3.el7.x86_64
Found initrd image: /boot/initramfs-3.10.0-862.2.3.el7.x86_64.img
done


32. Убедимся, что rd.lvm.lv теперь у нас ссылается на VolGroup00/LogVol00

[root@lvm /]# cat /boot/grub2/grub.cfg
#
# DO NOT EDIT THIS FILE
#
# It is automatically generated by grub2-mkconfig using templates
# from /etc/grub.d and settings from /etc/default/grub
#
...
### BEGIN /etc/grub.d/10_linux ###
menuentry 'CentOS Linux (3.10.0-862.2.3.el7.x86_64) 7 (Core)' --class centos --class gnu-linux --class gnu --class os --unrestricted $menuentry_id_option 'gnulinux-3.10.0-862.2.3.el7.x86_64-advanced-035a6686-d5ec-4c32-a568-4285fe011304' {
	load_video
	set gfxpayload=keep
	insmod gzio
	insmod part_msdos
	insmod xfs
	set root='hd4,msdos2'
	if [ x$feature_platform_search_hint = xy ]; then
	  search --no-floppy --fs-uuid --set=root --hint-bios=hd4,msdos2 --hint-efi=hd4,msdos2 --hint-baremetal=ahci4,msdos2  570897ca-e759-4c81-90cf-389da6eee4cc
	else
	  search --no-floppy --fs-uuid --set=root 570897ca-e759-4c81-90cf-389da6eee4cc
	fi
	linux16 /vmlinuz-3.10.0-862.2.3.el7.x86_64 root=/dev/mapper/VolGroup00-LogVol00 ro no_timer_check console=tty0 console=ttyS0,115200n8 net.ifnames=0 biosdevname=0 elevator=noop crashkernel=auto rd.lvm.lv=VolGroup00/LogVol00 rd.lvm.lv=VolGroup00/LogVol01 rhgb quiet 
	initrd16 /initramfs-3.10.0-862.2.3.el7.x86_64.img
}
if [ "x$default" = 'CentOS Linux (3.10.0-862.2.3.el7.x86_64) 7 (Core)' ]; then default='Advanced options for CentOS Linux>CentOS Linux (3.10.0-862.2.3.el7.x86_64) 7 (Core)'; fi;
### END /etc/grub.d/10_linux ###
...


33. Проверим разделы

[root@lvm /]# lsblk
NAME                    MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda                       8:0    0   10G  0 disk 
└─vg_root-lv_root       253:0    0   10G  0 lvm  
sdb                       8:16   0    2G  0 disk 
sdc                       8:32   0    1G  0 disk 
sdd                       8:48   0    1G  0 disk 
sde                       8:64   0   40G  0 disk 
├─sde1                    8:65   0    1M  0 part 
├─sde2                    8:66   0    1G  0 part /boot
└─sde3                    8:67   0   39G  0 part 
  ├─VolGroup00-LogVol01 253:1    0  1.5G  0 lvm  [SWAP]
  └─VolGroup00-LogVol00 253:2    0    8G  0 lvm  /


34. Обновим образ initrd

[root@lvm /]# cd /boot ; for i in `ls initramfs-*img`; do dracut -v $i `echo $i|sed "s/initramfs-//g; s/.img//g"` --force; done
Executing: /sbin/dracut -v initramfs-3.10.0-862.2.3.el7.x86_64.img 3.10.0-862.2.3.el7.x86_64 --force
...
*** Creating image file ***
*** Creating microcode section ***
*** Created microcode section ***
*** Creating image file done ***
*** Creating initramfs image file '/boot/initramfs-3.10.0-862.2.3.el7.x86_64.img' done ***


35. Создадим зеркальный том на разделах /dev/sdc и /dev/sdd

[root@lvm boot]# pvcreate /dev/sdc /dev/sdd
  Physical volume "/dev/sdc" successfully created.
  Physical volume "/dev/sdd" successfully created.


36. Создадим группу разделов

[root@lvm boot]# vgcreate vg_var /dev/sdc /dev/sdd
  Volume group "vg_var" successfully created


37. Создадим логический раздел lv_var

[root@lvm boot]# lvcreate -L 990M -m1 -n lv_var vg_var
  Rounding up size to full physical extent 992.00 MiB
  Logical volume "lv_var" created.


38. Создаем файловую систему ext4 на разделе lv_var

[root@lvm boot]# mkfs.ext4 /dev/vg_var/lv_var
mke2fs 1.42.9 (28-Dec-2013)
Filesystem label=
OS type: Linux
Block size=4096 (log=2)
Fragment size=4096 (log=2)
Stride=0 blocks, Stripe width=0 blocks
63488 inodes, 253952 blocks
12697 blocks (5.00%) reserved for the super user
First data block=0
Maximum filesystem blocks=260046848
8 block groups
32768 blocks per group, 32768 fragments per group
7936 inodes per group
Superblock backups stored on blocks: 
	32768, 98304, 163840, 229376

Allocating group tables: done                            
Writing inode tables: done                            
Creating journal (4096 blocks): done
Writing superblocks and filesystem accounting information: done


39. Смонтируем раздел 

[root@lvm boot]# mount /dev/vg_var/lv_var /mnt


40. Скопируем содержимое /var в раздел lv_var

[root@lvm boot]# cp -aR /var/* /mnt/


41. Создаем временный раздел /tmp/oldvar и перемещаем в него все данные из /var 

[root@lvm boot]# mkdir /tmp/oldvar && mv /var/* /tmp/oldvar


42. Проверим перемещенные данные

[root@lvm boot]# ls /tmp/oldvar
adm    db     games   kerberos  local  log   nis  preserve  spool  yp
cache  empty  gopher  lib       lock   mail  opt  run       tmp


43. Размонтируем раздел lv_var из /mnt и смонтируем его в /var

[root@lvm boot]# umount /mnt
[root@lvm boot]# mount /dev/vg_var/lv_var /var


44. Проверим данные

[root@lvm boot]# ls /var
adm    db     games   kerberos  local  log         mail  opt       run    tmp
cache  empty  gopher  lib       lock   lost+found  nis   preserve  spool  yp


45. Проверим исходное состояние fstab

[root@lvm boot]# cat /etc/fstab

#
# /etc/fstab
# Created by anaconda on Sat May 12 18:50:26 2018
#
# Accessible filesystems, by reference, are maintained under '/dev/disk'
# See man pages fstab(5), findfs(8), mount(8) and/or blkid(8) for more info
#
/dev/mapper/VolGroup00-LogVol00 /                       xfs     defaults        0 0
UUID=570897ca-e759-4c81-90cf-389da6eee4cc /boot                   xfs     defaults        0 0
/dev/mapper/VolGroup00-LogVol01 swap                    swap    defaults        0 0


46. Добавим в fstab автоматическое монтирование раздела lv_var в /var

[root@lvm boot]# echo "`blkid | grep var: | awk '{print $2}'` /var ext4 defaults 0 0" >> /etc/fstab


47. Проверим конечное состояние fstab

[root@lvm boot]# cat /etc/fstab

#
# /etc/fstab
# Created by anaconda on Sat May 12 18:50:26 2018
#
# Accessible filesystems, by reference, are maintained under '/dev/disk'
# See man pages fstab(5), findfs(8), mount(8) and/or blkid(8) for more info
#
/dev/mapper/VolGroup00-LogVol00 /                       xfs     defaults        0 0
UUID=570897ca-e759-4c81-90cf-389da6eee4cc /boot                   xfs     defaults        0 0
/dev/mapper/VolGroup00-LogVol01 swap                    swap    defaults        0 0
UUID="fba107f8-4fb3-4ecf-bada-309408b5d61e" /var ext4 defaults 0 0


48. Выйдем и перезагрузимся

[root@lvm boot]# exit
[root@lvm vagrant]# reboot


49. Проверим разделы после перезагрузки

[vagrant@lvm ~]$ lsblk
NAME                     MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda                        8:0    0   40G  0 disk 
├─sda1                     8:1    0    1M  0 part 
├─sda2                     8:2    0    1G  0 part /boot
└─sda3                     8:3    0   39G  0 part 
  ├─VolGroup00-LogVol00  253:0    0    8G  0 lvm  /
  └─VolGroup00-LogVol01  253:1    0  1.5G  0 lvm  [SWAP]
sdb                        8:16   0   10G  0 disk 
└─vg_root-lv_root        253:2    0   10G  0 lvm  
sdc                        8:32   0    2G  0 disk 
sdd                        8:48   0    1G  0 disk 
├─vg_var-lv_var_rmeta_0  253:3    0    4M  0 lvm  
│ └─vg_var-lv_var        253:7    0  992M  0 lvm  /var
└─vg_var-lv_var_rimage_0 253:4    0  992M  0 lvm  
  └─vg_var-lv_var        253:7    0  992M  0 lvm  /var
sde                        8:64   0    1G  0 disk 
├─vg_var-lv_var_rmeta_1  253:5    0    4M  0 lvm  
│ └─vg_var-lv_var        253:7    0  992M  0 lvm  /var
└─vg_var-lv_var_rimage_1 253:6    0  992M  0 lvm  
  └─vg_var-lv_var        253:7    0  992M  0 lvm  /var


50. Удалим раздел lv_root, группу разделов vg_root и физический раздел sdb

[vagrant@lvm ~]$ sudo su
[root@lvm vagrant]# lvremove /dev/vg_root/lv_root
Do you really want to remove active logical volume vg_root/lv_root? [y/n]: y
  Logical volume "lv_root" successfully removed
[root@lvm vagrant]# vgremove /dev/vg_root
  Volume group "vg_root" successfully removed
[root@lvm vagrant]# pvremove /dev/sdb
  Labels on physical volume "/dev/sdb" successfully wiped.


51. Проверим разделы после удаления

[root@lvm vagrant]# lsblk
NAME                     MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda                        8:0    0   40G  0 disk 
├─sda1                     8:1    0    1M  0 part 
├─sda2                     8:2    0    1G  0 part /boot
└─sda3                     8:3    0   39G  0 part 
  ├─VolGroup00-LogVol00  253:0    0    8G  0 lvm  /
  └─VolGroup00-LogVol01  253:1    0  1.5G  0 lvm  [SWAP]
sdb                        8:16   0   10G  0 disk 
sdc                        8:32   0    2G  0 disk 
sdd                        8:48   0    1G  0 disk 
├─vg_var-lv_var_rmeta_0  253:3    0    4M  0 lvm  
│ └─vg_var-lv_var        253:7    0  992M  0 lvm  /var
└─vg_var-lv_var_rimage_0 253:4    0  992M  0 lvm  
  └─vg_var-lv_var        253:7    0  992M  0 lvm  /var
sde                        8:64   0    1G  0 disk 
├─vg_var-lv_var_rmeta_1  253:5    0    4M  0 lvm  
│ └─vg_var-lv_var        253:7    0  992M  0 lvm  /var
└─vg_var-lv_var_rimage_1 253:6    0  992M  0 lvm  
  └─vg_var-lv_var        253:7    0  992M  0 lvm  /var


52. Выделим логический раздел LogVol_Home для /home на группе разделов VolGroup00

[root@lvm vagrant]# lvcreate -n LogVol_Home -L 2G /dev/VolGroup00
  Logical volume "LogVol_Home" created.


53. Создадим файловую систему xfs

[root@lvm vagrant]# mkfs.xfs /dev/VolGroup00/LogVol_Home
meta-data=/dev/VolGroup00/LogVol_Home isize=512    agcount=4, agsize=131072 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=1        finobt=0, sparse=0
data     =                       bsize=4096   blocks=524288, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0 ftype=1
log      =internal log           bsize=4096   blocks=2560, version=2
         =                       sectsz=512   sunit=0 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0


54. Смонтируем новый раздел в /mnt

[root@lvm vagrant]# mount /dev/VolGroup00/LogVol_Home /mnt/


55. Скопируем данные из /home в /mnt и удалим данные в /home 

[root@lvm vagrant]# cp -aR /home/* /mnt/
[root@lvm vagrant]# rm -rf /home/*


56. Размонтируем раздел LogVol_Home из /mnt и смонтируем его в /home

[root@lvm vagrant]# umount /mnt
[root@lvm vagrant]# mount /dev/VolGroup00/LogVol_Home /home/


57. Добавим в fstab автоматическое монтирование раздела LogVol_Home в /home

[root@lvm vagrant]# echo "`blkid | grep Home | awk '{print $2}'` /home xfs defaults 0 0" >> /etc/fstab


58. Проверим конечное состояние fstab

[root@lvm vagrant]# cat /etc/fstab

#
# /etc/fstab
# Created by anaconda on Sat May 12 18:50:26 2018
#
# Accessible filesystems, by reference, are maintained under '/dev/disk'
# See man pages fstab(5), findfs(8), mount(8) and/or blkid(8) for more info
#
/dev/mapper/VolGroup00-LogVol00 /                       xfs     defaults        0 0
UUID=570897ca-e759-4c81-90cf-389da6eee4cc /boot                   xfs     defaults        0 0
/dev/mapper/VolGroup00-LogVol01 swap                    swap    defaults        0 0
UUID="fba107f8-4fb3-4ecf-bada-309408b5d61e" /var ext4 defaults 0 0
UUID="dfb160c8-3f42-4ec0-a81c-3886d03ab704" /home xfs defaults 0 0


59. Создадим 20 файлов в каталоге /home и проверим их наличие

[root@lvm vagrant]# touch /home/file{1..20}
[root@lvm vagrant]# ls /home
file1   file12  file15  file18  file20  file5  file8
file10  file13  file16  file19  file3   file6  file9
file11  file14  file17  file2   file4   file7  vagrant


60. Создадим снэпшот home_snap для раздела LogVol_Home

[root@lvm vagrant]# lvcreate -L 100M -s -n home_snap /dev/VolGroup00/LogVol_Home
  Rounding up size to full physical extent 128.00 MiB
  Logical volume "home_snap" created.


61. Проверим состояние разделов

[root@lvm vagrant]# lsblk
NAME                            MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda                               8:0    0   40G  0 disk 
├─sda1                            8:1    0    1M  0 part 
├─sda2                            8:2    0    1G  0 part /boot
└─sda3                            8:3    0   39G  0 part 
  ├─VolGroup00-LogVol00         253:0    0    8G  0 lvm  /
  ├─VolGroup00-LogVol01         253:1    0  1.5G  0 lvm  [SWAP]
  ├─VolGroup00-LogVol_Home-real 253:8    0    2G  0 lvm  
  │ ├─VolGroup00-LogVol_Home    253:2    0    2G  0 lvm  /home
  │ └─VolGroup00-home_snap      253:10   0    2G  0 lvm  
  └─VolGroup00-home_snap-cow    253:9    0  128M  0 lvm  
    └─VolGroup00-home_snap      253:10   0    2G  0 lvm  
sdb                               8:16   0   10G  0 disk 
sdc                               8:32   0    2G  0 disk 
sdd                               8:48   0    1G  0 disk 
├─vg_var-lv_var_rmeta_0         253:3    0    4M  0 lvm  
│ └─vg_var-lv_var               253:7    0  992M  0 lvm  /var
└─vg_var-lv_var_rimage_0        253:4    0  992M  0 lvm  
  └─vg_var-lv_var               253:7    0  992M  0 lvm  /var
sde                               8:64   0    1G  0 disk 
├─vg_var-lv_var_rmeta_1         253:5    0    4M  0 lvm  
│ └─vg_var-lv_var               253:7    0  992M  0 lvm  /var
└─vg_var-lv_var_rimage_1        253:6    0  992M  0 lvm  
  └─vg_var-lv_var               253:7    0  992M  0 lvm  /var


62. Удалим первые 10 файлов и проверим их отсутствие

[root@lvm vagrant]# rm -f /home/file{1..10}
[root@lvm vagrant]# ls /home
file11  file13  file15  file17  file19  vagrant
file12  file14  file16  file18  file20


63. Размонтируем LogVol_Home

[root@lvm vagrant]# umount /home


64. Выполним восстановление раздела LogVol_Home из снэпшота home_snap

[root@lvm vagrant]# lvconvert --merge /dev/VolGroup00/home_snap
  Merging of volume VolGroup00/home_snap started.
  VolGroup00/LogVol_Home: Merged: 100.00%


65. Проверим состояние разделов

[root@lvm vagrant]# lsblk
NAME                       MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda                          8:0    0   40G  0 disk 
├─sda1                       8:1    0    1M  0 part 
├─sda2                       8:2    0    1G  0 part /boot
└─sda3                       8:3    0   39G  0 part 
  ├─VolGroup00-LogVol00    253:0    0    8G  0 lvm  /
  ├─VolGroup00-LogVol01    253:1    0  1.5G  0 lvm  [SWAP]
  └─VolGroup00-LogVol_Home 253:2    0    2G  0 lvm  
sdb                          8:16   0   10G  0 disk 
sdc                          8:32   0    2G  0 disk 
sdd                          8:48   0    1G  0 disk 
├─vg_var-lv_var_rmeta_0    253:3    0    4M  0 lvm  
│ └─vg_var-lv_var          253:7    0  992M  0 lvm  /var
└─vg_var-lv_var_rimage_0   253:4    0  992M  0 lvm  
  └─vg_var-lv_var          253:7    0  992M  0 lvm  /var
sde                          8:64   0    1G  0 disk 
├─vg_var-lv_var_rmeta_1    253:5    0    4M  0 lvm  
│ └─vg_var-lv_var          253:7    0  992M  0 lvm  /var
└─vg_var-lv_var_rimage_1   253:6    0  992M  0 lvm  
  └─vg_var-lv_var          253:7    0  992M  0 lvm  /var


66. Смонтируем /home

[root@lvm vagrant]# mount /home


67. Проверим состояние разделов

[root@lvm vagrant]# lsblk
NAME                       MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda                          8:0    0   40G  0 disk 
├─sda1                       8:1    0    1M  0 part 
├─sda2                       8:2    0    1G  0 part /boot
└─sda3                       8:3    0   39G  0 part 
  ├─VolGroup00-LogVol00    253:0    0    8G  0 lvm  /
  ├─VolGroup00-LogVol01    253:1    0  1.5G  0 lvm  [SWAP]
  └─VolGroup00-LogVol_Home 253:2    0    2G  0 lvm  /home
sdb                          8:16   0   10G  0 disk 
sdc                          8:32   0    2G  0 disk 
sdd                          8:48   0    1G  0 disk 
├─vg_var-lv_var_rmeta_0    253:3    0    4M  0 lvm  
│ └─vg_var-lv_var          253:7    0  992M  0 lvm  /var
└─vg_var-lv_var_rimage_0   253:4    0  992M  0 lvm  
  └─vg_var-lv_var          253:7    0  992M  0 lvm  /var
sde                          8:64   0    1G  0 disk 
├─vg_var-lv_var_rmeta_1    253:5    0    4M  0 lvm  
│ └─vg_var-lv_var          253:7    0  992M  0 lvm  /var
└─vg_var-lv_var_rimage_1   253:6    0  992M  0 lvm  
  └─vg_var-lv_var          253:7    0  992M  0 lvm  /var


68. Убедимся, что удаленные файлы восстановлены

[root@lvm vagrant]# ls /home
file1   file12  file15  file18  file20  file5  file8
file10  file13  file16  file19  file3   file6  file9
file11  file14  file17  file2   file4   file7  vagrant
