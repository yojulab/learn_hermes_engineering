# 17. 팀원 로컬 개발 온보딩 (Windows)

> **대상**: Windows에서 AutoGT를 로컬로 띄우고 개발하려는 팀원.
> **이중 용도**: ① 사람이 따라 하는 셋업 절차서, ② 각자의 생성형 AI에게 작업을 시킬 때
> 붙여 넣는 **공통 머리말**(§7)의 원본. 작업별 지시서는 이 문서를 전제로 짧게 쓴다.
> **관리**: CTO. 절차가 바뀌면 이 문서를 같은 PR에서 갱신한다 (docs 우선 방침).

---

## 1. 레벨 구성 — 자기 작업에 맞는 것만 띄운다

| 레벨 | 구성 | 대상 작업 | 필요한 것 |
|---|---|---|---|
| **Level 1** | mongodb + backend | Spring 도메인 개발 (notification·attendance 등) | WSL2 우분투 + Docker |
| **Level 2** | + ai + frontend | 화면·탐지 흐름까지 보는 개발 | + 모델 파일 2개, (탐지를 볼 때만) 폰 카메라 |
| CTO 전용 | GPU 분리(B)·정확도 실측·실데이터 | 파인튜닝, 회귀 게이트, 평가셋 | — (요청은 CTO에게) |

정확도 측정은 팀원 로컬에서 **불가능한 것이 정상**이다 — 평가셋·등록 데이터가 CTO 맥에만
있기 때문(§3). 기능 개발·검증은 전부 가능하다.

---

## 2. 공통 준비 (1회) — WSL2 우분투 24.04 + Docker Engine

> **왜 이 경로인가 (CTO 확정 2026-08-22)**: Docker Desktop을 쓰지 않는다 — WSL2 우분투 안에
> Docker Engine을 직접 설치한다. Desktop 없이 가볍게 가고, 셸·경로·명령이 리눅스 그대로라
> 문서·CI·프로덕션과 명령이 일치한다. **모든 작업(클론·빌드·테스트·compose)은 우분투 셸 안에서.**

1. **WSL2 + 우분투 24.04** — PowerShell(관리자)에서:
   ```
   wsl --install -d Ubuntu-24.04
   ```
   재부팅 후 우분투가 열리면 사용자 계정을 만든다. (이미 WSL이 있으면 `wsl -l -v`로 버전 2인지 확인)
2. **Docker Engine + compose 플러그인** — 우분투 셸에서:
   ```
   curl -fsSL https://get.docker.com | sudo sh
   sudo usermod -aG docker $USER
   ```
   그 후 셸을 새로 연다(그룹 반영). `docker compose version`이 나오면 성공.
   (최근 WSL은 systemd 기본 활성이라 데몬이 자동으로 뜬다 — `docker ps`가 데몬 오류를 내면
   `sudo systemctl enable --now docker`)
3. **클론은 우분투 홈(`~`) 안에** — `/mnt/c`(윈도우 디스크) 쪽은 I/O가 몇 배 느리고 권한이 꼬인다:
   ```
   sudo apt-get install -y git
   git clone https://github.com/dun0315667-stack/AutoGT.git ~/AutoGT
   cd ~/AutoGT && git checkout dev
   ```
   프라이빗 리포라 인증이 필요하다 — GitHub PAT(HTTPS) 또는 `gh auth login`.
4. (backend 개발자) **JDK 21 Temurin** — 우분투 셸에서:
   ```
   sudo apt-get install -y wget apt-transport-https gpg
   wget -qO - https://packages.adoptium.net/artifactory/api/gpg/key/public | sudo gpg --dearmor -o /usr/share/keyrings/adoptium.gpg
   echo "deb [signed-by=/usr/share/keyrings/adoptium.gpg] https://packages.adoptium.net/artifactory/deb noble main" | sudo tee /etc/apt/sources.list.d/adoptium.list
   sudo apt-get update && sudo apt-get install -y temurin-21-jdk
   ```
