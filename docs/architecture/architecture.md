# 시스템 아키텍처

HELBOTIN은 React/Vite 프론트엔드, Django REST API, PostgreSQL + pgvector, Neo4j, LangGraph 기반 AI 워크플로우로 구성됩니다.

## 전체 구조

```mermaid
flowchart LR
    User["사용자"] --> Browser["React/Vite Frontend"]
    Browser --> Nginx["Nginx Reverse Proxy"]
    Nginx --> Django["Django REST API"]

    Django --> Auth["Auth / Session / device_uuid"]
    Django --> Routine["LangGraph Routine Agent"]
    Django --> Chatbot["LangGraph RAG Chatbot"]

    Routine --> Neo4j["Neo4j Graph DB"]
    Chatbot --> PGVector["PostgreSQL + pgvector"]
    Django --> Postgres["PostgreSQL RDB"]

    Postgres --> Data["Exercise / Routine / Chat / User Tables"]
    Neo4j --> GraphData["Exercise Relation Graph"]
    PGVector --> Embedding["OpenAI Embedding Vector"]

    Django --> LLM["OpenAI / Groq / Ollama"]
```

## 구성 요소

| 영역 | 구성 |
| --- | --- |
| Frontend | React 18, Vite, React Router, Lucide React |
| Backend | Django 4.2, Django REST Framework |
| Auth | 세션/JWT 계열 인증, 로그인 사용자와 게스트 device_uuid 지원 |
| Primary DB | PostgreSQL 16, pgvector |
| Graph DB | Neo4j, APOC 중심 운영 |
| AI Workflow | LangGraph 기반 루틴 추천 그래프와 RAG 챗봇 그래프 |
| Streaming | Django `StreamingHttpResponse` 기반 SSE 토큰 스트리밍 |
| Deployment | AWS EC2, Docker Compose, Nginx, Certbot HTTPS |

## AI 루틴 추천 구조

```mermaid
flowchart TD
    Start(["START"]) --> Supervisor["Supervisor Agent"]

    Supervisor -->|프로필 정리| Profile["User Profile Tool"]
    Supervisor -->|추천 조건 생성| Params["Recommendation Param Agent"]
    Supervisor -->|운동 후보 검색| Search["Graph Search Tool"]
    Supervisor -->|루틴 초안 생성| Composition["Routine Composition Agent"]
    Supervisor -->|루틴 검증| Validation["Routine Validation Agent"]
    Supervisor -->|사용자 검토| Review["Final Human Review Node"]
    Supervisor -->|루틴 수정| Revision["Routine Revision Agent"]
    Supervisor -->|완료| End(["END"])

    Profile --> Supervisor
    Params --> Supervisor
    Search --> Supervisor
    Composition --> Supervisor
    Validation --> Supervisor
    Review --> Supervisor
    Revision --> Supervisor
```

| Agent / Tool | 역할 |
| --- | --- |
| Supervisor Agent | 상태와 검증 결과를 보고 다음 노드 라우팅 |
| User Profile Tool | 프론트 설문 데이터를 추천 가능한 프로필로 정규화 |
| Recommendation Param Agent | 목표, 레벨, 장비, 통증, 척추 부하 조건을 검색 파라미터로 변환 |
| Graph Search Tool | Neo4j 운동 그래프에서 부위별 후보 조회 |
| Routine Composition Agent | 후보 운동만 사용해 주간 루틴 초안 구성 |
| Routine Validation Agent | 누락 부위, 장비 불일치, 통증 위험, 움직임 편중 검증 |
| Routine Revision Agent | 사용자 피드백에 따라 운동 제외, 대체, 재구성 수행 |

## RAG 챗봇 구조

```mermaid
flowchart TD
    Start["사용자 질문"] --> Classify["질문 유형 분류"]
    Classify -->|운동 무관| Out["범위 제한 응답"]
    Classify -->|대화 회상| Recall["대화 기록 기반 응답"]
    Classify -->|일반 추천| General["벡터 검색"]
    Classify -->|특정 운동| Specific["운동명 키워드 검색"]
    Classify -->|통증/부상| Injury["근육 필터 + 안전 검색"]
    General --> Generate["LLM 답변 생성"]
    Specific --> Generate
    Injury --> Generate
    Recall --> Stream["SSE 스트리밍"]
    Out --> Stream
    Generate --> Stream
```

## 관련 문서

- [RAG 프로젝트 정리](./RAG_프로젝트_정리.md)
- [RAG 챗봇 기술명세](./RAG_챗봇_기술명세.md)
- [루틴 추천 서비스](./ROUTINE_RECOMMENDATION_SERVICE.md)
- [GraphDB 명세](./planfit_graphdb_spec.md)
