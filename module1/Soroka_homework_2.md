# Домашнє завдання №2. Файлова система і права доступу

Виконано у WSL (Ubuntu).

## Завдання 1. Ієрархія каталогів Linux

```bash
cd /
ls -la | head -n 20

cd /etc
ls -la | head -n 20

cd /home
ls -la
```

Фрагмент результату (`/home`):

```text
total 12
drwxr-xr-x  3 root    root    4096 Mar 21 21:03 .
drwxr-xr-x 22 root    root    4096 Mar 21 21:04 ..
drwxr-x---  3 Volodymyr Volodymyr 4096 Mar 21 21:04 Volodymyr
```

## Завдання 2. Файли, каталоги та посилання

```bash
# виконано під користувачем Volodymyr
mkdir -p ~/lab2
cd ~/lab2
touch file.txt
cp file.txt file_copy.txt
mv file_copy.txt file_renamed.txt
ln file.txt file_hardlink.txt
ln -s file.txt file_symlink.txt
find ~ -name "file.txt"
ls -li ~/lab2
```

Результат:

```text
/home/Volodymyr/lab2/file.txt

total 0
46487 -rw-rw-r-- 2 Volodymyr Volodymyr 0 Mar 21 21:04 file.txt
46487 -rw-rw-r-- 2 Volodymyr Volodymyr 0 Mar 21 21:04 file_hardlink.txt
46488 -rw-rw-r-- 1 Volodymyr Volodymyr 0 Mar 21 21:04 file_renamed.txt
46493 lrwxrwxrwx 1 Volodymyr Volodymyr 8 Mar 21 21:04 file_symlink.txt -> file.txt
```

## Завдання 3. Права доступу

```bash
ls -l ~/lab2/file.txt
chmod 444 ~/lab2/file.txt
ls -l ~/lab2/file.txt
chmod u+w ~/lab2/file.txt
ls -l ~/lab2/file.txt
umask
umask 022
umask
```

Результат:

```text
-rw-rw-r-- 2 Volodymyr Volodymyr 0 Mar 21 21:04 /home/Volodymyr/lab2/file.txt
-r--r--r-- 2 Volodymyr Volodymyr 0 Mar 21 21:04 /home/Volodymyr/lab2/file.txt
-rw-r--r-- 2 Volodymyr Volodymyr 0 Mar 21 21:04 /home/Volodymyr/lab2/file.txt
0002
0022
```

## Завдання 4. Користувачі

```bash
sudo useradd -m -s /bin/bash Volodymyr
sudo usermod -aG sudo Volodymyr
id Volodymyr
getent passwd Volodymyr
```

Результат:

```text
uid=1000(Volodymyr) gid=1000(Volodymyr) groups=1000(Volodymyr),27(sudo)
Volodymyr:x:1000:1000::/home/Volodymyr:/bin/bash
```
