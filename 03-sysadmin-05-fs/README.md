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

7. Соберите `mdadm` RAID0 на второй паре маленьких разделов.

8. Создайте 2 независимых PV на получившихся md-устройствах.

9. Создайте общую volume-group на этих двух PV.

10. Создайте LV размером 100 Мб, указав его расположение на PV с RAID0.

11. Создайте `mkfs.ext4` ФС на получившемся LV.

12. Смонтируйте этот раздел в любую директорию, например, `/tmp/new`.

13. Поместите туда тестовый файл, например `wget https://mirror.yandex.ru/ubuntu/ls-lR.gz -O /tmp/new/test.gz`.

14. Прикрепите вывод `lsblk`.

15 Протестируйте целостность файла:

    ```bash
    root@vagrant:~# gzip -t /tmp/new/test.gz
    root@vagrant:~# echo $?
    0
    ```

16. Используя pvmove, переместите содержимое PV с RAID0 на RAID1.

17. Сделайте `--fail` на устройство в вашем RAID1 md.

18. Подтвердите выводом `dmesg`, что RAID1 работает в деградированном состоянии.

19. Протестируйте целостность файла, несмотря на "сбойный" диск он должен продолжать быть доступен:

    ```bash
    root@vagrant:~# gzip -t /tmp/new/test.gz
    root@vagrant:~# echo $?
    0
    ```

20. Погасите тестовый хост, `vagrant destroy`.

---