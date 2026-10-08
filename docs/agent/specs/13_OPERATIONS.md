# 13. 운영 / 런북 (Runbook)

> 문서 상태: ✅ 작성 (2026-08-10) — GPU 서버 배포 전 선행 문서
> **장애가 났을 때 이 문서 하나만 보고 복구할 수 있어야 한다.** 새 장애를 겪을 때마다 2장과 5장에 추가한다.
>
> ⚠ 이 문서에는 **주소·계정·토큰의 실제 값을 적지 않는다.** 전부 리포 루트 `.env`에 있고, 그 파일은 커밋하지 않는다.
> 서버 접속 정보는 팀 DM으로만 공유한다 (rules/02 시크릿 관리).

---

## 1. 서비스 운영 정보

| 항목 | 내용 |
|---|---|
| 실행 위치 | 학원 GPU 서버 1대 (Docker Compose). 개발은 팀원 PC에서 같은 compose로 실행 |
| 배포 단위 | 리포 루트 `docker-compose.yml` — `frontend`(Nginx) · `backend`(Spring) · `ai`(FastAPI) · `mongodb` 4개 |
| 시작 방법 | 리포 루트에서 `docker compose up -d --build` |
| 중지 방법 | `docker compose down` (DB 데이터 유지) / `docker compose down -v` (**DB 볼륨까지 삭제 = 개인정보 파기**, 6장 참조) |
| 접속 주소 | `http://<서버>:<FRONTEND_PORT>` — GPU 서버는 **3051** (반별 포트 대역 XX51~XX99, rules/06). 로컬 개발은 `.env`에서 자유(예: 8090) |
| 정상 동작 확인 | 아래 "헬스체크" |
| 로그 위치 | **파일이 아니라 컨테이너 표준출력** — `docker compose logs -f <서비스>` |
| 설정 파일 | 리포 루트 `.env` 한 곳 (키 목록은 `.env.example`). Spring도 `spring.config.import`로 같은 파일을 읽는다 |
| 의존 외부 서비스 | ① LMS 출결 스냅샷 API ② 솔라피(문자 발송 — 알림톡 전환 검토 중, rules/05) ③ Slack(알림) — 셋 다 `.env`로 설정 |

### 최초 1회 (서버를 새로 세울 때)

```bash
cp .env.example .env      # 값 채우기 — 특히 MONGO_* 비밀번호는 실제 값으로
docker compose up -d --build
cd database && ./apply.sh # 컬렉션·인덱스·TTL·계정 권한 (멱등, 몇 번 돌려도 안전)
```

- `apply.sh`를 빼먹으면 **TTL 인덱스가 없어 개인정보가 자동 삭제되지 않는다** (rules/04) — 반드시 실행한다.
  - **빼먹었는지는 `ai` 기동 로그로 확인한다** (2026-08-13 추가). 정상이면 `TTL 인덱스 확인 — 9개 컬렉션 정상`,
    빠졌으면 `TTL 인덱스 없음: <컬렉션…>`이 ERROR로 뜬다. 서버는 그래도 뜬다 — 개인정보가 늦게 지워지는 것보다
    퇴실 감시가 통째로 서는 쪽이 당장의 피해가 크고, 나중에 `apply.sh`를 돌리면 소급 적용되기 때문이다(만료 시각은 문서에 이미 들어 있다).
    ```bash
    docker compose logs ai | grep TTL
    ```
- 시드(`./seed.sh`)는 **개발·시연용 합성 데이터**다. 실운영 DB에는 넣지 않는다.

### 헬스체크

```bash
docker compose ps                                    # 5개 모두 Up (healthy) — backend·ai도 이제 포함
curl -s http://<서버>:<포트>/api/v1/health            # Spring + Mongo
curl -s http://<서버>:<포트>/api/v1/detection/status  # FastAPI
```

