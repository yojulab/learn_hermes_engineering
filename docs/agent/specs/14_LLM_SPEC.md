# 14. LLM 기능 명세 (관제 어시스턴트·가상 조교)

> 문서 상태: 🔄 작성 (2026-08-11, v4) — 기능·실행·안전장치·**모델 gemma4:26b 확정**. LLM은 **다음 주 고도화 기간**에 구현(팀회의).
> 배경: 8/10 강사 LLM 추가 지시 + Slack 제안 4종 → 8/11 팀회의로 **우선순위 2기능 확정**.
> 기존 문서(rules/05)의 LLM 용도(SMS 문구 생성 단독)는 이 문서로 **대체**된다 (99_DECISIONS 2026-08-10).
>
> **⚠ 2026-08-11 팀회의 반영 (우선순위 2기능):**
> - **① 문자 답장 이벤트 → Slack 알림** (= 3장 LLM-003 가상 조교의 수신 감지 부분). 학원 발송 문자에 답장이 오면 수신을 API 이벤트로 감지해 Slack 알림.
> - **② 검색/알리바이 어시스턴트** (= LLM-001 확장). tool calling으로 MongoDB 조회 → "○○ 몇 시 퇴실" + **로컬 드라이브 사진 첨부**(역학조사·알리바이식). **"검색엔진"=별도 인프라 아님, LLM 툴콜+MongoDB 조회.**
> - 모델 = **gemma4:26b**(GPU 서버 Ollama, 검증 완료). ~~qwen3·gemma3~~ 후보 종결.
> - **웹발신(031)**: 개발/시연은 솔라피+`SMS_MODE=test` 유지. **학원 전용 SMS 연동은 실배포 시점에 어댑터 교체** (지금 발송 경로 변경 없음).
> - LLM-004(넛지)는 로드맵 유지하되 우선순위 낮음이었으나, **2026-08-25 구현 완료**(이슈 #225, A안 채택 — 45자 강제 검증). LLM-005(사유 플래그)도 별도로 우선순위 낮음 상태에서 진행됨(#226).

---

## 0. 결정 요약

| 항목 | 결정 | 근거 |
|---|---|---|
| 근거 확보 방식 | **tool calling + context window 직접 주입. 벡터 검색 안 씀** | 출결 데이터는 정형 — 실제 API 호출 결과로만 답해야 수치 할루시네이션이 구조적으로 차단된다. 강사도 동일 방침("Vector 없이 Context Window만으로 해결") |
| 실행 위치 | **학원 GPU 서버에서 Ollama 구동**, webapp(FastAPI)에 assistant 라우터 | 강사 지시(학원 GPU 활용) + 학생 이름이 외부 API로 나가지 않음(개인정보) + 과금 없음 |
| 저장소 | **MongoDB 유지. pgvector·Neo4j 모두 보류** | 벡터 검색을 쓰지 않으므로 pgvector도 도입 이유가 없다. 그래프 관계는 전부 1단계라 Neo4j는 얻는 쿼리 없이 운영 부담만 는다 |
| backend 영향 | 최소 — 어시스턴트·조교의 도구는 기존 Spring API를 호출. 신규는 예외 제안 승인 흐름(LLM-003) 정도 | 팀원 코드 무손상 원칙. 팀원들이 만든 API가 LLM의 도구로 재사용되는 그림 |
| 쓰기 작업 원칙 | **LLM은 제안까지만, 실행은 사람 승인 후** (LLM-003) | 학생 문자·사유 텍스트는 신뢰할 수 없는 입력 — 3장 참조 |

## 1. 기능 로드맵 (강사 제안 4종 + 어시스턴트)

| ID | 기능 | 유래 | 단계 | 예상 |
|---|---|---|---|---|
| **LLM-001** | **관제 자연어 어시스턴트** — 대시보드에서 "오늘 안 나간 사람 누구야?" → tool calling으로 실데이터 답변 | 기존 계획 | **Phase 1** | 2~3일 |
| **LLM-002** | **일일 이상 흐름 브리핑** — 어제 하루 JSON(명단·알람·예외·문자 이력)을 프롬프트에 통째로 주입 → 사람의 언어로 된 Anomaly Report를 **PDF 리포트로 만들어 메일(SMTP) 발송** (2026-08-25 변경, 99_DECISIONS) | 강사 ③ → **민선아**(#224) | **Phase 1** | 1일 |
| **LLM-003** | **양방향 가상 조교** — 학생 답장("화장실이에요")을 LLM이 판독 → **예외 등록을 '제안'으로 생성 → 행정실 원클릭 승인 시 API-005 실행** | 강사 ① | Phase 2 | 2~3일 |
| **LLM-004** | **넛지 동적 메시지** — 고정 골격(기관명·요청 행동) + LLM이 어투 한 줄 변주 | 강사 ② | Phase 2 — **✅ 구현 완료(8/25, #225)** | 1일 |
| **LLM-005** | **예외 사유 확인 권장 플래그** — 예외 사유 텍스트의 타당성을 Few-shot 규정으로 판별, 의심 건만 대시보드 하이라이트 | 강사 ④ | Phase 3 | 1일 |

- Phase 1이 발표 데모의 중심. 8/31까지 안 되는 것은 뒤에서부터 잘라낸다.
- ⚠ **pgvector 런북 Q&A(구 LLM-003)는 보류** — 벡터 불요 방침으로 우선순위 상실. **Neo4j도 보류** — "그래프로만 풀리는 질문"이 생기기 전까지 다시 제안하지 않는다.
- ⚠ LLM-003의 **문자 실수신**은 별도 확인 항목 — 솔라피 수신(인바운드) 지원 여부·수신 번호 요건을 확인해야 한다. 확인 전까지 시연은 **수신 시뮬레이터**(웹훅 엔드포인트에 수동 POST)로 진행 — LLM 판독·제안·승인 흐름은 동일하게 데모된다.

## 2. 아키텍처

```
[React 헤더 어시스턴트 패널]
    │  질문: POST /api/v1/assistant/query
    │  응답: SSE (text/event-stream) — 토큰 단위 스트리밍
    ▼
[webapp FastAPI — routers/assistant.py]
    │  StreamingResponse(media_type="text/event-stream")
    │  Ollama /api/chat (stream: true) ↔ 도구 실행 루프
    ▼                                    ▼
[Ollama — GPU 서버(운영)]         [Spring backend 기존 조회 API]
                                          │
               LLM-003 승인 시에만 → API-005 (예외 등록, 사람 클릭이 트리거)

[학생 답장 SMS] ── (수신 웹훅/시뮬레이터) ──→ [webapp FastAPI]
```

> **⚠ 2026-08-14 전환: HTTP JSON → SSE 스트리밍 (CTO 결정, 이슈 #152)**
> - **질문**: 클라이언트가 `POST /api/v1/assistant/query` (일반 JSON body)
> - **응답**: `Content-Type: text/event-stream` — Ollama stream 모드의 토큰을 SSE `data:` 이벤트로 중계
> - **프론트**: 헤더 영역 어시스턴트 패널에서 `fetch` + `ReadableStream` (또는 `EventSource`)으로 수신
> - **이유**: LLM 답변은 서버→클라이언트 단방향 스트리밍 — SSE가 정확한 용도. WebSocket은 양방향이라 과잉
> - 기존 WebSocket(Spring 이벤트 피드, API-008)과 **공존** — 용도가 다름(03_FRONTEND)

- **배포(compose)**: Ollama를 `docker-compose.yml`의 서비스로 편입 — 프론트·백엔드·AI·DB와 함께 `compose up` 한 번에 뜬다. `ai`는 `OLLAMA_BASE_URL=http://ollama:11434`로 호출, 모델은 `ollama-pull` 원샷 서비스가 볼륨(`ollama_models`)에 받아둔다.
  - ⚠ **실행 위치 = 호스트(e8000)**. 배정 컨테이너 `gpu_user16` **안에서는 compose가 불가**(2026-08-10 실측: 도커 데몬·소켓·nvidia-container-toolkit 모두 없음, 컨테이너에 특권 부여는 학원 권한). Docker-in-Docker는 포기하고 호스트에 올린다 — **호스트 배포 허용은 대표님 확인 필요**.
  - ⚠ **GPU 번호 고정**: compose는 기본 GPU 0을 잡는데 공용 서버라 다른 조와 겹친다. 우리 배정(예: 3번)을 `.env`의 `OLLAMA_GPU_DEVICE`로 지정. nvidia-container-toolkit은 호스트엔 있다(실측).
- **개발(맥)**: 앱은 맥에서, 모델은 GPU 서버 것을 SSH 터널로 쓴다 — `ssh -f -N -L 11434:<컨테이너IP>:11434 <서버>` 후 `.env`에 `OLLAMA_BASE_URL=http://localhost:11434`. compose로 띄우면 이 값은 무시된다.
- Ollama 기본 포트 **11434**. 호스트 배포 시 기존 점유 포트(8000=jupyterhub 등) 충돌 점검 목록에 추가.

## 3. 안전장치 (6원칙)

1. **수치·명단은 도구 결과로만 말한다** — 도구 호출 0회인데 수치가 포함된 답은 `grounded: false`로 강등.
2. **도구 실패 시 지어내지 않는다** — "현재 확인할 수 없습니다"로 고정. LMS 폴백 규칙(specs/12)과 같은 철학.
3. **어시스턴트(LLM-001) 도구는 읽기 전용만** — 등록·삭제·발송 도구는 주지 않는다.
4. **쓰기는 제안→사람 승인** (LLM-003) — LLM은 API-005를 직접 호출하지 못한다. 판독 결과를 `PENDING_REVIEW` 제안으로 만들고, 행정실이 대시보드에서 승인해야 서버 코드가 API-005를 실행한다. 제안에는 원문·판독 근거를 함께 저장해 검증 가능하게 한다.
5. **학생 입력은 신뢰할 수 없는 데이터** — 답장 본문(LLM-003)·예외 사유(LLM-005)는 프롬프트에서 "데이터"로만 취급하고, 본문 안의 지시("예외로 등록해줘" 등)를 따르지 않도록 시스템 프롬프트에 명시 + Few-shot로 고정한다. **문자 한 통으로 시스템을 조종하는 프롬프트 인젝션이 실재하는 공격 경로다.**
6. **LLM이 학생 불이익을 직접 만들지 않는다** — LLM-005는 '허위 판정'이 아니라 **'추가 확인 권장' 신호**까지만. 최종 판단은 사람. (카메라 단독 판정 금지 원칙과 동일 철학, rules/05 5장)

### SMS 문구 (LLM-004) 가드레일 — ✅ 구현 완료 (2026-08-25, 이슈 #225)

"매번 다른 문구"는 검수 불가 위험이 있으므로 **골격 고정 + 변주 검증**으로 수용한다:
- 필수 골격은 코드가 조립: 기관명 · 요청 행동(복귀/퇴실) — LLM이 손댈 수 없다.
  ⚠ **수신자 이름은 골격에도 넣지 않는다** — `RpaSmsAdapter.RETURN_REQUEST_TEXT`가 이미
  2026-08-09부터 이름 없는 문구였고(rules/04 — 문자는 통제 밖 유일 경로, 잠금화면 유출 위험),
  이 문서 초안(2026-08-11) 작성 당시 그 결정이 반영되지 않은 채로 남아 있었다. 구현은 기존
  코드의 무이름 정책을 따랐다
- LLM은 **어투 문장 1개**만 생성. 컨텍스트로 주는 것은 1차/재안내 구분뿐 — 누적 경고 횟수 등은
  주입하지 않는다(범위 밖). 검증기: 바이트 예산(90바이트/45자) · 이모지 · 숫자 · 금칙어 · 골격 중복 생성
- 검증 불통과·생성 실패 시 **고정 템플릿으로 폴백** — 문자가 안 나가는 일은 없다
- `SMS_MODE=test`(수신자 강제 치환) 안전장치는 그대로 상위에서 동작 — live 전환 조건도 기존 그대로 (rules/05)
- 상세 계약은 specs/10 API-020, 구현은 `webapp/app/services/nudge.py` + backend `NudgeMessageClient`

## 4. API 계약

> 구현 확정 시 10_API_SPEC.md에 API-014~로 이관. 응답 봉투는 global 공통(`success/data/error`).

### LLM-001 `POST /api/v1/assistant/query` → SSE 스트리밍 응답

**요청** (일반 JSON POST — 2026-08-25 `history` 추가):
```json
{ "question": "오늘 퇴실 안 찍은 사람 누구야?",
  "history": [ { "role": "user", "content": "…" }, { "role": "assistant", "content": "…" } ] }
```

- `history`(선택, ≤20턴): 패널의 이전 대화 — 서버는 무상태 유지, 클라이언트가 실어 보낸다.
  서비스가 최근 8턴·턴당 1,000자로 다이어트해 system과 현재 질문 사이에 끼운다.
  **맥락(정정·후속 질문)용이지 사실 근거가 아니다** — 수치는 여전히 도구로만 (프롬프트 원칙 8).
- 시스템 프롬프트에 **오늘 날짜(KST)를 동적 주입**한다 (2026-08-25) — 없으면 모델이
  "8월 24일"의 연도를 학습 시점(2024)으로 지어내 도구에 잘못된 date를 넘긴다(실측).

**응답**: `Content-Type: text/event-stream` (SSE)

```text
data: {"type":"token","content":"현재"}

data: {"type":"token","content":" 미퇴실"}

data: {"type":"token","content":" 3명입니다"}

...

data: {"type":"tool_call","tool":"get_today_summary","status":"ok"}

data: {"type":"done","grounded":true}

```

- 각 `data:` 라인은 JSON 한 줄 — `type` 필드로 구분(`token` | `tool_call` | `error` | `done`)
- `done` 이벤트 수신 시 스트림 종료
- 에러 발생 시 `{"type":"error","code":"ASSISTANT_001","message":"..."}` 후 스트림 종료
- FastAPI 구현: `StreamingResponse(generator(), media_type="text/event-stream")`
- 프론트: `fetch` + `ReadableStream` 파싱(헤더 패널 내 점진적 렌더링)

> **⚠ 이전 계약(JSON 일괄 응답)은 폐기** — 이슈 #152 참조
응답 200:
```json
{
  "success": true,
  "data": {
    "answer": "현재 미퇴실 3명입니다: 홍길동, 김철수, 이영희. (18:42 기준)",
    "grounded": true,
    "tool_calls": [ { "tool": "get_today_summary", "status": "ok" } ],
    "reasoning": "get_today_summary 결과를 보면 …"
  },
  "error": null
}
```

- `reasoning`(2026-08-14, PR #148): gemma4 사고 과정(thinking) — 답변이 어떤 도구 결과를 근거로 나왔는지 추적용. 모델이 안 주면 `null`.

### LLM-001 `POST /api/v1/assistant/query/stream` — **화면 기본 경로** (2026-08-14 재설계)

같은 요청 본문, 응답은 SSE(`text/event-stream`). 대용량 질의도 첫 토큰부터 보이고, 바이트가 계속 흐르므로 프록시 타임아웃에 걸리지 않는다.

```
data: {"type": "reasoning", "delta": "…"}     ← 사고 과정 (선택적, 여러 번)
data: {"type": "answer",    "delta": "…"}     ← 답변 본문 (여러 번)
data: {"type": "tool", "tool": "get_today_summary", "status": "ok"}
data: {"type": "done", "data": { /query와 같은 data }}     ← 항상 마지막
data: {"type": "error", "code": "ASSISTANT_001", "message": "…"}   ← 실패 시 (스트림을 연 뒤라 HTTP는 200)
```

### LLM-002 브리핑 `POST /api/v1/briefing/run` · `GET /api/v1/briefing/status` (2026-08-25, #224)

자동 실행(아침 tick)과 별개인 **수동 입구**다. 개발·시연에서 09:00을 기다릴 수 없고, 배치가 놓친 날을 다시 만들 방법도 있어야 한다.

**요청**
```json
{ "date": "2026-08-25", "dry_run": true, "course": "심화_AI기반 지능형 솔루션 개발 과정" }
```
- `date`(선택): 생략 시 **어제**. 오늘까지 허용하고 미래는 거절(`BRIEFING_002`) — 없는 데이터를 0으로 읽어 "평온한 하루"가 되기 때문이다. 형식 오류는 `BRIEFING_001`
- **`dry_run` 기본 `true`** — 이 엔드포인트는 부르면 학생 실명이 담긴 메일이 나갈 수 있다(C안, 99_DECISIONS 2026-08-25). 빈 본문이면 리포트만 만들고 아무것도 보내지 않는다. **보내려면 `dry_run: false`를 명시**해야 하고, 그것마저 `BRIEFING_ENABLED=true`가 아니면 발송되지 않는다(스위치 2단)
- `course`(선택, #260): 생략 시 `BRIEFING_COURSE` 설정을 따른다. ⚠ **그 설정의 기본값이 비어 있지 않다** — 아무것도 지정하지 않으면 `심화_AI기반 지능형 솔루션 개발 과정` 범위로 나간다(#260 요청). **학원 전체로 되돌리려면 `BRIEFING_COURSE=`를 빈 값으로 명시**한다. 값을 주면 **명단뿐 아니라 집계도 그 강의 범위**로 낸다 — 숫자는 전체인데 이름만 좁히면 "미퇴실 12명"에 이름이 3명만 실린 리포트가 되고, 본문의 수를 사실과 대조하는 검증기가 이를 불일치로 보아 고정 서식으로 떨어뜨린다. ⚠ `attendance_history.course_name`과 **정확히 일치**로 비교한다(부분 일치로 하면 다른 과정 학생 실명이 섞인다 — 0건으로 나오는 쪽이 낫다)

**응답 data**
```json
{ "date": "2026-08-25", "course": null, "generated_by_llm": true, "pdf_bytes": 35038,
  "sent": false, "attendance_warn": 23, "attendance_danger": 4,
  "warnings": [], "body": "…" }
```
- 어느 단계가 폴백했는지 남긴다 — `generated_by_llm: false`면 고정 서식(LLM 미기동·검증 불통과), `pdf_bytes: 0`이면 한글 폰트를 못 찾아 본문만 나간다. "문장이 밋밋하다"의 원인을 로그 없이 가른다
- `attendance_warn`·`attendance_danger`는 리포트에 실린 출석률 확인 권장 **인원수**다(#226). ⚠ **줄 수가 아니라 사람 수**다 — 절에는 안내 한 줄·"외 N명"·표본부족 안내·각주가 섞여 있어 줄을 세면 인원과 어긋난다(실측 25줄 대 27명, #250). 둘 다 0이면 그 절이 리포트에 없다는 뜻이다
- `course`는 이 리포트가 집계한 범위다(#260). `null`이면 학원 전체. 값이 있으면 그 리포트의 모든 수치가 그 강의 범위이며, 그 사실이 본문 머리줄·PDF 절·메일 제목·각주에 함께 실린다 — 전체 브리핑과 나란히 놓였을 때 숫자가 다른 이유가 리포트 안에서 설명돼야 한다
- `warnings`는 이력 소실 경고(#230) — 그날 수치를 단정하지 않게 하는 근거다

`GET /status`는 운영 확인용(13_OPERATIONS): `enabled`·`include_names`·`recipient_count`·`smtp_host`·`smtp_configured`·`send_hour`·`font`·`course`(기본 집계 범위, 빈 문자열이면 학원 전체). ⚠ **수신 주소·비밀번호는 내보내지 않는다** — 이 서버엔 인증이 없어(rules/02) 상태에 실린 주소는 곧 공개된 주소다.

**자동 실행**: webapp 안의 1분 tick + 시각 가드(`BRIEFING_SEND_HOUR`, 기본 9시). specs/12 §0의 스케줄러 결정(Spring `@Scheduled`)은 **출결 업무 배치**의 것이고, 브리핑이 쓰는 재료(LLM·통계 도구)가 전부 webapp에 있어 Spring이 깨우는 구성은 내부 API와 backend 변경을 더 만든다. webapp에 같은 형태의 주기 루프 선례가 있다(`_enroll_cleanup_loop`, 이슈 #96). ⚠ **앱이 그 시각 이후에 뜨면 그날은 건너뛴다** — 마지막 실행 날짜를 메모리에만 두기 때문이다. 중복 발송보다 낫다고 보고, 놓친 날은 위 수동 실행으로 만든다.

### LLM-003 수신 웹훅 `POST /api/v1/assistant/sms-reply` (시뮬레이터 겸용)

```json
{ "from": "010-XXXX-XXXX", "text": "저 화장실이에요 금방 들어가요" }
```
→ LLM 판독 → 제안 생성(`PENDING_REVIEW`) → WS 이벤트로 대시보드에 표시 → 행정실 승인/반려.
제안 문서: `{ student_id, 제안 유형(OUTING 등), 원문, 판독 요약, 상태 }` — 저장 위치·필드는 rules/04 갱신 시 확정.

에러 코드 (도메인 `ASSISTANT`): `ASSISTANT_001` Ollama 연결 실패(503) · `ASSISTANT_003` 질문 비어 있음/1,000자 초과(400)
- ~~`ASSISTANT_002` 타임아웃~~ 폐기(2026-08-14) — 실측 결과 "타임아웃"의 실체는 대용량 명단의 정상 계산이었다. 앱 타임아웃을 제거하고(연결 5초만 유지) 스트리밍·도구 다이어트로 해결. 8장 참조

## 5. 도구 정의 (LLM-001 — 읽기 전용 6종)

> 경로·파라미터는 10_API_SPEC의 해당 API를 그대로 따른다. 어시스턴트용 새 API를 만들지 않는다.

| 도구명 | 매핑 | 용도 |
|---|---|---|
| `get_today_summary` | API-003 | 당일 출결 요약·미퇴실 명단 |
| `list_alerts` | API-004 | 미태그 포착 알람 목록 |
| `list_exceptions` | API-006 | 당일 예외 목록 |
| `search_student` | API-010 | 학생 검색 (LMS 실시간) |
| `get_detection_status` | API-011 | 탐지 파이프라인 상태 |
| `get_checkout_windows` | API-013 | 오늘의 퇴실 창 |
| `get_checkout_history` | ⚠ **MongoDB 직접 조회** (`attendance_history`, 읽기 전용) | "○○ 몇 시에 퇴실했어?" — 특정 학생 입·퇴실 시각 |
| `get_attendance_stats` | ⚠ MongoDB 직접 조회 (`detection_alerts`+`attendance_history`) | 최근 N일 미퇴실 포착 통계·요일 분포 (#223, 2026-08-25) |
| `get_student_history` | ⚠ MongoDB 직접 조회 (`detection_alerts`+`attendance_exceptions`) | 특정 학생 최근 N주 반복 이력 — "저번에도 그랬어?" (#223) |
| `get_sms_history` | ⚠ MongoDB 직접 조회 (`notifications`) | 최근 N일 문자 발송 이력·성공률·재발송 (#223) |

- 쓰기 계열(API-005 등록·API-007 삭제·발송)은 **의도적으로 제외** — 3장 원칙 3·4.
- ⚠ **통계 도구 3종(2026-08-25, #223)은 `get_checkout_history` 예외의 연장이다** — 기간 집계는 Spring에 API가 없고 LLM-002·005의 선행 조건이라 읽기 전용 직접 조회로 확장. 집계·결론 필드(`top_weekday`·`missed_count` 등)까지 **코드가 계산**하고 모델은 문장화만 한다(§3.1). ⚠ **예외 하나(#260)**: 도구 3종은 강의 인자를 받지 않으므로, LLM-002가 `course`로 범위를 좁힌 리포트를 낼 때만 브리핑이 **같은 규칙으로 직접 센다**(미퇴실=카메라 포착 기준·사람 단위 중복 제거·재발송은 같은 학생 같은 날 2건째부터). 범위를 지정하지 않은 기본 경로는 도구 3종 그대로다. 옳은 최종 형태는 도구가 `course`를 받는 것이지만 그건 어시스턴트의 도구 계약 변경이라 별건으로 둔다. "미퇴실" 집계는 `detection_alerts`(카메라 포착) 기준이며 그 한계를 결과 `안내` 필드로 모델에 전달한다 — `attendance_history.checked_out_at`은 마감 시 항상 채워지므로(기본 18:00) 미퇴실 판정 소스가 아니다. 팀원 로컬 개발용 가상 데이터는 `tools/seed_demo_data.py`.
- ⚠ **`get_checkout_history`는 최초의 DB 직접 조회 예외다** (통계 3종이 그 연장) (2026-08-14, 99_DECISIONS). 실측에서 "○○ 몇 시 퇴실" 질문에 답할 도구가 없어 모델이 미퇴실 명단 부재를 "퇴실 안 함"으로 지어냈다. 정답이 든 `attendance_history`에는 Spring 공개 API가 없고, 신설은 backend 도메인 작업이라 **읽기 전용 직접 조회**로 결정 — webapp이 `daily_roster`를 직접 읽는 기존 선례(DetectionRecorder 잔존 게이트)의 연장이다. LLM ②(검색/알리바이)용 Spring API가 생기면 이 도구 내부만 그쪽으로 갈아탄다.
- **도구 결과는 다이어트해서 넣는다** (2026-08-14 실측): 미퇴실 66~69명 전체 JSON을 주입하면 응답이 **90초+** — 원인은 타임아웃이 아니라 입력량이다. `get_today_summary`는 집계 그대로 + 명단 상위 N명(`ASSISTANT_LIST_CAP`, 기본 15) + "외 N명" 표시, `search_student`도 같은 상한. 학번(12자리)은 답변 재료가 아니라 아예 뺀다. 모델은 프롬프트 원칙 6에 따라 생략 표시를 그대로 "외 N명"으로 전달한다

## 6. 모델 선정 — ✅ gemma4:26b 확정 (2026-08-10)

- 상태: ✅ **확정 — `LLM_MODEL=gemma4:26b`** (학원 GPU 서버 gpu_user16, Ollama 0.32.6)
- 검토한 후보:

| 후보 | 판단 |
|---|---|
| **gemma4:26b** ✅ | 채택 — 청약메이트 gemma 계열 운영 경험의 연장, 한국어 자연스러움, **네이티브 tool calling 실측 통과** |
| gemma4:12b/31b | 12b는 품질 여유 부족 우려, 31b(~20GB)도 가능하나 26b가 품질·VRAM 균형 우위. 필요 시 31b로 상향 여지 |
| qwen3 계열 | 제외 — **한국어 질의에 중국어 누출**(CTO 직접 확인) + 파인튜닝 난이도. tool calling은 강하나 우리 용도(한국어 관제)에 부적합 |

- **실측 근거 (2026-08-10, gpu_user16 · L40S 46GB)**:

| 항목 | 결과 |
|---|---|
| GPU 로드 | CUDA, `100% GPU`, **VRAM 19GB / 46GB** (얼굴 추론 동시 구동해도 절반 이상 여유) |
| 툴콜 선택 | "오늘 퇴실 안 찍은 사람?" → `get_today_summary` 정확 호출, **1.7s** |
| 도구결과→최종답변 | 자연스러운 한국어("홍길동, 김철수, 이영희입니다"), **1.9s** |
| 범위 밖 질문 | "내일 날씨?" → 도구 미호출 + **"모릅니다"** (지어내기 0건), 1.8s |
| 응답 지연 | 전부 2초 이내 — 명세 p95 15s 기준 크게 밑돎. ⚠ **소명단 기준이었다** — 실데이터(미퇴실 66~69명) 주입 실측(2026-08-14, GPU 서버)에서 **90초+**로 반증됨. 도구 다이어트·스트리밍 도입의 근거 (5·8장) |

- gemma3의 약점이던 네이티브 tools 미지원이 **gemma4에서 해결됨을 실측으로 확인**. JSON 프롬프트 우회 불필요.
- L40S 46GB 사이징(Q4 근사): 12B ≈ 9GB · 26B ≈ **17GB(실측 19GB 상주)** · 31B ≈ 20GB.
- **정식 게이트(어시스턴트 20문항·조교 인젝션 10문항)는 골격 구현 후 회귀 테스트로 상시 수행** — 위 스모크 3종은 확정을 위한 예비 검증이며, 프롬프트·도구 확정 시 전체 문항으로 재측정한다.
  - **골든 회귀 게이트 = `tools/eval_assistant_golden.py`** (2026-08-25 착수). 실제 gemma에 골든 질문을 태워 모델층 회귀를 속성으로 검증한다 — ① 날짜/연도(어제·지난주를 올해로) ② 멀티턴 코어퍼런스("그 학생") ③ grounding ④ 할루시네이션(빈 결과에 수치 안 지어냄). 목킹 단위테스트(`test_assistant.py`)가 못 잡는 **모델 판단**을 잡는다. **CI 미실행**(실 ollama+DB 필요) — 프롬프트·도구·`LLM_MODEL` 바꾸는 PR 전 `docker compose exec ai python tools/eval_assistant_golden.py --repeat 3 --threshold 0.67`로 돌리고 결과를 PR에 붙인다. 초기 베이스라인 5/5 통과(2026-08-25, GPU 서버).

## 7. 환경변수 (.env — 키 추가 시 .env.example에도 반영)

```bash
# === LLM 어시스턴트 (specs/14) ===
ASSISTANT_ENABLED=false          # 기본 꺼짐 — Ollama 없는 환경에서 다른 기능에 영향 주지 않게
OLLAMA_BASE_URL=                 # 개발: http://localhost:11434 / 운영: GPU 서버
LLM_MODEL=gemma4:26b             # 6장 실측 확정 (2026-08-10)
ASSISTANT_LIST_CAP=15            # 도구 결과 명단 상한 — 5장 다이어트 (2026-08-14)

# === 일일 브리핑 (LLM-002, #224 — 2026-08-25) ===
BRIEFING_ENABLED=false           # 자동 발송 스위치. 이 값이 곧 스케줄 스위치다
BRIEFING_INCLUDE_NAMES=true      # 리포트 실명 포함 (C안 채택, 99_DECISIONS). 학번은 무관하게 금지
BRIEFING_RECIPIENTS=             # 쉼표 구분. 실명이 실리므로 여기서만 정한다(리포·이슈 금지)
BRIEFING_SEND_HOUR=9             # 자동 발송 시각(KST 시) — 1분 tick + 시각 가드
BRIEFING_FONT_PATH=              # 비우면 nanum(컨테이너)→AppleGothic→맑은고딕 순으로 탐색
BRIEFING_COURSE=심화_AI기반 지능형 솔루션 개발 과정   # 집계 범위(정확히 일치). **기본값이 이 값**이고, 빈 값으로 명시해야 학원 전체 (#260)
SMTP_HOST=smtp.gmail.com
SMTP_PORT=587                    # STARTTLS
SMTP_USER=
SMTP_PASSWORD=                   # Gmail 앱 비밀번호 16자리 (계정 비밀번호 아님). 커밋 금지
SMTP_FROM=                       # 비우면 SMTP_USER
# LLM_TIMEOUT_SECONDS는 폐기 (2026-08-14) — 8장 타임아웃 항목 참조
```

## 8. 비기능·운영

| 항목 | 기준 |
|---|---|
| 동시성 | 1~2 (행정실 상시 화면 1대 전제) — 큐잉 없이 단순 처리 |
| 타임아웃 | **연결 5s / 생성 무제한** (2026-08-14 — 30s·`ASSISTANT_002` 폐기). 실측: 66~69명 명단 질의는 사고 포함 90초+가 **정상 계산**이다. 화면 경로는 스트리밍이라 대기가 보이고, Nginx `proxy_read_timeout 300s`(델타 사이 간격에만 적용)·`proxy_buffering off`가 상한이다. ⚠ vite dev proxy는 타임아웃이 없어 이 제약이 로컬에서 재현되지 않는다 |
| 모델 웜 유지 | **업무시간(08~22시)만 상주** (2026-08-25). 웜이면 응답 2~5초지만 ollama 기본 keep_alive 5분이라 띄엄띄엄 질의마다 **콜드 재로드 ~80초**를 문다(실측). 앱 keep_alive(`LLM_KEEP_ALIVE`)는 짧게(10m) 두고 — 길면 야간 질의가 GPU를 붙잡는다 — **하트비트 `ops/ollama_warm.sh`가 08~22시에만 빈 프롬프트로 예열**한다. 창 밖엔 자연 언로드로 GPU 반환(야간 셧다운과 정합). cron: `*/10 * * * * ops/ollama_warm.sh`(창 판단은 스크립트가). 첫 예열(08시경)은 하트비트가 무므로 사용자 대기 아님 |
| 장애 격리 | 어시스턴트가 죽어도 대시보드 본 기능·SMS 발송은 무관 (`ASSISTANT_ENABLED=false`와 동일 동작) |
| 헬스체크 | `/api/v1/assistant/status` — Ollama 연결·모델 로드 여부 (13_OPERATIONS 추가 예정) |
| 프롬프트 | `webapp/app/prompts/<name>_prompt.py` 모듈 상수 + git 버전 관리 (rules/05 4장) — 서비스가 직접 import, 로더·폴백 없음. 실존 3종(어시스턴트·브리핑·넛지). `.txt`→`.py` 전환 2026-08-28(#244·99_DECISIONS) |
| 로깅 | 질문·도구명·성공 여부만. 도구 결과(학생 명단)·답장 원문은 로그 제외, 대화 이력 무상태 |

## 9. 구현 분담·순서

| 순서 | 작업 | 담당 |
|---|---|---|
| 1 | 이 명세 리뷰·확정 (강사 제안 반영본) | CTO |
| 2 | 맥 로컬 Ollama로 LLM-001 골격 (라우터·도구 루프·프롬프트) | CTO + Claude (webapp은 CTO 영역) |
| 3 | 모델 선정 게이트 실측 — 맥에서 예비, GPU 서버에서 본 측정 | Claude 실행, CTO 판정 |
| 4 | LLM-002 브리핑 (BAT 연계) → LLM-003 조교(시뮬레이터) → LLM-004 넛지 | CTO + Claude |
| 5 | 프론트: 어시스턴트 패널 + 제안 승인 UI | 프론트 담당 팀원과 협의 |
| 6 | 솔라피 수신 지원 확인 → 실수신 연동 여부 결정 / GPU 서버 Ollama 배포 | CTO |
