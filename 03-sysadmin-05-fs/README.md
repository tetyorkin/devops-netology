# Домашнее задание к занятию "3.5. Файловые системы"

1. Узнайте о [sparse](https://ru.wikipedia.org/wiki/%D0%A0%D0%B0%D0%B7%D1%80%D0%B5%D0%B6%D1%91%D0%BD%D0%BD%D1%8B%D0%B9_%D1%84%D0%B0%D0%B9%D0%BB) (разряженных) файлах.
    * Узнал


2. Могут ли файлы, являющиеся жесткой ссылкой на один объект, иметь разные права доступа и владельца? Почему?
    * Нет, так как ссылка происходит на inode, которая и хранит права доступа
    


3. Сделайте `vagrant destroy` на имеющийся инстанс Ubuntu. Замените содержимое Vagrantfile следующим:

    ```bash
    Vagrant.configure("2") do |config|
      config.vm.box = "bento/ubuntu-20.04"
      config.vm.provider :virtualbox do |vb|
        lvm_experiments_disk0_path = "/tmp/lvm_experiments_disk0.vmdk"
        lvm_experiments_disk1_path = "/tmp/lvm_experiments_disk1.vmdk"
        vb.customize ['createmedium', '--filename', lvm_experiments_disk0_path, '--size', 2560]
        vb.customize ['createmedium', '--filename', lvm_experiments_disk1_path, '--size', 2560]
        vb.customize ['storageattach', :id, '--storagectl', 'SATA Controller', '--port', 1, '--device', 0, '--type', 'hdd', '--medium', lvm_experiments_disk0_path]
        vb.customize ['storageattach', :id, '--storagectl', 'SATA Controller', '--port', 2, '--device', 0, '--type', 'hdd', '--medium', lvm_experiments_disk1_path]
      end
    end
    ```

    Данная конфигурация создаст новую виртуальную машину с двумя дополнительными неразмеченными дисками по 2.5 Гб.
    * Создал, действительно создалась vm с двумя неразмеченными дисками по 2.5 Гб.
        ```bash
            vagrant@vagrant:~$ lsblk
            NAME                 MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
            sda                    8:0    0   64G  0 disk 
            ├─sda1                 8:1    0  512M  0 part /boot/efi
            ├─sda2                 8:2    0    1K  0 part 
            └─sda5                 8:5    0 63.5G  0 part 
            ├─vgvagrant-root   253:0    0 62.6G  0 lvm  /
            └─vgvagrant-swap_1 253:1    0  980M  0 lvm  [SWAP]
            sdb                    8:16   0  2.5G  0 disk 
            sdc                    8:32   0  2.5G  0 disk  
      
       ```

4. Используя `fdisk`, разбейте первый диск на 2 раздела: 2 Гб, оставшееся пространство.
   *
   ```bash
   root@vagrant:~# fdisk /dev/sdb
   Welcome to fdisk (util-linux 2.34).
   Changes will remain in memory only, until you decide to write them.
   Be careful before using the write command.
   
   Device does not contain a recognized partition table.
   Created a new DOS disklabel with disk identifier 0x78db68b4.
   Command (m for help): n
   Partition type
      p   primary (0 primary, 0 extended, 4 free)
      e   extended (container for logical partitions)
   Select (default p): p
   Partition number (1-4, default 1): 
   First sector (2048-5242879, default 2048): 
   Last sector, +/-sectors or +/-size{K,M,G,T,P} (2048-5242879, default 5242879): +2G
   
   Created a new partition 1 of type 'Linux' and of size 2 GiB.
   
   Command (m for help): n
   Partition type
      p   primary (1 primary, 0 extended, 3 free)
      e   extended (container for logical partitions)
   Select (default p): p
   Partition number (2-4, default 2): 
   First sector (4196352-5242879, default 4196352): 
   Last sector, +/-sectors or +/-size{K,M,G,T,P} (4196352-5242879, default 5242879): +100%FREE
   Last sector, +/-sectors or +/-size{K,M,G,T,P} (4196352-5242879, default 5242879): 100%
   Last sector, +/-sectors or +/-size{K,M,G,T,P} (4196352-5242879, default 5242879): 
   
   Created a new partition 2 of type 'Linux' and of size 511 MiB.
   
   Command (m for help): w
   The partition table has been altered.
   Calling ioctl() to re-read partition table.
   Syncing disks.
   
   root@vagrant:~# lsblk
   NAME                 MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
   sda                    8:0    0   64G  0 disk 
   ├─sda1                 8:1    0  512M  0 part /boot/efi
   ├─sda2                 8:2    0    1K  0 part 
   └─sda5                 8:5    0 63.5G  0 part 
     ├─vgvagrant-root   253:0    0 62.6G  0 lvm  /
     └─vgvagrant-swap_1 253:1    0  980M  0 lvm  [SWAP]
   sdb                    8:16   0  2.5G  0 disk 
   ├─sdb1                 8:17   0    2G  0 part 
   └─sdb2                 8:18   0  511M  0 part 
   sdc                    8:32   0  2.5G  0 disk 
   ```


5. Используя `sfdisk`, перенесите данную таблицу разделов на второй диск.
   *
   ```bash
   root@vagrant:~# sfdisk -d /dev/sdb | sfdisk /dev/sdc
   Checking that no-one is using this disk right now ... OK
   
   Disk /dev/sdc: 2.51 GiB, 2684354560 bytes, 5242880 sectors
   Disk model: VBOX HARDDISK   
   Units: sectors of 1 * 512 = 512 bytes
   Sector size (logical/physical): 512 bytes / 512 bytes
   I/O size (minimum/optimal): 512 bytes / 512 bytes
   
   >>> Script header accepted.
   >>> Script header accepted.
   >>> Script header accepted.
   >>> Script header accepted.
   >>> Created a new DOS disklabel with disk identifier 0x78db68b4.
   /dev/sdc1: Created a new partition 1 of type 'Linux' and of size 2 GiB.
   /dev/sdc2: Created a new partition 2 of type 'Linux' and of size 511 MiB.
   /dev/sdc3: Done.
   
   New situation:
   Disklabel type: dos
   Disk identifier: 0x78db68b4
   
   Device     Boot   Start     End Sectors  Size Id Type
   /dev/sdc1          2048 4196351 4194304    2G 83 Linux
   /dev/sdc2       4196352 5242879 1046528  511M 83 Linux
   
   The partition table has been altered.
   Calling ioctl() to re-read partition table.
   Syncing disks.
   root@vagrant:~# lsblk
   NAME                 MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
   sda                    8:0    0   64G  0 disk 
   ├─sda1                 8:1    0  512M  0 part /boot/efi
   ├─sda2                 8:2    0    1K  0 part 
   └─sda5                 8:5    0 63.5G  0 part 
     ├─vgvagrant-root   253:0    0 62.6G  0 lvm  /
     └─vgvagrant-swap_1 253:1    0  980M  0 lvm  [SWAP]
   sdb                    8:16   0  2.5G  0 disk 
   ├─sdb1                 8:17   0    2G  0 part 
   └─sdb2                 8:18   0  511M  0 part 
   sdc                    8:32   0  2.5G  0 disk 
   ├─sdc1                 8:33   0    2G  0 part 
   └─sdc2                 8:34   0  511M  0 part    
   ```


6. Соберите `mdadm` RAID1 на паре разделов 2 Гб.
   *
   ```bash
   vagrant@vagrant:~$ sudo mdadm --create /dev/md1 --level=1 --raid-devices=2 /dev/sdb1 /dev/sdc1
   mdadm: Note: this array has metadata at the start and
       may not be suitable as a boot device.  If you plan to
       store '/boot' on this device please ensure that
       your boot-loader understands md/v1.x metadata, or use
       --metadata=0.90
   Continue creating array? y
   mdadm: Defaulting to version 1.2 metadata
   mdadm: array /dev/md1 started.
   vagrant@vagrant:~$ lsblk
   NAME                 MAJ:MIN RM  SIZE RO TYPE  MOUNTPOINT
   sda                    8:0    0   64G  0 disk  
   ├─sda1                 8:1    0  512M  0 part  /boot/efi
   ├─sda2                 8:2    0    1K  0 part  
   └─sda5                 8:5    0 63.5G  0 part  
     ├─vgvagrant-root   253:0    0 62.6G  0 lvm   /
     └─vgvagrant-swap_1 253:1    0  980M  0 lvm   [SWAP]
   sdb                    8:16   0  2.5G  0 disk  
   ├─sdb1                 8:17   0    2G  0 part  
   │ └─md1                9:1    0    2G  0 raid1 
   └─sdb2                 8:18   0  511M  0 part  
   sdc                    8:32   0  2.5G  0 disk  
   ├─sdc1                 8:33   0    2G  0 part  
   │ └─md1                9:1    0    2G  0 raid1 
   └─sdc2                 8:34   0  511M  0 part  
   vagrant@vagrant:~$ sudo mdadm /dev/md1
   /dev/md1: 2045.00MiB raid1 2 devices, 0 spares. Use mdadm --detail for more detail.   
   ```
   

7. Соберите `mdadm` RAID0 на второй паре маленьких разделов.
   *
   ```bash
   vagrant@vagrant:~$ sudo mdadm --create /dev/md2 --level=0 --raid-devices=2 /dev/sdb2 /dev/sdc2
   mdadm: Defaulting to version 1.2 metadata
   mdadm: array /dev/md2 started.
   vagrant@vagrant:~$ sudo mdadm /dev/md2
   /dev/md2: 1018.00MiB raid0 2 devices, 0 spares. Use mdadm --detail for more detail.   
   ```

8. Создайте 2 независимых PV на получившихся md-устройствах.
   *
   ```bash
   vagrant@vagrant:~$ sudo pvcreate /dev/md1
     Physical volume "/dev/md1" successfully created.
   vagrant@vagrant:~$ sudo pvcreate /dev/md2
     Physical volume "/dev/md2" successfully created.
   vagrant@vagrant:~$ pvls
   -bash: pvls: command not found
   vagrant@vagrant:~$ pvs
     WARNING: Running as a non-root user. Functionality may be unavailable.
     /run/lock/lvm/P_global:aux: open failed: Permission denied
   vagrant@vagrant:~$ sudo pvs
     PV         VG        Fmt  Attr PSize    PFree   
     /dev/md1             lvm2 ---    <2.00g   <2.00g
     /dev/md2             lvm2 ---  1018.00m 1018.00m
     /dev/sda5  vgvagrant lvm2 a--   <63.50g       0    
   ```


9. Создайте общую volume-group на этих двух PV.
   *
   ```bash
   vagrant@vagrant:~$ sudo vgcreate vg1 /dev/md1 /dev/md2
     Volume group "vg1" successfully created
   vagrant@vagrant:~$ sudo vgs
     VG        #PV #LV #SN Attr   VSize   VFree 
     vg1         2   0   0 wz--n-  <2.99g <2.99g
     vgvagrant   1   2   0 wz--n- <63.50g     0 
   ```

10. Создайте LV размером 100 Мб, указав его расположение на PV с RAID0.
   
   ```bash
   vagrant@vagrant:~$ sudo lvcreate -L100 -n lv1 vg1 /dev/md1
     Logical volume "lv1" created.
   vagrant@vagrant:~$ lsblk
   NAME                 MAJ:MIN RM  SIZE RO TYPE  MOUNTPOINT
   sda                    8:0    0   64G  0 disk  
   ├─sda1                 8:1    0  512M  0 part  /boot/efi
   ├─sda2                 8:2    0    1K  0 part  
   └─sda5                 8:5    0 63.5G  0 part  
     ├─vgvagrant-root   253:0    0 62.6G  0 lvm   /
     └─vgvagrant-swap_1 253:1    0  980M  0 lvm   [SWAP]
   sdb                    8:16   0  2.5G  0 disk  
   ├─sdb1                 8:17   0    2G  0 part  
   │ └─md1                9:1    0    2G  0 raid1 
   │   └─vg1-lv1        253:2    0  100M  0 lvm   
   └─sdb2                 8:18   0  511M  0 part  
     └─md2                9:2    0 1018M  0 raid0 
   sdc                    8:32   0  2.5G  0 disk  
   ├─sdc1                 8:33   0    2G  0 part  
   │ └─md1                9:1    0    2G  0 raid1 
   │   └─vg1-lv1        253:2    0  100M  0 lvm   
   └─sdc2                 8:34   0  511M  0 part  
     └─md2                9:2    0 1018M  0 raid0 
   ```

11. Создайте `mkfs.ext4` ФС на получившемся LV.
   ```bash
   vagrant@vagrant:~$ sudo mkfs -t ext4 /dev/mapper/vg1-lv1
   mke2fs 1.45.5 (07-Jan-2020)
   Creating filesystem with 25600 4k blocks and 25600 inodes
   
   Allocating group tables: done                            
   Writing inode tables: done                            
   Creating journal (1024 blocks): done
   Writing superblocks and filesystem accounting information: done
   ```    

12. Смонтируйте этот раздел в любую директорию, например, `/tmp/new`.
   ```bash
   vagrant@vagrant:~$ mkdir /tmp/new
   vagrant@vagrant:~$ sudo mount /dev/mapper/vg1-lv1 /tmp/new
   ```

13. Поместите туда тестовый файл, например `wget https://mirror.yandex.ru/ubuntu/ls-lR.gz -O /tmp/new/test.gz`.
   ```bash
   vagrant@vagrant:~$ cd /tmp/new   
   vagrant@vagrant:/tmp/new$ sudo wget https://mirror.yandex.ru/ubuntu/ls-lR.gz -O /tmp/new/test.gz
   --2021-09-10 08:14:02--  https://mirror.yandex.ru/ubuntu/ls-lR.gz
   Resolving mirror.yandex.ru (mirror.yandex.ru)... 213.180.204.183, 2a02:6b8::183
   Connecting to mirror.yandex.ru (mirror.yandex.ru)|213.180.204.183|:443... connected.
   HTTP request sent, awaiting response... 200 OK
   Length: 21006478 (20M) [application/octet-stream]
   Saving to: ‘/tmp/new/test.gz’
   
   /tmp/new/test.gz                                          100%[=====================================================================================================================================>]  20.03M  2.98MB/s    in 11s     
   
   2021-09-10 08:14:13 (1.90 MB/s) - ‘/tmp/new/test.gz’ saved [21006478/21006478]
```

14. Прикрепите вывод `lsblk`.
   ```bash
   vagrant@vagrant:/tmp/new$ lsblk
   NAME                 MAJ:MIN RM  SIZE RO TYPE  MOUNTPOINT
   sda                    8:0    0   64G  0 disk  
   ├─sda1                 8:1    0  512M  0 part  /boot/efi
   ├─sda2                 8:2    0    1K  0 part  
   └─sda5                 8:5    0 63.5G  0 part  
     ├─vgvagrant-root   253:0    0 62.6G  0 lvm   /
     └─vgvagrant-swap_1 253:1    0  980M  0 lvm   [SWAP]
   sdb                    8:16   0  2.5G  0 disk  
   ├─sdb1                 8:17   0    2G  0 part  
   │ └─md1                9:1    0    2G  0 raid1 
   │   └─vg1-lv1        253:2    0  100M  0 lvm   /tmp/new
   └─sdb2                 8:18   0  511M  0 part  
     └─md2                9:2    0 1018M  0 raid0 
   sdc                    8:32   0  2.5G  0 disk  
   ├─sdc1                 8:33   0    2G  0 part  
   │ └─md1                9:1    0    2G  0 raid1 
   │   └─vg1-lv1        253:2    0  100M  0 lvm   /tmp/new
   └─sdc2                 8:34   0  511M  0 part  
     └─md2                9:2    0 1018M  0 raid0 
   ```

15. Протестируйте целостность файла:
   ```bash
   vagrant@vagrant:/tmp/new$ gzip -t test.gz
   vagrant@vagrant:/tmp/new$ echo $?
   0
   ```

16. Используя pvmove, переместите содержимое PV с RAID0 на RAID1.
   ```bash
   vagrant@vagrant:/tmp/new$ sudo pvmove /dev/md1 /dev/md2
     /dev/md1: Moved: 20.00%
     /dev/md1: Moved: 100.00%
   vagrant@vagrant:/tmp/new$ lsblk
   NAME                 MAJ:MIN RM  SIZE RO TYPE  MOUNTPOINT
   sda                    8:0    0   64G  0 disk  
   ├─sda1                 8:1    0  512M  0 part  /boot/efi
   ├─sda2                 8:2    0    1K  0 part  
   └─sda5                 8:5    0 63.5G  0 part  
     ├─vgvagrant-root   253:0    0 62.6G  0 lvm   /
     └─vgvagrant-swap_1 253:1    0  980M  0 lvm   [SWAP]
   sdb                    8:16   0  2.5G  0 disk  
   ├─sdb1                 8:17   0    2G  0 part  
   │ └─md1                9:1    0    2G  0 raid1 
   └─sdb2                 8:18   0  511M  0 part  
     └─md2                9:2    0 1018M  0 raid0 
       └─vg1-lv1        253:2    0  100M  0 lvm   /tmp/new
   sdc                    8:32   0  2.5G  0 disk  
   ├─sdc1                 8:33   0    2G  0 part  
   │ └─md1                9:1    0    2G  0 raid1 
   └─sdc2                 8:34   0  511M  0 part  
     └─md2                9:2    0 1018M  0 raid0 
       └─vg1-lv1        253:2    0  100M  0 lvm   /tmp/new
   ```

17. Сделайте `--fail` на устройство в вашем RAID1 md.
   ```bash
   vagrant@vagrant:/tmp/new$ sudo mdadm /dev/md1 --fail /dev/sdb1
   mdadm: set /dev/sdb1 faulty in /dev/md1
   ```

18. Подтвердите выводом `dmesg`, что RAID1 работает в деградированном состоянии.
   ```bash
   [ 9959.995902] md/raid1:md1: Disk failure on sdb1, disabling device.
                  md/raid1:md1: Operation continuing on 1 devices.
   ```

19. Протестируйте целостность файла, несмотря на "сбойный" диск он должен продолжать быть доступен:
   ```bash
   vagrant@vagrant:/tmp/new$ gzip -t test.gz
   vagrant@vagrant:/tmp/new$ echo $?
   0
   ```

20. Погасите тестовый хост, `vagrant destroy`.
    *
    Погасил
---