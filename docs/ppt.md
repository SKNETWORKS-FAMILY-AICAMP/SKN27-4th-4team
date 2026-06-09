---
marp: true
theme: default
paginate: true
size: 16:9
backgroundColor: #080808
color: #FFFFFF
style: |
  @import url('https://fonts.googleapis.com/css2?family=Bebas+Neue&family=Noto+Sans+KR:wght@400;500;700;900&display=swap');

  :root {
    --yellow: #FFD700;
    --yellow-2: #C8A200;
    --bg: #080808;
    --panel: rgba(255,255,255,0.075);
    --panel-2: rgba(255,215,0,0.12);
    --line: rgba(255,215,0,0.34);
    --muted: rgba(255,255,255,0.78);
    --white: #FFFFFF;
  }

  section {
    font-family: 'Noto Sans KR', sans-serif;
    padding: 38px 48px;
    background:
      linear-gradient(135deg, rgba(255,215,0,0.08), transparent 34%),
      radial-gradient(circle at 88% 14%, rgba(255,215,0,0.16), transparent 24%),
      #080808;
    color: #FFFFFF;
  }

  h1, h2, h3, p {
    margin: 0;
    letter-spacing: 0;
  }

  section h1 {
    font-size: 82px;
    font-weight: 900;
    line-height: 1.02;
    color: var(--yellow) !important;
  }

  section h2 {
    font-size: 54px;
    font-weight: 900;
    line-height: 1.08;
    margin-bottom: 22px;
    color: var(--yellow) !important;
  }

  section h3 {
    font-size: 24px;
    font-weight: 900;
    color: var(--yellow) !important;
    margin-bottom: 10px;
  }

  p, li {
    font-size: 23px;
    line-height: 1.48;
  }

  strong {
    color: var(--yellow);
    font-weight: 900;
  }

  code {
    color: #080808;
    background: var(--yellow);
    border-radius: 4px;
    padding: 2px 7px;
    font-weight: 900;
  }

  .brand {
    font-family: 'Bebas Neue', sans-serif;
    color: var(--yellow);
    letter-spacing: 3px;
    font-size: 50px;
  }

  .eyebrow {
    color: var(--yellow);
    font-size: 15px;
    font-weight: 900;
    letter-spacing: 3px;
    text-transform: uppercase;
    margin-bottom: 16px;
  }

  .lead {
    color: var(--muted);
    font-size: 27px;
    max-width: 880px;
    margin-top: 18px;
  }

  .muted {
    color: var(--muted);
  }

  .yellow {
    color: var(--yellow);
  }

  .split {
    display: grid;
    grid-template-columns: 0.92fr 1.08fr;
    gap: 42px;
    align-items: center;
    height: 100%;
  }

  .split-even {
    display: grid;
    grid-template-columns: 1fr 1fr;
    gap: 26px;
    align-items: stretch;
  }

  .grid-2 {
    display: grid;
    grid-template-columns: repeat(2, 1fr);
    gap: 18px;
  }

  .grid-3 {
    display: grid;
    grid-template-columns: repeat(3, 1fr);
    gap: 18px;
  }

  .grid-4 {
    display: grid;
    grid-template-columns: repeat(4, 1fr);
    gap: 14px;
  }

  .api-card {
    min-height: 168px;
    display: flex;
    align-items: center;
    justify-content: space-between;
    gap: 22px;
  }

  .api-copy {
    min-width: 0;
  }

  .api-glyph {
    width: 76px;
    height: 76px;
    border-radius: 50%;
    border: 2px solid var(--yellow);
    display: flex;
    align-items: center;
    justify-content: center;
    flex: none;
    background: rgba(255,215,0,0.10);
    box-shadow: 0 0 24px rgba(255,215,0,0.16);
  }

  .api-glyph::before {
    content: "";
    width: 44px;
    height: 44px;
    display: block;
    background-image: var(--icon);
    background-position: center;
    background-repeat: no-repeat;
    background-size: contain;
  }

  .icon-exercise {
    --icon: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 64 64'%3E%3Cg fill='none' stroke='%23FFD700' stroke-width='5' stroke-linecap='round' stroke-linejoin='round'%3E%3Cpath d='M20 32h24'/%3E%3Cpath d='M12 23v18M20 20v24M44 20v24M52 23v18'/%3E%3C/g%3E%3C/svg%3E");
  }

  .icon-chatbot {
    --icon: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 64 64'%3E%3Cg fill='none' stroke='%23FFD700' stroke-width='5' stroke-linecap='round' stroke-linejoin='round'%3E%3Cpath d='M13 16h38a6 6 0 0 1 6 6v20a6 6 0 0 1-6 6H27L14 56V48h-1a6 6 0 0 1-6-6V22a6 6 0 0 1 6-6Z'/%3E%3Cpath d='M23 31h.1M32 31h.1M41 31h.1'/%3E%3C/g%3E%3C/svg%3E");
  }

  .icon-auth {
    --icon: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 64 64'%3E%3Cg fill='none' stroke='%23FFD700' stroke-width='5' stroke-linecap='round' stroke-linejoin='round'%3E%3Ccircle cx='28' cy='22' r='10'/%3E%3Cpath d='M10 56c3-12 12-18 28-18'/%3E%3Cpath d='M42 49l6 6 12-16'/%3E%3C/g%3E%3C/svg%3E");
  }

  .icon-routine {
    --icon: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 64 64'%3E%3Cg fill='none' stroke='%23FFD700' stroke-width='5' stroke-linecap='round' stroke-linejoin='round'%3E%3Cpath d='M20 10h24l5 7v43H15V17l5-7Z'/%3E%3Cpath d='M24 27h20M24 39h20M24 51h13'/%3E%3Cpath d='M24 10v10h20V10'/%3E%3C/g%3E%3C/svg%3E");
  }

  .card {
    background: var(--panel);
    border: 1px solid rgba(255,255,255,0.12);
    border-top: 3px solid var(--yellow);
    border-radius: 8px;
    padding: 21px;
  }

  .card p {
    color: var(--muted);
    font-size: 19px;
  }

  .tile {
    background: rgba(255,215,0,0.12);
    border: 1px solid var(--line);
    border-radius: 8px;
    padding: 20px;
  }

  .tile p {
    font-size: 19px;
    color: rgba(255,255,255,0.82);
  }

  .big {
    font-size: 68px;
    font-weight: 900;
    color: var(--yellow);
    line-height: 1;
  }

  .kicker {
    font-size: 18px;
    color: var(--muted);
    margin-top: 8px;
  }

  .quote {
    font-size: 44px;
    font-weight: 900;
    line-height: 1.22;
  }

  .flow {
    display: flex;
    gap: 12px;
    align-items: center;
    margin: 16px 0;
  }

  .node {
    flex: 1;
    min-height: 82px;
    display: flex;
    align-items: center;
    justify-content: center;
    text-align: center;
    border-radius: 8px;
    background: rgba(255,255,255,0.07);
    border: 1px solid rgba(255,255,255,0.15);
    font-size: 19px;
    font-weight: 900;
  }

  .node.hot {
    background: var(--yellow);
    color: #080808;
  }

  .arrow {
    color: var(--yellow);
    font-weight: 900;
    font-size: 28px;
  }

  .screen {
    border: 1px solid rgba(255,255,255,0.12);
    border-radius: 8px;
    background: linear-gradient(180deg, rgba(255,255,255,0.08), rgba(255,255,255,0.03));
    min-height: 310px;
    padding: 18px;
    position: relative;
    overflow: hidden;
  }

  .screen::before {
    content: "";
    position: absolute;
    inset: 0;
    background:
      linear-gradient(90deg, transparent, rgba(255,255,255,0.24), transparent),
      repeating-linear-gradient(0deg, rgba(255,255,255,0.03) 0 1px, transparent 1px 36px);
    pointer-events: none;
  }

  .diagram-img {
    display: block;
    width: 100%;
    max-height: 560px;
    object-fit: contain;
    margin: 8px auto 0;
    border-radius: 8px;
    background: rgba(255,255,255,0.96);
    padding: 14px;
  }

  .mock-nav {
    height: 18px;
    width: 45%;
    background: var(--yellow);
    border-radius: 999px;
    margin-bottom: 24px;
  }

  .mock-line {
    height: 14px;
    border-radius: 999px;
    background: rgba(255,255,255,0.20);
    margin: 12px 0;
  }

  .mock-card {
    height: 72px;
    border-radius: 8px;
    background: rgba(255,255,255,0.18);
    border: 1px solid rgba(255,255,255,0.10);
    margin-top: 16px;
  }

  .tag {
    display: inline-block;
    color: var(--yellow);
    border: 1px solid rgba(255,215,0,0.38);
    border-radius: 999px;
    padding: 5px 11px;
    font-size: 15px;
    font-weight: 900;
    margin-right: 7px;
    margin-bottom: 8px;
  }

  .bar {
    height: 14px;
    border-radius: 999px;
    background: rgba(255,255,255,0.20);
    overflow: hidden;
  }

  .bar > span {
    display: block;
    height: 100%;
    background: linear-gradient(90deg, var(--yellow), var(--yellow-2));
  }

  .mini-list {
    display: grid;
    gap: 12px;
  }

  .mini {
    display: flex;
    align-items: center;
    gap: 12px;
    font-size: 22px;
    font-weight: 800;
  }

  .dot {
    width: 11px;
    height: 11px;
    border-radius: 999px;
    background: var(--yellow);
    flex: none;
  }

  .section-title {
    display: flex;
    height: 100%;
    align-items: center;
  }
