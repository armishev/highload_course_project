# Исходный код диаграммы: Физическая схема БД (раздел 6)

```mermaid
flowchart TB
    subgraph PG["PostgreSQL — Patroni: primary + sync standby МСК, async standby СПб (без шардирования)"]
        direction TB
        U["users\n─────────────\nPK: user_id\nUNIQUE: email, phone"]
        US["user_sessions\n─────────────\nPK: session_id\nFK: user_id\nIDX: user_id, expires_at"]
        MT["meetings\n─────────────\nPK: meeting_id\nFK: owner_id (nullable)\nUNIQUE: join_code\n+ participant_count, has_recording, has_transcript"]
        MI["meeting_invitees\n─────────────\nPK: id\nFK: meeting_id, user_id (nullable)\nIDX: meeting_id, user_id"]
        MP["meeting_participants\n─────────────\nPK: (id, joined_at)\nFK: meeting_id, user_id (nullable)\nIDX: meeting_id\nIDX: (user_id, joined_at DESC)\nPARTITION BY RANGE(joined_at), по месяцам"]
        CM["chat_messages\n─────────────\nPK: (message_id, created_at)\nFK: meeting_id, sender_id (nullable)\nIDX: (meeting_id, created_at)\nPARTITION BY RANGE(created_at), по месяцам"]
        RT["recordings / transcripts\n─────────────\nPK: recording_id / transcript_id\nFK: meeting_id\nIDX: meeting_id\nstorage_url → Object Storage"]
    end

    subgraph REDIS["Redis Cluster МСК — 3 master + 3 replica, слоты по ключу"]
        direction TB
        LS["live_meeting_state\n─────────────\nКлюч: meeting:{meeting_id}:state\nroster, mic/cam/hand · TTL встречи"]
        PR["participant_routing\n─────────────\nКлюч: participant:{participant_id}\nSFU, presence · TTL присутствия"]
        MA["meeting_allocation\n─────────────\nКлюч: meeting:{meeting_id}:alloc\nhome POP, SFU · TTL встречи"]
    end

    subgraph CH["ClickHouse — 2 shard × 2 replica, Keeper на 3 площадках"]
        QE["media_qos_events\n─────────────\nORDER BY (meeting_id, ts)\nPARTITION BY toYYYYMMDD(ts)\nшард: meeting_id · TTL 30 дней"]
    end

    subgraph CEPH["Yandex Object Storage — managed S3, класс cold, lifecycle 12 мес"]
        direction TB
        RB["recording_blobs\n─────────────\nbucket: recordings\n~2701 ТБ/год"]
        TRB["transcript_blobs\n─────────────\nbucket: transcripts\n~0.96 ТБ/год"]
        AV["avatars\n─────────────\nbucket: avatars\norigin для CDN"]
    end

    subgraph KAFKA["Kafka — 3 broker, RF=3, min.insync.replicas=2"]
        direction TB
        KJ["recording_jobs / transcription_jobs\n─────────────\npartition by meeting_id\nretention 3 дня"]
        KQ["qos_ingest / meeting_events\n─────────────\npartition by meeting_id\nretention 3 дня"]
    end

    U -->|"user_id FK"| US
    U -->|"owner_id FK"| MT
    U -->|"user_id FK"| MI
    U -->|"user_id FK"| MP
    U -->|"sender_id FK"| CM
    U -->|"avatar_url"| AV

    MT -->|"meeting_id FK"| MI
    MT -->|"meeting_id FK"| MP
    MT -->|"meeting_id FK"| CM
    MT -->|"meeting_id FK"| RT
    MT -->|"meeting_id"| LS
    MT -->|"meeting_id"| MA
    MP -->|"participant_id"| PR

    RT -->|"storage_url"| RB
    RT -->|"storage_url"| TRB
    RT -.->|"recording_finished"| KJ
    KQ -.->|"ingest"| QE
```