5. 리포 루트에 `.env` 생성 — §4/§5의 샘플을 복사. **`.env`는 절대 커밋하지 않는다** (.gitignore에 이미 있음)

> 브라우저 확인은 **Windows 쪽 브라우저에서 `http://localhost:3051` 그대로** — WSL2가 localhost를
> 자동 포워딩한다. 에디터는 VS Code의 WSL 원격(Remote-WSL)이 편하다.
> 줄바꿈(CRLF) 걱정은 안 해도 된다 — `.gitattributes`가 git 수준에서 LF를 강제한다 (rules/08).

---

## 3. 경계 — 팀원 로컬에 **오지 않는 것** (여기만은 예외 없음)

| 없는 것 | 대체 수단 |
|---|---|
| 등록 임베딩·얼굴 사진·평가셋 (동의 기반 PII — CTO 맥 밖 반출 금지) | **자기 얼굴을 직접 등록**해서 테스트 (SCR-004 화면 등록, §5) |
| SOLAPI 키·문자 수신자 매핑 (개인 전화번호) | **빈 값으로 둔다** — 셋 중 하나라도 비면 발송을 시도하지 않는 설계라, 빈 값 = 실발송 0 |
| Slack 봇 토큰 | 빈 값 (Slack 발송 실패는 FAILED 기록으로만 남고 나머지 흐름은 정상) |
| 파이 카메라 (학원 망 전용) | 폰의 IP Webcam 앱으로 동일 MJPEG 구성 (§5) |

추가 금지: `docker compose down -v` (볼륨 파기 — 8/31 파기 절차 외 금지),
`MATCH_THRESHOLD`·`MATCH_MARGIN`·`MATCH_TOPK` 등 정확도 손잡이 변경(rules/05 — 변경은 CTO 실측 게이트 경유),
모델(.onnx)·registry.json·얼굴 사진의 커밋.

---

## 4. Level 1 — backend 개발 (mongodb + backend)

### .env (샘플 그대로 시작해도 된다)

```dotenv
# === AutoGT 팀원 로컬 Level 1 (Windows) — 커밋 금지 ===
MONGO_ROOT_USER=admin
MONGO_ROOT_PASSWORD=localdev
MONGO_APP_USER=autogt
MONGO_APP_PASSWORD=localdev
MONGO_VIEW_USER=autogt_view
MONGO_VIEW_PASSWORD=localdev
MONGO_DB_NAME=project3
MONGO_PORT=27051
FRONTEND_PORT=3051

# LMS 스냅샷 API — 실제 URL은 리포에 적지 않는다. CTO에게 받아 채운다 (외부 URL이라 집에서도 닿는다)
LMS_SNAPSHOT_API_URL=<CTO에게 받는다>
ATTENDANCE_SCHEDULER_ENABLED=true

# 탐지는 끈다 — backend 개발에 카메라·ai 불필요
DETECTION_ENABLED=false

# 문자·Slack — 전부 빈 값 = 실발송 0 (설계상 키가 비면 발송을 시도하지 않는다)
SMS_MODE=test
SMS_TEST_RECIPIENT=
SLACK_BOT_TOKEN=
SLACK_CHANNEL_ID=
SOLAPI_API_KEY=
SOLAPI_API_SECRET=
SOLAPI_FROM_NUMBER=
```

### 기동·확인

```
docker compose -f docker-compose.yml -f docker-compose.mac-split.yml up -d mongodb backend
```

- `mac-split` 오버라이드를 얹는 이유: 평소 backend는 호스트 포트를 열지 않는데(nginx가 유일
  창구, rules/06), Level 1은 frontend 없이 backend API를 직접 찔러야 해서 **127.0.0.1 한정**으로
  8080을 연다. 파일 이름은 B 구성(맥) 용도로 지어졌지만 내용은 "backend 로컬 개방" 그 자체라 재사용한다.
- 확인:
  ```
  docker inspect --format "{{.State.Health.Status}}" autogt-backend      → healthy
  curl http://127.0.0.1:8080/api/v1/health
  ```
