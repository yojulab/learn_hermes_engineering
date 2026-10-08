# META_PROMPT — 바이브 코딩 하네스 구성 프롬프트 모음

> **사람이 복사해서 AI에게 붙여넣는 문서**다. `< >` 부분만 프로젝트에 맞게 바꾼다.
> 근거: 이전 프로젝트(axfoundly, UnifiedMessagingService, codeindocker, Formatly, bidsenses 등)에서 반복 입력한 지시를 표준화했다.
> 반복 지시 중 상시 규칙이 된 것은 이미 `rules/`의 🧩 메타 기본값으로 들어 있으므로, 프롬프트에 다시 적을 필요가 없다.

---

## [A] 초기 구성 프롬프트 (새 프로젝트 첫 입력)

```text
docs/agent/AGENT_GUIDE.md 를 읽고 이 프로젝트의 하네스를 초기 구성해줘.

[입력]
- 업무정의서: @docs/<업무정의서 파일명>.md
- 참조 자료: <없으면 삭제 — 예: @docs/<화면 목업>.png, @docs/<예제 데이터>.csv>
- 실행 환경: <예: 개발 컨테이너(Linux) / macOS / WSL>
- DB: <예: MongoDB — 별도 docker 컨테이너, MONGODB_URI=mongodb://host.docker.internal:27017, MONGODB_DBNAME=<프로젝트>_dev>
- 서버 포트: <예: 3110>
- 사용 에이전트: <예: Claude Code + Antigravity(Gemini)>

[진행 방식]
1. AGENT_GUIDE의 Phase 0~4 순서대로 진행
2. 업무정의서로 rules/ 📋 항목을 먼저 채우고, specs/ 목록 표에 기능·화면·API·배치를 📝로 등록
3. 남은 ❓·🔸만 한 주제씩 선택지와 추천안으로 질문
4. 🧩 메타 기본값은 업무정의서와 충돌할 때만 바꾸고 99_DECISIONS에 기록
5. rules 확정 후 하네스 파일(CLAUDE.md, AGENTS.md, .claude/settings.json, .mcp.json,
   .env.example, restart_server.sh, stop_server.sh, .vscode/launch.json, playwright 설정) 생성
6. 글로벌 프롬프트는 그대로 유지, 기존 하네스 파일은 병합
7. 빈 서버 기동 + Playwright 스모크 통과 확인 후 "개발 시작" 확인 요청
```

### 변형 A-1. 업무정의서 없이 시작
```text
docs/agent/AGENT_GUIDE.md 를 읽고 인터뷰를 시작해줘. 업무정의서는 없어.
인터뷰가 끝나면 답변 내용을 docs/업무정의서.md 로 정리해 두고 Phase 3부터 진행해줘.
```

### 변형 A-2. 기존 프로젝트에 하네스 이식
```text
docs/agent/AGENT_GUIDE.md 기준으로 현재 프로젝트에 맞게 하네스 프롬프트 재구성해줘.
- 기존 코드·설정(.agents/, .claude/, .windsurfrules, GEMINI.md 등)을 먼저 분석해 rules/에 현재 상태로 기록
- 중복 하네스 파일은 병합, 불필요한 파일은 목록으로 보여주고 확인 후 삭제
- 폴더 위치 이동 포함, 글로벌 프롬프트는 그대로 유지
- 업무정의서: @docs/<파일>.md (없으면 삭제)
```

---

## [B] 개발 진행 프롬프트

### B-1. 업무정의서 전체 구현
```text
@docs/<업무정의서>.md 대로 마무리까지 구현.
- 부족한 부분은 판단해 진행 (판단한 내용은 보고에 명시)
- 기능마다 specs 먼저 → 구현 → Playwright 검증 → 업무 묶음별 커밋
```

### B-2. 기능 단위 요청
```text
# <기능명>
- 업무지시서(specs)에 정리 기록 후 참조해 구현
- 관련 skill·MCP 활용해 검증까지 판단해 진행

## 요구사항
- <요구 1>
- <요구 2>
```

### B-3. 검증과 이슈 해결
```text
<기능/흐름> 검증과 이슈 해결
- 흐름: <예: 회원가입 → 로그인 → 파일 업로드 → 설정 → 발송>
- 테스트 데이터: <예: @docs/<예제>.csv — 실데이터 아니면 생성해 사용>
- 실발송/실결제는 하지 말고 확인 요청   ← 필요 시 삭제
```

### B-4. 반복 자가 수정
```text
페이지 연관성 검증과 빌드·린트 오류가 모두 해결될 때까지 5회 내에서 반복 수정.
5회 안에 못 끝내면 원인과 남은 이슈 보고.
```

---

## [C] 하네스 유지보수 프롬프트

```text
<새 상시 규칙> 을 하네스에 명기해 개발 시 적용되게 구성
- 원본은 docs/agent/rules/ 해당 문서에 🧩 또는 ✅ 항목으로 추가
- CLAUDE.md / AGENTS.md 요약에도 반영
- 기존 규칙과 충돌하면 99_DECISIONS 기록
```

자주 쓰던 예:
- "특정 업무 묶음마다 commit 진행, 하네스 프롬프트에 명기" → 이미 `rules/08` 🧩
- "검증은 playwright(headless)로, pytest 제외" → 이미 `rules/07` 🧩
- "의뢰자 소통용 UI 태그 채번, 운영에선 숨김" → 이미 `rules/03` 🧩
- ".env 는 root 한 파일로 통일" → 이미 `rules/06` 🧩
- "검증 완료 시 git push/pull" → 이미 `rules/08` 🧩

---

## [D] 메타 기본값 요약 (🧩 — rules에 이미 반영됨)

| 영역 | 기본값 | 원본 |
|---|---|---|
| 문서 | 에이전트 문서는 `docs/agent/`, `docs/` 루트는 사용자 지침서·업무정의서. 한국어 작성 | AGENT_GUIDE |
| 환경변수 | 루트 `.env` 단일 파일 + `.env.example`, 서브 프로젝트도 루트 파일을 읽음 | 06 |
| 서버 제어 | `restart_server.sh` / `stop_server.sh` 루트 표준 스크립트, 포트 충돌 시 기존 프로세스 정리 | 06 |
| 디버그 | `.vscode/launch.json` 서버 디버그 구성 | 06 |
| DB | 별도 Docker 컨테이너의 DB에 접속 (앱이 DB 컨테이너를 소유하지 않음) | 04·06 |
| 검증 | Playwright headless E2E가 기본 게이트, pytest 등 단위 테스트는 요청 시만 | 07 |
| 자가 수정 | 검증 실패 시 최대 5회 반복 수정 후 보고 | 07 |
| Git | 업무 묶음 단위 커밋(Conventional Commits + 한국어), 검증 통과 후 작업 브랜치 push, main 반영은 확인 후 | 08 |
| UI 태그 | 주요 요소에 `data-ui-id="<영역>-<유형>-<식별>"` 채번, 개발 모드에서만 배지 표시 | 03 |
| 네이밍 | 경계면(파일·DB·JSON) snake_case, 언어 내부는 언어 표준 | 09 |
| MCP | 프로젝트 MCP는 루트 `.mcp.json` 한 곳. 최신 문서는 Context7 우선 | AGENT_GUIDE |
| 보안 | 문서·커밋·히스토리에 실제 시크릿 금지 | AGENT_GUIDE |
