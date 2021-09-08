# Домашнее задание к занятию "3.4. Операционные системы, лекция 2"

1. На лекции мы познакомились с [node_exporter](https://github.com/prometheus/node_exporter/releases). В демонстрации его исполняемый файл запускался в background. Этого достаточно для демо, но не для настоящей production-системы, где процессы должны находиться под внешним управлением. Используя знания из лекции по systemd, создайте самостоятельно простой [unit-файл](https://www.freedesktop.org/software/systemd/man/systemd.service.html) для node_exporter:

    * поместите его в автозагрузку,
    * предусмотрите возможность добавления опций к запускаемому процессу через внешний файл (посмотрите, например, на `systemctl cat cron`),
    * удостоверьтесь, что с помощью systemctl процесс корректно стартует, завершается, а после перезагрузки автоматически поднимается.

-
   + Нужно скачать и распаковать `node_exporter-1.2.2.linux-amd64.tar.gz`, далее скопировать исполняемый файл `node_exporter` в директорию `/usr/bin/`
   
   + Создать юнит для node_exporter. Для этого в директории `/etc/systemd/system/` создать файл `node_exporter.service` со следующим содержимым:
   ```bash
         [Unit]
         Description=node_exporter
         Documentation=https://github.com/prometheus/node_exporter
         After=network-online.target
         
         [Service]
         EnvironmentFile=/etc/node-exporter.conf
         ExecStart=/usr/bin/node-exporter $EXTRA_OPTS
         KillMode=process
         Restart=on-failure
         
         [Install]
         WantedBy=multi-user.target
   ```
   + Запускаем `systemctl start node_exporter` и добавляем юнит в список автозапуска `systemctl enable node_exporter`
   + Непонятно для чего использовать `systemctl cat cron` по сути мы просто посмотрели содержимое юнита cron

2. Ознакомьтесь с опциями node_exporter и выводом `/metrics` по-умолчанию. Приведите несколько опций, которые вы бы выбрали для базового мониторинга хоста по CPU, памяти, диску и сети.
   * Выбрал следующие опции: 
   `--collector.cpu`, `--collector.meminfo`, `--collector.diskstats`, `--collector.netstat`. Их нужно положить в файл `/etc/node-exporter.conf`


3. Установите в свою виртуальную машину [Netdata](https://github.com/netdata/netdata). Воспользуйтесь [готовыми пакетами](https://packagecloud.io/netdata/netdata/install) для установки (`sudo apt install -y netdata`). После успешной установки:
    * в конфигурационном файле `/etc/netdata/netdata.conf` в секции [web] замените значение с localhost на `bind to = 0.0.0.0`,
    * добавьте в Vagrantfile проброс порта Netdata на свой локальный компьютер и сделайте `vagrant reload`:

    ```bash
    config.vm.network "forwarded_port", guest: 19999, host: 19999
    ```

    После успешной перезагрузки в браузере *на своем ПК* (не в виртуальной машине) вы должны суметь зайти на `localhost:19999`. Ознакомьтесь с метриками, которые по умолчанию собираются Netdata и с комментариями, которые даны к этим метрикам.
    * Ознакомился установил
    
    ![image info](./img/net-data.png)

    
4. Можно ли по выводу `dmesg` понять, осознает ли ОС, что загружена не на настоящем оборудовании, а на системе виртуализации?
   * Да, можно
    ```bash
        root@vagrant:~# dmesg | grep -i virt
        [    0.000000] DMI: innotek GmbH VirtualBox/VirtualBox, BIOS VirtualBox 12/01/2006
        [    0.002392] CPU MTRRs all blank - virtualized system.
        [    0.066350] Booting paravirtualized kernel on KVM
        [    2.349698] systemd[1]: Detected virtualization oracle
   ```

5. Как настроен sysctl `fs.nr_open` на системе по-умолчанию? Узнайте, что означает этот параметр. Какой другой существующий лимит не позволит достичь такого числа (`ulimit --help`)?
   * `fs.nr_open` - Максимальное количество файловых дескрипторов, которые может выделить процесс
    ```bash
        root@vagrant:~# sysctl fs.nr_open  
        fs.nr_open = 1048576
    ```
    И
    ```bash
        root@vagrant:~# ulimit -Hn
        1048576
    ```
    

6. Запустите любой долгоживущий процесс (не `ls`, который отработает мгновенно, а, например, `sleep 1h`) в отдельном неймспейсе процессов; покажите, что ваш процесс работает под PID 1 через `nsenter`. Для простоты работайте в данном задании под root (`sudo -i`). Под обычным пользователем требуются дополнительные опции (`--map-root-user`) и т.д.
   * Подключаемся новой сессией к vm, вводим команду `unshare -f --pid --mount-proc ping 8.8.4.4`
   * на старой сессии вводим команду `ps -ax | grep ping` для обнаружения `PID`:
    ```bash 
        root@vagrant:~# ps -ax | grep ping
        1336 pts/0    S+     0:00 unshare -f --pid --mount-proc ping 8.8.4.4
        1337 pts/0    S+     0:00 ping 8.8.4.4
        1367 pts/1    S+     0:00 grep --color=auto ping
   ```
   * далее 
    ```bash 
        root@vagrant:~# nsenter --target 1337 --pid --mount
        root@vagrant:/# ps ax
            PID TTY      STAT   TIME COMMAND
            1 pts/0    S+     0:00 ping 8.8.4.4
            2 pts/1    S      0:00 -bash
            11 pts/1    R+     0:00 ps ax
   ```
   как видно из вывода у процесса ping PID = 1


7. Найдите информацию о том, что такое `:(){ :|:& };:`. Запустите эту команду в своей виртуальной машине Vagrant с Ubuntu 20.04 (**это важно, поведение в других ОС не проверялось**). Некоторое время все будет "плохо", после чего (минуты) – ОС должна стабилизироваться. Вызов `dmesg` расскажет, какой механизм помог автоматической стабилизации. Как настроен этот механизм по-умолчанию, и как изменить число процессов, которое можно создать в сессии?
   * Данная команда `:(){ :|:& };:` это 'fork bomb' в рекурсии вызывает сама себя и создаёт новы процессы.
   * Скорее всего помогло это: 
     ```bash
      [ 2451.953531] cgroup: fork rejected by pids controller in /user.slice/user-0.slice/session-3.scope
     ```
   * Как я понял когда количество процессов становится равно `ulimit -u` отрабатывает `cgroup` и не даёт создать новые процессы, можно увеличить командой `ulimit -u <кол-во процессов>`

---