---

<div class="brand">HELBOTIN</div>

# 같은 부위,<br />다른 자극.

<p class="lead">운동 백과, AI 루틴 추천, AI 운동 상담을 연결한 피트니스 루틴 관리 서비스</p>

<br />

<span class="tag">AI Fitness</span>
<span class="tag">RAG</span>
<span class="tag">GraphDB</span>
<span class="tag">LangGraph</span>

---

## 발표 목차

<div class="grid-3">
  <div class="tile">
    <div class="big">01</div>
    <h3>문제인식</h3>
    <p>루틴이 지속되지 않는 이유</p>
  </div>
  <div class="tile">
    <div class="big">02</div>
    <h3>타겟 사용자</h3>
    <p>누구의 문제를 해결하는가</p>
  </div>
  <div class="tile">
    <div class="big">03</div>
    <h3>해결방안</h3>
    <p>AI 루틴 파트너 컨셉</p>
  </div>
  <div class="tile">
    <div class="big">04</div>
    <h3>핵심 기능</h3>
    <p>백과, 추천, 상담</p>
  </div>
  <div class="tile">
    <div class="big">05</div>
    <h3>기술스택</h3>
    <p>서비스를 구성한 기술</p>
  </div>
  <div class="tile">
    <div class="big">06</div>
    <h3>검증/시연</h3>
    <p>테스트 결과와 실제 화면</p>
  </div>