| 대상 | 정상 응답 |
|---|---|
| Spring + Mongo | `{"success":true,"data":{"server":"up","mongo_db":"up"},...}` — `mongo_db`까지 함께 알려준다 |
| FastAPI | `{"success":true,"data":{"enabled":...,"monitoring":...},...}` |
| 화면 | 브라우저 접속 → 시계가 흐르고, 카메라 점·"퇴실 감시" 뱃지가 보인다 |

> ⚠ **`/health`(맨 앞에 `/api/v1`이 없는 주소)로 확인하지 말 것.**
> Nginx가 FastAPI로 넘기는 건 `/api/v1/streams`와 `/api/v1/detection/`뿐이라, `/health`는 SPA 폴백에 걸려
> **AI 서버가 죽어 있어도 HTML과 함께 200을 돌려준다.** 상태 코드만 보는 감시 스크립트라면 영영 초록불이다.
> 밖에서 AI 서버 생사를 보는 주소는 `/api/v1/detection/status`다.

> `backend`·`ai`에도 compose healthcheck가 붙어 있다(2026-08-14, 이슈 #116) — `docker compose ps`에서
> `(healthy)`가 안 뜨면 그 즉시 확인할 것. `restart: unless-stopped`는 healthcheck가 있어야 "멈춘 채
> 살아 있는" 프로세스도 재시작한다. 위 curl은 컨테이너 밖에서 직접 확인할 때 그대로 쓴다.
>
> **죽어도 사람이 화면을 봐야만 아는 시간대(새벽·주말·점심)를 위해** `ops/healthwatch.sh`가 5분
> 주기로 서비스 생사를 확인하고, 상태가 바뀔 때만(플랩 방지) 배치 실패 알림(#117)과 같은
> Slack 채널로 알린다. ⚠ **판정은 위 curl이 아니라 `docker inspect`로 compose healthcheck
> 결과를 읽는다** (2026-08-20 로컬 우선 전환 — backend·ai가 호스트 포트를 안 열어 호스트
> curl은 항상 실패한다, 99_DECISIONS #116). **CTO 맥북 crontab** 등록: `ops/healthwatch.sh`
> 파일 머리말 참조. (위 curl 주소는 컨테이너 안에서·직접 확인할 때 여전히 유효하다.)

### DB를 직접 볼 때 — 시각은 KST로 변환해서 읽는다 (2026-08-14)

몽고는 시각을 **UTC로 저장**하고 mongosh는 그대로 보여준다 — `ISODate('…T09:02Z')`는 **18:02 KST**다.
화면·어시스턴트는 전부 KST로 변환해 표시하므로 사용자용 표면에는 문제가 없고, 이건 mongosh로
직접 열어 보는 개발자용 주의사항이다. ⚠ 저장을 KST로 바꾸면 안 된다 — TTL(개인정보 자동 삭제)이
실제 시각과 비교되므로 9시간 어긋난다.

사람이 읽을 조회는 `$dateToString`에 타임존을 주면 된다 (예: 특정 학생의 오늘 입·퇴실):

```bash
docker exec -it autogt-mongo mongosh -u admin -p --authenticationDatabase admin project3 --eval 'db.attendance_history.aggregate([{$match:{student_name:"<이름>"}},{$project:{_id:0,student_name:1,입실:{$dateToString:{date:"$check_in_time",format:"%Y-%m-%d %H:%M",timezone:"Asia/Seoul"}},퇴실:{$dateToString:{date:"$checked_out_at",format:"%Y-%m-%d %H:%M",timezone:"Asia/Seoul"}}}}]).forEach(printjson)'
```

### 화면으로 보는 상태 (헤더)

행정실이 상시 띄워 두는 화면이 1차 모니터링이다.

| 표시 | 뜻 |
|---|---|
| `● 퇴실 감시 중` | 탐지가 실제로 돌고 있고 지금이 퇴실 창 안이다 (마우스를 올리면 이유가 보인다) |
| `● 퇴실 감시 대기` | 탐지가 꺼졌거나 · 기동 실패했거나 · 지금이 퇴실 창 밖이다 |
| `● 퇴실 확인 불가` | AI 서버(webapp)에 연결하지 못했다 |
| `⚠ 실시간 연결 끊김` | WebSocket이 끊겼다 — **화면이 자동 갱신되지 않는다** |
| `카메라: ○cam_…` | 흰 점 = 미연결. 전부 흰 점이면 카메라·네트워크를 본다 |
| `등록된 카메라 없음` | `.env`의 `CAMERA_SOURCES`가 비어 있다 |

---

## 2. 장애 대응 절차

> 순서는 항상 **① 어디가 죽었는지 좁힌다 → ② 그 서비스만 다시 띄운다 → ③ 안 되면 전체 재시작**이다.
> 컨테이너가 죽어도 `restart: unless-stopped`라 도커가 자동으로 다시 띄운다 — 그런데도 안 되는 상황이 여기 있는 경우다.

### 증상: 화면이 아예 안 열린다

1. 확인
   - `docker compose ps` — `frontend`가 있는가
   - 서버에서 그 포트를 다른 프로그램이 쓰고 있지 않은가 (`FRONTEND_PORT` 확인)
2. 조치
   - `docker compose up -d frontend`
   - 포트 충돌이면 `.env`의 `FRONTEND_PORT`를 바꾸고 `docker compose up -d frontend` — 단 GPU 서버에서는 **반별 대역(XX51~XX99) 안에서만** 고를 것 (rules/06)

### 증상: 화면은 뜨는데 출결 수치가 안 보인다 ("서버 연결 실패" 배너)

1. 확인
   - `curl .../api/v1/health` → 200이 아닌가
   - `docker compose logs --tail 100 backend`
   - MongoDB가 healthy인가 (`docker compose ps`)
2. 조치
   - Mongo가 죽었으면 `docker compose up -d mongodb` 후 `docker compose restart backend`
   - Spring이 기동 실패면 로그의 첫 예외를 본다. **`.env` 값 오타(특히 `MONGO_*`)가 가장 흔하다**
   - 이때도 **라이브 뷰와 실시간 이벤트는 계속 보여야 한다** — 안 보이면 AI 서버도 같이 죽은 것이다

### 증상: 라이브 뷰가 "미연결"이다

1. 확인
   - 헤더 카메라 점이 전부 흰색인가 → 카메라 쪽 문제
   - `docker compose logs --tail 100 ai` — 재접속 로그가 반복되는가
   - 서버에서 카메라 URL에 직접 닿는가 (`curl -I <카메라 URL>`)
2. 조치
   - 카메라·네트워크 복구가 우선이다. AI 서버는 2초→30초로 간격을 늘려 가며 계속 재접속한다
   - `.env`의 `CAMERA_SOURCES` 형식 확인: `cam_12f_1=rtsp://...;cam_1f=...` (세미콜론 구분)
   - **`.env`만 고치면 반영되지 않는다** — `docker compose up -d --force-recreate ai`
   - ⚠ **내 자리에서는 스트림이 열리는데 서버 curl만 실패하면 망 배치 문제다** — 학원엔 같은 192.168.0.x 대역 라우터가 두 개라(메인 LAN vs 서버실 공유기) IP가 비슷해도 서로 못 볼 수 있다. 카메라 랜선을 서버 공유기로 옮길 것 (rules/06 "카메라 연결" 배치 조건, 2026-08-14)

### 증상: 문자가 나가지 않는다

1. 확인
   - `docker compose logs backend | grep -i solapi` — `solapi_not_configured`가 보이면 키 누락
   - 알람 화면에서 그 건의 발송 상태를 본다: `발송 대기`(아직 확정 전) / `실패` / `대상 아님`
   - 발신번호 자체 점검: `cd RPAs && python check_sender.py` (1건만 보내고 최종 상태까지 확인)
2. 조치
   - **접수 성공은 발송 성공이 아니다.** 화면이 `실패`로 바뀌었다면 통신망에서 막힌 것이다
     - `3113` = 번호도용문자차단서비스 → 수신자가 통신사에 해지 요청해야 한다 (전파에 시차가 있다)
   - `.env`의 `SOLAPI_*` 3개를 확인하고 `docker compose up -d --force-recreate backend`
   - `SMS_MODE`가 `test`면 실제 학생이 아니라 `SMS_TEST_RECIPIENT`로 나간다 — 의도한 상태인지 확인

### 증상: 아침 브리핑 메일이 안 온다 (LLM-002, #224)

1. 확인
   - `curl localhost:8000/api/v1/briefing/status` — 여기서 대부분 갈린다
     - `enabled: false` → 발송 스위치가 꺼져 있다 (기본값이 꺼짐이다)
     - `smtp_configured: false` → `SMTP_USER`·`SMTP_PASSWORD`가 비어 있다
     - `recipient_count: 0` → `BRIEFING_RECIPIENTS`가 비어 있다
     - `font: ""` → 한글 폰트를 못 찾은 것. **메일은 나가되 첨부 PDF 없이 본문만 간다** (한글이 □로 깨진 PDF보다 낫다는 판단, specs/14)
   - `docker compose logs ai | grep -i 브리핑` — 발송 성공 시 `브리핑 메일 발송 — 수신 N명` 한 줄이 남는다
2. 조치
   - **앱이 발송 시각 이후에 떴으면 그날은 건너뛴 것이다** — 마지막 실행 날짜를 메모리에만 두기 때문이고, 중복 발송보다 낫다는 의도된 동작이다. 놓친 날은 손으로 만든다:
     ```
     curl -X POST localhost:8000/api/v1/briefing/run -H 'Content-Type: application/json' \
          -d '{"date":"2026-08-26","dry_run":false}'
     ```
     ⚠ `dry_run`은 **기본이 true**다 — 빼먹으면 리포트만 만들고 아무것도 안 나간다(실명이 나가는 경로라 일부러 그렇게 뒀다)
   - `.env`를 고쳤으면 `docker compose up -d`로 올린다 — **`restart`는 환경변수를 다시 안 읽는다**
   - 문장이 밋밋하면(고정 서식) LLM이 빠진 것이다: Ollama가 안 닿았거나, 생성문이 검증기에 걸려 되돌아간 것. 로그에 사유가 남는다 (`사실에 없는 수` / `권고 표현` / `미퇴실을 다른 사건으로 바꿔 부름`). **메일 자체는 어느 경우에도 나간다**

### 증상: "LMS 연동 이상" 배너가 떠 있다

1. 확인
   - `docker compose logs --tail 100 backend | grep -i lms`
   - `.env`의 `LMS_SNAPSHOT_API_URL`이 비어 있지 않은가
   - 서버에서 그 주소에 닿는가
2. 조치
   - 화면은 **마지막 동기화 데이터로 계속 동작한다** — 즉시 조치가 필요한 장애는 아니다
   - LMS가 복구되면 다음 폴링(퇴실 시간대 1분, 그 외 5~10분)에 자동으로 배너가 사라진다
   - 배너가 오래 남으면 학원 담당자에게 LMS API 상태를 문의한다

### 증상: 헤더에 `⚠ 실시간 연결 끊김`이 계속 떠 있다

1. 확인
   - `curl .../api/v1/health` → Spring이 살아 있는가
   - 브라우저 새로고침으로 사라지는가
2. 조치
   - Spring이 살아 있는데도 계속이면 `docker compose restart backend`
   - 화면은 재연결을 계속 시도한다(최대 15초 간격). **끊긴 동안의 이벤트는 재전송되지 않지만**, 다시 붙는 순간 전체를 재조회한다

### 증상: 예외 등록 화면의 학생 목록만 안 나온다

- 이 화면만 LMS를 실시간으로 부른다. 대시보드가 멀쩡한데 여기만 실패하는 것은 **정상적인 증상 분리**다.
- 위 "LMS 연동 이상" 절차와 같다.

### 공통 재시작 절차

```bash
docker compose ps                  # 1) 어디가 죽었는지 확인
docker compose logs --tail 100 <서비스>   # 2) 첫 예외를 본다
docker compose up -d <서비스>       # 3) 그 서비스만 다시
docker compose down && docker compose up -d --build   # 4) 그래도 안 되면 전체 (DB 데이터는 유지된다)
```

> ⚠ 급하다고 `down -v`를 치지 않는다 — **DB 볼륨이 지워진다.** 그날 명단은 LMS 폴링으로 곧 복구되지만,
> 행정실이 등록한 예외와 발송 이력은 돌아오지 않는다.

---

## 3. 백업 / 복구

| 항목 | 내용 |
|---|---|
| 백업 대상 | **익명 통계(`daily_stats`)만.** 개인정보 컬렉션은 백업하지 않는다 |
| 백업 금지 이유 | 퇴실 즉시 삭제·TTL 자동 삭제 정책이 백업본 때문에 무력화된다 (rules/04·06) |
| 백업 주기 / 방법 | 정기 백업 없음. 발표 자료용으로 필요할 때만 아래 명령으로 덤프 |
| 백업 보관 위치 | 담당자 PC (팀 공유 금지 — 통계라도 리포에 커밋하지 않는다) |
| 복구 절차 | 아래 |
| 복구 테스트 마지막 확인일 | ⬜ 미실시 — GPU 서버 배포 후 1회 실시할 것 |

```bash
# 익명 통계만 덤프 (개인정보 없음)
docker exec autogt-mongo mongoexport --quiet \
  -u <root계정> -p <비밀번호> --authenticationDatabase admin \
  -d project3 -c daily_stats --jsonArray > daily_stats.json
```

### 복구 절차

1. **스키마·인덱스**: `cd database && ./apply.sh` — 컬렉션·TTL·계정 권한이 다시 만들어진다
2. **당일 명단**: 복구하지 않는다. 원천이 LMS이므로 다음 폴링(최대 10분)에 자동으로 다시 채워진다
3. **예외·발송 이력**: 복구 대상이 아니다 (백업이 없다). 행정실에 그날 등록한 예외를 다시 넣도록 안내한다
4. **통계**: 덤프본이 있으면 `mongoimport`로 되돌린다

---

## 4. 정기 점검 항목

| 주기 | 점검 내용 |
|---|---|
| 매일 (퇴실 시간대 전) | 헤더 3종 — 퇴실 감시 뱃지 / 카메라 점 / 실시간 연결. 셋 다 정상이어야 그날 탐지가 돈다 |
| 매일 (마감 후) | 알람 화면에서 `실패` 건이 있는지 — 있으면 사유(3113 등)를 확인하고 수신자에게 안내 |
| 매주 | `docker compose ps`로 재시작 흔적 확인 · 솔라피 콘솔에서 잔액·발송 실패율 확인 |
| 매주 | 디스크 여유 — 도커 로그는 로테이션 설정이 없어 계속 쌓인다 (`docker system df`) |
| 매일 (오전) | 브리핑 메일이 왔는지 — 안 왔으면 §2 "아침 브리핑 메일이 안 온다". ⚠ **Gmail 앱 비밀번호는 계정 비밀번호를 바꾸면 무효가 된다** — 그때 조용히 실패하므로 메일이 안 온 날은 이것부터 의심한다 |
| 과정 종료 시 (8/31) | 브리핑 발송을 끈다(`BRIEFING_ENABLED=false`) — 리포트에 **학생 실명이 실려 외부(Google)로 나가는 유일한 경로**다(99_DECISIONS 2026-08-25). 메일함에 남은 리포트도 함께 정리한다 |
| 시간표 변경 시 | 별도 조치 불필요 — 탐지 창은 LMS 시간표에서 자동으로 계산된다. **`.env`의 `DETECTION_WINDOW`는 비워 둔다**(2026-08-11): 값을 넣으면 Spring을 못 받은 날 그 고정값으로 도는데, 고정값은 시간표가 바뀌면 낡아 틀린 시간에 카메라를 돌린다 |

---

## 5. 장애 회고 (포스트모템)

### 2026-08-09. Slack 알림이 도커에서만 나가지 않았다

- **증상 / 영향**: 로컬(gradle 실행)에서는 Slack 발송이 확인됐는데, 도커로 띄우면 로그에 "미설정 — 발송 스킵"만 남고 한 건도 나가지 않았다. 예외도 에러도 없어서 한동안 알아채지 못했다.
- **원인**: `docker-compose.yml`의 `environment:`는 **화이트리스트**다. `.env`에 값이 있어도 거기 적지 않으면 컨테이너로 전달되지 않는다. `SLACK_BOT_TOKEN`·`SLACK_CHANNEL_ID`가 빠져 있었다.
- **조치**: compose에 누락 키를 추가하고, `application.yml`이 참조하는 환경변수 전수를 대조했다(3개 발견).
- **재발 방지**: `EnvironmentContractTest` 추가 — `application.yml` ↔ `docker-compose.yml` ↔ `.env.example` 세 곳이 어긋나면 테스트가 깨진다. 사람이 눈으로 대조하는 것으로는 다시 놓친다.

### 2026-08-09. 문자 21건이 전부 실패했는데 화면에는 "발송"으로 남아 있었다

- **증상 / 영향**: 솔라피 콘솔에는 21건 전원 실패(`3113` 번호도용문자차단)인데, 우리 DB와 화면에는 전부 `SENT`였다. 화면만 보면 "다 잘 나갔다"로 읽히는 조용한 실패였다.
- **원인**: **접수 성공(statusCode 2000)은 발송 성공이 아니다.** 통신망 단계의 실패는 접수 응답에 나타나지 않고 나중에 붙는다.
- **조치**: 접수 때 받은 `message_id`로 분 단위 대조 폴링(`SmsDeliveryReconcileService`)을 돌려 최종 상태를 확정하고, 실패면 화면 상태를 정정하도록 했다.
- **재발 방지**: 발송 대기 구간을 `PENDING`이라는 정식 상태로 만들어 화면이 "아직 모름"과 "성공"을 구분해 보여준다. 통계(`sms_sent_count`)도 접수가 아니라 **도착 확정** 기준으로만 센다.

---

## 6. 프로젝트 종료 시 파기 절차 (2026-08-31)

> 개인정보(수강생 실명·연락처)를 다루는 시스템이라, 종료는 "끄기"가 아니라 **파기**다.

1. **DB 볼륨 삭제** — `docker compose down -v` (컨테이너·네트워크·`mongo_data` 볼륨 제거)
2. **이미지 삭제** — `docker image rm` 으로 `autogt-*` 이미지 제거 (`docker system prune -a` 도 가능)
3. **`.env` 회수** — 서버와 팀원 PC 양쪽에서 삭제. 특히 `SMS_TEST_RECIPIENT`(팀원 실번호)
4. **외부 키 폐기** — 솔라피 API Key/Secret 삭제, Slack Bot 토큰 폐기, 발신번호 등록 해지
5. **얼굴 등록부** — `registered_faces` 컬렉션은 1의 볼륨 삭제로 함께 사라진다. 별도 JSON(`REGISTRY_FILE`)을 쓴 PC가 있으면 그 파일도 삭제
6. **확인** — `docker volume ls`에 `autogt_mongo_data`가 없는지, 팀원 PC에 `.env`·등록부 사본이 없는지 서로 확인
