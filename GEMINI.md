# GEMINI.md — eCare24 이러닝 프로젝트 하네스 헌장

> **Antigravity & Gemini 하네스 엔지니어링 시스템 가동 중**
> 본 프로젝트는 Claude Code의 `revfactory/harness` 및 `ralph-loop` 아키텍처를 Google Antigravity 및 Gemini 실행 환경에 맞춰 완벽히 재구성하고, **`TASK_SPECIFICATION_ECARE24.md` (Rev 1.2)** 업무지시서와 **`caregiver_terminology_dictionary.md`** 임상 전문 용어 사전, 그리고 **의뢰자 소통용 UI 태깅 인스펙션 시스템**에 최적화된 하네스 규칙 체계를 준수합니다.

---

## 1. 하네스 시스템 트리거 및 라우팅 포인터

사용자의 요청 키워드에 따라 아래 스킬과 룰을 즉시 참조하여 작업을 수행합니다:

| 사용자 요청 키워드 | 대상 스킬 / 룰 | 설명 |
| :--- | :--- | :--- |
| **"하네스 구성해줘"**, **"하네스 설계"**, **"에이전트 팀 만들어줘"** | [.agents/skills/harness/SKILL.md](file:///apps/project_ecare24_elearning/.agents/skills/harness/SKILL.md) | 프로젝트 도메인 분석 후 최적의 에이전트 팀과 스킬 설계 |
| **"하네스 회고"**, **"하네스 피드백 반영"**, **"하네스 진화"** | [.agents/skills/harness-evolve/SKILL.md](file:///apps/project_ecare24_elearning/.agents/skills/harness-evolve/SKILL.md) | 실행 델타 포착, 원리 일반화, 룰 및 에이전트 지속 개선 |
| **"루프 실행"**, **"ralph loop"**, **"끝날 때까지 반복"**, **"자율 루프"** | [.agents/skills/ralph-loop/SKILL.md](file:///apps/project_ecare24_elearning/.agents/skills/ralph-loop/SKILL.md) | Ralph Loop 자율 반복 실행, 완료 약속 및 서킷 브레이커 관리 |
| **UI 일련번호 태깅 & 단축키 인스펙션** | [.agents/rules/ui-tagging-inspection.md](file:///apps/project_ecare24_elearning/.agents/rules/ui-tagging-inspection.md) | 페이지/태그 일련번호 체계, `Alt+I` 단축키 토글, 뱃지 클릭 시 복사 |
| **간병 임상 용어 사전 & 10대 골든 룰** | [.agents/rules/caregiver-terminology.md](file:///apps/project_ecare24_elearning/.agents/rules/caregiver-terminology.md) | [caregiver_terminology_dictionary.md](file:///apps/project_ecare24_elearning/docs/caregiver_terminology_dictionary.md) 기반 임상 전문 용어, 약어 및 10대 골든 룰 |
| **eCare24 도메인 & PRD 규격** | [.agents/rules/ecare24-domain.md](file:///apps/project_ecare24_elearning/.agents/rules/ecare24-domain.md) | 16대 질환군, 실버 UX, 콜센터 3분할, 자격증, 간병24TV, DB DDL 스펙 |
| **Git 커밋 규약 & 업무 종료 수칙** | [.agents/rules/git-workflow.md](file:///apps/project_ecare24_elearning/.agents/rules/git-workflow.md) | 업무 종료 시 Git Commit 필수화, 원자적 커밋, 메시지 컨벤션 |
| **하네스 아키텍처 & 품질 원칙** | [.agents/rules/harness-engineering.md](file:///apps/project_ecare24_elearning/.agents/rules/harness-engineering.md) | 6대 팀 패턴, 5대 품질 검증, 점진적 공개 원칙 |
| **루프 엔지니어링 원칙 & 서킷 브레이커** | [.agents/rules/loop-engineering.md](file:///apps/project_ecare24_elearning/.agents/rules/loop-engineering.md) | Iteration > Perfection, 상태 디스크 영속화, 서킷 브레이커 규정 |

---

## 2. eCare24 핵심 업무 지시 원칙 및 개발 가드레일

1. **기획 및 상세 아키텍처 수립 단계 준수 (비개발 원칙)**:
   - **※ 본 단계에서는 실제 소스코드 개발을 진행하지 않습니다.**
   - 개발팀, 기획팀, 인프라팀이 상호 합의하고 즉시 스프린트에 착수할 수 있도록 구체적인 스펙, WBS, 인터페이스 정의 및 아키텍처 수립에 집중합니다.
2. **의뢰자 소통용 UI 일련번호 태깅 및 단축키 뱃지 복사 규정**:
   - 모든 화면과 컴포넌트 태그에 고유 일련번호(`[P01-SEC01-BTN01]`)를 부여합니다.
   - 단축키 **`Alt + I`**로 뱃지 표시/숨김 토글을 지원하고, **뱃지 클릭 시 일련번호가 클립보드에 자동 복사(Copy)**되어 의뢰자가 피드백에 1초 만에 활용할 수 있도록 구현합니다.
3. **간병 실무 필수 전문 용어 및 10대 골든 룰 준수**:
   - 모든 교안, CBT 시험 문항, 콜센터 상담 템플릿 작성 시 [caregiver_terminology_dictionary.md](file:///apps/project_ecare24_elearning/docs/caregiver_terminology_dictionary.md)의 전문 용어(피딩, 엘튜브, PEG, 폴리, 유린백, 체변, 앙와위/측위 등)와 **"현장 압축 10대 골든 룰"**(탈건착환, q2hr 체변, 유린백 방광 하단 거치, 석션 15초 이내 등)을 절대 준수합니다.
4. **알맞은 업무 종료 시 Git Commit 의무화**:
   - 모든 에이전트는 기획/아키텍처 문서 생성, 룰 갱신, 검증 완료 등 유의미한 작업 단위를 완수했을 때 **반드시 원자적 Git Commit을 수행**합니다.
   - 커밋 전 `.env` 등 민감 정보 누출 방지 및 `.gitignore` 적용 여부를 엄격히 확인합니다.
5. **4대 프로젝트 타겟 분리 구조**:
   - `frontend_user/` (수강생 포털, 실버 친화적 초직관 UX, WCAG AAA)
   - `frontend_admin/` (관리자 & 콜센터 CS 3분할 원스크린, CTI 단축키 연동)
   - `backend/` (NestJS Clean Architecture, 8대 핵심 모듈, PostgreSQL 16+ DDL)
   - `mobileApp/` (React Native 실버 터치, 오프라인 교안 캐싱, 숏폼 스와이프)
   - `tests/` (Playwright Headless 중요 화면 자동 캡처 슈트)

---

## 3. eCare24 하네스 전담 에이전트 팀 (Roles)

| 에이전트 롤 | 모델 티어 | 주 임무 및 전담 산출물 |
| :--- | :--- | :--- |
| **System Architect** | Gemini Pro | 전체 4대 멀티 프로젝트 구조, DB DDL 스키마, .env 규격, .vscode 통합 디버그 환경 설계 |
| **Silver UX Specialist** | Gemini Pro/Flash | 수강생 포털(`frontend_user`) 및 모바일앱 실버 접근성(폰트 조절 `[가-/가/가+]`, 고대비, OMR CBT, 이어보기, 용어 툴팁, **UI 일련번호 태깅 명세**) |
| **Callcenter CS Specialist** | Gemini Pro/Flash | 관리자 및 콜센터 포털(`frontend_admin`) 3분할 원스크린 레이아웃, CTI 팝업 연동, 상담 단축키(Alt+1~3, Alt+S), 임상 약어 템플릿, **콜센터 UI 일련번호 설계** |
| **Caregiver Cert Specialist** | Gemini Pro | 16대 질환군 4단계 커리큘럼, 직능원(pqi.or.kr) 민간자격 관리규정, CBT 출제 및 과락 판정 엔진, 임상 용어 Care Point 반영 |
| **Media TV24 Specialist** | Gemini Flash | 간병24TV 유튜브 API 연동, 1분 숏폼 스와이프 플레이어 및 무료 교안 유입 퍼널 설계 |
| **Quality & E2E Assurer** | Gemini Pro | Playwright Headless 16개 질환/콜센터 자동 캡처 시나리오, **UI 일련번호 기반 안정적 셀렉터 검증**, 보안/개인정보 암호화 감사 |

---

## 4. 하네스 변경 이력 (Harness Changelog)

| 일자 | 변경 내용 | 대상 | 사유 |
| :--- | :--- | :--- | :--- |
| **2026-10-12** | Gemini 맞춤형 하네스 규칙 체계 최초 수립 | 전체 하네스 시스템 | Claude Code `harness@harness-marketplace` 기반 Gemini/Antigravity 환경으로 완벽 재구성 |
| **2026-10-12** | 루프 엔지니어링(Ralph Loop) 체계 추가 구성 | `ralph-loop` 스킬, `loop-engineering.md`, `hooks.json` | `ralph-loop@claude-plugins-official` 아키텍처를 Antigravity Stop Hook, Fresh Context Bash 러너 및 서킷 브레이커로 완벽 재구성 |
| **2026-10-12** | eCare24 PRD 사양 반영 및 Git Commit 의무화 | `ecare24-domain.md`, `git-workflow.md`, `GEMINI.md`, `AGENTS.md` | `TASK_SPECIFICATION_ECARE24.md` (Rev 1.2) 업무지시서 요구사항 완벽 반영 및 업무 종료 시 Git commit 룰 탑재 |
| **2026-10-12** | 간병 실무 필수 전문 용어 사전 룰 연동 | `caregiver-terminology.md`, `ecare24-domain.md`, `GEMINI.md`, `AGENTS.md` | `caregiver_terminology_dictionary.md` 기반 임상 전문 용어 및 10대 골든 룰 준수 체계 탑재 |
| **2026-10-12** | 의뢰자 소통용 UI 태깅 & 인스펙션 뱃지 시스템 구축 | `ui-tagging-inspection.md`, `ecare24-domain.md`, `GEMINI.md`, `AGENTS.md` | 의뢰자-에이전트 간 정밀 피드백을 위한 페이지/태그 일련번호 체계, `Alt+I` 단축키 토글, 뱃지 클릭 복사(Copy) 규격 수립 |
