# Домашнє завдання №7. Аналіз життєвого циклу контейнера

Виконано у WSL (Ubuntu).

Команди Docker виконувались у локальному контейнерному середовищі.

Обраний варіант: `A — Аналіз життєвого циклу контейнера`.

## Завдання 1. Запуск контейнера

```bash
docker run -d --name module7-app -p 8080:8080 python:3.12-slim sh -c "python -u -m http.server 8080"
docker ps --filter name=module7-app
```

Результат:

```text
9b884ca6bf36836753d54a2e997958f13346946e0d598524ea91a419eca135ca

CONTAINER ID   IMAGE              COMMAND                  CREATED          STATUS          PORTS                                         NAMES
9b884ca6bf36   python:3.12-slim   "sh -c 'python -u -m…"   31 seconds ago   Up 30 seconds   0.0.0.0:8080->8080/tcp, [::]:8080->8080/tcp   module7-app
```

Контейнер запущено у фоновому режимі з простим HTTP-сервером Python. Порт `8080` контейнера опубліковано на порт `8080` хоста.

## Завдання 2. Який процес є PID 1

```bash
docker exec module7-app sh -c 'cat /proc/1/cmdline | tr "\0" " "'
docker exec module7-app sh -c 'cat /proc/1/status | sed -n "1,12p"'
docker exec module7-app sh -c 'cat /proc/1/task/1/children'
```

Результат:

```text
sh -c python -u -m http.server 8080

Name:	sh
Umask:	0022
State:	S (sleeping)
Tgid:	1
Ngid:	0
Pid:	1
PPid:	0
TracerPid:	0
Uid:	0	0	0	0
Gid:	0	0	0	0
FDSize:	64
Groups:	0

7
```

Процесом `PID 1` усередині контейнера є `sh`, а не `python`. Це відбувається тому, що контейнер запущено командою `sh -c "python -u -m http.server 8080"`, тобто саме оболонка `sh` стає головним процесом контейнера, а HTTP-сервер Python запускається як її дочірній процес з PID `7`.

## Завдання 3. Завершення контейнера

```bash
docker stop module7-app
docker ps -a --filter name=module7-app
docker inspect module7-app --format '{{.State.Status}} {{.State.ExitCode}} {{.State.FinishedAt}}'
docker inspect module7-app --format '{{.Path}} {{join .Args " "}}'
```

Результат:

```text
module7-app

CONTAINER ID   IMAGE              COMMAND                  CREATED              STATUS                        PORTS     NAMES
9b884ca6bf36   python:3.12-slim   "sh -c 'python -u -m…"   About a minute ago   Exited (137) 23 seconds ago             module7-app

exited 137 2026-04-19T13:17:18.950380842Z
sh -c python -u -m http.server 8080
```

Команда `docker stop` спочатку надсилає головному процесу контейнера сигнал `SIGTERM` за замовчуванням. У цьому прикладі контейнер завершився з кодом `137`, що означає примусове завершення після `SIGKILL`. Це сталося тому, що `PID 1` був `sh`, а не сам Python-сервер, і головний процес не завершив контейнер коректно до завершення grace period. Якщо процес ігнорує `SIGTERM` або не обробляє його належним чином, Docker після очікування надсилає `SIGKILL` і примусово зупиняє контейнер.

## Завдання 4. Логи контейнера

```bash
docker logs module7-app
```

Результат:

```text
Serving HTTP on 0.0.0.0 port 8080 (http://0.0.0.0:8080/) ...
```

Логи, які показує `docker logs`, беруться зі стандартних потоків `stdout` і `stderr` головного процесу контейнера. У цьому випадку Python HTTP-сервер вивів службове повідомлення про старт у стандартний вивід, тому Docker зберіг його і показав через `docker logs`.

## Висновок

Життя контейнера визначається життям його головного процесу `PID 1`. Поки цей процес працює, контейнер залишається активним. У цьому прикладі контейнер завершився після `docker stop`, тому що Docker спочатку відправив `SIGTERM`, а потім примусово завершив контейнер через `SIGKILL`, що видно з коду виходу `137`.
