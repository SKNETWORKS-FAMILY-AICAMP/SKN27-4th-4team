# 피트니스 RAG 챗봇 — 프로젝트 정리

> 운동 추천·설명을 제공하는 대화형 RAG 챗봇.
> 핵심 스택: **PostgreSQL + pgvector** (하이브리드 검색), **LangGraph** (대화 그래프), **OpenAI** (임베딩·LLM), **Django** (백엔드).

---

## 1. 시스템 개요

사용자가 운동에 대해 질문하면, 질문 유형을 분류한 뒤 운동 데이터를 검색하고 LLM이 답변을 토큰 단위로 스트리밍한다. 대화 맥락(이전에 무엇을 물었는지)을 기억해 후속 질문도 처리한다.

구성은 크게 두 부분이다.

- **오프라인 인덱싱 파이프라인**: 운동 데이터를 적재 → 임베딩 → PostgreSQL에 저장.
- **런타임 대화 그래프(LangGraph)**: 질문 분류 → 유형별 검색 → 답변 생성.

---

## 2. 데이터 파이프라인 (오프라인 인덱싱)

| 단계 | 파일 | 역할 |
|------|------|------|
| 로더 | `loader.py` | DB 행을 언패킹해 자연어 `page_content` + `metadata`(검색 필터용)로 변환, LangChain `Document`로 묶음 |
| 분할 | `splitter.py` | 50자 미만 문서 필터(의미 없는 청크 방지) — 확인 결과 해당 문서 없음 |
| 임베딩 | `embedding.py` | `page_content`만 뽑아 `text-embedding-3-small`로 벡터(1536차원) 일괄 변환 |
| 저장 | `pgvectordb.py` | `exercises` 테이블의 `embedding` 컬럼에 `UPDATE ... SET embedding = %s::vector` |

`description / starting_position / movement / breathing / related_exercises` 등 누락 항목을 DB에 추가해 답변 품질을 보강했다.

---

## 3. LangGraph 구성의 진화

설계는 단계적으로 발전했다. (상세 다이어그램은 [rag_evolution_all.html](../ppt/rag_evolution_all.html) 참고)

1. **기본 그래프** — `START → retrieve → generate → END`
2. **조건 분기(conditional edge)** — `classify`로 질문 유형별 `retrieve` 분기(general/specific/injury)
3. **Adaptive RAG** — 운동 무관 질문용 `out_of_scope`, 부상 분기에 rerank 추가
4. **recall 노드 추가** — 대화 내용을 묻는 질문을 검색 없이 처리
5. **최종** — 검색 경로에 query rewriting을 더해 참조 질문까지 정확 처리

---

## 4. 정리

기존 스키마(운동 테이블 + 임베딩 컬럼)를 살리는 방향으로 RAG 검색을 직접 구현했고, 대화형 사용에서 드러난 **회상 질문 오염**이라는 핵심 결함을 "검색이 필요한 질문"과 "대화 기록만으로 답할 질문"을 분리하는 구조로 해결했다. 이를 통해 멀티턴 대화에서도 맥락을 정확히 유지하는 챗봇을 완성했다.
