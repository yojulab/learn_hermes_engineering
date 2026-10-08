# 01. 기술 스택 및 버전

> 문서 상태: ❓ 작성 전
> 버전을 명시하지 않으면 AI마다 다른 버전을 가정해 코드가 꼬인다. 반드시 구체적인 버전까지 결정한다.
> 업무정의서에 스택 지정이 없으면 💬 인터뷰로 추천안을 제시한다. 최신 stable 버전은 Context7 MCP 등으로 확인 후 제안한다.

## 1. 개발 환경

### 개발 OS / 환경 📋 💬
- 상태: ❓ 미결정
- 선택지: 개발 컨테이너(Linux) / WSL(Ubuntu) / macOS / Windows — 혼용 시 각각 기재
- 결정:
- 결정일:

### 사용 AI 에이전트 🧩
- 상태: 🧩 기본값
- 결정: Claude Code CLI + Antigravity(Gemini) 병용. 하네스 원본은 `docs/agent/`, 진입점은 `CLAUDE.md`·`AGENTS.md`(필요 시 `GEMINI.md`)
- 결정일:

## 2. 백엔드

### 언어 및 버전 📋 💬
- 상태: ❓ 미결정
- 선택지 예: Node.js 22 LTS(TypeScript) / Python 3.12 / Java 21 / Go 1.23 / 없음(프론트만)
- 결정:
- 이유:
- 결정일:

### 프레임워크 및 버전 📋 💬
- 상태: ❓ 미결정
- 선택지 예: Next.js(API Routes) / Express / NestJS / FastAPI / Spring Boot 3.x
- 결정:
- 결정일:

### 빌드 도구 / 패키지 매니저 💬
- 상태: ❓ 미결정
- 선택지 예: npm / pnpm / uv / pip+venv / Gradle
- 결정:
- 결정일:

## 3. 프론트엔드

### 언어 및 버전 💬
- 상태: ❓ 미결정
- 선택지 예: TypeScript 5.x / JavaScript
- 결정:
- 결정일:

### 프레임워크 및 버전 📋 💬
- 상태: ❓ 미결정
- 선택지 예: Next.js 15 / React 19 + Vite / Vue 3 / 서버 템플릿(Jinja 등) / 없음(API만)
- 결정:
- 결정일:

### Node.js 버전 / 패키지 매니저 💬
- 상태: ❓ 미결정
- 선택지 예: Node 22 LTS + npm
- 결정:
- 결정일:

## 4. 데이터베이스 (버전만 — 상세 규칙은 04_DATABASE.md)

### DBMS 및 버전 🧩 📋
- 상태: 🧩 기본값
- 결정: MongoDB (별도 Docker 컨테이너에서 실행 중인 인스턴스에 접속) — 업무정의서에 다른 DBMS가 있으면 덮어쓴다
- 결정일:

## 5. 필수 도구 버전 요약표

> 모든 결정이 끝나면 AI가 채운다. 개발 시 AI는 이 표의 버전을 기준으로 코드를 작성한다.

| 구분 | 도구 | 버전 | 확인 명령어 |
|---|---|---|---|
| 개발 환경 |  |  |  |
| 백엔드 언어 |  |  |  |
| 백엔드 프레임워크 |  |  |  |
| 프론트 프레임워크 |  |  |  |
| Node.js |  |  | `node --version` |
| DBMS | MongoDB |  | `mongosh --version` |
| E2E 검증 | Playwright |  | `npx playwright --version` |

## 6. 버전 정책

### 버전 고정 방식 🧩
- 상태: 🧩 기본값
- 결정: lockfile 커밋 필수 (`package-lock.json` 등). Python은 버전 고정 `requirements.txt` 또는 `uv.lock`
- 결정일:
