# Проект по дисциплине «Технологии и методы программирования»

**Репозиторий:** [crypto-suite-lab](https://github.com/avariceJS/crypto-suite-lab) (PD Crypto Suite)

## Участники (учебная группа: **251-353**)

1. Следников Александр  
2. Кемплинг Екатерина  

---

## Документация к проекту

| Раздел | Описание |
|--------|----------|
| [Архитектура](Architecture) | Компоненты системы, клиент–сервер |
| [Сборка и запуск](Build-and-Run) | CMake, Qt 6, Docker |
| [Протокол](Protocol) | JSON-команды TCP |
| [Роли и безопасность](Roles-and-Security) | user / admin |
| [Тестирование](Testing) | стратегия, Qt Test |
| [Стеганография](Steganography) | LSB-алгоритм |
| [Метод хорд](Chord-Method) | численное решение уравнений |
| [FAQ](FAQ) | типовые вопросы |

Файлы тест-плана и артефакты: каталог [`docs/testing`](https://github.com/avariceJS/crypto-suite-lab/tree/main/docs/testing) в основном репозитории.

---

## Структура Git (схема ветвления)

Ниже — упрощённая модель: стабильная ветка `main`, интеграция в `develop`, задачи в `feature/*`, документация в `docs`.

```mermaid
gitGraph
    commit id: "инициализация"
    branch develop
    checkout develop
    commit id: "интеграция"
    branch feature-algorithms
    checkout feature-algorithms
    commit id: "алгоритмы core"
    checkout develop
    merge feature-algorithms
    branch feature-server
    checkout feature-server
    commit id: "TCP SQLite"
    checkout develop
    merge feature-server
    branch feature-client
    checkout feature-client
    commit id: "Qt GUI"
    checkout develop
    merge feature-client
    branch docs
    checkout docs
    commit id: "Wiki UML"
    commit id: "тест-план"
    checkout develop
    commit id: "Docker тесты"
    checkout main
    merge develop
    commit id: "Ver 1.0"
    checkout develop
    commit id: "полировка"
    checkout main
    merge develop
    commit id: "Ver 1.1"
```

---

## Основные ветки

| Ветка | Назначение | Ответственные |
|-------|------------|----------------|
| `main` | Стабильный код, пригодный к сдаче / демонстрации | Следников А., Кемплинг Е. |
| `develop` | Интеграция фич и промежуточная проверка | оба участника |
| `feature/*` | Отдельные задачи (сервер, клиент, алгоритмы, Docker, тесты) | оба участника |
| `docs` | Документация, диаграммы, материалы Wiki | оба участника |

---

## Логика работы с ветками

**Проект состоит из основных частей:**

- клиент (Qt Widgets);  
- сервер (Qt TCP + SQLite);  
- общее ядро (`core`) и документация (`docs/`).

**Общий принцип:**

- актуальная документация и артефакты — в репозитории (`docs/`, при необходимости отдельная ветка `docs`);  
- разработка идёт от `develop`; прямые коммиты в `main` без ревью нежелательны;  
- задачи выполняются во временных ветках `feature/…` с последующим merge в `develop`, затем в `main` при готовности релиза.

**Этапы работы над задачей:**

1. Создать ветку `feature/<краткое-название>` от `develop`.  
2. После реализации и локальной проверки — merge (или PR) в `develop`.  
3. После проверки всего приложения — merge `develop` → `main` (релиз).  
4. При дефектах — исправления снова через `develop`.

Такая схема упрощает совместную работу и снижает риск поломки стабильной ветки.

---

## Диаграмма классов — сервер

Соответствует основным типам в коде (`server/`, `Database` — синглтон).

```mermaid
classDiagram
    class QTcpServer
    class Server {
        +start(quint16 port) bool
        #incomingConnection(qintptr)
    }
    class ClientHandler {
        -QTcpSocket* sock_
        -QString login_, role_
        +dispatch(QJsonObject) QJsonObject
        +onReadyRead()
        +onDisconnected()
    }
    class Database {
        <<singleton>>
        +instance() Database&
        +open(path) bool
        +authenticate(...) bool
        +registerUser(...) bool
        +listUsers()
        +listLogs()
    }

    QTcpServer <|-- Server
    Server --> ClientHandler : создаёт на connect
    ClientHandler --> Database : запросы к БД
```

---

## Диаграмма классов — клиент

```mermaid
classDiagram
    class MainWindow {
        +вкладки крипто, админ
        +onVigenereEnc/Dec()
        +onSha384()
        +refreshUsers()
    }
    class LoginDialog {
        +doLogin()
        +doRegister()
    }
    class ClientCore {
        <<singleton>>
        +instance() ClientCore&
        +connectTo(host, port) bool
        +request(QJsonObject) QJsonObject
        +login(user, pass) bool
    }

    MainWindow --> ClientCore : JSON-запросы
    LoginDialog --> ClientCore : вход / регистрация
```

---

## Use-case диаграмма

```mermaid
flowchart TB
    subgraph actors["Акторы"]
        G[Гость]
        U[Пользователь]
        A[Администратор]
    end
    subgraph cases["Действия"]
        REG[Регистрация]
        AUTH[Вход]
        VIG[Виженер SHA Хорды]
        STEG[Стеганография локально]
        ADM1[Список пользователей / блокировка]
        ADM2[Журнал событий]
    end
    S[(Сервер)]

    G --> REG --> S
    G --> AUTH --> S
    AUTH -.-> U
    AUTH -.-> A
    U --> VIG --> S
    U --> STEG
    A --> VIG --> S
    A --> STEG
    A --> ADM1 --> S
    A --> ADM2 --> S
```

---

## Стратегии тестирования

- Подробности: страница **[Тестирование](Testing)**.  
- Артефакты (чек-листы, тест-кейсы, дефекты): **[docs/testing в репозитории](https://github.com/avariceJS/crypto-suite-lab/tree/main/docs/testing)**.

---

## Прогресс в работе (примерный хронологический план)

| Период | Содержание работ |
|--------|------------------|
| Неделя 1 | Репозиторий GitHub, структура CMake, Wiki, диаграммы *(Следников А., Кемплинг Е.)* |
| Неделя 2 | Сервер: TCP, протокол JSON, SQLite-синглтон, авторизация *(Следников А., Кемплинг Е.)* |
| Неделя 3 | Клиент Qt, синглтон сетевого слоя, вкладки функций *(Следников А., Кемплинг Е.)* |
| Неделя 4 | Docker, роли admin/user, админ-таблицы, UnitTest (`tests/`) *(Следников А., Кемплинг Е.)* |
| Неделя 5 | Doxygen, тест-план и тест-кейсы, доработка документации *(Следников А., Кемплинг Е.)* |

---

## Архитектура

![Архитектура](https://raw.githubusercontent.com/avariceJS/crypto-suite-lab/main/docs/uml/architecture.svg)

```mermaid
flowchart LR
    subgraph client["Хост клиента"]
        UI[pd_client Qt Widgets]
        CC[ClientCore singleton]
        UI --> CC
    end
    subgraph server["Хост сервера"]
        SR[pd_server]
        TS[QTcpServer]
        CH[ClientHandler]
        DB[(Database singleton)]
        SQL[(SQLite)]
        SR --> TS --> CH --> DB --> SQL
    end
    CC <-->|"TCP JSON, порт 5555"| TS
```

---

**Ответственные за Git:** Следников Александр, Кемплинг Екатерина.
