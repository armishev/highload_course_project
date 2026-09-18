# Исходный код диаграммы: Схема DNS-балансировки (раздел 3)

```mermaid
flowchart TD
    U["Клиент (Web / iOS / Android)"]
    R["Рекурсивный DNS-резолвер<br/>(+EDNS Client Subnet)"]
    G["Geo-DNS на PowerDNS<br/>(МСК + СПб)"]
    CDN["Yandex Cloud CDN<br/>(static, VOD-записи)"]

    subgraph POP["Ближайший POP"]
        VIP["Вход в POP<br/>L4 → L7 nginx"]
        WS["ws.telemost.ru<br/>(Signaling)"]
        SFU["SFU-ноды (медиаслой)"]
    end

    subgraph MSK["МСК (control plane)"]
        VIPM["Вход в МСК<br/>L4 → L7 nginx"]
        API["api.telemost.ru<br/>(Control API)"]
        O["Оркестратор<br/>(Application Based LB, least load)"]
    end

    U -->|"1. DNS-запрос"| R
    R --> G
    G -.->|"2. ws → IP ближайшего POP, api → IP МСК"| U
    U -->|"3. WSS на вход POP"| VIP
    U -->|"3. HTTPS на вход МСК"| VIPM
    VIPM --> API
    VIP --> WS
    WS -->|"4. запрос на медиа"| O
    O -->|"5. SFU по загрузке"| WS
    WS -->|"6. IP:port SFU клиенту"| U
    U -.->|"7. медиа SRTP/UDP напрямую"| SFU
    U -->|"static / VOD"| CDN
```
