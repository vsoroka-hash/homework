# Домашнє завдання №3. Процеси, пріоритети та моніторинг ресурсів

Виконано у WSL (Ubuntu).

## Завдання 1. Огляд активних процесів

```bash
ps aux | head -n 10
top -b -n 1 -o %MEM | head -n 15
echo $$
```

Фрагмент результату:

```text
USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root         1  0.0  0.1 169684 11520 ?        Ss   17:10   0:01 /sbin/init
root       612  0.0  0.0   2800  1792 ?        Ss   17:10   0:00 /init
root       613  0.0  0.0   2800   188 ?        S    17:10   0:00 plan9 --control-socket 5 --log-level 4
Volodymyr  655  0.0  0.0  21540  5488 pts/0    Ss   17:10   0:00 /bin/bash
Volodymyr 1024  1.2  3.8 2654320 312640 pts/0  Sl+  17:22   0:04 code

top - 17:24:51 up 14 min,  1 user,  load average: 0.18, 0.11, 0.08
Tasks:  92 total,   1 running,  91 sleeping,   0 stopped,   0 zombie
MiB Mem :   7820.0 total,   2140.4 free,   2612.8 used,   3066.8 buff/cache

  PID USER      PR  NI    VIRT    RES    SHR S  %CPU %MEM     TIME+ COMMAND
 1024 Volodymyr 20   0 2654320 312640 126408 S   1.3  3.9   0:04.11 code
 1288 Volodymyr 20   0 1422568 184920  92384 S   0.7  2.3   0:02.17 chrome

655
```

Процес з найбільшим споживанням RAM: `code` (PID `1024`).

PID поточної оболонки: `655`.

## Завдання 2. Робота у фоні та керування процесами

```bash
sleep 1000 &
jobs
fg %1
# Ctrl+Z
kill -9 %1
nohup sleep 300 > nohup.out 2>&1 &
jobs
```

Результат:

```text
[1] 2148
[1]+  Running                 sleep 1000 &

sleep 1000
^Z
[1]+  Stopped                 sleep 1000
[1]+  Killed                  sleep 1000

[2] 2173
[2]+  Running                 nohup sleep 300 > nohup.out 2>&1 &
```

## Завдання 3. Пріоритети та обмеження

```bash
nice -n 10 sleep 400 &
ps -o pid,ni,comm -p $!

sleep 500 &
ps -o pid,ni,comm -p $!
renice 15 -p $!
ps -o pid,ni,comm -p $!

ulimit -a
```

Результат:

```text
[1] 2418
    PID  NI COMMAND
   2418  10 sleep

[2] 2450
    PID  NI COMMAND
   2450   0 sleep
2450 (process ID) old priority 0, new priority 15
    PID  NI COMMAND
   2450  15 sleep

real-time non-blocking time  (microseconds, -R) unlimited
core file size              (blocks, -c) 0
data seg size               (kbytes, -d) unlimited
scheduling priority                 (-e) 0
file size                   (blocks, -f) unlimited
pending signals                     (-i) 255382
max locked memory           (kbytes, -l) 16384
max memory size             (kbytes, -m) unlimited
open files                          (-n) 1024
pipe size                (512 bytes, -p) 8
stack size                  (kbytes, -s) 8192
cpu time                   (seconds, -t) unlimited
max user processes                  (-u) 30944
virtual memory              (kbytes, -v) unlimited
file locks                          (-x) unlimited
```

## Завдання 4. Моніторинг ресурсів

```bash
df -h
free -h
```

Результат:

```text
Filesystem      Size  Used Avail Use% Mounted on
/dev/sdc       1007G   19G  938G   2% /
tmpfs           782M  1.3M  781M   1% /run

               total        used        free      shared  buff/cache   available
Mem:           7.6Gi       2.6Gi       2.1Gi       120Mi       3.0Gi       4.7Gi
Swap:          2.0Gi          0B       2.0Gi
```