- 테스트(= PR 전 필수 게이트): `backend` 폴더(우분투 셸)에서 `./gradlew clean test`
- IDE(IntelliJ)에서 Spring만 직접 띄우고 싶으면: mongodb만 compose로 올리고, `.env`의
  IDE용 주석 처리된 값(`MONGO_URI=...localhost:27051...`) 방식을 쓴다. ⚠ compose로 전환할 때는
  그 줄을 반드시 다시 주석 처리 — compose가 `.env` 값을 존중해서 컨테이너 안이 `localhost`를
  찾다 죽는다 (2026-08-20 실측 함정, 99_DECISIONS).

---

## 5. Level 2 — 전 스택 (+ ai + frontend)

### 모델 파일 배치 (1회)

`webapp/models/`에 두 파일을 넣는다 — **팀 드라이브 `AutoGT-models/`에서 받는다** (CTO가 업로드).
자세한 검증 절차는 `webapp/README.md` "앙상블 모델 배치".

| 파일 | 확인 |
|---|---|
| `glintr100.onnx` | 배치 후 `docker compose logs ai`에 지문 불일치 ERROR가 없어야 함 |
| `adaface_ir101.onnx` | 〃 (`models/CHECKSUMS.txt` 대조는 기동 시 자동) |

### .env — Level 1 샘플에 아래를 추가/변경

```dotenv
# 탐지를 켤 때만 (화면·기록 조회만 볼 거면 false 유지)
DETECTION_ENABLED=true
# 폰에 "IP Webcam"류 앱 설치 → 같은 Wi-Fi에서 폰 IP 확인 → 아래에 기입
CAMERA_SOURCES=cam_dev=http://<폰IP>:8080/video
# 시간표 밖 시각에 탐지를 보려면 (개발 후 반드시 지운다 — 남기면 아무 때나 카메라가 돈다)
DETECTION_WINDOW_FORCE=true
DETECTION_WINDOW=09:00-23:59

# 식별 운영점 — 손대지 않는다 (rules/05, C-1 채택값)
ARCFACE_MODEL=glintr100.onnx
ADAFACE_MODEL=adaface_ir101.onnx
MATCH_THRESHOLD=0.35
MATCH_MARGIN=0
MATCH_TOPK=5
REGISTRY_FILE=

# 사진 서명 URL 키 — 아무 문자열이나, 비우면 재기동마다 링크가 만료된다
PHOTO_URL_SECRET=dev-<본인이니셜>-아무문자열

# LLM 어시스턴트 — 로컬은 끔 (ollama는 GPU 서버 전용)
ASSISTANT_ENABLED=false
```

### 기동·확인·자기 얼굴 등록

```
docker compose up -d mongodb backend ai frontend
```

⚠ `ollama`는 올리지 않는다 — Windows Docker에 nvidia 런타임이 없으면 기동 실패하고, 로컬에서 돌릴 이유도 없다.

1. `http://localhost:3051` 접속 → 대시보드
2. **얼굴 등록(SCR-004)**: 등록 화면에서 자기 얼굴을 등록한다 — 이것이 팀원 로컬의 유일한
   갤러리다 (실데이터는 오지 않는다, §3)
3. 탐지 확인: 폰 카메라 앞에 서면 개발자 뷰에 bbox·유사도가 뜬다.
   `http://localhost:3051/api/v1/detection/status`로 `registered=1`, 카메라 `connected=true` 확인

### Windows 함정 모음

