# 09. 네이밍 규칙

> 문서 상태: 🧩 기본값 적용
> **원칙: 시스템 간 경계면(파일·DB·API·데이터)은 snake_case로 통일**하고, 각 언어 코드 내부는 언어 표준을 따른다.
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
