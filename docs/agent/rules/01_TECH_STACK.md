# 01. 기술 스택 및 버전

> 문서 상태: ✅ 완료 (2026-08-04)
> 버전을 명시하지 않으면 AI마다 다른 버전을 가정해 코드가 꼬인다. 반드시 구체적인 버전까지 결정한다.

## 1. 개발 환경

### 개발 OS / 환경
- 상태: ✅ 결정 (2026-08-05 변경 — 99_DECISIONS 참고)
- 선택지: Windows / WSL(Ubuntu) / macOS / Linux — 혼용 시 각각 기재
- 결정: **개발은 Windows 유지** (팀 전체 통일 — IDE에서 직접 실행, DB만 로컬 Docker). **배포는 Docker Compose로 전 구성요소** — 학원 GPU 서버에 frontend·backend·ai·mongodb를 올린다 (2026-08-07 변경, 상세는 06)
- 이유: 개발 환경 통일은 2026-08-05 강사 지침. 컨테이너 범위는 2026-08-07 3팀 CTO 협의로 전면 확대 (이전: DB 용도로만 / 그 이전: WSL2+Ubuntu 24.04)
- 결정일: 2026-08-07 (개발 환경은 2026-08-05)

## 2. 백엔드

### 언어 및 버전
- 상태: ✅ 결정
- 선택지 예: Java 21 / Kotlin 2.x / Node.js 22 LTS / Python 3.12 / Go 1.23
- 결정: **투트랙** — Python 3.12 + Java 21 LTS
- 이유: Python = AI 모델·영상 파이프라인 생태계(얼굴 탐지 툴·OpenCV), Java = 업무 API·취업 어필. 두 스택 모두 사용 확정(2026-08-04 인터뷰)
- 결정일: 2026-08-04

### 프레임워크 및 버전
- 상태: ✅ 결정
- 선택지 예: Spring Boot 3.x / NestJS / Express / FastAPI / Django
- 결정: FastAPI(최신 stable) + Spring Boot 3.3.x 병행. **역할 경계 확정** — FastAPI: 얼굴 탐지·영상 파이프라인·탐지 이벤트 발행 / Spring Boot: 출결 판정·예외처리·대시보드 API·알림 트리거
- 탐지·식별 모델: YOLO 전제(~08-04) → 얼굴 특화 툴 비교 테스트(08-05) → **2026-08-06 확정: 탐지 YuNet + 식별 SFace** (둘 다 opencv_zoo ONNX, Apache-2.0). 선정 근거(라이선스·CPU 6.4ms·저샷 정확도)와 실측표는 `05_AI.md`
- 이유: 경계가 직관적이고 각 언어 강점 활용 (2026-08-04 인터뷰 확정)
- 결정일: 2026-08-04

### 빌드 도구 / 패키지 매니저
- 상태: ✅ 결정
- 선택지 예: Gradle(Kotlin DSL) / Maven / npm / pnpm / uv / poetry
- 결정: Python = pip + venv / Java = Gradle / 프론트 = npm
- 이유: 팀 익숙도 우선 (2026-08-04 인터뷰)
- 결정일: 2026-08-04

## 3. 프론트엔드

### 언어 및 버전
- 상태: ✅ 결정 (버전은 제안값)
- 선택지 예: TypeScript 5.x / JavaScript(ES2022+)
- 결정: TypeScript 5.x
- 이유: React와 표준 조합, 타입 안정성
- 결정일: 2026-08-04

### 프레임워크 및 버전
- 상태: ✅ 결정
- 선택지 예: Next.js 15 / React 19 / Vue 3 / NiceGUI / 없음(API만)
- 결정: React (버전 제안: 19 + Vite). 실시간 출결 현황 갱신은 **WebSocket** (2026-08-04 03_FRONTEND 인터뷰에서 확정)
- 이유: 학원 공통 체크리스트 FrontEnd 요건 충족 + 취업 어필
- 결정일: 2026-08-04

### Node.js 버전 / 패키지 매니저
- 상태: ✅ 결정
- 선택지 예: Node 22 LTS + npm / pnpm / yarn
- 결정: Node 22 LTS + npm
- 이유: 표준 조합, lockfile(package-lock.json) 기본 제공
- 결정일: 2026-08-04

## 4. 데이터베이스 (버전만 — 상세 규칙은 04_DATABASE.md)

### DBMS 및 버전
- 상태: ✅ 결정 (2026-08-05 변경 — 99_DECISIONS 참고)
- 선택지 예: PostgreSQL 16 / MySQL 8.4 / SQLite / MongoDB 8
- 결정: **MongoDB** (버전 제안: 8.x — Docker 공식 이미지 최신 stable, 개발 시작 시 고정)
- 이유: 2026-08-05 회의 강사 확정 지침 — 짧은 기간에 사전 스키마 설계 없이 JSON 바로 저장 가능. 탐지 이벤트·프레임 인덱스와도 부합. 운영: 학원 GPU 서버의 Docker + MongoDB에 팀원 IP 접속
- ✅ 충돌 해소(2026-08-05): 8/4에 기록했던 "PostgreSQL vs 강사 NoSQL 권장" 충돌은 회의에서 MongoDB로 확정되며 해소
- 결정일: 2026-08-05

## 5. 필수 도구 버전 요약표

> 모든 결정이 끝나면 AI가 아래 표를 채운다. 개발 시작 시 AI는 이 표의 버전을 기준으로 코드를 작성한다.

| 구분 | 도구 | 버전 | 확인 명령어 |
|---|---|---|---|
| 개발 환경 | Windows (+ Docker는 DB 용도만) | 10/11 | `winver` |
| 백엔드 언어 (AI/영상) | Python | 3.12.x | `python3 --version` |
| 백엔드 프레임워크 (AI/영상) | FastAPI | 최신 stable (개발 시작 시 고정) | `pip show fastapi` |
| 백엔드 언어 (업무) | Java | 21 LTS | `java --version` |
| 백엔드 프레임워크 (업무) | Spring Boot | 3.3.x | `gradle dependencies` |
| 빌드 도구 | pip+venv / Gradle / npm | - | - |
| 프론트 언어 | TypeScript | 5.x | `npx tsc --version` |
| 프론트 프레임워크 | React + Vite | 19 / 6 | `npm list react vite` |
| Node.js | Node | 22 LTS | `node --version` |
| DBMS | MongoDB (학원 GPU 서버 Docker) | 8.x (개발 시작 시 고정) | `mongod --version` |

## 6. 버전 정책

### 버전 고정 방식
- 상태: ✅ 결정
- 선택지: lockfile 커밋 필수 / 마이너 버전까지 고정 / LTS만 사용
- 결정: lockfile 커밋 필수 — `package-lock.json`(npm), `gradle.lockfile`(Gradle 잠금 활성화), Python은 pip 특성상 lockfile이 없으므로 **버전 고정된 `requirements.txt`**(`pip freeze` 기반)를 커밋. (운영 보완 2026-08-05: 직접 의존성은 `requirements.in`에 사람이 관리, `requirements.txt`는 freeze 자동 생성물 — 갱신 절차는 requirements.in 상단 주석 참조)
- 이유: 5명 팀 환경 동일성 보장
- 결정일: 2026-08-04
