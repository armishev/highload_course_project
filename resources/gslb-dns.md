# Исходный код диаграммы: Схема DNS-балансировки (раздел 3)

```mermaid
flowchart TD
    U["Клиент Web / iOS / Android"]
    R["Рекурсивный DNS-резолвер"]
    G["Geo-DNS PowerDNS — МСК и СПб"]
    CDN["CDN — файлы сайта и готовые записи"]

    subgraph EDGE["Ближайший доступный POP"]
        VIP["Вход — L4 и L7 nginx"]
        SIG["Signaling — ws.telemost.ru"]
        TURN["TURN — ретранслятор медиа"]
    end

    subgraph CONTROL["МСК — основной контур управления"]
        VIPM["Вход — L4 и L7 nginx"]
        API["Control API — api.telemost.ru"]
        O["Оркестратор — размещение встреч"]
    end

    subgraph MEDIA["POP размещения встречи"]
        SFU["SFU выбранной встречи"]
    end

    U -->|"1. Запрос адреса сервиса"| R
    R -->|"Запрос при отсутствии ответа в кэше"| G
    G -.->|"2. DNS-ответ"| R
    R -.->|"Адрес сервиса"| U

    U <-->|"3. HTTPS"| VIPM
    VIPM <--> API

    U <-->|"3. WSS — служебные сообщения"| VIP
    VIP <--> SIG

    SIG -->|"4. Найти или назначить SFU для встречи"| O
    O -->|"5. Выбранный SFU"| SIG

    SIG <-->|"Согласование медиасоединения"| SFU

    U <-->|"6. Основной медиапуть — SRTP/UDP"| SFU
    U <-->|"6. Запасной медиапуть"| TURN
    TURN <-->|"Пересылка медиапакетов"| SFU

    U <-->|"HTTPS — загрузка файлов и записей"| CDN
```
