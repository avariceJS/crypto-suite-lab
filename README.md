# PD Project — Crypto Suite (Vigenère + SHA‑384 + Chord Method + Steganography)

Учебный курсовой проект на **C++ / Qt 6**: клиент-серверное приложение, реализующее
четыре алгоритма (шифр Виженера, хеш SHA‑384, метод хорд для нахождения корня
уравнения, LSB-стеганография — внедрение сообщения в картинку), с авторизацией,
ролями (user/admin), БД (SQLite, синглтон), несколькими клиентами и Docker-инфраструктурой.

![architecture](docs/uml/architecture.png)

## Возможности

| №   | Функция                                          | Где используется       |
| --- | ------------------------------------------------ | ---------------------- |
| 1   | Шифр Виженера (encrypt/decrypt)                  | клиент ↔ сервер        |
| 2   | SHA-384 (hash сообщения / хеш паролей)           | сервер (auth) + клиент |
| 3   | Метод хорд — численное решение уравнений         | клиент (UI ввода f(x)) |
| 4   | LSB-стеганография — скрытие сообщения в PNG      | клиент                 |
| 5   | Авторизация / регистрация                        | сервер                 |
| 6   | Роли user / admin                                | сервер + клиент        |
| 7   | Админ-функции (список пользователей, блокировка) | сервер + клиент        |
| 8   | Логи операций в таблице                          | клиент                 |

## Структура репозитория

```
pd/
├── core/         — общая библиотека алгоритмов (Vigenere, SHA384, Chord, Steg)
├── server/       — Qt TCP-сервер + SQLite (singleton)
├── client/       — Qt Widgets GUI (singleton)
├── tests/        — UnitTest (QtTest)
├── docs/
│   ├── wiki/     — страницы для GitHub Wiki
│   ├── uml/      — диаграммы (PlantUML + PNG)
│   ├── requirements.md
│   └── testing/  — тест-план, чек-лист, тест-кейсы, дефекты (CSV/XLS)
├── docker-compose.yml
├── Doxyfile
└── CMakeLists.txt
```

## Ветки git

| Ветка                     | Назначение                       |
| ------------------------- | -------------------------------- |
| `main`                    | стабильный релиз                 |
| `develop`                 | основная разработка              |
| `feature/core-algorithms` | алгоритмы шифрования/хеширования |
| `feature/server`          | TCP-сервер и БД                  |
| `feature/client`          | GUI-клиент                       |
| `feature/docker`          | контейнеризация                  |
| `feature/tests`           | модульные тесты                  |

Создание веток: `bash scripts/setup_branches.sh`

## Запуск

Нужны **CMake ≥ 3.16**, компилятор с **C++17**, **Qt 6** с модулями:  
`Core`, `Network`, `Sql`, `Widgets`, `Gui`, `Test` (модуль Test — только для юнит-тестов).

Склонируйте репозиторий и перейдите в каталог проекта:

```bash
git clone https://github.com/<ваш-аккаунт>/<имя-репозитория>.git
cd <имя-репозитория>
```

### macOS (Homebrew)

```bash
brew install qt cmake
export CMAKE_PREFIX_PATH="$(brew --prefix qt)"
cmake -S . -B build -DCMAKE_BUILD_TYPE=Release
cmake --build build -j
```

При проблемах с поиском Qt CMake сообщит об ошибке `find_package(Qt6)` — проверьте, что `CMAKE_PREFIX_PATH` указывает на префикс Homebrew Qt.

Запуск (два терминала):

```bash
# терминал 1 — сервер (оставить окно открытым)
./build/server/pd_server --port 5555 --db pd_server.db

# терминал 2 — клиент
./build/client/pd_client
```

В диалоге входа: хост `127.0.0.1`, порт `5555`. Учётная запись администратора по умолчанию после первого старта сервера: **admin / admin**.

Проверка, что сервер слушает порт:

```bash
lsof -nP -iTCP:5555 -sTCP:LISTEN
```

Юнит-тесты:

```bash
ctest --test-dir build/tests --output-on-failure
```

Если GUI клиента не находит плагины Qt, можно задать (обычно достаточно автоматического поиска в коде сервера под Homebrew):

```bash
export QT_PLUGIN_PATH="$(brew --prefix qt)/share/qt/plugins"
```

### Linux (пример: Ubuntu / Debian)

```bash
sudo apt update
sudo apt install cmake build-essential qt6-base-dev
cmake -S . -B build -DCMAKE_BUILD_TYPE=Release
cmake --build build -j
```

Если сервер не открывает SQLite (ошибка драйвера QSQLITE), установите пакет с плагином SQL для Qt 6 — имя зависит от дистрибутива (ищите в репозитории по запросам `qt6` и `sqlite`).

Если `find_package(Qt6)` не находит Qt, укажите префикс явно (путь зависит от дистрибутива), например:

```bash
cmake -S . -B build -DCMAKE_BUILD_TYPE=Release -DCMAKE_PREFIX_PATH=/usr/lib/x86_64-linux-gnu/cmake/Qt6
```

Запуск сервера и клиента — те же команды `./build/server/pd_server ...` и `./build/client/pd_client`.

### Windows

Установите **Qt 6** (Qt Online Installer) и **CMake**, откройте **x64 Native Tools** (или среду с компилятором MSVC/MinGW).

```bat
cmake -S . -B build -DCMAKE_BUILD_TYPE=Release -DCMAKE_PREFIX_PATH=C:\Qt\6.x.x\msvc2019_64
cmake --build build --config Release
build\Release\pd_server.exe --port 5555 --db pd_server.db
build\Release\pd_client.exe
```

Точный путь к `CMAKE_PREFIX_PATH` возьмите из каталога установки Qt (папка, где лежит `lib/cmake/Qt6`).

### Docker

Сборка и запуск через контейнеры (см. `docker-compose.yml`):

```bash
docker compose up --build
```

## Сборка (кратко)

Локально, если Qt уже виден CMake:

```bash
cmake -S . -B build -DCMAKE_BUILD_TYPE=Release
cmake --build build -j
ctest --test-dir build/tests --output-on-failure
```

## Документация

- **Doxygen**: `doxygen Doxyfile` → `docs/doxygen/html/index.html`
- **Wiki**: см. `docs/wiki/Home.md`
- **Требования**: `docs/requirements.md`
- **UML**: `docs/uml/`

## Лицензия

MIT
