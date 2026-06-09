# HELBOTIN

<p align="center">
  <strong>SKN27 4th PROJECT · AI Fitness Routine & Exercise Consultation Service</strong><br />
  운동 라이브러리, 개인 맞춤 루틴 추천, AI 운동 챗봇을 연결한 운동 루틴 관리 웹 서비스
</p>

---

## 목차

1. [프로젝트 개요](#1-프로젝트-개요)
2. [팀 소개](#2-팀-소개-skn27-4팀)
3. [주제 선정 배경](#3-주제-선정-배경)
4. [타겟 사용자](#4-타겟-사용자)
5. [서비스 모델](#5-서비스-모델)
6. [시스템 아키텍처](#6-시스템-아키텍처)
7. [AI Agent 구조](#7-ai-agent-구조)
8. [데이터 구성](#8-데이터-구성)
9. [기술 스택](#9-기술-스택)
10. [한계점 및 개선 방향](#10-한계점-및-개선-방향)

---

## 1. 프로젝트 개요

HELBOTIN은 **헬스 보이 루틴**이라는 의미를 담은 AI 운동 루틴 추천 서비스입니다. 운동 라이브러리, 개인 맞춤 운동 루틴 추천, AI 운동 챗봇을 통해 사용자가 더 쉽고 꾸준하게 운동을 이어갈 수 있도록 돕습니다.

운동을 꾸준히 이어가기 위해서는 **다양성**이 필요합니다. 같은 운동 루틴을 반복하면 몸이 해당 동작에 적응하여 칼로리 소모가 줄어들고, 근육 성장이 정체되는 **플래토(Plateau) 현상**이 발생할 수 있습니다. 또한 단조로운 루틴은 운동에 대한 흥미 자체를 떨어뜨립니다.

HELBOTIN은 이러한 문제를 해결하고, 운동 초보자부터 숙련자까지 누구나 자신의 상태에 맞게 효과적으로 운동할 수 있는 환경을 만들기 위해 개발되었습니다.

> "같은 부위, 다른 자극. 뻔한 루틴을 깨다."

---

## 2. 팀 소개 (SKN27 4팀)

<table>
  <tr>
    <td align="center"><b>이재희 (팀장)</b></td>
    <td align="center"><b>김필주</b></td>
    <td align="center"><b>김경수</b></td>
    <td align="center"><b>주연중</b></td>
    <td align="center"><b>박창제</b></td>
  </tr>
  <tr>
    <td align="center"><img width="140" height="160" alt="이재희" src="frontend/public/이재희.gif" /></td>
    <td align="center"><img width="140" height="160" alt="주연중" src="frontend/public/주연중.gif" /></td>
    <td align="center"><img width="140" height="160" alt="박창제" src="frontend/public/박창제.gif" /></td>
    <td align="center"><img width="140" height="160" alt="김필주" src="frontend/public/김필주.gif" /></td>
    <td align="center"><img width="140" height="160" alt="김경수" src="frontend/public/김경수.gif" /></td>
  </tr>
  <tr>
    <td align="center">Project Manager</td>
    <td align="center">AI 루틴 추천</td>
    <td align="center">RAG 기반 챗봇</td>
    <td align="center">GraphDB / 데이터 구조 구축</td>
    <td align="center">Auth 인증 / 사용자 관리</td>
  </tr>
  <tr>
   <td align="center">서비스 기획<br/>프론트엔드<br/>AWS 배포</td>
  <td align="center">사용자 조건 분석<br/>맞춤 루틴 생성<br/>추천 결과 저장·리뷰</td>
  <td align="center">운동 상담 챗봇<br/>RAG 검색 파이프라인<br/>SSE 스트리밍 응답</td>
  <td align="center">Neo4j 그래프 구축<br/>운동 후보 조회<br/>유사·대체 운동 관계</td>
  <td align="center">회원가입·로그인<br/>세션/JWT 관리<br/>게스트 사용자 처리</td>
</tr>
</table>

---

## 3. 주제 선정 배경

### 3.1 목적 및 배경

운동 효과를 높이기 위해서는 같은 부위라도 다양한 방식으로 자극을 주는 것이 중요합니다. 하지만 많은 사용자는 익숙한 운동만 반복하게 되고, 그 결과 다음과 같은 문제가 발생합니다.

- 신체가 반복 동작에 적응해 운동 효율이 떨어짐
- 근육 성장이 정체되는 플래토 현상이 발생함
- 루틴이 단조로워져 운동 흥미가 감소함
- 초보자는 어떤 운동부터 시작해야 할지 판단하기 어려움
- 부상이나 통증이 있을 때 어떤 운동을 피해야 하는지 알기 어려움

### 3.2 문제 정의

| 대상        | 문제                                                        |
| ----------- | ----------------------------------------------------------- |
| 운동 경험자 | 반복되는 루틴으로 인한 정체기 및 흥미 저하                  |
| 운동 초보자 | 어떤 운동을 어떻게 시작해야 할지 모름                       |
| 공통        | 자신의 상태(부상, 체력 수준)에 맞는 운동 정보를 얻기 어려움 |

### 3.3 해결 방안

HELBOTIN은 세 가지 핵심 기능으로 위 문제를 해결합니다.

| 해결 기능                | 설명                                                              |
| ------------------------ | ----------------------------------------------------------------- |
| 운동 라이브러리          | 다양한 운동 정보를 제공하여 루틴에 변화를 주고 정체기 탈출을 도움 |
| 개인 맞춤 운동 루틴 추천 | 사용자의 목표, 체력 수준, 가용 시간에 맞는 주간 루틴 자동 구성    |
| AI 운동 챗봇             | 운동 동작, 자세, 부상 시 대체 운동 등 운동 관련 질문에 즉시 답변  |

---

## 4. 타겟 사용자

HELBOTIN은 운동은 꾸준히 하고 싶지만, 매주 어떤 루틴을 해야 할지 고민하는 사용자를 대상으로 합니다.

| 사용자 유형 | 겪는 문제 | HELBOTIN이 제공하는 가치 |
| --- | --- | --- |
| 운동 초보자 | 어떤 운동부터 시작해야 할지 모름 | 목표와 수준에 맞는 첫 루틴 제공 |
| 정체기를 겪는 경험자 | 반복 루틴으로 자극과 흥미가 감소함 | 같은 부위에 새로운 운동 자극 제공 |
| 통증·부상 이력 사용자 | 피해야 할 운동을 구분하기 어려움 | 통증 부위를 고려한 대체 운동 제안 |
| 시간이 불규칙한 사용자 | 요일과 시간에 맞춰 루틴을 짜기 어려움 | 가용 시간 기반 주간 루틴 구성 |
| 운동 정보를 찾는 사용자 | 자세, 호흡, 주의사항이 흩어져 있음 | 운동 영상과 상세 가이드를 한곳에 제공 |

---

## 5. 서비스 모델

### 5.1 핵심 기능

| 기능               | 설명                                                                         |
| ------------------ | ---------------------------------------------------------------------------- |
| 운동 라이브러리    | 900개 이상의 운동 데이터를 카테고리, 기구, 난이도, 검색어로 탐색             |
| 운동 상세 정보     | 운동 영상, 설명, 시작 자세, 동작, 호흡, 주의사항, 관련 운동 제공             |
| 맞춤 루틴 추천     | 나이, 성별, 수준, 통증 부위, 요일, 운동 부위, 목표, 시간 기반 주간 루틴 생성 |
| 루틴 관리          | 요일별 운동 완료 체크, 세트/반복수 확인, 대체 운동 선택, 데일리 메모 저장    |
| AI 운동 상담       | 운동 질문을 분류하고 RAG 검색 결과를 바탕으로 스트리밍 답변 제공             |
| 사용자/게스트 세션 | 로그인 사용자와 device_uuid 기반 게스트의 상담·루틴 데이터 유지              |

### 5.2 차별점

- 통증 부위 기반 운동 제외 및 대체 추천
- PostgreSQL 벡터 검색과 근육 브릿지 테이블을 활용한 운동 상담
- Neo4j 그래프 검색 기반 루틴 후보 탐색
- LangGraph를 이용한 추천 생성, 검증, 사람 검토, 수정 루프
- 상담 세션, 루틴, 수행 기록을 분리 저장하는 지속 관리 구조

---

## 6. 시스템 아키텍처

### 6.1 현재 구성

- **Frontend** : React 18 + Vite 기반 SPA
- **Backend** : Django 4.2 + Django REST Framework
- **Auth** : 세션/JWT 계열 인증, 로그인 사용자와 게스트 device_uuid 지원
- **Primary DB** : PostgreSQL + pgvector
- **Graph DB** : Neo4j + APOC + Graph Data Science
- **AI Workflow** : LangGraph 기반 루틴 추천 그래프와 RAG 챗봇 그래프
- **Streaming** : Django `StreamingHttpResponse` 기반 SSE 토큰 스트리밍
- **Container** : Docker Compose로 DB, Neo4j, Backend, Frontend 구성

### 6.2 주요 API

| API                                     | 역할                              |
| --------------------------------------- | --------------------------------- |
| `GET /api/exercises/`                   | 운동 목록 조회                    |
| `GET /api/exercises/featured/`          | 홈 화면 추천 운동 조회            |
| `GET /api/exercises/<id>/`              | 운동 상세 정보 조회               |
| `GET/POST /api/sessions/`               | 상담 세션 조회 및 생성            |
| `GET/POST /api/sessions/<id>/messages/` | 상담 메시지 조회 및 SSE 답변 생성 |
| `GET/POST /api/routines/`               | 주간 루틴 조회 및 저장            |
| `POST /api/routines/recommend/`         | AI 루틴 추천 시작                 |
| `POST /api/routines/recommend/review/`  | 추천안 승인·수정 피드백 처리      |

---

## 7. AI Agent 구조

### 7.1 루틴 추천 워크플로우

1. 사용자 설문 입력을 추천 프로필로 변환
2. Supervisor Agent가 다음 실행 단계를 결정
3. Recommendation Param Agent가 검색 조건을 구조화
4. Graph Search Tool이 Neo4j에서 운동 후보를 조회
5. Routine Composition Agent가 후보 안에서 주간 루틴 초안 생성
6. Routine Validation Agent가 안전성, 장비, 분할 부위, 운동 다양성 검증
7. Final Human Review 단계에서 사용자 검토 요청
8. 수정 요청이 있으면 Revision Agent 또는 재검색 루프로 반영

### 7.2 루틴 추천 Agent

| Agent / Tool               | 처리 내용                                                     |
| -------------------------- | ------------------------------------------------------------- |
| Supervisor Agent           | 상태와 검증 결과를 보고 다음 노드 라우팅                      |
| User Profile Tool          | 프론트 설문 데이터를 추천 가능한 프로필로 정규화              |
| Recommendation Param Agent | 목표, 레벨, 장비, 통증, 척추 부하 조건을 검색 파라미터로 변환 |
| Graph Search Tool          | Neo4j 운동 그래프에서 부위별 후보 조회                        |
| Composition Agent          | 후보 운동만 사용해 주간 루틴 초안 구성                        |
| Validation Agent           | 누락 부위, 장비 불일치, 통증 위험, 움직임 편중 검증           |
| Revision Agent             | 사용자 피드백에 따라 운동 제외, 대체, 재구성 수행             |

### 7.3 운동 상담 RAG 워크플로우

1. 사용자 질문 유형 분류
2. 운동 무관 질문은 즉시 제한 응답
3. 이전 대화 맥락이 필요한 질문은 히스토리 기반 응답
4. 일반 추천, 특정 운동, 통증/부상 질문을 분기
5. pgvector 유사도 검색, 키워드 검색, 근육 필터를 조합
6. 검색된 운동 데이터와 대화 히스토리 기반 답변 생성
7. SSE로 프론트엔드에 토큰 단위 스트리밍

### 7.4 안전성 처리

- 통증/부상 질문에서는 고급 난이도 운동을 제외
- 근육 브릿지 테이블로 아픈 부위와 연결된 운동을 필터링
- 무릎, 손목, 허리처럼 근육명이 아닌 관절 표현은 대표 근육 키워드로 보정
- 데이터에 없는 운동 효과나 안전성은 단정하지 않도록 프롬프트 제한

---

## 8. 데이터 구성

**저장소**

- PostgreSQL
- pgvector
- Neo4j

**데이터 출처 및 적재**

- `backend/data/planfit_exercises_enriched.json`
- `backend/data/planfit_exercises.json`
- `backend/data/exercise_edges.json`
- `backend/db/load_exercises.py`
- `backend/db/load_muscles.py`
- `backend/db/build_graph.py`

**ERD 구성 요소**

- 사용자 (`users`)
- 통증 로그 (`user_pain_logs`)
- 운동 마스터 (`exercises`)
- 상담 세션 (`chat_sessions`)
- 상담 메시지 (`chat_messages`)
- 주간 루틴 스케줄러 (`weekly_schedulers`)
- 일별 루틴 (`daily_routines`)
- 근육 마스터 (`muscles`)
- 근육 관계 (`muscle_relations`)
- 운동-근육 브릿지 (`exercise_muscles`)

---

## 9. 기술 스택

### Frontend

![React](https://img.shields.io/badge/React-18.3.1-61DAFB?style=for-the-badge&logo=react&logoColor=black)
![Vite](https://img.shields.io/badge/Vite-5.4.1-646CFF?style=for-the-badge&logo=vite&logoColor=white)
![React Router](https://img.shields.io/badge/React_Router-7.16.0-CA4245?style=for-the-badge&logo=reactrouter&logoColor=white)
![Lucide React](https://img.shields.io/badge/Lucide_React-1.17.0-F56565?style=for-the-badge)
![React Markdown](https://img.shields.io/badge/React_Markdown-10.1.0-000000?style=for-the-badge&logo=markdown&logoColor=white)

### Backend

![Django](https://img.shields.io/badge/Django-4.2.29-092E20?style=for-the-badge&logo=django&logoColor=white)
![DRF](https://img.shields.io/badge/Django_REST_Framework-3.x-A30000?style=for-the-badge&logo=django&logoColor=white)
![Simple JWT](https://img.shields.io/badge/Simple_JWT-5.5.1-000000?style=for-the-badge)
![django-cors-headers](https://img.shields.io/badge/django--cors--headers-enabled-092E20?style=for-the-badge&logo=django&logoColor=white)
![psycopg2](https://img.shields.io/badge/psycopg2--binary-enabled-336791?style=for-the-badge&logo=postgresql&logoColor=white)

### AI / Agent

![LangGraph](https://img.shields.io/badge/LangGraph-1.0+-1C3C3C?style=for-the-badge)
![LangChain](https://img.shields.io/badge/LangChain-Core_1.0+-1C3C3C?style=for-the-badge)
![OpenAI](https://img.shields.io/badge/OpenAI-supported-412991?style=for-the-badge&logo=openai&logoColor=white)
![Ollama](https://img.shields.io/badge/Ollama-supported-000000?style=for-the-badge)
![Groq](https://img.shields.io/badge/Groq-supported-F55036?style=for-the-badge)
![Sentence Transformers](https://img.shields.io/badge/Sentence_Transformers-rerank-FF6F00?style=for-the-badge)

### Data

![PostgreSQL](https://img.shields.io/badge/PostgreSQL-16-4169E1?style=for-the-badge&logo=postgresql&logoColor=white)
![pgvector](https://img.shields.io/badge/pgvector-enabled-336791?style=for-the-badge)
![Neo4j](https://img.shields.io/badge/Neo4j-latest-4581C3?style=for-the-badge&logo=neo4j&logoColor=white)
![Graph Data Science](https://img.shields.io/badge/Neo4j_GDS-enabled-4581C3?style=for-the-badge&logo=neo4j&logoColor=white)

### DevOps

![Docker](https://img.shields.io/badge/Docker_Compose-enabled-2496ED?style=for-the-badge&logo=docker&logoColor=white)
![Node.js](https://img.shields.io/badge/Node.js-frontend_runtime-5FA04E?style=for-the-badge&logo=nodedotjs&logoColor=white)
![Python](https://img.shields.io/badge/Python-3.10+-3776AB?style=for-the-badge&logo=python&logoColor=white)

---

## 10. 한계점 및 개선 방향

- 운동 추천 결과에 대한 정량 평가 지표와 테스트 케이스 확대
- 루틴 추천 결과를 실제 사용자 피드백으로 재학습·개선하는 루프 보강
- 상담 RAG에서 운동별 단일 벡터 저장 구조를 다중 청크 검색 구조로 개선
- 통증·부상 관련 답변에 의료적 한계를 더 명확히 표시
- 프론트엔드 반응형 UI와 모바일 운동 기록 경험 개선
- Neo4j 그래프 데이터 구축 자동화와 운영 모니터링 강화

---

## 실행 방법

```bash
docker compose up --build
```

- Frontend: `http://localhost:5173`
- Backend: `http://localhost:8000`
- Neo4j Browser: `http://localhost:7474`

---

## 참고 데이터

- Planfit 기반 운동 데이터
- 자체 정제 운동 상세 가이드 및 영상 경로
- 근육 마스터·근육 관계 데이터
- 운동-근육 브릿지 및 Neo4j 운동 그래프
