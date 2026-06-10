# API 명세

## 운동 API

| Method | Endpoint | 설명 |
| --- | --- | --- |
| GET | `/api/exercises/` | 운동 목록 조회. `full=1` 사용 시 상세 필드 포함 |
| GET | `/api/exercises/featured/` | 홈 화면 대표 운동 조회. `limit` 지원 |
| GET | `/api/exercises/<exercise_id>/` | 운동 상세 정보 조회 |

## 상담 API

| Method | Endpoint | 설명 |
| --- | --- | --- |
| GET | `/api/sessions/` | 상담 세션 목록 조회 |
| POST | `/api/sessions/` | 새 상담 세션 생성 |
| GET | `/api/sessions/<session_id>/` | 상담 세션 상세 조회 |
| PATCH | `/api/sessions/<session_id>/` | 세션 제목 수정 |
| DELETE | `/api/sessions/<session_id>/` | 세션 삭제 |
| GET | `/api/sessions/<session_id>/messages/` | 상담 메시지 조회 |
| POST | `/api/sessions/<session_id>/messages/` | 사용자 메시지 저장 및 SSE 답변 생성 |

## 인증 API

| Method | Endpoint | 설명 |
| --- | --- | --- |
| GET | `/api/auth/csrf/` | CSRF 쿠키 발급 |
| POST | `/api/auth/register/` | 회원가입 |
| POST | `/api/auth/login/` | 로그인 |
| POST | `/api/auth/logout/` | 로그아웃 |
| GET | `/api/auth/me/` | 현재 사용자 조회 |
| GET | `/api/auth/check-nickname/` | 닉네임 중복 확인 |
| GET | `/api/auth/check-email/` | 이메일 중복 확인 |

## 루틴 API

| Method | Endpoint | 설명 |
| --- | --- | --- |
| GET | `/api/routines/` | 주차별 루틴 조회 |
| POST | `/api/routines/` | 주차별 루틴 저장 |
| POST | `/api/routines/recommend/` | AI 루틴 추천 생성 |
| POST | `/api/routines/recommend/review/` | 추천 루틴 승인/수정 피드백 처리 |