</div>

---

## 문제인식

<p class="quote">운동이 어려운 이유는<br />의지가 아니라 <strong>계속할 구조</strong>가 없기 때문입니다.</p>

<br />

<div class="split-even">
  <div class="tile">
    <div class="big">01</div>
    <h3>반복 루틴</h3>
    <p>같은 운동만 반복하면 자극과 흥미가 동시에 줄어듭니다.</p>
  </div>
  <div class="tile">
    <div class="big">02</div>
    <h3>정보 분산</h3>
    <p>운동 자세, 호흡, 주의사항, 대체 운동 정보가 흩어져 있습니다.</p>
  </div>
  <div class="tile">
    <div class="big">03</div>
    <h3>통증 변수</h3>
    <p>통증이 있을 때 어떤 운동을 피해야 하는지 판단하기 어렵습니다.</p>
  </div>
  <div class="tile">
    <div class="big">04</div>
    <h3>설계 부담</h3>
    <p>목표, 요일, 시간에 맞춰 매주 루틴을 짜는 일이 번거롭습니다.</p>
  </div>
</div>

---

## 타겟 사용자

<p class="quote">HELBOTIN은 운동을 시작하거나,<br />다시 꾸준히 이어가려는 사용자를 위한 서비스입니다.</p>

<br />

<div class="grid-3">
  <div class="card">
    <h3>운동 초보자</h3>
    <p>운동 순서와 시작 방법이 필요한 사용자</p>
  </div>
  <div class="card">
    <h3>정체기 경험자</h3>
    <p>새로운 자극과 운동 조합이 필요한 사용자</p>
  </div>
  <div class="card">
    <h3>통증 이력 사용자</h3>
    <p>통증 부위를 고려한 대체 운동이 필요한 사용자</p>
  </div>
  <div class="card">
    <h3>바쁜 사용자</h3>
    <p>요일과 가능 시간에 맞춰 루틴을 받고 싶은 사용자</p>
  </div>
  <div class="card">
    <h3>운동 정보를 찾는 사용자</h3>
    <p>영상, 자세, 호흡, 주의사항을 한곳에서 보고 싶은 사용자</p>
  </div>
</div>

---

## 해결방안

<div class="section-title">
  <div>
    <div class="eyebrow">SERVICE CONCEPT</div>
    <h1>운동 정보를<br />개인 루틴으로 바꾸는<br /><strong>AI 파트너</strong></h1>
    <p class="lead">정보 검색, 운동 관계 탐색, 루틴 생성과 검증을 하나의 흐름으로 연결했습니다.</p>
  </div>
