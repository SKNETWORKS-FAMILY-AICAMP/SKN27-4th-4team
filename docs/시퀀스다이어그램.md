# 시퀀스 다이어그램

## AI 챗봇 RAG 시퀀스

```mermaid
sequenceDiagram
    actor User as 사용자
    participant FE as Frontend
    participant API as Django API
    participant Graph as LangGraph RAG
    participant DB as PostgreSQL + pgvector
    participant LLM as LLM

    User->>FE: 질문 입력
    FE->>API: POST /api/sessions/{id}/messages/
    API->>DB: 사용자 메시지 저장
    API->>Graph: 질문 + 세션 히스토리 전달
    Graph->>Graph: 질문 유형 분류

    alt 운동 무관
        Graph-->>API: 범위 안내 응답
    else 회상 질문
        Graph->>DB: 대화 기록 조회
        Graph-->>API: 기록 기반 응답
    else 운동 데이터 필요
        Graph->>DB: 벡터/키워드/부상 필터 검색
        DB-->>Graph: 관련 운동 문서
        Graph->>LLM: 검색 결과 기반 답변 생성
        LLM-->>Graph: 답변 토큰
    end

    loop 스트리밍
        Graph-->>API: 답변 조각
        API-->>FE: SSE 전송
        FE-->>User: 실시간 표시
    end

    API->>DB: 봇 메시지 저장
```

## 루틴 추천 및 피드백 시퀀스

```mermaid
sequenceDiagram
    actor User as 사용자
    participant FE as Frontend
    participant API as Django API
    participant Agent as LangGraph Routine Agent
    participant Neo4j as Neo4j GraphDB
    participant LLM as LLM

    User->>FE: 설문 입력
    FE->>API: POST /api/routines/recommend/
    API->>Agent: 추천 워크플로우 시작
    Agent->>Agent: 사용자 프로필/추천 조건 생성
    Agent->>Neo4j: 조건 기반 운동 후보 검색
    Neo4j-->>Agent: 후보 운동 반환
    Agent->>LLM: 후보 기반 루틴 초안 생성
    LLM-->>Agent: routine_draft
    Agent->>LLM: 루틴 검증
    LLM-->>Agent: validation_result
    Agent-->>API: needs_review + thread_id
    API-->>FE: 루틴 초안 반환
    FE-->>User: 루틴 검토 화면 표시

    opt 수정 요청
        User->>FE: 피드백 입력
        FE->>API: POST /api/routines/recommend/review/
        API->>Agent: thread_id로 수정 재개
        Agent->>LLM: 기존 루틴 + 피드백 반영
        Agent-->>API: 수정된 루틴 반환
        API-->>FE: 수정 결과
    end
```
