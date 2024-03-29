# Домашнее задание к занятию "3.2. Работа в терминале, лекция 2"

1. Какого типа команда `cd`? Попробуйте объяснить, почему она именно такого типа; опишите ход своих мыслей, если считаете что она могла бы быть другого типа.
   * Команда `cd` встроена в оболочку shell
2. Какая альтернатива без pipe команде `grep <some_string> <some_file> | wc -l`? `man grep` поможет в ответе на этот вопрос. Ознакомьтесь с [документом](http://www.smallo.ruhr.de/award.html) о других подобных некорректных вариантах использования pipe.
   * grep -c <some_string> <some_file>
3. Какой процесс с PID `1` является родителем для всех процессов в вашей виртуальной машине Ubuntu 20.04?
   * systemd
4. Как будет выглядеть команда, которая перенаправит вывод stderr `ls` на другую сессию терминала?
   * `ls 2>/dev/pts/x`, где `x` - номер сессии куда необходимо перенаправить вывод
5. Получится ли одновременно передать команде файл на stdin и вывести ее stdout в другой файл? Приведите работающий пример.
   * да, получится. пример `cat <test 1>test_2`
6. Получится ли находясь в графическом режиме, вывести данные из PTY в какой-либо из эмуляторов TTY? Сможете ли вы наблюдать выводимые данные?
   * Да, получится. Да, данные наблюдать сможем нужно знать номер существующей сессии tty и вывести сообщение на неё `echo "test" > /dev/ttyx`, где `x` - номер сессии tty
7. Выполните команду `bash 5>&1`. К чему она приведет? Что будет, если вы выполните `echo netology > /proc/$$/fd/5`? Почему так происходит?
   * `bash 5>&1` создаётся новый файловый дескриптор 5 и перенаправит его на stdout.
    `echo netology > /proc/$$/fd/5`  выведет `netology` так как перенаправляем в дескриптор 5, который перенаправляется в stdout, что было сделано предыдущим шагом. 
     
8. Получится ли в качестве входного потока для pipe использовать только stderr команды, не потеряв при этом отображение stdout на pty? Напоминаем: по умолчанию через pipe передается только stdout команды слева от `|` на stdin команды справа.
Это можно сделать, поменяв стандартные потоки местами через промежуточный новый дескриптор, который вы научились создавать в предыдущем вопросе.
   * Если правильно понял, то будет примерно так: `ls /no-such-file /etc/passwd 3>&2 2>&1 1>&3 | wc -l`
   * `3>&2` промежуточный дескриптор перенаправляем в stderr. `2>&1` stderr перенаправляем в stdout и `1>&3` stdout перенаправляем в новый дескриптор.

9. Что выведет команда `cat /proc/$$/environ`? Как еще можно получить аналогичный по содержанию вывод?
   * Команда `cat /proc/$$/environ` выведет переменные для текущего процесса. Альтернатива команды `printenv` и `env`

10. Используя `man`, опишите что доступно по адресам `/proc/<PID>/cmdline`, `/proc/<PID>/exe`.
    * `/proc/<PID>/cmdline` - Этот файл содержит полную командную строку запуска процесса, кроме тех процессов, что полностью ушли в своппинг, а также тех, что превратились в зомби. В этих двух случаях в файле ничего нет, то есть чтение этого файла вернет 0 символов. Аргументы командной строки в этом файле указаны как список строк, каждая из которых завешается нулевым символом, с добавочным нулевым байтом после последней строки.
      `/proc/<PID>/exe` - является символьной ссылкой, содержащей фактическое полное имя выполняемого файла.

11. Узнайте, какую наиболее старшую версию набора инструкций SSE поддерживает ваш процессор с помощью `/proc/cpuinfo`.
    * `cat /proc/cpuinfo | grep sse` sse4_2

12. При открытии нового окна терминала и `vagrant ssh` создается новая сессия и выделяется pty. Это можно подтвердить командой `tty`, которая упоминалась в лекции 3.2. Однако:

    ```bash
	vagrant@netology1:~$ ssh localhost 'tty'
	not a tty
    ```

	Почитайте, почему так происходит, и как изменить поведение.
    * По умолчанию, при запуске команд на удаленном хосте по ssh, tty не выделяется для удаленного сеанса.  
    * если для неинтерактивной SSH-сессии всё-таки нужно выделить терминал, используется ключ `-t`, `ssh -t localhost 'tty'`

13. Бывает, что есть необходимость переместить запущенный процесс из одной сессии в другую. Попробуйте сделать это, воспользовавшись `reptyr`. Например, так можно перенести в `screen` процесс, который вы запустили по ошибке в обычной SSH-сессии.
    * команда `reptyr <PID>` у меня не заработала, не понял почему, читал мануал, смотрел видосы что с sudo, что с ключем -T, ругается на Permission denied. Пробовал на разных vm
    * p.s команда заработала только от пользователя root.

14. `sudo echo string > /root/new_file` не даст выполнить перенаправление под обычным пользователем, так как перенаправлением занимается процесс shell'а, который запущен без `sudo` под вашим пользователем. Для решения данной проблемы можно использовать конструкцию `echo string | sudo tee /root/new_file`. Узнайте что делает команда `tee` и почему в отличие от `sudo echo` команда с `sudo tee` будет работать.
    
    * `tee` – чтение с stdin и вывод в stdout и в file. 
    * команда `echo string | sudo tee /root/new_file` работает так как запись в файл выполняется от пользователя root.