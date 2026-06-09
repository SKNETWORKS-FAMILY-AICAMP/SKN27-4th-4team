# 피트니스 RAG · 챗봇 — 기술 명세

> 운동 추천·설명 대화형 챗봇의 RAG 검색 및 LangGraph 대화 엔진 명세.
> 스택: **PostgreSQL + pgvector**(벡터 검색) · **LangGraph**(대화 그래프) · **OpenAI**(임베딩·LLM) · **Django**(API/SSE).

## 목차

1. 데이터 스키마
2. RAG 인덱싱 파이프라인
3. LangGraph 구성
4. 구현 이슈 & 해결
5. 핵심 함수
6. 역할 분담
7. 보완 필요 데이터

---

## 1. 데이터 스키마

### 1-1. `exercises` — 운동 마스터 _(약 920개, 중복 id 3개 제거)_

| 컬럼 | 타입 | 설명 |
|------|------|------|
| `exercise_id` | INT PK | 운동 식별자 |
| `name_kor` / `name_eng` | VARCHAR | 운동명 |
| `category` | VARCHAR | 분류 (하체·코어·등·어깨·가슴·스트레칭·유산소·이두·삼두·전완근) |
| `target_primary` | VARCHAR | 주 타겟 근육 |
| `target_secondary` | JSONB | 보조 타겟 근육 배열 |
| `equipment` | VARCHAR | 장비 (`body` 등) |
| `difficulty` | ENUM | `초급` / `중급` / `고급` (`fitness_level_enum`) |
| `default_duration_min` | INT | 기본 운동 시간(분) |
| `description`, `starting_position`, `movement`, `breathing`, `related_exercises`, `guide`, `caution` | TEXT | 동작·설명 본문 |
| `embedding` | **vector(1536)** | OpenAI `text-embedding-3-small` 벡터 |

**인덱스**: `embedding`에 HNSW (`vector_cosine_ops`), `category`·`difficulty` B-tree.

### 1-2. 대화 테이블

| 테이블 | 주요 컬럼 | 역할 |
|--------|-----------|------|
| `chat_sessions` | `session_id` PK, `device_uuid`, `title` | 대화 세션 |
| `chat_messages` | `message_id` PK, `session_id` FK, `sender`(`user`/`bot`), `content`, `created_at` | 메시지 영속 저장 |

### 1-3. 근육 그래프 _(부상 분기용)_

| 테이블 | 역할 |
|--------|------|
| `muscles` | 근육 마스터 (`name_kor`, `muscle_group`) |
| `exercise_muscles` | 운동–근육 브릿지 (부상 부위 근육을 쓰는 운동 정밀 제외에 사용) |

---

## 2. RAG 인덱싱 파이프라인 _(오프라인)_

| 단계 | 모듈 | 입력 → 출력 | 비고 |
|------|------|-------------|------|
| 로드 | `loader.py` | DB 행 → `List[Document]` | 행을 자연어 `page_content` + `metadata`로 변환 |
| 분할 | `splitter.py` | `Document` → 청크 | 50자 미만 필터(무의미 청크 방지). **현재 런타임 임베딩에는 미사용** |
| 임베딩 | `embedding.py` | `page_content` → `vector(1536)` | `text-embedding-3-small` 일괄 변환, **운동 1건=벡터 1개**(청킹 X) |
| 저장 | `pgvectordb.py` | 벡터 → `exercises.embedding` | `UPDATE ... SET embedding=%s::vector WHERE exercise_id=%s`, 중복 id 가드 |

> **설계 결정**: `PGVector.from_documents`(별도 vectorstore 테이블 생성) 대신, 운동 테이블에 직접 만든 `embedding` 컬럼을 재사용해 유사도 검색을 직접 구현.

---

## 3. LangGraph 구성

### 3-1. 상태 — `RAGState`

| 필드 | 타입 | 설명 |
|------|------|------|
| `messages` | list (`add_messages`) | 대화 히스토리(자동 누적) |
| `session_id` | int | 세션 ID (recall이 전체 히스토리 조회) |
| `question` | str | 사용자 질문 원문 |
| `search_query` | str | 히스토리 반영해 재작성한 검색용 질문 |
| `query_type` | str | 분류 결과 |
| `retrieved_docs` | List[Document] | 검색 문서 |
| `context` / `answer` / `sources` | str / str / List[str] | 컨텍스트·최종 답변·참고 운동명 |