| 증상 | 원인·처방 |
|---|---|
| 폰 카메라 `connected=false` | 폰과 PC가 **같은 Wi-Fi**인지, Windows 방화벽/공유기 AP 격리 확인. 브라우저에서 `http://<폰IP>:8080/video`가 먼저 열려야 한다 |
| ai가 mongo를 못 찾음 | `.env`에 `MONGO_URI=`를 직접 쓴 경우 — 지워라. compose가 그 값을 존중해 컨테이너 안 `localhost`로 붙는다 (§4 함정과 동일) |
| WSL 메모리 폭식 | `%UserProfile%\.wslconfig`에 `[wsl2] memory=6GB` 정도로 상한 |
| `pytest`가 `ZoneInfo`/tzdata 오류로 죽음 | 우분투 셸이 아니라 **Windows 파이썬**으로 돌렸다는 신호 — 우분투 셸에서 실행할 것 (그래도 필요하면 requirements.txt의 win32 전용 tzdata가 받쳐준다) |
| 이 문서와 다른 환경·도구를 AI가 권함 | 따르지 말 것 — **이 문서(WSL2 우분투 24.04 + Docker Engine)가 유일한 공식 경로**다. 지시서 밖 제안은 거절하고 CTO에게 (§7 범위 규율) |
| 명령이 `command not found` | Windows PowerShell에서 리눅스 명령을 쳤다는 신호 — 이 문서의 모든 명령은 **우분투 셸** 기준이다 |

---

## 6. 개발 → PR 흐름 (rules/08 요약)

1. `dev`에서 브랜치: `feat/<이니셜>-<작업명>` (예: `feat/SA-missed-checkout-...`)
2. 로컬 전체 테스트 통과가 PR 전제: backend는 `gradlew clean test`, webapp은 `pytest`
3. PR 본문: 무엇을/왜 + 확인(테스트 결과 로그 붙이기) — 템플릿 체크박스 채우기
4. **머지는 CTO 전담.** 리뷰 없이 머지 금지, 본인 머지 금지

---

## 7. AI 지시서 공통 머리말 — 아래 블록을 그대로 AI에게 붙여넣고 시작한다

> 사용법: 새 AI 세션을 열면 **가장 먼저** 아래 블록 + 그날의 작업 지시서를 붙여넣는다.
> AI가 "추가로 이것도 고치겠다"고 하면 **거절하고 CTO에게 물어본다.**
> 에러가 나면 요약하지 말고 **에러 전문**을 AI에게 붙여넣는다.
> 한 번에 다 시키지 말고 **계획을 먼저 시키고 → 사람이 확인 → 구현** 순서로 간다.

```text
너는 AutoGT(CCTV 출결 자동 관리, Spring backend + FastAPI ai + React frontend + MongoDB)
리포에서 작업한다. 코드를 쓰기 전에 반드시 다음을 순서대로 읽어라:
1) docs/agent/AGENT_GUIDE.md
2) docs/agent/rules/ 중 이번 작업 도메인의 문서 (backend면 02, 테스트는 07, 협업은 08)
3) 작업 지시서가 지정하는 specs 문서

[범위 규율]
- 작업 지시서에 명시된 파일·패키지만 수정한다. 그 밖을 고치고 싶으면 **멈추고 이유를 보고**한다.
- 지시서에 없는 리팩터링·의존성 추가·설정 변경을 임의로 하지 않는다.

[절대 금지 — 위반 시 작업 전체 무효]
- .env, *.onnx 모델, registry.json, 얼굴 사진·개인 전화번호를 커밋/출력에 포함
- MATCH_THRESHOLD / MATCH_MARGIN / MATCH_TOPK 등 정확도 파라미터 변경
- docker compose down -v (볼륨 파기)
- git push --force, dev/main 직접 push, PR 머지 (머지는 CTO만 한다)
- SMS_MODE를 test 외 값으로 변경

[완료 기준]
- 지시서의 완료 조건 체크리스트를 전부 만족
- backend 변경 시 `gradlew clean test` 전체 통과 로그 제시 / webapp 변경 시 `pytest` 통과
- 변경이 문서(rules/specs)와 어긋나면 같은 작업에서 해당 문서도 갱신 (docs 우선 방침)
- 결과는 feat/* 브랜치 커밋까지. PR 본문 초안(무엇을/왜/확인)을 작성해 제시
```

---

## 8. 문제가 생기면

1. 이 문서 §5 함정 표 → 2. `docker compose logs <서비스>` 를 AI에게 전문 붙여넣기 →
3. 그래도 안 되면 로그와 함께 CTO에게. **셋업 문제로 30분 이상 소모하지 말 것.**
