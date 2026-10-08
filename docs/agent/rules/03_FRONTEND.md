# 03. 프론트엔드 설계 규칙

> 문서 상태: ✅ 완료 (2026-08-04)
> React 19 + Vite 6 + TypeScript 5.x (버전은 `01_TECH_STACK.md` 참고). 행정실 대시보드가 주 화면.

## 1. 기본 구조

### 렌더링 방식
- 상태: ✅ 결정
- 선택지: SPA(CSR) / SSR / SSG / 서버 렌더링 GUI(NiceGUI 등)
- 결정: SPA(CSR) + React Router
- 이유: 내부 대시보드라 SEO 불필요, Vite 조합으로 가장 단순
- 결정일: 2026-08-04

### 라우팅 규칙
- 상태: ✅ 결정
- 결정: React Router — `/` 출결 현황 대시보드, `/alerts` 미태그 포착 알람, `/exceptions` 예외 처리 등록, **`/enroll` 얼굴 등록(SCR-004, 2026-08-12 추가)**. 상세 라우트는 개발하며 추가
- 결정일: 2026-08-04 (2026-08-12 `/enroll` 추가)

### 폴더 구조
- 상태: ✅ 결정 (2026-08-05 CTO 확정 — hooks·styles 폴더 없음)
- 결정: features 도메인 기준 — 백엔드 도메인별 패키지 구조(02)와 동일 철학.
  **hooks 폴더는 두지 않는다** — 데이터 훅(TanStack Query)은 `features/*/api/`에, 테마 훅은 `app/providers.tsx`에 함께 둔다.
  **styles 폴더도 두지 않는다** — 전역 CSS·테마 토큰은 `src/global.css` 단일 파일로 관리(컴포넌트 스타일은 각자 *.module.css)
```
src/
├── app/              # main.tsx · providers.tsx(QueryClient+테마) · router.tsx
├── pages/            # 라우트별 페이지 — feature 컴포넌트 조립만 담당 (Dashboard·Alerts·Exceptions)
├── components/
│   ├── layout/       # Header·Sidebar·PageLayout (+테마 토글)
│   └── ui/           # Card·Table·Badge 등 공통 UI (+ *.module.css)
├── features/
│   ├── attendance/   # 출결 현황 — components/ api/ types/
│   ├── detection/    # 미태그 포착 알람 (동일 하위 구조)
│   └── exception/    # 예외 처리 등록 (동일 하위 구조)
├── lib/              # axios 클라이언트(api.ts)·WebSocket 클라이언트(ws.ts)·유틸
├── mocks/            # 목업 데이터 (8/11 목업용 — API 연동 전까지 features/*/api가 반환)
├── types/            # 공통 타입
└── global.css        # 전역 CSS·라이트/다크 테마 토큰
```
- 결정일: 2026-08-04 (2026-08-05 구조 확정)

## 2. 상태 관리

### 서버 상태 (API 데이터)
- 상태: ✅ 결정
- 선택지: TanStack Query / SWR / 직접 fetch
- 결정: TanStack Query
- 이유: 캐싱·로딩·에러·자동 재요청 처리. WebSocket 수신 이벤트는 쿼리 캐시 무효화/갱신으로 연결
- 결정일: 2026-08-04

### 클라이언트 상태 (UI 상태)
- 상태: ✅ 결정
- 선택지: 컴포넌트 로컬 state만 / Zustand / Jotai / Redux Toolkit
- 결정: 컴포넌트 로컬 state만 (필요해지면 Zustand 추가 — 그때 99_DECISIONS 기록)
- 이유: 대시보드 규모에 전역 상태 라이브러리는 과함. 단, 테마(라이트/다크)는 Context + localStorage로 관리
- 결정일: 2026-08-04

## 3. 스타일링

### 스타일링 방식
- 상태: ✅ 결정
- 선택지: Tailwind CSS / CSS Modules / styled-components / UI 라이브러리(shadcn/ui, MUI 등)
- 결정: **CSS Modules**
- 이유: 팀 선택(2026-08-04 인터뷰) — 의존성 최소, 순수 CSS 역량 어필. 컴포넌트는 직접 제작
- 결정일: 2026-08-04

### 디자인 기준
- 상태: ✅ 결정
- 결정: **라이트 + 다크모드 모두 지원** — CSS 변수(디자인 토큰)를 `styles/`에 정의하고 `data-theme` 속성 토글 방식(CSS Modules와 호환). 초기값은 시스템 설정 따름 + 수동 토글 제공, 선택은 localStorage 저장. 색상 팔레트는 대시보드 첫 화면 설계 시 AI가 제안 → 확인
- 결정일: 2026-08-04

## 4. API 통신

### HTTP 클라이언트 / 공통 처리
- 상태: ✅ 결정
- 선택지: fetch 래퍼 / axios (+ 인터셉터로 토큰·에러 공통 처리)
- 결정: axios + 인터셉터 (공통 응답 포맷 `{success,data,error}` 언래핑·에러 공통 처리). 실시간 수신은 **WebSocket** (Spring 서버 연결, 재연결 로직 포함 — 2026-08-04 확정, 01의 "WebSocket 또는 SSE" 항목 해소)
- 결정일: 2026-08-04

### API 타입 관리
- 상태: ✅ 결정
- 선택지: 수동 타입 정의 / OpenAPI 코드 생성 / 백엔드와 타입 공유
- 결정: 수동 타입 정의 (features/*/types) — 여유 시 Swagger(OpenAPI) 기반 코드 생성으로 전환
- 결정일: 2026-08-04

## 5. 지원 범위

### 대상 디바이스 / 반응형
- 상태: ✅ 결정
- 선택지: 데스크톱만 / 모바일 우선 / 완전 반응형
- 결정: 데스크톱 전용 (행정실 모니터 상시 표시가 주 사용 시나리오 — 유저 스토리 ② 참고)
- 결정일: 2026-08-04

### 지원 브라우저
- 상태: ✅ 결정
- 결정: 최신 Chrome·Edge만
- 결정일: 2026-08-04
