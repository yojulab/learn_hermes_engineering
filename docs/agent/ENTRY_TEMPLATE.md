# ENTRY_TEMPLATE — 진입점 파일 템플릿

> AGENT_GUIDE Phase 3에서 AI가 이 템플릿으로 리포 루트 파일을 만든다. `< >`는 rules 결정값으로 채운다.
> **원본은 docs/agent/** — 진입점 파일에는 요약과 링크만 두고, 규칙을 바꿀 때는 docs/agent를 먼저 고친다.

---

## 1. `AGENTS.md` (표준 진입점 — 모든 AI 공통)

```markdown
# AGENTS.md — <프로젝트명>

이 리포에서 작업하는 모든 AI 에이전트(도구 무관)는 이 파일과 `docs/agent/AGENT_GUIDE.md`를 먼저 읽는다.
세부 규칙의 원본은 `docs/agent/rules/`, 기능 명세는 `docs/agent/specs/`, 업무 원문은 `docs/<업무정의서>.md`.

## 프로젝트
- <한 줄 소개> (rules/00)
- 스택: <언어·프레임워크·DB 버전> (rules/01)

## 명령어
- 서버 재시작: `./restart_server.sh`  /  중지: `./stop_server.sh`
- 접속: http://localhost:<PORT>
- E2E 검증: `<npx playwright test>`
- 빌드·린트: `<명령>`

## 반드시 지킬 규칙
1. **문서 우선**: 새 API·화면·배치·컬렉션은 docs/agent/specs에 먼저 쓰고 구현한다.
2. **업무 용어 사전**: 이름은 rules/09 4장의 식별자만 쓴다. 새 업무 용어는 사전에 먼저 등록한다. 동의어 혼용 금지.
3. **표기**: 파일·DB·JSON은 snake_case, URL은 케밥 복수, 코드 내부는 언어 표준 (rules/09).
4. **환경변수**: 루트 `.env` 하나만 쓴다. 키 추가 시 `.env.example` 갱신. 실제 시크릿을 코드·문서·커밋에 쓰지 않는다.
5. **검증**: 구현 후 `restart_server.sh` → Playwright headless E2E → 빌드·린트. 실패 시 수정·재검증 최대 5회, 넘으면 보고.
6. **업무 묶음별 커밋**: 묶음(WBS 작업 1건 / 스펙 1건 / 버그 1건 / 문서·하네스 변경)이 검증을 통과할 때마다 즉시 커밋한다. 묶어서 나중에 커밋하지 않는다. 형식: `feat|fix|docs|chore|refactor: 한국어 요약 (관련 ID)` (rules/08)
7. **브랜치**: 작업 브랜치 `<vibe_coding>`에서만 커밋. 검증 후 작업 브랜치 push. main 반영·force push는 확인 후.
8. **UI 태그**: 주요 요소에 `data-ui-id="<영역>-<유형>-<식별>"`, specs/11 대장에 등록, 운영 빌드에선 배지 숨김 (rules/03).
9. **먼저 확인할 것**: 데이터 삭제·DB 초기화, 실발송·실결제, main 반영, 확정된 결정 변경.
10. **보고**: 한 일 / 검증 결과(실패도 그대로) / 커밋 목록 / 판단해서 정한 것 / 남은 일.

## 실행 못 하는 AI라면 (AGENT_GUIDE 0-2)
- 파일을 못 쓰면 바꿀 파일의 경로와 전체 내용을 출력한다.
- 명령을 못 돌리면 실행할 명령(서버·E2E·git commit)을 순서대로 출력하고 결과를 요청한다. 검증 전에는 "미검증"으로 보고한다.
```

---

## 2. 도구별 포인터 파일 (rules/01 "사용 AI 도구"에 있는 것만 생성)

규칙을 복사하지 않고 AGENTS.md로 보낸다. 도구가 다른 파일 임포트를 지원하면 임포트, 아니면 안내 문장.

### `CLAUDE.md` (Claude Code)
```markdown
@AGENTS.md

이 파일은 포인터다. 규칙은 AGENTS.md와 docs/agent/에 있다.
```

### `GEMINI.md` (Gemini CLI)
```markdown
@AGENTS.md

이 파일은 포인터다. 규칙은 AGENTS.md와 docs/agent/에 있다.
```

### `.agents/rules/project.md` (Antigravity) · `.windsurfrules` (Windsurf) · `.github/copilot-instructions.md` (Copilot) · `.cursor/rules/project.mdc` (Cursor)
```markdown
이 리포의 작업 규칙은 루트 `AGENTS.md`에 있다. 작업 전에 반드시 `AGENTS.md`와 `docs/agent/AGENT_GUIDE.md`를 읽고 따른다.
```
- Cursor `.mdc`는 frontmatter에 `alwaysApply: true`를 넣는다.
- 도구가 임포트 문법(`@파일`)을 지원하지 않거나 확실하지 않으면 안내 문장만 쓴다.

### 웹 채팅형 AI (ChatGPT·Gemini·Claude.ai 웹)
- 파일 생성 없음. 대화를 시작할 때 `AGENTS.md`, `docs/agent/AGENT_GUIDE.md`, 작업에 필요한 rules·specs를 첨부한다.
- 프로젝트 기능(ChatGPT Projects, Gemini Gems, Claude Projects 등)이 있으면 `AGENTS.md` 내용을 지침(instructions)에 넣고 docs/agent를 지식 파일로 올린다.