### 3-2. 노드 정의

| 노드 | 역할 | 출력 |
|------|------|------|
| `classify` | 질문 유형 분류 (히스토리 포함, temp=0) | `query_type` |
| `recall` | **검색 없이** 전체 대화 기록만으로 답 | `answer` |
| `out_of_scope` | 운동 무관 질문 거부(고정 메시지) | `answer` |
| `retrieve_general` | 카테고리·장비·운동명 힌트 추출 → 벡터검색(단일 운동이면 정확검색 위임) | `retrieved_docs` |
| `retrieve_specific` | 운동명 추출 → 키워드 정확검색(실패 시 벡터 폴백) | `retrieved_docs` |
| `retrieve_injury` | 부상 부위→근육 매핑 제외 + 고급 난이도 제외 + rerank | `retrieved_docs` |
| `generate` | 운동 데이터 + 히스토리로 답변 스트리밍 생성 | `answer` |

### 3-3. 그래프 흐름

```
START → classify ─┬─ recall ─────────────→ END
                  ├─ out_of_scope ───────→ END
                  ├─ retrieve_general ─┐
                  ├─ retrieve_specific ─┼→ generate → END
                  └─ retrieve_injury ──┘
```
- `classify`에서 `query_type`에 따라 conditional edge로 분기.
- 검색이 필요한 3개 노드는 검색 직전 `_rewrite_query`로 참조 표현을 실제 운동명으로 치환.

### 3-4. 질문 유형 _(constants `QUERY_TYPES`)_

| 유형 | 라우팅 | 정의 |
|------|--------|------|
| `specific` | `retrieve_specific` | 특정 운동명이 포함된 질문 |
| `general` | `retrieve_general` | 추천·루틴, 또는 이전 대화 운동의 정보를 묻는 참조 질문 |
| `injury` | `retrieve_injury` | 통증·부상·대체 운동 |
| `recall` | `recall` | 대화 내용 자체(이름·순서)를 묻는 질문 |
| `out_of_scope` | `out_of_scope` | 운동 무관 질문 |

### 3-5. 주요 설정값 _(constants)_

| 키 | 값 |
|----|----|
| LLM / 임베딩 | `gpt-4o-mini` / `text-embedding-3-small` |
| `LLM_TEMPERATURE` (생성/분류) | `0.7` / `0` |
| `RETRIEVE_LIMIT` / `RETRIEVE_SPECIFIC_LIMIT` | `8` / `3` |
| `MAX_HISTORY_TURNS` | `15` (최근 30개 메시지) |
| rerank | `Qwen/Qwen3-Reranker-0.6B`, `ENABLE_RERANK` 기본 `False`, `TOP_N=5` |

---

## 4. 구현 이슈 & 해결

### 4-1. 임베딩 청크 덮어쓰기

스키마는 운동당 벡터 1개인데 청킹 시 같은 `exercise_id`에 여러 청크가 UPDATE되어 마지막 청크만 남았다. → 청킹 제거, 운동 문서 전체를 1벡터로 임베딩 + 중복 id 가드.

### 4-2. 거리 연산자 ↔ 인덱스 불일치

HNSW는 cosine(`vector_cosine_ops`)인데 쿼리는 L2(`<->`)를 써서 인덱스 미적용 위험. → 쿼리를 cosine(`<=>`)으로 통일.

### 4-3. 회상 질문 검색 오염 _(핵심)_

| 항목 | 내용 |
|------|------|
| 문제 | "처음 물어본 운동이 뭐야?"에 실제 첫 운동(데드리프트)이 아니라 변형(스태거드 데드리프트) 응답 |
| 원인 | 회상 질문도 벡터검색을 거쳐 유사 변형이 컨텍스트를 오염, generate가 검색 운동명을 출력 |
| 해결 | `recall` 유형·노드 신설 → 검색 없이 전체 대화 기록만으로 답 (운동 데이터 미주입) |
| 효과 | 회상 질문 정확도 확보, 긴 대화에서도 유지, 불필요한 검색 제거로 지연·비용 감소 |

### 4-4. 참조+속성 결합 질문

"처음 물어본 운동의 호흡법은?" → 검색 직전 `_rewrite_query`로 "데드리프트 호흡법"으로 치환 후 검색.

### 4-5. 변형 운동 오선택

