# Planfit Graph DB — 기술 명세

> Neo4j 기반 5분할 운동 추천 그래프  
> `build_graph.py` · `queries.py`

---

## 목차

1. [노드 정의](#1-노드-정의)
2. [관계 정의](#2-관계-정의)
3. [그래프 스키마 다이어그램](#3-그래프-스키마-다이어그램)
4. [데이터 전처리 이슈](#4-데이터-전처리-이슈)
5. [핵심 쿼리](#5-핵심-쿼리)
6. [DB 역할 분담](#6-db-역할-분담)

---

## 1. 노드 정의

그래프는 **5종류의 노드 레이블**을 사용한다.  
`Exercise`가 핵심 노드이며, 나머지 4개는 정적 참조 노드다.

---

### 1-1. Exercise _(667개)_

5분할에 속하는 운동 노드. 전체 enriched 데이터(969개) 중 `SPLIT_MAP`에 매핑되는 운동만 생성된다.

| Property | Type | Example | Description |
|---|---|---|---|
| `id` | Integer | `1001` | 고유 식별자 (기본키) |
| `name_kor` | String | `데드리프트` | 한국어 운동명 |
| `name_eng` | String | `Deadlift` | 영문 운동명 |
| `split_day` | String | `BACK` | 분할 카테고리 코드 |
| `equipment` | String | `barbell` | 사용 기구 (raw 키) |
| `difficulty` | Integer | `2` | 난이도 숫자 (1~3) |
| `difficulty_label` | String | `intermediate` | 난이도 레이블 |
| `spine_loading` | String | `상 / 중 / 하` | 척추 부하 수준 |
| `spine_loading_level` | Integer | `1~3` | 정규화값 — 하=1, 중=2, 상=3 |
| `cal_per_min` | Float | `5.2` | 분당 칼로리 소모량 |
| `place_type` | String | `gym / home` | 운동 장소 유형 |
| `home_friendly` | String | `Y / N` | 홈트레이닝 가능 여부 |
| `target_primary` | String | `대퇴사두근` | 주 타겟 근육명 (원문 텍스트) |
| `target_secondary` | String[] | `["대둔근"]` | 부 타겟 근육명 배열 |
| `is_machine_based` | Boolean | `false` | 머신 기반 여부 (파생 필드) |
| `video_url` | String | `https://...` | 운동 영상 링크 |
| `image_url` | String | `https://...` | 운동 이미지 링크 |
| `tag` | String | `compound` | 운동 태그 |
| `description` | String | `...` | 운동 설명 |

---

### 1-2. BodyPart _(11개, 고정)_

운동이 자극하는 신체 부위. `UPPER` 6개 · `LOWER` 4개 · `CORE` 1개.

| id | name_ko | category | primary_muscles (매핑 키워드) |
|---|---|---|---|
| `bp_chest` | 가슴 | UPPER | 대흉근, 대흉근(상부/하부), 소흉근 |
| `bp_back` | 등 | UPPER | 광배근, 척추기립근, 능형근, 승모근 |
| `bp_shoulder` | 어깨 | UPPER | 삼각근, 삼각근(전/측/후면) |
| `bp_biceps` | 이두 | UPPER | 상완이두근, 이두근 |
| `bp_triceps` | 삼두 | UPPER | 삼두근, 상완삼두근 |
| `bp_forearm` | 전완근 | UPPER | 전완굴근, 전완신근, 전완근 |
| `bp_core` | 코어 | CORE | 복직근, 외/내복사근, 횡복근, 하복부 |
| `bp_quad` | 대퇴사두근 | LOWER | 대퇴사두근, 전퇴부 |
| `bp_hamstring` | 햄스트링 | LOWER | 대퇴이두근, 반건/반막양근, 햄스트링 |
| `bp_glute` | 둔근 | LOWER | 대둔근, 중둔근, 소둔근, 둔근 |
| `bp_calf` | 종아리 | LOWER | 비복근, 가자미근 |

> **매핑 로직** — `get_bp_id(target)` 함수가 `primary_muscles` 배열을 순서대로 탐색해 첫 매칭 노드를 반환한다.  
> 하체 세분류(`bp_quad`, `bp_hamstring`, `bp_glute`, `bp_calf`)가 앞에 배치되어 구체 노드가 먼저 매칭되고, 매칭 실패 시 `None`을 반환한다.

---

### 1-3. Equipment _(8개, 고정)_

| id | raw (필터 키) | type | gym_required |
|---|---|---|---|
| `eq_barbell` | `barbell` | FREE_WEIGHT | true |
| `eq_dumbbell` | `dumbbell` | FREE_WEIGHT | false |
| `eq_machine` | `machine` | MACHINE | true |
| `eq_body` | `body` | BODYWEIGHT | false |
| `eq_pullupbar` | `pull_up_bar` | BODYWEIGHT | false |
| `eq_band` | `band` | BAND | false |
| `eq_kettlebell` | `kettlebell` | FREE_WEIGHT | false |
| `eq_normal` | `normal` | OTHER | false |

---

### 1-4. SplitDay _(5개, 고정)_

| split_day | name | order | weekday |
|---|---|---|---|
| `CHEST` | 가슴 | 1 | 월 |
| `BACK` | 등 | 2 | 화 |
| `LEG` | 하체 | 3 | 수 |
| `SHOULDER` | 어깨 | 4 | 목 |
| `ARM` | 팔 | 5 | 금 |

---

### 1-5. IntensityLevel _(3개, 고정)_

| level | 누적 포함 범위 |
|---|---|
| `beginner` | beginner |
| `intermediate` | beginner + intermediate |
| `advanced` | beginner + intermediate + advanced |

> 쿼리 레이어의 `DIFFICULTY_MAP`이 이 누적 포함 로직을 구현한다.

---

## 2. 관계 정의

총 **8종류**의 방향성 있는 엣지. `Exercise → 참조 노드` 5개와 `Exercise → Exercise` 3개로 구성된다.

### 2-1. 전체 요약

| Relationship | From | To | Description |
|---|---|---|---|
| `TARGETS_PRIMARY` | Exercise | BodyPart | 주 운동 부위 연결 |
| `TARGETS_SECONDARY` | Exercise | BodyPart | 보조 운동 부위 연결 |
| `REQUIRES_EQUIPMENT` | Exercise | Equipment | 필요 기구 연결 |
| `HAS_INTENSITY` | Exercise | IntensityLevel | 난이도 분류 연결 |
| `PART_OF_SPLIT` | Exercise | SplitDay | 분할 루틴 소속 (프로퍼티 있음) |
| `SIMILAR_TO` | Exercise | Exercise | 유사 동작 운동 — **양방향** |
| `SUBSTITUTE_FOR` | Exercise | Exercise | 대체 운동 — 장소 호환 기준 |
| `PROGRESSION_OF` | Exercise | Exercise | 점진 과부하 경로 |

---

### 2-2. PART_OF_SPLIT 프로퍼티

운동이 분할 요일에 소속될 때 세트/반복/시간 정보를 엣지에 저장한다.

| Property | Default | Type |
|---|---|---|
| `sets` | `3` | Integer |
| `reps` | `"10"` | String |
| `duration_min` | `15` | Integer |

---

### 2-3. SUBSTITUTE_FOR 프로퍼티

`exercise_edges.json` 기반. 헬스장 ↔ 홈트레이닝 장소 호환 여부를 엣지에 저장한다.

| Property | Type | Description |
|---|---|---|
| `from_place` | String | 출발 운동의 장소 유형 (`gym` / `home`) |
| `to_place` | String | 도착 운동의 장소 유형 (`gym` / `home`) |
| `same_place` | Boolean | 동일 장소 대체 여부 |

---

### 2-4. SIMILAR_TO 프로퍼티

`related_exercises` 필드를 정규식으로 파싱해 **양방향** 생성된다.

| Property | Value | Description |
|---|---|---|
| `score` | `1.0` | 유사도 점수 (현재 고정값) |
| `basis` | `"planfit_related"` | 유사도 산출 근거 |

---

### 2-5. PROGRESSION_OF 프로퍼티

동일 `split_day` + 동일 주 타겟 근육 그룹 내에서 `difficulty` 순으로 자동 연결된다.  
`beginner → intermediate → advanced` 방향.

| Property | Value | Description |
|---|---|---|
| `type` | `"LOAD"` | 과부하 유형 (현재 LOAD 고정) |

---

## 3. 그래프 스키마 다이어그램

```
┌─────────────────────────────────────────────────────────────────────┐
│                         참조 노드 (정적)                              │
│                                                                     │
│   ┌────────────┐   ┌─────────────┐   ┌────────────────┐            │
│   │  BodyPart  │   │  Equipment  │   │ IntensityLevel │            │
│   │────────────│   │─────────────│   │────────────────│            │
│   │ id         │   │ id          │   │ level          │            │
│   │ name_ko    │   │ raw         │   │ (3개 고정)     │            │
│   │ category   │   │ type        │   └───────▲────────┘            │
│   │ (11개 고정)│   │ gym_required│           │ HAS_INTENSITY        │
│   └──────▲─────┘   └──────▲──────┘           │                     │
│          │                │                  │                     │
│  TARGETS_│PRIMARY         │ REQUIRES_EQUIPMENT│                    │
│  TARGETS_SECONDARY        │                  │                     │
└──────────┼────────────────┼──────────────────┼─────────────────────┘
           │                │                  │
           │         ┌──────┴──────────────────┴──────┐
           │         │           Exercise              │
           └─────────│─────────────────────────────────│
                     │  id              split_day       │
                     │  name_kor        equipment       │
                     │  difficulty      spine_loading   │
                     │  cal_per_min     place_type      │
                     │  home_friendly   target_primary  │
                     │  (667개)                         │
                     └───┬───────┬────────────┬────────┘
                         │       │            │
              ┌──────────┘       │            └──────────────┐
              │                  │                           │
              │ SIMILAR_TO       │ SUBSTITUTE_FOR            │ PROGRESSION_OF
              │ (양방향)         │ (장소 호환)               │ (난이도 순)
              ▼                  ▼                           ▼
           Exercise           Exercise                   Exercise
                     │
                     │ PART_OF_SPLIT
                     │ {sets, reps, duration_min}
                     ▼
              ┌─────────────┐
              │  SplitDay   │
              │─────────────│
              │ split_day   │
              │ weekday     │
              │ order       │
              │ (5개 고정)  │
              └─────────────┘
```

### 노드 수 요약

| Label | 수량 | 성격 |
|---|---|---|
| Exercise | 667개 | 동적 (빌드 시 생성) |
| BodyPart | 11개 | 정적 |
| Equipment | 8개 | 정적 |
| IntensityLevel | 3개 | 정적 |
| SplitDay | 5개 | 정적 |

---

## 4. 데이터 전처리 이슈

빌드 파이프라인에서 발생하는 실질적인 전처리 문제들을 정리한다.

---

### 4-1. 카테고리 → split_day 통합 (`SPLIT_MAP`)

원본 JSON의 `category` 필드가 한국어 세분류로 되어 있어 5분할 코드로 변환이 필요하다.  
특히 **팔 운동 3개 카테고리가 `ARM`으로 통합**된다.

```python
SPLIT_MAP = {
    "가슴": "CHEST",
    "등":   "BACK",
    "하체": "LEG",
    "어깨": "SHOULDER",
    "이두": "ARM",   # ← 3개가
    "삼두": "ARM",   # ← 하나로
    "전완근": "ARM", # ← 통합
}
```

> `SPLIT_MAP`에 없는 카테고리(복근, 유산소 등)는 필터링되어 **969개 → 667개**로 축소된다.

---

### 4-2. cal_per_min 누락 처리 (`CAL_BASE` 폴백)

`estimated_cal_per_min` 값이 없는 운동에 분할별 기본값을 대입한다.  
하체가 높고 팔이 낮은 것은 실제 운동 강도를 반영한다.

```python
CAL_BASE = {
    "CHEST":    5.2,
    "BACK":     5.2,
    "LEG":      6.4,  # 하체 최고
    "SHOULDER": 4.7,
    "ARM":      4.1,  # 팔 최저
}
```

---

### 4-3. spine_loading 정규화 (`SPINE_LOADING_LEVEL`)

원문의 한국어 값(`상/중/하`)을 수치로 변환해 `spine_loading_level` 필드를 추가한다.  
원본 필드(`spine_loading`)는 변경하지 않고 **병행 저장**한다.

```python
SPINE_LOADING_LEVEL = {"하": 1, "중": 2, "상": 3}
```

---

### 4-4. BodyPart 매핑 실패 문제

`get_bp_id(target)` 함수는 `target_primary` 문자열 안에 키워드가 포함되어 있는지를 탐색한다.  
원본 데이터의 근육명 표기가 일치하지 않으면 `None`을 반환하고 **엣지가 생성되지 않는다**.

```python
def get_bp_id(target: str) -> str | None:
    for bp in BODY_PARTS:
        if any(m in target for m in bp["primary_muscles"]):
            return bp["id"]
    return None  # 매핑 실패 → TARGETS_PRIMARY 엣지 없음
```

> **주의** — 매핑 실패 시 해당 운동은 `TARGETS_PRIMARY` 엣지 없이 생성된다.  
> 추후 매핑 커버리지 통계 확인 및 키워드 보완이 필요하다.

---

### 4-5. related_exercises 파싱 (`SIMILAR_TO` 소스)

`related_exercises` 필드가 `"1234(운동명), 5678(운동명)"` 형태의 문자열이라  
정규식으로 ID만 추출한 뒤 양방향 `SIMILAR_TO` 엣지를 생성한다.

```python
def parse_related(s: str) -> list[int]:
    return [int(m) for m in re.findall(r'(\d+)\(', s or "")]
```

> 존재하지 않는 ID 또는 5분할 외 ID는 `all_ids` 집합으로 검증해 제외한다.  
> 자기 자신(`rid != d["id"]`)도 명시적으로 제외한다.

---

### 4-6. SUBSTITUTE_FOR 엣지 5분할 필터링

`exercise_edges.json`의 원본 엣지(3,170개) 중 **출발·도착 노드가 모두 5분할 내 운동인 엣지만** 생성한다.

```python
five_edges = [
    e for e in edges
    if e["from"] in valid_ids and e["to"] in valid_ids
]
```

> 5분할 외 운동(복근, 유산소 등)을 참조하는 엣지는 자동으로 제거된다.

---

### 4-7. PROGRESSION_OF 엣지 생성 조건

동일 `split_day` + 동일 `target_primary` BodyPart ID를 기준으로 그룹화한 뒤,  
`difficulty_label`이 **다른** 인접 운동 사이에만 엣지를 생성한다.  
같은 난이도 운동끼리는 연결하지 않는다.

```python
if a.get("difficulty_label") != b.get("difficulty_label"):
    # PROGRESSION_OF 엣지 생성
```

---

### 4-8. Docker / 로컬 환경 Neo4j URI 자동 전환

Docker 컨테이너 외부에서 실행 시 `neo4j` 호스트명이 조회되지 않으면  
`localhost`로 자동 폴백한다.

```python
try:
    socket.getaddrinfo(parsed.hostname, parsed.port or 7687)
except socket.gaierror:
    if parsed.hostname == "neo4j":
        NEO4J_URI = parsed._replace(netloc="localhost:...").geturl()
```

---

## 5. 핵심 쿼리

`GraphQuery` 클래스가 제공하는 주요 메서드와 실제 Cypher를 정리한다.

---

### 5-1. 분할별 운동 조회 — `get_exercises_by_split()`

사용자의 분할·장비·난이도·척추 조건을 조합해 운동 목록을 반환하는 **가장 핵심적인 쿼리**.

```python
gq.get_exercises_by_split(
    split_day = "CHEST",
    spine     = "mid",      # "all" | "mid" | "low"
    equip     = ["dumbbell", "body"],
    level     = "intermediate",
    limit     = 5,
)
```

```cypher
MATCH (e:Exercise {split_day: $split_day})
WHERE e.spine_loading    IN $spine   -- SPINE_MAP 변환값
  AND e.equipment        IN $equip
  AND e.difficulty_label IN $diff    -- DIFFICULTY_MAP 누적 포함
OPTIONAL MATCH (e)-[:TARGETS_PRIMARY]->(primary:BodyPart)
OPTIONAL MATCH (e)-[:TARGETS_SECONDARY]->(secondary:BodyPart)
RETURN e.*, collect(DISTINCT primary.id), collect(DISTINCT secondary.id)
ORDER BY e.difficulty ASC, e.cal_per_min DESC
LIMIT $limit
```

**필터 매핑 규칙**

| 파라미터 | 입력 | 실제 필터값 |
|---|---|---|
| `spine = "all"` | → | `["상", "중", "하"]` |
| `spine = "mid"` | → | `["중", "하"]` |
| `spine = "low"` | → | `["하"]` |
| `level = "intermediate"` | → | `["beginner", "intermediate"]` |
| `level = "advanced"` | → | `["beginner", "intermediate", "advanced"]` |

---

### 5-2. 대체 운동 조회 — `get_substitutes()`

특정 운동의 `SUBSTITUTE_FOR` 엣지를 탐색해 대체 운동 목록을 반환한다.  
`same_place_only=True`로 설정하면 헬스장→헬스장 또는 홈→홈 대체만 조회한다.

```python
gq.get_substitutes(
    exercise_id     = 1001,
    same_place_only = False,
    spine           = "low",
)
```

```cypher
MATCH (:Exercise {id: $id})-[r:SUBSTITUTE_FOR]->(sub:Exercise)
WHERE sub.spine_loading IN $spine
  AND (NOT $same_place OR r.same_place = true)
RETURN sub.id, sub.name_kor, sub.equipment, sub.spine_loading,
       r.from_place, r.to_place, r.same_place
ORDER BY sub.spine_loading ASC, sub.cal_per_min DESC
LIMIT 10
```

---

### 5-3. 1-hop 관계 탐색 — `get_related_exercises()`

`SUBSTITUTE_FOR`, `SIMILAR_TO`, `PROGRESSION_OF` 중 하나를 지정해  
여러 운동의 1-hop 이웃을 한 번에 조회한다. 관계 타입은 화이트리스트로 검증된다.

```python
gq.get_related_exercises(
    exercise_ids      = [1001, 1002],
    relationship_type = "SIMILAR_TO",
    split_day         = "BACK",
    spine             = "all",
    level             = "intermediate",
    limit             = 20,
)
```

```cypher
MATCH (source:Exercise)-[r:SIMILAR_TO]->(related:Exercise {split_day: $split_day})
WHERE source.id IN $exercise_ids
  AND related.spine_loading    IN $spine
  AND related.equipment        IN $equip
  AND related.difficulty_label IN $diff
RETURN source.id AS relation_source_id,
       type(r)   AS relation_type,
       coalesce(r.score, 0.0) AS relation_score,
       related.*
ORDER BY relation_score DESC, related.difficulty ASC
LIMIT $limit
```

> 허용 관계 타입: `SUBSTITUTE_FOR` | `SIMILAR_TO` | `PROGRESSION_OF`  
> 그 외 값이 들어오면 빈 리스트 반환 (SQL Injection 방지 목적)

---

### 5-4. 점진 과부하 경로 — `get_progression()`

특정 운동에서 출발해 `PROGRESSION_OF` 엣지를 최대 `steps`번 따라가며  
난이도 순 경로를 배열로 반환한다.

```python
gq.get_progression(exercise_id=1001, steps=3)
# → [{"id": 1001, "difficulty_label": "beginner"}, {"id": 1050, "difficulty_label": "intermediate"}, ...]
```

```cypher
MATCH path = (start:Exercise {id: $id})
             -[:PROGRESSION_OF*1..$steps]->(adv:Exercise)
RETURN [n IN nodes(path) | {
    id: n.id, name_kor: n.name_kor,
    difficulty: n.difficulty, difficulty_label: n.difficulty_label
}] AS chain
LIMIT 1
```

---

### 5-5. 척추 안전 운동 — `get_spine_safe()`

재활·부상 방지 목적. `spine_loading = "하"` 운동만 조회한다.

```python
gq.get_spine_safe(split_day="LEG", spine="low")
```

```cypher
MATCH (e:Exercise {split_day: $split_day})
WHERE e.spine_loading IN $spine
RETURN e.id, e.name_kor, e.equipment, e.spine_loading,
       e.home_friendly, e.cal_per_min, e.difficulty_label
ORDER BY e.cal_per_min DESC
```

---

### 5-6. 주간 칼로리 합산 — `get_weekly_calories()`

분할별 예상 칼로리를 체중으로 보정해 반환한다. 기준 체중은 70 kg.

```python
gq.get_weekly_calories(weight=80.0)
```

```cypher
MATCH (e:Exercise)-[rel:PART_OF_SPLIT]->(s:SplitDay)
RETURN s.split_day AS split, s.name, s.order, s.weekday,
       round(SUM(e.cal_per_min * ($weight / 70.0) * rel.duration_min)) AS kcal
ORDER BY s.order
```

---

### 5-7. 분할 통계 — `get_split_stats()`

분할별 운동 수, 평균 칼로리, 척추 부하 분포를 한 번에 조회한다.  
주로 데이터 검증 및 대시보드 용도.

```cypher
MATCH (e:Exercise)
WHERE e.split_day IS NOT NULL
RETURN e.split_day AS split,
       count(e)    AS total,
       round(avg(e.cal_per_min) * 10) / 10 AS avg_cal,
       sum(CASE WHEN e.spine_loading = '상' THEN 1 ELSE 0 END) AS spine_high,
       sum(CASE WHEN e.spine_loading = '중' THEN 1 ELSE 0 END) AS spine_mid,
       sum(CASE WHEN e.spine_loading = '하' THEN 1 ELSE 0 END) AS spine_low
ORDER BY split
```

---

## 6. DB 역할 분담

그래프 DB(Neo4j)가 담당하는 역할과 담당하지 않는 영역을 명확히 구분한다.

---

### 6-1. Graph DB가 담당하는 것

| 역할 | 구체적 내용 |
|---|---|
| **운동 메타데이터 저장소** | Exercise 노드의 모든 속성값 (이름, 칼로리, 척추 부하, 영상 URL 등) |
| **조건 기반 운동 필터링** | split_day + equipment + spine_loading + difficulty_label 복합 조건 조회 |
| **운동 간 관계 탐색** | SUBSTITUTE_FOR / SIMILAR_TO / PROGRESSION_OF 1-hop 이웃 탐색 |
| **점진 과부하 경로** | difficulty 순 chain 경로 (`*1..N` 가변 깊이 탐색) |
| **분할별 칼로리 집계** | PART_OF_SPLIT 엣지의 duration_min 기반 체중 보정 칼로리 계산 |
| **척추 안전 운동 필터** | spine_loading 기반 재활·부상 방지 운동 조회 |

---

### 6-2. Graph DB가 담당하지 않는 것

| 역할 | 담당 주체 |
|---|---|
| 사용자 계정·인증 정보 | RDBMS (예: PostgreSQL) |
| 사용자 운동 기록·히스토리 | RDBMS |
| 루틴 저장·즐겨찾기 | RDBMS |
| 사용자 맞춤 추천 로직 (ML) | 애플리케이션 레이어 |
| 운동 데이터 원본 관리 | JSON 파일 → 빌드 스크립트로 주입 |
| 실시간 트랜잭션 처리 | RDBMS |

---

### 6-3. 빌드 파이프라인 역할 (`build_graph.py`)

Graph DB는 **읽기 전용에 가까운** 구조로 운영된다.  
데이터 변경은 빌드 스크립트 재실행으로만 이루어진다.

```
[JSON 원본 데이터]
  planfit_exercises_enriched.json  (969개)
  exercise_edges.json              (3,170개)
          │
          ▼
[build_graph.py — GraphBuilder]
  0. DB 초기화    → DETACH DELETE 전체
  1. 인덱스 생성  → IF NOT EXISTS
  2. 정적 노드    → BodyPart / Equipment / IntensityLevel / SplitDay
  3. Exercise     → 667개 MERGE
  4~7. 참조 엣지  → TARGETS / REQUIRES / HAS_INTENSITY / PART_OF_SPLIT
  8~10. 관계 엣지 → SIMILAR_TO / SUBSTITUTE_FOR / PROGRESSION_OF
          │
          ▼
[Neo4j Graph DB]  ← 애플리케이션이 읽기 전용으로 사용
          │
          ▼
[queries.py — GraphQuery]
  운동 추천 API 레이어
```

---

### 6-4. 인덱스 전략

| Index | Property | 목적 |
|---|---|---|
| `ex_id` | `Exercise.id` | 단건 조회 기본키 |
| `ex_split` | `Exercise.split_day` | 분할 필터링 (가장 빈번) |
| `ex_spine` | `Exercise.spine_loading` | 척추 조건 필터 |
| `ex_equip` | `Exercise.equipment` | 기구 조건 필터 |
| `ex_place` | `Exercise.place_type` | 장소 조건 필터 |
| `ex_spine_level` | `Exercise.spine_loading_level` | 정규화 수치 정렬 |
| `bp_id` | `BodyPart.id` | 엣지 생성 시 MATCH |
| `eq_id` | `Equipment.id` | 엣지 생성 시 MATCH |
| `il_level` | `IntensityLevel.level` | 난이도 노드 조회 |
| `sd_split` | `SplitDay.split_day` | 분할 노드 조회 |
