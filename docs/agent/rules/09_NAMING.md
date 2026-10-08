# 09. 네이밍 규칙

> 문서 상태: ❓ 작성 전 — 1~3장 표기 규칙은 🧩 기본값, 4~5장 업무 용어 사전은 📋 업무정의서로 작성
> **원칙: 시스템 간 경계면(파일·DB·API·데이터)은 snake_case로 통일**하고, 각 언어 코드 내부는 언어 표준을 따른다.
> 두 층으로 구성된다. **1~3장 표기 규칙**(어떻게 쓰나 — 🧩 공통)과 **4~5장 업무 용어 사전·코드 체계**(무엇이라 부르나 — 📋 업무정의서로 작성).
> 스택이 확정되면(01) 아래 표에서 쓰지 않는 언어 행은 지우고, 프레임워크 관례(예: Next.js 라우트 폴더 케밥케이스)와 충돌하는 행은 관례 쪽으로 조정한다.

## 1. 기본 원칙

### 프로젝트 표준 케이스 🧩
- 상태: 🧩 기본값
- 결정: 경계면 snake_case, 언어 내부는 언어 표준
- 이유: 산출물(파일·DB·JSON) 일관성. 언어 내부까지 강제하면 생태계 관례·린터와 충돌
- 결정일:

## 2. 적용 표

| 대상 | 규칙 | 예시 |
|---|---|---|
| 파일·폴더명 (공통) | snake_case (프레임워크 관례 우선) | `mail_sender.py`, `address_book/` |
| DB 컬렉션·필드 | snake_case, 컬렉션 복수형 | `mail_jobs`, `created_at` |
| API JSON 필드 | snake_case | `{"recipient_count": 3}` |
| 환경변수 | UPPER_SNAKE_CASE | `MONGODB_URI`, `SERVER_PORT` |
| URL 경로 | 케밥케이스 + 복수형 리소스 | `/api/v1/mail-jobs` |
| Git 브랜치 | `feat/케밥-설명` | `feat/csv-import` |
| UI 태그 | `<영역>-<유형>-<식별>` 대문자 (03) | `PLT-BTN-TEST` |
| 스펙 ID | `API-001`, `SCR-001`, `BAT-001`, `LLM-001`, `T-001` | |
| Python 코드 | PEP 8 (snake_case) | `def send_mail():` |
| TS/React 코드 | 컴포넌트 PascalCase, 변수·함수 camelCase | `MailForm.tsx`, `fetchJobs()` |
| Java 코드 | 클래스 PascalCase, 변수·메서드 camelCase | `MailService` |

## 3. 경계면 변환 규칙

### TS ↔ JSON 🧩
- 상태: 🧩 기본값
- 결정: API 타입은 JSON 그대로 snake_case로 선언, 변환 레이어 없음. 프론트 내부 전용 변수만 camelCase
- 결정일:

### Python ↔ JSON 🧩
- 상태: 🧩 기본값
- 결정: Pydantic 기본(snake_case) 그대로
- 결정일:

### Java ↔ JSON 🧩
- 상태: 🧩 기본값 (Java 미사용 시 삭제)
- 결정: Jackson `PropertyNamingStrategies.SNAKE_CASE` 전역 설정
- 결정일:

---

## 4. 업무 용어 사전 (Domain Glossary) 📋

> 업무정의서의 업무 용어를 **하나의 영문 식별자**로 고정하는 표. AI가 같은 개념을 파일마다 다른 이름(member/user/customer)으로 짓는 것을 막는다.
> 업무정의서 흡수(AGENT_GUIDE Phase 1)에서 AI가 초안을 만들고 🔸 추정으로 표시 → 인터뷰에서 확인받아 ✅.

- 상태: ❓ 미결정
- 결정일:

### 사전 운영 규칙 🧩
- **1용어 1식별자**: 한 업무 용어에는 영문 식별자 하나만 쓴다. 동의어(`member`/`user`)를 혼용하지 않는다. 금지 동의어는 표에 적는다
- **사전 먼저**: 코드·DB·API·UI 태그에 새 업무 용어가 필요하면 이 표에 먼저 추가한 뒤 쓴다
- **업무정의서 표현 우선**: 업무정의서(또는 의뢰자)가 쓰는 한국어 표현을 그대로 `업무 용어` 칸에 둔다. 화면 문구도 이 표현을 쓴다
- **식별자 짓기**: 영어 단수 명사, 업계 표준 용어 우선. 어색한 직역·로마자 표기(`hoewon`)는 쓰지 않는다. 약어는 5장 약어 표에 있는 것만
- **파생 규칙**: 컬렉션 = snake_case 복수, API 경로 = 케밥 복수, 클래스 = PascalCase, UI 영역 코드 = 3자 대문자
- **변경**: 확정된 식별자를 바꾸면 전체 치환 + 99_DECISIONS 기록 (DB 필드명은 마이그레이션 필요)

### 엔티티 (업무 대상)

| 업무 용어 (한국어) | 식별자 | 컬렉션/테이블 | API 리소스 | 클래스·타입 | UI 영역 | 금지 동의어 | 정의 |
|---|---|---|---|---|---|---|---|
| (예: 회원) | `member` | `members` | `/api/v1/members` | `Member` | `MBR` | user, customer | 가입한 서비스 이용자 |

### 상태값 (enum)

> 상태값은 DB·API에 **영문 UPPER_SNAKE_CASE 코드**로 저장하고, 화면 표시용 한국어 라벨은 이 표가 원본이다.

| 대상 엔티티.필드 | 코드 | 화면 라벨 | 설명 |
|---|---|---|---|
| (예: `mail_job.status`) | `PENDING` | 대기 | 발송 예약됨 |

### 역할·권한

| 업무 용어 | 코드 | 설명 |
|---|---|---|
| (예: 관리자) | `admin` | 모든 기능 접근 |

### 업무 행위 (동사)

> 함수·API 동작 이름에 쓰는 동사. 같은 행위에 다른 동사를 섞지 않는다 (예: 발송은 `send`만 — `dispatch`·`deliver` 금지).

| 업무 행위 | 동사 | 예시 |
|---|---|---|
| (예: 발송) | `send` | `send_mail_job()`, `POST /api/v1/mail-jobs/{id}/send` |

## 5. 코드 체계 · 약어 📋

### 문서 ID 체계 🧩

| 대상 | 형식 | 정의 문서 |
|---|---|---|
| WBS 작업 | `T-001` | rules/08 |
| API | `API-001` | specs/10 |
| 화면 | `SCR-001` | specs/11 |
| 배치 | `BAT-001` | specs/12 |
| LLM 기능 | `LLM-001` | specs/14 |
| UI 태그 | `<UI 영역>-<유형>-<식별>` | rules/03, specs/11 |
| 에러 코드 | `UPPER_SNAKE_CASE` (예: `MEMBER_NOT_FOUND`) | rules/02 |

### 업무 코드·번호 체계 📋
- 상태: ❓ 미결정 (업무에 사람이 읽는 번호가 없으면 ➖)
- 결정: (업무정의서에 주문번호·접수번호·과정코드 등이 있으면 형식을 정한다. 예: `ORD-YYYYMMDD-0001`)
- 결정일:

### 허용 약어
| 약어 | 원어 | 비고 |
|---|---|---|
| `id` | identifier | |
| `url` | uniform resource locator | |
| `csv` | comma-separated values | |

