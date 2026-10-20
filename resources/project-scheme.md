# Исходный код диаграммы: Схема проекта (раздел 10)

Четыре сценария показаны отдельно. Повторённые Kafka, Redis, PostgreSQL, Object Storage, клиент и SFU обозначают те же компоненты.

PNG и SVG с фиксированным расположением блоков собираются из единого описания:

```bash
python3 scripts/render_project_scheme.py
```

Нужны Python 3, Node.js и Chromium-браузер. Путь к браузеру можно передать через `--browser`. Сетевые запросы для сборки не нужны.

## 01   ПОДКЛЮЧЕНИЕ И УПРАВЛЕНИЕ

```mermaid
%%{init: {"theme":"base", "layout":"elk", "look":"classic", "flowchart":{"curve":"linear"}}}%%
flowchart LR
    DNS1["Geo-DNS<br/>PowerDNS<br/>МСК + СПб"]:::edge
    C1["Клиент<br/>Web · iOS · Android"]:::client
    INAPI1["Вход API<br/>L4 + nginx<br/>МСК, резерв СПб"]:::edge
    INWS1["Вход WSS<br/>L4 + nginx<br/>Ближайший живой POP"]:::edge
    API1["Control API<br/>Java · права и токены<br/>МСК"]:::service
    SIG1["Сигналинг LiveKit<br/>Выбор SFU встречи<br/>Региональный POP"]:::service
    ORC1["Оркестратор<br/>Java · управление<br/>МСК"]:::service
    PG1["PostgreSQL<br/>Данные приложения<br/>МСК; резерв СПб"]:::store
    RD1["Redis<br/>Состояние LiveKit<br/>МСК"]:::queue

    C1 <-->|"DNS"| DNS1
    C1 <-->|"HTTPS"| INAPI1
    C1 <-->|"WSS"| INWS1
    INAPI1 <--> API1
    INWS1 <--> SIG1
    API1 -->|"Команды"| ORC1
    ORC1 <-->|"Server API"| SIG1
    API1 <-->|"Данные приложения"| PG1
    SIG1 <-->|"Комнаты и маршрутизация"| RD1

    classDef client fill:#ffffff,stroke:#9aa6b2,color:#273647
    classDef edge fill:#e8f5ef,stroke:#89b4a0,color:#273647
    classDef service fill:#eaf2ff,stroke:#94afd2,color:#273647
    classDef media fill:#e5f6f1,stroke:#7eb4a7,color:#273647
    classDef store fill:#fff3df,stroke:#c6aa79,color:#273647
    classDef queue fill:#f2eafa,stroke:#b397c8,color:#273647
    linkStyle 0 stroke:#85939c
    linkStyle 1 stroke:#426b9e
    linkStyle 2 stroke:#426b9e
    linkStyle 3 stroke:#426b9e
    linkStyle 4 stroke:#426b9e
    linkStyle 5 stroke:#426b9e
    linkStyle 6 stroke:#426b9e
    linkStyle 7 stroke:#ae803e
    linkStyle 8 stroke:#8c63a8
```

## 02   МЕДИАПУТЬ И КАЧЕСТВО ЗВОНКА

```mermaid
%%{init: {"theme":"base", "layout":"elk", "look":"classic", "flowchart":{"curve":"linear"}}}%%
flowchart LR
    C2["Клиент<br/>Тот же участник"]:::client
    SIG2["Сигналинг LiveKit<br/>Согласование соединения"]:::service
    SFU2["SFU встречи<br/>LiveKit · один узел на встречу<br/>POP размещения встречи"]:::media
    RD2["Redis<br/>Загрузка и состояние узлов"]:::queue
    TURN2["TURN<br/>coturn · региональный POP"]:::edge
    KF2["Kafka<br/>События качества"]:::queue
    CH2["ClickHouse<br/>Аналитика качества звонков"]:::store

    C2 <-->|"SRTP / UDP напрямую"| SFU2
    SIG2 <--> SFU2
    SFU2 <-->|"Состояние"| RD2
    C2 <-.->|"Если прямой путь недоступен"| TURN2
    TURN2 <-->|"Relay"| SFU2
    SFU2 -->|"QoS через сборщик"| KF2
    KF2 -->|"QoS"| CH2

    classDef client fill:#ffffff,stroke:#9aa6b2,color:#273647
    classDef edge fill:#e8f5ef,stroke:#89b4a0,color:#273647
    classDef service fill:#eaf2ff,stroke:#94afd2,color:#273647
    classDef media fill:#e5f6f1,stroke:#7eb4a7,color:#273647
    classDef store fill:#fff3df,stroke:#c6aa79,color:#273647
    classDef queue fill:#f2eafa,stroke:#b397c8,color:#273647
    linkStyle 0 stroke:#288976
    linkStyle 1 stroke:#426b9e
    linkStyle 2 stroke:#8c63a8
    linkStyle 3 stroke:#288976
    linkStyle 4 stroke:#288976
    linkStyle 5 stroke:#8c63a8
    linkStyle 6 stroke:#8c63a8
```

