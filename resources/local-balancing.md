# Исходный код диаграммы: Схема локальной балансировки (раздел 4)

```mermaid
%%{init: {
    "theme": "base",
    "layout": "elk",
    "look": "classic",
    "fontFamily": "Arial, sans-serif",
    "themeVariables": {
        "fontSize": "14px",
        "primaryColor": "#f5f5f5",
        "primaryTextColor": "#333333",
        "primaryBorderColor": "#dddddd",
        "lineColor": "#999999",
        "secondaryColor": "#f5f5f5",
        "tertiaryColor": "#fafafa",
        "clusterBkg": "#fafafa",
        "clusterBorder": "#e5e5e5",
        "edgeLabelBackground": "#ffffff"
    },
    "flowchart": {
        "curve": "linear",
        "look": "classic",
        "wrappingWidth": 200,
        "nodeSpacing": 55,
        "rankSpacing": 75,
        "padding": 18
    },
    "themeCSS": ".node rect { rx: 12px; ry: 12px; stroke-width: 1px; } .cluster rect { rx: 14px; ry: 14px; stroke-width: 1px; } .edgeLabel .labelBkg { background-color: #ffffff; border: 1px solid #e5e5e5; border-radius: 10px; padding: 3px 8px; } .edgeLabel .labelBkg:has(.edgeLabel:empty) { border: 0; padding: 0; } .edgeLabel p { margin: 0; } .flowchart-link { stroke-width: 1px; stroke-linecap: round; stroke-linejoin: round; }"
}}%%
flowchart TD
    C["Клиенты Web / iOS / Android"]
    CDN["CDN — файлы сайта и записи"]

    subgraph POP["Один POP"]
        VIP["Общий IP — пара L4-узлов IPVS"]

        subgraph L7POOL["Пул nginx — один рабочий и один резервный"]
            L7A["Рабочий nginx"]
            L7R["Резервный nginx"]
        end

        SIG["Signaling — в каждом POP"]
        API["Control API — только МСК или резерв СПб"]
        SFU["SFU встреч на этой площадке"]
        TURN["TURN — ретранслятор"]
    end

    C <-->|"HTTPS / WSS"| VIP

    VIP <-->|"Основной путь"| L7A
    VIP <-.->|"При отказе рабочего nginx"| L7R

    L7A <--> SIG
    L7A <--> API
    L7R <-.-> SIG
    L7R <-.-> API

    C <-->|"Медиа напрямую — SRTP/UDP"| SFU
    C <-->|"Если прямое подключение недоступно"| TURN
    TURN <-->|"Пересылка медиапакетов"| SFU

    C <-->|"HTTPS"| CDN
```
