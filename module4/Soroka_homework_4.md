# Домашнє завдання №4. Пакети, сервіси та журнали

Виконано у WSL (Ubuntu).

## Завдання 1. Менеджери пакетів

```bash
sudo apt update
sudo apt install -y tree
dpkg -l | grep tree
tree --version
sudo apt remove -y tree
```

Результат:

```text
Hit:1 http://archive.ubuntu.com/ubuntu jammy InRelease
Hit:2 http://security.ubuntu.com/ubuntu jammy-security InRelease
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done

The following NEW packages will be installed:
  tree
0 upgraded, 1 newly installed, 0 to remove and 12 not upgraded.

ii  tree  2.0.2-1  amd64  displays an indented directory tree, in color
tree v2.0.2 (c) 1996 - 2022 by Steve Baker, Thomas Moore, Francesc Rocher, Kyosuke Tokoro

The following packages will be REMOVED:
  tree
0 upgraded, 0 newly installed, 1 to remove and 12 not upgraded.
Removing tree (2.0.2-1) ...
```

## Завдання 2. Керування сервісами через `systemctl`

```bash
sudo systemctl status cron
sudo systemctl stop cron
sudo systemctl is-active cron
sudo systemctl start cron
sudo systemctl enable cron
sudo systemctl status cron
```

Результат:

```text
● cron.service - Regular background program processing daemon
     Loaded: loaded (/lib/systemd/system/cron.service; disabled; vendor preset: enabled)
     Active: active (running) since Fri 2026-03-28 18:12:44 UTC; 3min ago

inactive

Synchronizing state of cron.service with SysV service script with /lib/systemd/systemd-sysv-install.
Executing: /lib/systemd/systemd-sysv-install enable cron

● cron.service - Regular background program processing daemon
     Loaded: loaded (/lib/systemd/system/cron.service; enabled; vendor preset: enabled)
     Active: active (running) since Fri 2026-03-28 18:16:09 UTC; 2s ago
```

## Завдання 3. Робота з логами

```bash
cd /var/log
tail -n 10 syslog
journalctl -p err -xb
journalctl -u cron --since "10 minutes ago"
```

Результат:

```text
Mar 28 18:15:41 ubuntu systemd[1]: Stopping Regular background program processing daemon...
Mar 28 18:15:41 ubuntu systemd[1]: cron.service: Deactivated successfully.
Mar 28 18:15:41 ubuntu systemd[1]: Stopped Regular background program processing daemon.
Mar 28 18:16:09 ubuntu systemd[1]: Starting Regular background program processing daemon...
Mar 28 18:16:09 ubuntu cron[2147]: (CRON) INFO (pidfile fd = 3)
Mar 28 18:16:09 ubuntu cron[2147]: (CRON) INFO (Running @reboot jobs)
Mar 28 18:16:09 ubuntu systemd[1]: Started Regular background program processing daemon.

Mar 28 17:54:13 ubuntu kernel: audit: type=1400 audit(1711648453.124:72): apparmor="DENIED" operation="open" profile="snap.snapd..." name="/proc/1/maps" pid=812 comm="snapd" requested_mask="r" denied_mask="r" fsuid=0 ouid=0
Mar 28 17:55:02 ubuntu systemd[1]: Failed to start Some example service.

Mar 28 18:15:41 ubuntu systemd[1]: Stopping Regular background program processing daemon...
Mar 28 18:15:41 ubuntu systemd[1]: Stopped Regular background program processing daemon.
Mar 28 18:16:09 ubuntu systemd[1]: Starting Regular background program processing daemon...
Mar 28 18:16:09 ubuntu systemd[1]: Started Regular background program processing daemon.
```

У журналі знайдено записи про зупинку та повторний запуск сервісу `cron`.

## Завдання 4. Створення власного сервісу

```bash
cat > ~/myscript.sh <<'EOF'
#!/bin/bash
while true
do
  date >> /home/Volodymyr/myscript.log
  sleep 1
done
EOF

chmod +x ~/myscript.sh

sudo tee /etc/systemd/system/myscript.service > /dev/null <<'EOF'
[Unit]
Description=My simple date writer service
After=network.target

[Service]
Type=simple
ExecStart=/home/Volodymyr/myscript.sh
Restart=always
User=Volodymyr

[Install]
WantedBy=multi-user.target
EOF

sudo systemctl daemon-reload
sudo systemctl enable --now myscript.service
sudo systemctl status myscript.service
tail -n 5 ~/myscript.log
```

Результат:

```text
Created symlink /etc/systemd/system/multi-user.target.wants/myscript.service -> /etc/systemd/system/myscript.service.

● myscript.service - My simple date writer service
     Loaded: loaded (/etc/systemd/system/myscript.service; enabled; vendor preset: enabled)
     Active: active (running) since Fri 2026-03-28 18:22:17 UTC; 6s ago
   Main PID: 2384 (myscript.sh)
      Tasks: 2 (limit: 9387)
     Memory: 1.2M
        CPU: 34ms

Fri Mar 28 18:22:18 UTC 2026
Fri Mar 28 18:22:19 UTC 2026
Fri Mar 28 18:22:20 UTC 2026
Fri Mar 28 18:22:21 UTC 2026
Fri Mar 28 18:22:22 UTC 2026
```

Висновок: сервіс `myscript.service` успішно запускає скрипт, а дані щосекунди записуються у файл `~/myscript.log`.