</div>

---

## 핵심 기능

<div class="grid-3">
  <div class="card">
    <h3>운동 백과</h3>
    <p>운동 영상, 설명, 시작 자세, 동작, 호흡, 주의사항 제공</p>
  </div>
  <div class="card">
    <h3>AI 루틴 추천</h3>
    <p>목표, 수준, 통증, 요일, 시간을 반영한 주간 루틴 생성</p>
  </div>
  <div class="card">
    <h3>AI 운동 상담</h3>
    <p>운동 질문을 분류하고 RAG 검색 기반 답변 스트리밍</p>
  </div>
  <div class="card">
    <h3>루틴 관리</h3>
    <p>완료 체크, 세트/반복 확인, 대체 운동 선택, 메모 저장</p>
  </div>
  <div class="card">
    <h3>게스트 세션</h3>
    <p>비로그인 사용자도 device_uuid 기반으로 상담과 루틴 유지</p>
  </div>
  <div class="card">
    <h3>도메인 배포</h3>
    <p>AWS EC2, Docker Compose, Nginx, HTTPS 기반 서비스 운영</p>
  </div>
</div>

<br />

<p class="quote">핵심은 정보를 보여주는 것을 넘어<br />사용자의 루틴으로 <strong>실행 가능하게 만드는 것</strong></p>

---

## 기술스택

<div class="grid-4">
  <div class="card">
    <h3>Frontend</h3>
    <p>React, Vite, React Router</p>
  </div>
  <div class="card">
    <h3>Backend</h3>
    <p>Django, DRF, Gunicorn</p>
  </div>
  <div class="card">
    <h3>Database</h3>
    <p>PostgreSQL, pgvector, Neo4j</p>
  </div>
  <div class="card">
    <h3>AI / DevOps</h3>
    <p>LangGraph, OpenAI, Docker, AWS EC2</p>
  </div>
</div>

<br />

<div class="flow">
  <div class="node hot">React</div>
  <div class="arrow">→</div>
  <div class="node">Django API</div>
  <div class="arrow">→</div>
  <div class="node">RAG / Agent</div>
  <div class="arrow">→</div>
  <div class="node">PostgreSQL / Neo4j</div>
</div>

---

## AI/RAG 설계

<div class="grid-3">
  <div class="card">
    <h3>PostgreSQL</h3>
    <p>사용자, 상담, 루틴, 운동 마스터 데이터 저장</p>
  </div>
  <div class="card">
    <h3>pgvector</h3>
    <p>운동 설명 임베딩 저장 및 의미 기반 검색</p>
  </div>
  <div class="card">
    <h3>Neo4j</h3>
    <p>유사 운동, 대체 운동, progression 관계 탐색</p>
  </div>
</div>

<br />

<div class="mini-list">
  <div class="mini"><span class="dot"></span> RDB는 사용자와 기록을 안정적으로 관리</div>
  <div class="mini"><span class="dot"></span> Vector DB는 질문과 운동 정보를 의미로 연결</div>
  <div class="mini"><span class="dot"></span> Graph DB는 운동 간 관계를 추천 후보로 활용</div>
</div>

---

<div class="section-title">
  <div>
    <div class="eyebrow">RAG CHATBOT</div>
    <h1>질문 유형에 따라<br />검색 방식을<br /><strong>다르게 적용했습니다</strong></h1>
  </div>
</div>

---

## RAG 파이프라인

<div class="flow">
  <div class="node hot">Data Loader</div>
  <div class="arrow">→</div>
  <div class="node">Chunk 변환</div>
  <div class="arrow">→</div>
  <div class="node hot">Embedding</div>
  <div class="arrow">→</div>
  <div class="node">pgvector 저장</div>
</div>

<br />

<div class="flow">
  <div class="node">질문</div>
  <div class="arrow">→</div>
  <div class="node hot">분류</div>
  <div class="arrow">→</div>
  <div class="node">검색</div>
  <div class="arrow">→</div>
  <div class="node">생성</div>
  <div class="arrow">→</div>
  <div class="node">스트리밍</div>
</div>

<br />

<p class="muted">Data Loader에서 적재된 데이터를 Chunk 단위로 나누고, 임베딩 후 pgvector에 저장해 검색 가능한 지식 베이스를 구성합니다.</p>

---

## RAG FLOW

<img class="diagram-img" src="./rag_flow.png" alt="RAG 챗봇 질문 라우팅 흐름" />

