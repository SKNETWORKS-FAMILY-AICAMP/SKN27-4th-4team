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

## Neo4j GraphDB

```mermaid
flowchart LR
    Exercise["Exercise<br/>운동"] -->|TARGETS_PRIMARY| BodyPart["BodyPart<br/>타겟 부위"]
    Exercise -->|TARGETS_SECONDARY| BodyPart
    Exercise -->|REQUIRES_EQUIPMENT| Equipment["Equipment<br/>사용 장비"]
    Exercise -->|HAS_INTENSITY| Intensity["IntensityLevel<br/>난이도"]
    Exercise -->|PART_OF_SPLIT| SplitDay["SplitDay<br/>분할 루틴"]
    Exercise -->|SIMILAR_TO| Similar["Exercise<br/>유사 운동"]
    Exercise -->|SUBSTITUTE_FOR| Substitute["Exercise<br/>대체 운동"]
    Exercise -->|PROGRESSION_OF| Progression["Exercise<br/>난이도 progression"]
```

### 노드

| 노드 | 역할 |
| --- | --- |
| Exercise | 루틴 추천 대상 운동 |
| BodyPart | 주/보조 타겟 부위 |
| Equipment | 운동에 필요한 장비 |
| IntensityLevel | 초급/중급/고급 난이도 |
| SplitDay | 가슴, 등, 하체, 어깨, 팔 분할 |

### 관계

| 관계 | 의미 |
| --- | --- |
| `TARGETS_PRIMARY` | 운동의 주 타겟 부위 |
| `TARGETS_SECONDARY` | 운동의 보조 타겟 부위 |
| `REQUIRES_EQUIPMENT` | 운동 수행에 필요한 장비 |
| `HAS_INTENSITY` | 운동 난이도 |
| `PART_OF_SPLIT` | 5분할 루틴의 어느 요일/부위에 속하는지 |
| `SIMILAR_TO` | 유사한 자극 또는 동작을 가진 운동 |
| `SUBSTITUTE_FOR` | 통증, 장비, 장소 조건에 따라 대체 가능한 운동 |
| `PROGRESSION_OF` | 난이도나 숙련도 progression 관계 |
