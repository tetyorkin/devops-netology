# Домашнее задание к занятию "3.3. Операционные системы, лекция 1"

1. Какой системный вызов делает команда `cd`? В прошлом ДЗ мы выяснили, что `cd` не является самостоятельной  программой, это `shell builtin`, поэтому запустить `strace` непосредственно на `cd` не получится. Тем не менее, вы можете запустить `strace` на `/bin/bash -c 'cd /tmp'`. В этом случае вы увидите полный список системных вызовов, которые делает сам `bash` при старте. Вам нужно найти тот единственный, который относится именно к `cd`. Обратите внимание, что `strace` выдаёт результат своей работы в поток stderr, а не в stdout.
   * системный вызов `chdir("/tmp")`, можно узнать с помощью команды `strace /bin/bash -c 'cd /tmp'`

2. Попробуйте использовать команду `file` на объекты разных типов на файловой системе. Например:
    ```bash
    vagrant@netology1:~$ file /dev/tty
    /dev/tty: character special (5/0)
    vagrant@netology1:~$ file /dev/sda
    /dev/sda: block special (8/0)
    vagrant@netology1:~$ file /bin/bash
    /bin/bash: ELF 64-bit LSB shared object, x86-64
    ```
    Используя `strace` выясните, где находится база данных `file` на основании которой она делает свои догадки.
   * 
   ```bash
   root@vagrant:~# strace -o output file /dev/tty && cat output | grep openat
   /dev/tty: character special (5/0)
   openat(AT_FDCWD, "/etc/ld.so.cache", O_RDONLY|O_CLOEXEC) = 3
   openat(AT_FDCWD, "/lib/x86_64-linux-gnu/libmagic.so.1", O_RDONLY|O_CLOEXEC) = 3
   openat(AT_FDCWD, "/lib/x86_64-linux-gnu/libc.so.6", O_RDONLY|O_CLOEXEC) = 3
   openat(AT_FDCWD, "/lib/x86_64-linux-gnu/liblzma.so.5", O_RDONLY|O_CLOEXEC) = 3
   openat(AT_FDCWD, "/lib/x86_64-linux-gnu/libbz2.so.1.0", O_RDONLY|O_CLOEXEC) = 3
   openat(AT_FDCWD, "/lib/x86_64-linux-gnu/libz.so.1", O_RDONLY|O_CLOEXEC) = 3
   openat(AT_FDCWD, "/lib/x86_64-linux-gnu/libpthread.so.0", O_RDONLY|O_CLOEXEC) = 3
   openat(AT_FDCWD, "/usr/lib/locale/locale-archive", O_RDONLY|O_CLOEXEC) = 3
   openat(AT_FDCWD, "/etc/magic.mgc", O_RDONLY) = -1 ENOENT (No such file or directory)
   openat(AT_FDCWD, "/etc/magic", O_RDONLY) = 3
   openat(AT_FDCWD, "/usr/share/misc/magic.mgc", O_RDONLY) = 3
   openat(AT_FDCWD, "/usr/lib/x86_64-linux-gnu/gconv/gconv-modules.cache", O_RDONLY) = 3   
   ```   
   * Судя по выводу команда `file` обращается к базам по путям: `/etc/magic.mgc`, `/etc/magic` и `/usr/share/misc/magic.mgc`
   

3. Предположим, приложение пишет лог в текстовый файл. Этот файл оказался удален (deleted в lsof), однако возможности сигналом сказать приложению переоткрыть файлы или просто перезапустить приложение – нет. Так как приложение продолжает писать в удаленный файл, место на диске постепенно заканчивается. Основываясь на знаниях о перенаправлении потоков предложите способ обнуления открытого удаленного файла (чтобы освободить место на файловой системе).
   * Необходимо найти `PID` этого приложения, далее номер дискриптора. Зная номер дискриптора и PID нужно направить пустую строку: `> /proc/<PID>/fd/<дискриптор>`


4. Занимают ли зомби-процессы какие-то ресурсы в ОС (CPU, RAM, IO)?
   * Зомби процессы не занимают никаких ресурсов, кроме как запись в таблице процессов
   

5. В iovisor BCC есть утилита `opensnoop`:
    ```bash
    root@vagrant:~# dpkg -L bpfcc-tools | grep sbin/opensnoop
    /usr/sbin/opensnoop-bpfcc
    ```
    
   На какие файлы вы увидели вызовы группы `open` за первую секунду работы утилиты? Воспользуйтесь пакетом `bpfcc-tools` для Ubuntu 20.04. Дополнительные [сведения по установке](https://github.com/iovisor/bcc/blob/master/INSTALL.md).
   * В первую секунду работы `opensnoop-bpfcc` был следующий вывод:
   ```bash
   root@vagrant:~# opensnoop-bpfcc
   PID    COMM               FD ERR PATH
   757    vminfo              4   0 /var/run/utmp
   584    dbus-daemon        -1   2 /usr/local/share/dbus-1/system-services
   584    dbus-daemon        18   0 /usr/share/dbus-1/system-services
   584    dbus-daemon        -1   2 /lib/dbus-1/system-services
   584    dbus-daemon        18   0 /var/lib/snapd/dbus-1/system-services/
   ```
   * что-то я не вкурил, что за утилита и для чего она нужна

6. Какой системный вызов использует `uname -a`? Приведите цитату из man по этому системному вызову, где описывается альтернативное местоположение в `/proc`, где можно узнать версию ядра и релиз ОС.
   * `uname -a` использует системный вызов `uname()`
   * в `man uname` я не нашёл информацию, но есть в `man proc`, цитата следующая:
   ```bash
    /proc/version
              This string identifies the kernel version that is currently running.  It includes the contents of /proc/sys/kernel/ostype, /proc/sys/kernel/osrelease and /proc/sys/kernel/version.  For example:
   ```

7. Чем отличается последовательность команд через `;` и через `&&` в bash? Например:
    ```bash
    root@netology1:~# test -d /tmp/some_dir; echo Hi
    Hi
    root@netology1:~# test -d /tmp/some_dir && echo Hi
    root@netology1:~#
    ```
    Есть ли смысл использовать в bash `&&`, если применить `set -e`?
   
    * `;` - логическое ИЛИ, выполняется или левая команда, или правая, или обе
    * `&&` - логическое И, команда с права выполняется только если выполнилась левая и вернёт код завершения 0
    * `set -e` - прекращает выполнение, если вернулся не нулевой результат, использовать с `&&` смысла нет
   

8. Из каких опций состоит режим bash `set -euxo pipefail` и почему его хорошо было бы использовать в сценариях?
   * `-e` - выход, если команда вернула ненулевой код
   * `-u` - проверяет инициализацию использующихся переменных
   * `-x` - выводит на экран этапы выполнения команд.
   * `-o pipefail` - будет возвращать значение команды, которая завершилась с ошибкой, либо 0, если все хорошо
9. Используя `-o stat` для `ps`, определите, какой наиболее часто встречающийся статус у процессов в системе. В `man ps` ознакомьтесь (`/PROCESS STATE CODES`) что значат дополнительные к основной заглавной буквы статуса процессов. Его можно не учитывать при расчете (считать S, Ss или Ssl равнозначными).
   * Больше всего процессов со статусом `I`, `I - Idle kernel thread` 
   ```bash
   root@vagrant:~# ps -e -o stat | sort | uniq -c
      8 I
     40 I<
      1 R+
     23 S
      2 S+
      1 Sl
      1 SLsl
      2 SN
      1 S<s
     16 Ss
      3 Ss+
      4 Ssl
      1 STAT
   ```
 
 ---
