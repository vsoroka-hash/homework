# Домашнє завдання №6. Bash-скрипт для бекапу логів

Виконано у WSL (Ubuntu).

Обраний варіант: `A — Скрипт бекапу логів`.

## Код скрипта

```bash
#!/bin/bash

LOCK_FILE="/tmp/backup.lock"

if [ "$#" -ne 2 ] || [ ! -d "$1" ] || [ ! -d "$2" ]; then
  echo "Usage: ./backup.sh <log_dir> <backup_dir>"
  exit 1
fi

LOG_DIR="$1"
BACKUP_DIR="$2"

if [ -e "$LOCK_FILE" ]; then
  echo "Backup already running"
  exit 1
fi

touch "$LOCK_FILE"
trap 'rm -f "$LOCK_FILE"' EXIT

TIMESTAMP="$(date +"%Y-%m-%d_%H-%M")"
ARCHIVE_NAME="logs_backup_${TIMESTAMP}.tar.gz"
ARCHIVE_PATH="${BACKUP_DIR}/${ARCHIVE_NAME}"

# Архівується вміст каталогу логів, а lock-файл видаляється при завершенні.
tar -czf "$ARCHIVE_PATH" -C "$LOG_DIR" .
STATUS=$?

if [ "$STATUS" -ne 0 ]; then
  echo "Backup failed"
  exit 2
fi

echo "Backup created: $(cd "$BACKUP_DIR" && pwd)/$ARCHIVE_NAME"
```

## Короткий опис

Скрипт `backup.sh` приймає два аргументи: каталог з логами та каталог для збереження резервної копії. Спочатку він перевіряє кількість аргументів і існування обох каталогів. Якщо перевірка не проходить, виводиться повідомлення про правильний формат запуску та скрипт завершується з кодом `1`.

Далі скрипт перевіряє наявність lock-файлу `/tmp/backup.lock`, щоб уникнути паралельного запуску. Якщо lock-файл уже існує, скрипт повідомляє, що бекап уже виконується, і завершує роботу. Якщо ні, створює lock-файл, формує ім'я архіву з поточною датою та часом і створює архів `.tar.gz` з усіх файлів каталогу логів. У разі помилки архівації виводиться `Backup failed` і скрипт завершується з кодом `2`. Якщо все успішно, скрипт виводить повний шлях до створеного архіву.

## Перевірка роботи з різними аргументами

```bash
chmod +x backup.sh
mkdir -p test_logs test_backup
echo "error line 1" > test_logs/app.log
echo "warning line 2" > test_logs/system.log
./backup.sh
./backup.sh test_logs missing_dir
touch /tmp/backup.lock
./backup.sh test_logs test_backup
rm -f /tmp/backup.lock
./backup.sh test_logs /root
./backup.sh test_logs test_backup
ls -1 test_backup
```

Результат:

```text
Usage: ./backup.sh <log_dir> <backup_dir>
Usage: ./backup.sh <log_dir> <backup_dir>
Backup already running
tar: Failed to open '/root/logs_backup_2026-04-19_16-01.tar.gz'
Backup failed
Backup created: /home/Volodymyr/homework/module6/test_backup/logs_backup_2026-04-19_16-01.tar.gz
logs_backup_2026-04-19_16-01.tar.gz
```

У перевірці показано запуск без аргументів, запуск із неіснуючим каталогом для бекапу, захист від паралельного запуску через lock-файл, приклад невдалого створення архіву та успішне створення резервної копії в каталозі `test_backup`.