---

<div class="section-title">
  <div>
    <div class="eyebrow">ROUTINE RECOMMENDATION</div>
    <h1>LLM이 임의로 만들지 않고<br />GraphDB 후보 안에서<br /><strong>루틴을 검증합니다</strong></h1>
  </div>
</div>

---

## 루틴 추천

<div class="flow">
  <div class="node hot">사용자 설문</div>
  <div class="arrow">→</div>
  <div class="node">조건 추출</div>
  <div class="arrow">→</div>
  <div class="node">Neo4j 후보 검색</div>
  <div class="arrow">→</div>
  <div class="node">후보 운동 선별</div>
</div>

<div class="flow">
  <div class="node">LLM 루틴 구성</div>
  <div class="arrow">→</div>
  <div class="node hot">검증</div>
  <div class="arrow">→</div>
  <div class="node">주간 루틴 제공</div>
  <div class="arrow">→</div>
  <div class="node">피드백 반영</div>
</div>

<br />

<div class="mini-list">
  <div class="mini"><span class="dot"></span> 사용자 설문을 조건으로 변환해 Neo4j에서 후보 운동을 조회</div>
  <div class="mini"><span class="dot"></span> LLM은 조회된 후보 안에서 주간 루틴을 구성</div>
  <div class="mini"><span class="dot"></span> 검증 결과와 사용자 피드백을 다음 수정 루프에 반영</div>
</div>

---

## 서비스 API 구조

<div class="grid-2">
    <div class="card api-card">
    <div class="api-copy">
      <h3>운동</h3>
      <p>`/api/exercises/`<br />목록, 대표 운동, 상세</p>
    </div>
    <div class="api-glyph icon-exercise"></div>
  </div>
  <div class="card api-card">
    <div class="api-copy">
      <h3>챗봇</h3>
      <p>`/api/sessions/`<br />세션, 메시지, SSE</p>
    </div>
    <div class="api-glyph icon-chatbot"></div>
  </div>
  <div class="card api-card">
    <div class="api-copy">
      <h3>인증</h3>
      <p>`/api/auth/`<br />가입, 로그인, 로그아웃</p>
    </div>
    <div class="api-glyph icon-auth"></div>
  </div>
  <div class="card api-card">
    <div class="api-copy">
      <h3>루틴</h3>
      <p>`/api/routines/`<br />저장, 추천, 검토</p>
    </div>
    <div class="api-glyph icon-routine"></div>
  </div>
</div>

---

## 테스트 및 검증

<div class="grid-3">
  <div class="tile">
    <div class="big">31</div>
    <p>서비스 API 테스트</p>
  </div>
  <div class="tile">
    <div class="big">29</div>
    <p>성공 / 정상 동작</p>
  </div>
  <div class="tile">
    <div class="big">2</div>
    <p>실패 케이스</p>
  </div>
</div>

<br />

<div class="mini-list">
  <div class="mini"><span class="dot"></span> 챗봇: 일반 / 특정 / 통증 / 범위 외 질문 정상 응답</div>
  <div class="mini"><span class="dot"></span> 운동 조회: 목록, 상세, 대표 운동, 빈 결과 처리 확인</div>
  <div class="mini"><span class="dot"></span> 인증: 비밀번호 정책 결함 2건 발견 후 개선 과제로 정리</div>
</div>

---

## 배포 구조

<div class="flow">
  <div class="node hot">Gabia DNS</div>
  <div class="arrow">→</div>
  <div class="node">AWS EC2</div>
  <div class="arrow">→</div>
  <div class="node">Nginx HTTPS</div>
  <div class="arrow">→</div>
  <div class="node">Docker Compose</div>
</div>

<br />

<div class="mini-list">
  <div class="mini"><span class="dot"></span> 도메인은 Gabia DNS에서 EC2 퍼블릭 IP로 연결했습니다.</div>
  <div class="mini"><span class="dot"></span> Nginx가 HTTPS 인증서와 외부 요청을 처리하고, 내부 서비스로 프록시합니다.</div>
  <div class="mini"><span class="dot"></span> Docker Compose로 애플리케이션과 데이터 서비스를 한 번에 재시작할 수 있게 구성했습니다.</div>
</div>

---

<div class="brand">HELBOTIN</div>

# 시연

<p class="lead">운동 목록, AI 상담, 맞춤 루틴 추천 흐름을 실제 서비스 화면에서 확인합니다.</p>

<br />

<span class="tag">https://sk-camp.cloud</span>


