# ERD 및 그래프 데이터 모델

## PostgreSQL ERD

```mermaid
erDiagram
    users ||--o{ user_pain_logs : logs
    users ||--o{ chat_sessions : owns
    users ||--o{ weekly_schedulers : owns

    chat_sessions ||--o{ chat_messages : contains
    chat_sessions ||--o{ weekly_schedulers : converts_to

    weekly_schedulers ||--o{ daily_routines : has
    exercises ||--o{ daily_routines : assigned

    exercises ||--o{ exercise_muscles : uses
    muscles ||--o{ exercise_muscles : mapped
    muscles ||--o{ muscle_relations : source
    muscles ||--o{ muscle_relations : target

    users {
        int user_id PK
        varchar email UK
        varchar password_hash
        varchar nickname
        enum fitness_level
        timestamp created_at
    }

    user_pain_logs {
        int pain_id PK
        int user_id FK
        uuid device_uuid
        varchar body_part
        smallint severity
        timestamp logged_at
    }

    exercises {
        int exercise_id PK
        varchar name_kor
        varchar name_eng
        varchar category
        varchar target_primary
        jsonb target_secondary
        varchar equipment
        enum difficulty
        int default_duration_min
        varchar video_url
        text guide
        text caution
        vector embedding
    }

    chat_sessions {
        int session_id PK
        int user_id FK
        uuid device_uuid
        varchar title
        jsonb extracted_conditions
        boolean is_converted
        timestamp created_at
    }

    chat_messages {
        int message_id PK
        int session_id FK
        enum sender
        text content
        timestamp created_at
    }

    weekly_schedulers {
        int scheduler_id PK
        int user_id FK
        uuid device_uuid
        int session_id FK
        int year
        int week_number
        varchar split_style
        varchar goal
        smallint session_min
        jsonb pain_parts
        jsonb work_days
        text weekly_review
    }

    daily_routines {
        int daily_routine_id PK
        int scheduler_id FK
        int exercise_id FK
        date scheduled_date
        varchar day_of_week
        float routine_order
        int recommended_sets
        int recommended_reps
        int target_sets
        int target_reps
        boolean is_custom_added
        boolean is_completed
        text daily_issue
    }

    muscles {
        int muscle_id PK
        varchar name_kor UK
        varchar name_eng
        varchar muscle_group
    }

    muscle_relations {
        int relation_id PK
        int source_muscle FK
        int target_muscle FK
        varchar relation_type
    }

    exercise_muscles {
        int exercise_id FK
        int muscle_id FK
        varchar role
    }
```

## Neo4j 그래프 스키마

```mermaid
flowchart LR
    Exercise["Exercise"] -->|TARGETS_PRIMARY| BodyPart["BodyPart"]
    Exercise -->|TARGETS_SECONDARY| BodyPart
    Exercise -->|REQUIRES_EQUIPMENT| Equipment["Equipment"]
    Exercise -->|HAS_INTENSITY| Intensity["IntensityLevel"]
    Exercise -->|PART_OF_SPLIT| SplitDay["SplitDay"]
    Exercise -->|SIMILAR_TO| Exercise
    Exercise -->|SUBSTITUTE_FOR| Exercise
    Exercise -->|PROGRESSION_OF| Exercise
```

| Label | 역할 |
| --- | --- |
| Exercise | 5분할 추천 대상 운동 |
| BodyPart | 주/보조 타겟 부위 |
| Equipment | 장비 필터 |
| IntensityLevel | 초급/중급/고급 난이도 |
| SplitDay | 가슴, 등, 하체, 어깨, 팔 분할 |

상세 그래프 명세는 [planfit_graphdb_spec.md](./planfit_graphdb_spec.md)를 참고합니다.