`keyword_search`를 **정확 일치 1순위 + 짧은(기본형) 이름 우선** 정렬로 보강, 단일 운동 질문은 의미검색 대신 이름검색으로 위임.

### 4-6. general 카테고리 ↔ DB 불일치

힌트 프롬프트가 DB에 없는 `팔`·`전신`을 가정 → 실제 카테고리(이두·삼두·전완근·유산소 등)에 맞추고 다중 카테고리 OR 지원.

---

## 5. 핵심 함수

### 5-1. `vector_search()` — 의미 검색

```python
def vector_search(query, limit=5, where_clause="", params=None) -> list:
    query_vector = _get_embedding_model().embed_query(query)
    sql = SELECT_COLUMNS + (f" WHERE {where_clause}" if where_clause else "")
    sql += " ORDER BY embedding <=> %s::vector LIMIT %s"   # cosine, 인덱스 사용
```

### 5-2. `keyword_search()` — 정확 검색 (정확 일치 우선)

```python
WHERE REPLACE(name_kor,' ','') ILIKE REPLACE(%s,' ','')
ORDER BY
    CASE WHEN REPLACE(name_kor,' ','') ILIKE REPLACE(%s,' ','') THEN 0 ELSE 1 END,
    LENGTH(name_kor)
```

### 5-3. `retrieve_injury()` — 부상 제외 검색

- `get_muscles_by_body_part()`로 부위 근육 조회 → 없으면 `JOINT_TO_MUSCLE_HINTS`(무릎→대퇴사두근·햄스트링 등)로 매핑
- `exercise_muscles` 브릿지로 해당 근육 운동 제외 + `difficulty <> '고급'`
- 결과를 CrossEncoder로 rerank → 상위 N

### 5-4. `_rewrite_query()` / `recall()` — 멀티턴

- `_rewrite_query`: 히스토리가 있을 때만 "그거/처음 물어본 운동" 등을 실제 운동명으로 재작성(temp=0)
- `recall`: `load_history(session_id)`로 **전체** 대화를 받아 검색 없이 답 생성

### 5-5. `generate()` + 스트리밍

- 답변 규칙: 운동 데이터 기반, 데이터 외 안전성 단정 금지, 부상 질문 시 안전 가이드 주입
- Django `StreamingHttpResponse`(SSE), `graph.stream(stream_mode="messages")`에서 `generate`·`recall` 노드 토큰만 전달

---

## 6. 역할 분담

**RAG/벡터검색이 담당하는 것**
- 운동 동작·자세·호흡법·주의사항 등 운동 정보 검색 및 근거 제공
- 질문 유형별 검색 전략(추천=벡터, 특정=키워드, 부상=제외+rerank)
- 대화 맥락 기반 질문 재작성 및 회상 응답

**담당하지 않는 것**
- 운동 데이터 원천 적재·정제(인덱싱 파이프라인 영역)
- 주간 스케줄/루틴 영속화(스케줄러·DailyRoutine 영역)
- 그래프 DB(Neo4j) 기반 관계 탐색(별도 시스템)

---

## 7. 보완 필요 데이터 _(작성 시 채워야 할 항목)_

> 아래는 코드만으로 확정하기 어려워, 명세 완성 전 직접 확인·기입이 필요한 값들입니다.

| 항목 | 현재 파악값 / 메모 | 확인 필요 |
|------|--------------------|-----------|
| 운동 데이터 건수 | 적재 로그상 **920개**(중복 3 제거) | 최종본 기준 재확인 |
| 카테고리 전체 | 하체·코어·등·어깨·가슴·스트레칭·유산소·이두·삼두·전완근(10종) | 변동 여부 |
| 모델 버전 핀 | `gpt-4o-mini`, `text-embedding-3-small` (스냅샷 미지정) | 운영 핀 날짜 명시 |
| rerank 운영값 | `ENABLE_RERANK` 기본 `False` | 운영 환경 실제 on/off |
| 평가 수치 | RAGAS·검색지표(Hit Rate/MRR) 측정함 | 최종 수치 표 삽입 |
| `muscles`/`exercise_muscles` 스키마 | `schema_muscles.sql` 별도 | 컬럼 상세 기입 |
| API 엔드포인트 | `/sessions`, `/messages`(SSE) 등 | 요청/응답 포맷 명세 추가 |
| 임베딩 재생성 절차 | 백엔드 기동 시 `pgvectordb.py` 자동 실행 | 운영 재인덱싱 주기 |
