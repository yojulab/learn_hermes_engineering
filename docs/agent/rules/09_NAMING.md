# 09. 네이밍 규칙

> 문서 상태: ✅ 완료 (2026-08-05)
> 프로젝트 전반의 네이밍 표준. **원칙: 팀이 주고받는 경계면(파일·DB·API·데이터)은 전부 snake_case로 통일**하고, 각 언어 코드 내부는 언어 표준을 따른다. (2026-08-05 CTO 확정)

## 1. 기본 원칙

### 프로젝트 표준 케이스
- 상태: ✅ 결정
- 결정: **snake_case** — 적용 범위는 아래 표 기준(경계면 전면 적용, 언어 내부는 표준 존중)
- 이유: 팀 산출물(파일·DB·JSON)의 일관성 확보. Java·TS 코드 내부까지 강제하면 Spring·React 생태계 관례·린터와 충돌하므로 경계면만 통일
- 결정일: 2026-08-05

## 2. 적용 표

| 대상 | 규칙 | 예시 |
|---|---|---|
| 파일·폴더명 (공통) | snake_case | `face_matcher.py`, `detection_events/` |
| MongoDB 컬렉션·필드 | snake_case 복수형 컬렉션 (04 참조) | `detection_events`, `created_at` |
| API JSON 필드 (요청·응답) | snake_case | `{"student_name": ..., "tagged_at": ...}` |
| Python 코드 (FastAPI) | snake_case (PEP 8 표준과 동일) | `def publish_event():` |
| 환경변수·.env 키 | UPPER_SNAKE_CASE | `MONGO_URI`, `SLACK_TOKEN` |
| URL 경로 | 케밥케이스 + 복수형 리소스 (02 결정 유지) | `/api/v1/attendance-records` |
| Git 브랜치 | `feat/케밥-설명` (08 결정 유지) | `feat/attendance-judge` |
| Java 코드 내부 | 언어 표준 — 클래스 PascalCase, 변수·메서드 camelCase | `AttendanceService`, `judgeAttendance()` |
| TS/React 코드 내부 | 언어 표준 — 컴포넌트 PascalCase, 변수·함수 camelCase | `AttendanceBoard.tsx`, `fetchRecords()` |
| CSS Modules 클래스 | camelCase (TS에서 `styles.xxx` 접근 편의) | `styles.alertCard` |

## 3. 경계면 변환 규칙

### Java(Spring) ↔ JSON 변환
- 상태: ✅ 결정
- 결정: Jackson `PropertyNamingStrategies.SNAKE_CASE` 전역 설정 — 코드 내부는 camelCase, 직렬화 시 자동으로 snake_case
- 결정일: 2026-08-05

### TS(React) ↔ JSON 변환
- 상태: ✅ 결정
- 결정: API 타입 정의(features/*/types)는 **JSON 그대로 snake_case로 선언** — 변환 레이어 없이 응답 필드를 직접 사용. 프론트 내부 전용 변수만 camelCase
- 결정일: 2026-08-05

### Python(FastAPI)
- 상태: ✅ 결정
- 결정: Pydantic 기본이 snake_case이므로 별도 설정 불필요
- 결정일: 2026-08-05
