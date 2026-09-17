# Исходный код диаграммы: Логическая схема БД (раздел 5)

```mermaid
erDiagram
    users ||--o{ user_sessions : "имеет"
    users ||--o{ meetings : "владеет"
    users ||--o{ meeting_invitees : "приглашён (nullable)"
    users ||--o{ meeting_participants : "участвует (nullable, гость)"
    users ||--o{ chat_messages : "отправляет (nullable, гость)"
    meetings ||--o{ meeting_invitees : "приглашает"
    meetings ||--o{ meeting_participants : "включает"
    meetings ||--o{ chat_messages : "содержит"
    meetings ||--o{ recordings : "порождает"
    meetings ||--o{ transcripts : "порождает"
    recordings ||--|| recording_blobs : "ссылается"
    transcripts ||--|| transcript_blobs : "ссылается"

    users {
        uuid user_id PK
        varchar email UK
        varchar phone UK
        varchar display_name
        varchar avatar_url
        jsonb settings_json
        timestamp created_at
        timestamp updated_at
    }
    user_sessions {
        uuid session_id PK
        uuid user_id FK
        varchar device_id
        varchar push_token
        varchar user_agent
        varchar ip
        timestamp created_at
        timestamp expires_at
    }
    meetings {
        uuid meeting_id PK
        uuid owner_id FK "nullable, гостевая встреча"
        varchar join_code UK
        enum type "instant / scheduled"
        varchar title
        timestamp scheduled_start
        enum status
        jsonb settings_json "lock, зал ожидания, запись"
        timestamp created_at
    }
    meeting_invitees {
        uuid id PK
        uuid meeting_id FK
        uuid user_id FK "nullable"
        varchar email "nullable, внешний приглашённый"
        enum status "invited / accepted"
    }
    meeting_participants {
        uuid id PK
        uuid meeting_id FK
        uuid user_id FK "nullable, гость"
        varchar display_name "для гостя"
        enum role "host / cohost / participant"
        timestamp joined_at
        timestamp left_at
        varchar device
    }
    chat_messages {
        uuid message_id PK
        uuid meeting_id FK
        uuid sender_id FK "nullable, гость"
        varchar text
        timestamp created_at
    }
    recordings {
        uuid recording_id PK
        uuid meeting_id FK
        varchar storage_url
        int duration_s
        bigint size_bytes
        enum resolution
        enum status
        timestamp created_at
    }
    transcripts {
        uuid transcript_id PK
        uuid meeting_id FK
        varchar storage_url
        varchar lang
        timestamp created_at
    }
    recording_blobs {
        varchar object_key PK
        bigint size_bytes
    }
    transcript_blobs {
        varchar object_key PK
        bigint size_bytes
    }
```
