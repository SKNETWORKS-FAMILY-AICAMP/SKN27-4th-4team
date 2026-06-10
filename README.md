# HELBOTIN

<p align="center">
  <strong>SKN27 4th PROJECT · AI Fitness Routine & Exercise Consultation Service</strong><br />
  운동 라이브러리, 개인 맞춤 루틴 추천, AI 운동 챗봇을 연결한 운동 루틴 관리 웹 서비스
</p>

---

## Demo

- Web: [https://sk-camp.cloud](https://sk-camp.cloud)
- 상세 API: [docs/api-spec.md](./docs/api-spec.md)

---

## Overview

### 1. 프로젝트 소개

HELBOTIN은 **헬스 보이 루틴**이라는 의미를 담은 AI 운동 루틴 추천 서비스입니다. 운동 라이브러리, 개인 맞춤 운동 루틴 추천, AI 운동 챗봇을 통해 사용자가 더 쉽고 꾸준하게 운동을 이어갈 수 있도록 돕습니다.

운동을 꾸준히 이어가기 위해서는 **다양성**이 필요합니다. 같은 운동 루틴을 반복하면 몸이 해당 동작에 적응하여 칼로리 소모가 줄어들고, 근육 성장이 정체되는 **플래토(Plateau) 현상**이 발생할 수 있습니다.

> "같은 부위, 다른 자극. 뻔한 루틴을 깨다."

### 2. 문제 정의

| 대상                  | 문제                                                   |
| --------------------- | ------------------------------------------------------ |
| 운동 경험자           | 반복되는 루틴으로 인한 정체기 및 흥미 저하             |
| 운동 초보자           | 어떤 운동을 어떻게 시작해야 할지 판단하기 어려움       |
| 통증·부상 이력 사용자 | 피해야 할 운동과 대체 운동을 구분하기 어려움           |
| 공통 사용자           | 운동 자세, 호흡, 주의사항 정보가 여러 곳에 흩어져 있음 |

### 3. 서비스 소개

| 핵심 기능           | 설명                                                  |
| ------------------- | ----------------------------------------------------- |
| 운동 라이브러리     | 900개 이상의 운동 정보, 영상, 자세 가이드 제공        |
| 개인 맞춤 루틴 추천 | 목표, 수준, 통증, 요일, 시간 기반 주간 루틴 자동 구성 |
| AI 운동 챗봇        | 운동 질문을 분류하고 RAG 검색 결과 기반 답변 제공     |
| 루틴 관리           | 수행 체크, 대체 운동 선택, 데일리 메모 저장           |

---

## Team

<table>
  <tr>
    <td align="center"><b>이재희 (팀장)</b></td>
    <td align="center"><b>김필주</b></td>
    <td align="center"><b>김경수</b></td>
    <td align="center"><b>주연중</b></td>
    <td align="center"><b>박창제</b></td>
  </tr>
  <tr>
    <td align="center"><img width="140" height="160" alt="이재희" src="docs/profiles/이재희.gif" /></td>
    <td align="center"><img width="140" height="160" alt="주연중" src="docs/profiles/주연중.gif" /></td>
    <td align="center"><img width="140" height="160" alt="박창제" src="docs/profiles/박창제.gif" /></td>
    <td align="center"><img width="140" height="160" alt="김필주" src="docs/profiles/김필주.gif" /></td>
    <td align="center"><img width="140" height="160" alt="김경수" src="docs/profiles/김경수.gif" /></td>
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

## Architecture

### 4. 시스템 아키텍처

React/Vite 프론트엔드, Django REST API, PostgreSQL + pgvector, Neo4j, LangGraph 기반 AI 워크플로우로 구성됩니다.

- 상세 문서: [docs/architecture.md](./docs/architecture.md)

### 5. AI 시스템

HELBOTIN의 AI 시스템은 두 축으로 구성됩니다.

- **AI 루틴 추천 Agent**: 사용자 조건을 기반으로 GraphDB 후보를 조회하고 루틴 초안을 생성·검증·수정합니다.
- **RAG 기반 챗봇**: 운동 질문을 분류하고 PostgreSQL + pgvector 검색 결과를 바탕으로 답변합니다.

- 상세 문서: [docs/architecture.md](./docs/architecture.md), [docs/rag-evaluation.md](./docs/rag-evaluation.md)

### 6. 데이터 모델

사용자, 운동 마스터, 상담 세션, 주간 루틴, 근육 관계, 운동-근육 브릿지 테이블로 구성됩니다.

- ERD: [docs/erd.md](./docs/erd.md)
- 시퀀스 다이어그램: [docs/sequence-diagram.md](./docs/sequence-diagram.md)

---

## Development

### 7. 기술 스택

| 영역     | 기술                                                                |
| -------- | ------------------------------------------------------------------- |
| Frontend | React 18, Vite, React Router, Lucide React, React Markdown          |
| Backend  | Django 4.2, Django REST Framework, Simple JWT, Gunicorn, WhiteNoise |
| Database | PostgreSQL 16, pgvector, Neo4j, APOC                                |
| AI       | OpenAI, LangChain, LangGraph, Groq, Ollama                          |
| DevOps   | Docker Compose, AWS EC2, Nginx, Certbot HTTPS                       |

### 8. 프로젝트 구조

```text
SKN27-4th-4team/
├── backend/
│   ├── api/
│   ├── config/
│   ├── data/
│   ├── db/
│   └── recommendation_service/
├── frontend/
│   ├── public/
│   └── src/
├── docs/
├── docker-compose.yml
├── docker-compose.prod.yml
└── README.md
```

### 9. 실행 방법

```bash
docker compose up --build
```

- Frontend: `http://localhost:5173`
- Backend: `http://localhost:8000`
- Neo4j Browser: `http://localhost:7474`

운영 배포는 [docs/AWS_DEPLOY.md](./docs/AWS_DEPLOY.md)를 참고합니다.

---

## Evaluation

### 10. 성능 평가

서비스 API, 루틴 추천, RAG 챗봇의 동작을 시나리오 기반으로 검증했습니다.

- 테스트 시나리오: [docs/test-scenario.md](./docs/test-scenario.md)
- RAG 품질평가: [docs/rag-evaluation.md](./docs/rag-evaluation.md)

### 11. 향후 계획

- 루틴 추천 결과에 대한 정량 평가 지표와 테스트 케이스 확대
- 통증·부상 조건의 GraphDB 후보 단계 필터링 강화
- 상담 RAG에서 운동별 단일 벡터 구조를 다중 청크 검색 구조로 개선
- RAGAS, Hit Rate, MRR 기반 RAG 품질 리포트 자동화
- 모바일 운동 기록 경험 및 반응형 UI 개선
- AWS 운영 환경에서 Secrets Manager 또는 Parameter Store 적용

---

## Appendix

### 12. 참고 문서

#### 상세 문서

- [아키텍처](./docs/architecture.md)
- [ERD](./docs/erd.md)
- [시퀀스 다이어그램](./docs/sequence-diagram.md)
- [API 명세](./docs/api-spec.md)
- [화면 설계](./docs/screen-design.md)
- [테스트 시나리오](./docs/test-scenario.md)
- [RAG 품질평가](./docs/rag-evaluation.md)
- [회고](./docs/retrospective.md)

#### 기존 참고 문서

- [루틴 추천 서비스](./docs/ROUTINE_RECOMMENDATION_SERVICE.md)
- [RAG 프로젝트 정리](./docs/RAG_프로젝트_정리.md)
- [RAG 챗봇 기술명세](./docs/RAG_챗봇_기술명세.md)
- [GraphDB 명세](./docs/planfit_graphdb_spec.md)
- [서비스 테스트 목록](./docs/SERVICE_AB_TEST_LIST.md)