## 03   ЗАПИСЬ И ВЫДАЧА ФАЙЛОВ

```mermaid
%%{init: {"theme":"base", "layout":"elk", "look":"classic", "flowchart":{"curve":"linear"}}}%%
flowchart LR
    SFU3["SFU встречи<br/>Тот же медиасервер"]:::media
    ORC3["Оркестратор<br/>Тот же сервис"]:::service
    REC3["Захват медиа<br/>SDK + FFmpeg<br/>Видео или только аудио"]:::service
    PG3["PostgreSQL<br/>Метаданные записи"]:::store
    OS3["Object Storage<br/>Видео и аудио"]:::store
    CDN3["CDN<br/>Статика и готовые записи"]:::store
    C3["Клиент<br/>Просмотр готовой записи"]:::client
    KF3["Kafka<br/>Задания ASR"]:::queue

    SFU3 -->|"Медиа"| REC3
    ORC3 -->|"Запуск / стоп"| REC3
    REC3 -->|"Метаданные"| PG3
    REC3 -->|"Файлы"| OS3
    OS3 -->|"Origin"| CDN3
    CDN3 -->|"HTTPS"| C3
    REC3 --> KF3

    classDef client fill:#ffffff,stroke:#9aa6b2,color:#273647
    classDef edge fill:#e8f5ef,stroke:#89b4a0,color:#273647
    classDef service fill:#eaf2ff,stroke:#94afd2,color:#273647
    classDef media fill:#e5f6f1,stroke:#7eb4a7,color:#273647
    classDef store fill:#fff3df,stroke:#c6aa79,color:#273647
    classDef queue fill:#f2eafa,stroke:#b397c8,color:#273647
    linkStyle 0 stroke:#288976
    linkStyle 1 stroke:#426b9e
    linkStyle 2 stroke:#ae803e
    linkStyle 3 stroke:#ae803e
    linkStyle 4 stroke:#ae803e
    linkStyle 5 stroke:#426b9e
    linkStyle 6 stroke:#8c63a8
```

## 04   РАСШИФРОВКА

```mermaid
%%{init: {"theme":"base", "layout":"elk", "look":"classic", "flowchart":{"curve":"linear"}}}%%
flowchart LR
    KF4["Kafka<br/>Тот же поток заданий ASR"]:::queue
    ASR4["ASR-диспетчер<br/>Java · МСК"]:::service
    SK4["SpeechKit<br/>Распознавание речи<br/>Yandex Cloud"]:::store
    OS4["Object Storage<br/>Аудио и расшифровки"]:::store
    PG4["PostgreSQL<br/>Метаданные расшифровки"]:::store

    KF4 -->|"Задание"| ASR4
    ASR4 <-->|"API"| SK4
    OS4 -->|"Аудио"| SK4
    ASR4 -->|"Текст расшифровки"| OS4
    ASR4 -->|"Статус и ссылка на результат"| PG4

    classDef client fill:#ffffff,stroke:#9aa6b2,color:#273647
    classDef edge fill:#e8f5ef,stroke:#89b4a0,color:#273647
    classDef service fill:#eaf2ff,stroke:#94afd2,color:#273647
    classDef media fill:#e5f6f1,stroke:#7eb4a7,color:#273647
    classDef store fill:#fff3df,stroke:#c6aa79,color:#273647
    classDef queue fill:#f2eafa,stroke:#b397c8,color:#273647
    linkStyle 0 stroke:#8c63a8
    linkStyle 1 stroke:#426b9e
    linkStyle 2 stroke:#ae803e
    linkStyle 3 stroke:#ae803e
    linkStyle 4 stroke:#ae803e
```
