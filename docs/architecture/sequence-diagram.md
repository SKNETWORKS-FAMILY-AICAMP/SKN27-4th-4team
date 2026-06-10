# 시퀀스 다이어그램

## AI 챗봇 RAG 시퀀스

```mermaid
sequenceDiagram
    actor User as 👤 사용자
    participant FE as 💻 Frontend
    participant View as 🌐 Backend
    participant CB as 🤖 chatbot.py
    participant Graph as 🔍 LangGraph
    participant DB as 🗄️ PostgreSQL

    User->>FE: ① 질문 입력
    FE->>View: ② 서버로 전달
    View->>DB: ③ 사용자 메시지 저장
    View->>CB: ④ 답변 생성 요청
    CB->>DB: ⑤ 이전 대화 기록 불러오기
    DB-->>CB: ↩ 대화 기록
    CB->>Graph: ⑥ 그래프 실행 (이전 대화 + 질문 + 세션 ID)

    Note over Graph: ⑦ 질문 유형 분류<br/>무관 / 회상 / 일반 / 특정 / 부상

    alt 운동 무관
        Graph-->>CB: 범위 안내 후 종료
    else 대화 내용을 묻는 질문 (회상)
        Graph->>DB: ⑧ 전체 대화 기록 조회
        DB-->>Graph: ↩ 전체 기록
        Note over Graph: ⑨ 대화 기록만으로 답 (검색 없음)
    else 운동 데이터가 필요한 질문
        Note over Graph: 검색 전 _rewrite_query<br/>참조 → 실제 운동명
        alt 일반 추천
            Graph->>DB: ⑩-A 조건 맞는 운동 검색
        else 특정 운동
            Graph->>DB: ⑩-B 운동명 정확 검색
        else 부상·통증
            Graph->>DB: ⑩-C 근육 조회 + 부위 제외 검색
            Note over Graph: ⑪ 관련도 재정렬(rerank) → 상위 5개
        end
        Note over Graph: ⑫ 운동 데이터 + 대화로 답 생성
    end

    loop ⑬ 답변 완성까지 (스트리밍)
        Graph-->>CB: 답변 조각(토큰)
        CB-->>View: 조각 전달
        View-->>FE: 실시간 전송
        FE-->>User: 글자 하나씩 표시
    end

    CB-->>View: ⑭ 완성 신호
    View->>DB: ⑮ 완성 답변 저장
    View-->>FE: ⑯ 완료 신호(메시지 ID)
    FE-->>User: ⑰ 최종 답변 화면 고정
```

## 루틴 추천 및 피드백 시퀀스

```mermaid
%%{init: {
  "themeVariables": {
    "fontSize": "18px",
    "fontFamily": "Arial"
  }
}}%%

sequenceDiagram
    participant User as 사용자
    participant FE as 프론트엔드
    participant API as Django API
    participant Graph as LangGraph
    participant DB as GraphDB
    participant LLM as LLM

    User->>FE: 설문 입력 후 "AI 추천 루틴 생성하기"
    FE->>API: POST /api/routines/recommend
    API->>Graph: 추천 워크플로우 시작

    rect rgb(235, 245, 255)
        Note over Graph,LLM: 추천 생성 루프
        Graph->>DB: 조건에 맞는 운동 후보 검색
        DB-->>Graph: 운동 후보 반환
        Graph->>LLM: 후보 기반 루틴 초안 생성
        LLM-->>Graph: routine_draft 반환
        Graph->>LLM: 루틴 내부 검증
        LLM-->>Graph: validation_result 반환
    end

    Graph-->>API: status=needs_review<br/>thread_id<br/>routine_draft<br/>validation_result
    API-->>FE: 생성된 루틴 반환
    FE-->>User: 루틴 표시
    FE-->>User: "추천 루틴 검토 및 수정" 패널 표시

    Note over FE,User: needs_review는 루틴 생성이 끝난 상태<br/>사용자는 루틴을 바로 확인할 수 있고, 필요하면 수정 요청을 보냄

    opt 사용자가 수정 요청을 입력하는 경우
        User->>FE: 수정 요청 입력<br/>예: "허리에 부담이 적게 해주세요."
        User->>FE: "수정하기" 클릭
        FE-->>User: "피드백 반영 중..."
        FE->>API: POST /api/routines/recommend/review<br/>thread_id + decision=revise + feedback
        API->>Graph: 같은 thread_id로 수정 재개

        Graph->>LLM: 기존 루틴에 피드백 반영
        LLM-->>Graph: 수정된 routine_draft 반환
        Graph->>LLM: 수정 루틴 내부 검증
        LLM-->>Graph: validation_result 반환

        Graph-->>API: status=needs_review<br/>같은 thread_id<br/>수정된 routine_draft<br/>validation_result
        API-->>FE: 수정된 루틴 초안 반환
        FE-->>User: "수정 요청 반영 완료"
        FE-->>User: "입력한 피드백을 바탕으로 추천 루틴을 다시 구성했습니다."
        FE-->>User: "변경된 루틴 확인하기"
    end
```
