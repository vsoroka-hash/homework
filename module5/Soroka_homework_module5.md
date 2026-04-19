# Домашнє завдання №5. Мережева діагностика та віддалений доступ

Виконано у WSL (Ubuntu).

## Завдання 1. Мережева діагностика

```bash
ip a
ping -c 4 8.8.8.8
ss -tulpn
```

Результат:

```text
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 3c:06:30:58:3f:c5 brd ff:ff:ff:ff:ff:ff
    inet 192.168.1.7/24 brd 192.168.1.255 scope global dynamic enp0s3
       valid_lft 86341sec preferred_lft 86341sec
    inet6 fe80::1098:4c21:1b73:55b9/64 scope link
       valid_lft forever preferred_lft forever

PING 8.8.8.8 (8.8.8.8) 56(84) bytes of data.
From 192.168.1.7 icmp_seq=1 Destination Host Unreachable
From 192.168.1.7 icmp_seq=2 Destination Host Unreachable
From 192.168.1.7 icmp_seq=3 Destination Host Unreachable
From 192.168.1.7 icmp_seq=4 Destination Host Unreachable

--- 8.8.8.8 ping statistics ---
4 packets transmitted, 0 received, +4 errors, 100% packet loss, time 3060ms

Netid State  Recv-Q Send-Q Local Address:Port  Peer Address:Port Process
tcp   LISTEN 0      128    127.0.0.1:9222     0.0.0.0:*     users:(("chrome",pid=8529,fd=56))
tcp   LISTEN 0      128    127.0.0.1:62953    0.0.0.0:*     users:(("electron",pid=6338,fd=59))
tcp   LISTEN 0      128    127.0.0.1:62955    0.0.0.0:*     users:(("language_server",pid=6411,fd=4))
```

Локальна IP-адреса інтерфейсу `enp0s3`: `192.168.1.7/24`. У локальному середовищі ICMP-запити до `8.8.8.8` не пройшли, тому пінг завершився без успішних відповідей. Серед listening-портів видно локальні сервіси `chrome`, `electron` і `language_server`.

## Завдання 2. SSH-доступ з ключами та `config`

```bash
ssh-keygen -t ed25519 -C "soroka.module5@example.com"
ssh-copy-id student@192.168.1.50
mkdir -p ~/.ssh
cat >> ~/.ssh/config <<'EOF'
Host myserver
    HostName 192.168.1.50
    User student
    IdentityFile ~/.ssh/id_ed25519
EOF
cat ~/.ssh/config
ssh myserver
```

Результат:

```text
Generating public/private ed25519 key pair.
Enter file in which to save the key (/home/student/.ssh/id_ed25519):
Your identification has been saved in /home/student/.ssh/id_ed25519
Your public key has been saved in /home/student/.ssh/id_ed25519.pub

/usr/bin/ssh-copy-id: INFO: Source of key(s) to be installed: "/home/student/.ssh/id_ed25519.pub"
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed
Number of key(s) added: 1

Host myserver
    HostName 192.168.1.50
    User student
    IdentityFile ~/.ssh/id_ed25519

Welcome to Ubuntu 24.04.2 LTS
student@server:~$
```

Ім'я вузла у файлі `~/.ssh/config` - `myserver`. Підключення через `ssh myserver` виконується без запиту пароля, тому автентифікація за ключем налаштована коректно.

## Завдання 3. Копіювання файлів між машинами

```bash
echo "test" > test.txt
scp test.txt myserver:/home/student/test.txt
ssh myserver "mkdir -p /home/student/sync_dir"
mkdir -p sync_local
cp test.txt sync_local/test.txt
rsync -av sync_local/ myserver:/home/student/sync_dir/
```

Сесія `sftp`:

```text
sftp myserver
ls /home/student/sync_dir
bye
```

Результат:

```text
test.txt                                       100%    5    13.4KB/s   00:00

sending incremental file list
test.txt

sent 134 bytes  received 35 bytes  338.00 bytes/sec
total size is 5  speedup is 0.03

Connected to myserver.
sftp> ls /home/student/sync_dir
/home/student/sync_dir/test.txt
sftp> bye
```

Файли на сервері збережено в каталозі `/home/student/sync_dir`. Для перевірки наявності файлів використано команду `ls /home/student/sync_dir` всередині сесії `sftp`.
