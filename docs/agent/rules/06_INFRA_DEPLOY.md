# 06. 배포 / 인프라

> 문서 상태: ✅ 완료 (2026-08-05)
> 학원 내부망 실행 전제. CCTV·LMS 연동이 내부망이라 외부 클라우드 배포 없음.

## 1. 배포 대상

### 실행 환경
- 상태: ✅ 결정 (2026-08-20 **로컬 우선으로 갱신** — 99_DECISIONS)
- 선택지: 로컬 실행만 / 클라우드 PaaS / 클라우드 IaaS / 홈서버·NAS
- 결정(2026-08-20): **로컬(CTO 맥북) 우선 배포** — `mongodb`·`backend`·`ai`·`frontend`는 맥북 compose로 돌리고, **GPU 서버에는 LLM(ollama)만** 남긴다(`COMPOSE_PROFILES=llm`). 어시스턴트가 필요하면 SSH 터널(`-L 11434`)로 끌어온다
  - 근거: ① 얼굴 파이프라인(YuNet+앙상블)은 처음부터 **CPU 실측으로 충분**해서(아래 "GPU는 아직 붙이지 않는다" 항목) GPU가 실제로 필요한 것은 LLM뿐이다 ② 실무 프레이밍 — GPU 노드는 비용이라 GPU가 필요한 워크로드만 올린다 ③ 엣지(라즈베리파이)를 맥북과 같은 강의실 망에 두면 이중 라우터 문제(위 카메라 절)가 소멸한다 ④ GPU 서버는 매일 저녁 꺼지므로 상시성 손실도 없다
  - ⚠ 로컬 운영의 대가: 시연·운영 중 **맥북이 켜져 있어야 한다**(절전 금지 — `caffeinate`), 대시보드 주소가 맥 IP(DHCP)를 따라 바뀔 수 있다
- 이전 결정(2026-08-05~08-19): 학원 GPU 서버 한 대에 전 스택 — CCTV·LMS가 내부망이라는 근거는 유지되나 배포 위치만 이동
- 결정일: 2026-08-05 / 2026-08-20(로컬 우선)

### GPU 분리 개발 (B) — 맥(front·back·mongo) ↔ GPU(ai·ollama) 터널 구성
- 상태: 🚧 초안 (2026-08-20, 99_DECISIONS) — 카메라 릴레이 실측 통과, 전체 통합 테스트 대기
- 배경: 개발을 학원 GPU에서 하고 파이(카메라)가 학원 상주라, "맥에 전 스택"보다 **ai를 GPU에**
  두는 편이 낫다는 판단. 단 **파이는 맥에만 닿고 GPU엔 안 닿으며**(강의실 vs 서버실 망), GPU는
  공인 IP지만 **집 NAT 뒤의 맥엔 못 닿는다.** 그래서 **맥을 유일한 다리**로 SSH 한 세션이 양방향을 나른다.
- 구성:
  - **맥**: `frontend`·`backend`·`mongodb` — `docker compose -f docker-compose.yml -f docker-compose.mac-split.yml up -d mongodb backend frontend` (+ `ops/tunnel.sh`)
  - **GPU**: `ai`(±`ollama`) — `docker compose -f docker-compose.yml -f docker-compose.gpu.yml up -d ai` (ai는 `network_mode: host`)
- **영상은 저장하지 않는다** — 파이 MJPEG는 터널로 흘러 ai가 라이브 처리 후 버린다. DB엔 **결과 + (검토용) 크롭 참조**만 (rules/04 그대로). "MJPEG를 DB에 저장" 안 함.
- 터널이 나르는 것 (맥 anchor, `ops/tunnel.sh`):

  | 흐름 | 포워딩 | 방향 |
  |---|---|---|
  | 카메라 | `-R 8080:<파이>:8080` | 파이 → (맥) → GPU ai |
  | 결과(API-009) | `-R 9080:127.0.0.1:8080` | GPU ai → (맥) → 맥 backend |
  | DB 적재 | `-R 9017:127.0.0.1:27051` | GPU ai → (맥) → 맥 mongo |
  | ai 호출 | `-L 8000:127.0.0.1:8000` | 맥 nginx/backend → (GPU) → ai |
  | LLM(선택) | `-L 11434:127.0.0.1:11434` | 맥 backend → (GPU) → ollama |

  → GPU의 ai는 전부 `localhost:<포트>`로 잡는다(host 네트워크). 주소는 GPU `.env`, nginx→ai는 맥 `.env`의 `AI_UPSTREAM`으로 (.env.example "GPU 분리 개발" 절).
- ⚠ **제약**: 이 다리는 **맥이 학원 망(파이 도달)에 있을 때만** 성립한다. 맥이 집에 가면 파이↔맥이 끊긴다 — 즉 이건 **"학원에서 개발할 때"용**이다.
- **프로덕션**: 실제 배포는 **단일 학원 컴퓨터**에 전 스택 — **터널 없이 전부 기본값(서비스 이름)**. 코드는 같고 `.env`만 다르다(설정 주도라 분기 없음). GPU 야간 셧다운 → 아침 기동 시 컨테이너 자동 기동(`restart: unless-stopped`).
- 근거가 된 실측: 2026-08-20 카메라 역터널 릴레이 성립(GPU가 `localhost:8080`에서 파이 `/health` 수신). 나머지 링크는 같은 SSH 포워딩이라 동일.

### 컨테이너화 여부
- 상태: ✅ 결정
- 선택지: 사용 안 함 / Docker / Docker Compose(멀티 서비스)
- 결정: **Docker Compose로 전 구성요소** — `frontend`(Nginx) · `backend`(Spring) · `ai`(FastAPI) · `mongodb` 4개 서비스를 올린다(2026-08-20부터 기본 위치는 로컬 맥북, 위 실행 환경 참조). 리포 루트 `docker-compose.yml`이 배포 단위. `ollama`는 **`llm` 프로파일 소속**이라 기본 up에 포함되지 않는다 — GPU 서버에서만 `.env`에 `COMPOSE_PROFILES=llm`
- 이유: 강사가 판단을 3팀 CTO 협의로 위임 → 전면 컨테이너화 합의 (2026-08-07). 발표 당일 4개 서버를 수동으로 띄우는 것보다 사고 확률이 낮다. (이전: DB만 Docker — 2026-08-05 강사 지침, 99_DECISIONS 참고)
- 결정일: 2026-08-07 (최초 2026-08-05)

#### 구성

| 서비스 | 기반 | 포트 | 역할 |
|---|---|---|---|
| `frontend` | Nginx (Vite 빌드 산출물) | **3051 — 외부 공개** | 브라우저가 보는 유일한 창구. `/api`·`/ws`를 뒤로 프록시 |
| `backend` | eclipse-temurin JRE 21 | 8080 (내부) | 업무 API·배치 |
| `ai` | python:3.12-slim + OpenCV | 8000 (내부) | 영상 수집·탐지. 카메라(Jetson)를 당겨온다 |
| `mongodb` | mongo:8.0 | **127.0.0.1에만** | 같은 Docker 네트워크 + 그 서버 자신에서만 접근 |

- ⚠ **반별 포트 대역 — 우리 반은 XX51~XX99** (2026-08-14 학원 정책): GPU 서버를 여러 반이 나눠 쓰게 되면서 다른 반이 우리 포트에 붙어 서비스가 죽는 사고가 났고, 반별 대역이 지정됐다. **호스트에 바인딩되는 포트만 대상**이다 — 컨테이너 내부 포트(backend 8080·ai 8000·ollama 11434)는 rootless 데몬의 네트워크 안에만 있어 다른 반과 충돌 자체가 불가능하므로 바꾸지 않는다.
  - 팀 관례: 헷갈리지 않게 대역 안에서 **끝자리 51로 통일** — frontend **3051**, mongo **27051**. compose 기본값도 이 값이라 `.env` 없이 올려도 정책을 어기지 않는다
  - 80·8080·27017 같은 **서비스 기본 포트는 쓰지 않는다** — 누구나 기본값으로 띄우는 번호라 충돌 1순위다 (80이 실제 사고 사례)
- **MongoDB 포트를 학원 LAN에 열지 않는다** — 앱이 전부 같은 네트워크에 있어 `mongodb:27017`로 붙으면 된다. 실명이 든 DB를 LAN에 노출할 이유가 없다.
  - 다만 compose는 `127.0.0.1:${MONGO_PORT}:27017`로 **그 서버 자신에게만** 연다 — 팀원이 IDE에서 앱을 띄우고 DB만 컨테이너로 쓰기 때문이다. 앞의 `127.0.0.1`이 핵심이고, 이걸 빼면(0.0.0.0) 그 순간 LAN에 열린다
  - 다른 PC에서 볼 땐 SSH 터널: `ssh -L 27017:localhost:27051 <서버>` (자기 로컬 쪽은 27017 그대로 둬도 된다)
  - ⚠ 127.0.0.1 바인딩은 **LAN**을 막을 뿐, 같은 호스트에 계정이 있는 다른 반 사용자는 여전히 `localhost:27051`로 닿는다 — 실명 DB의 마지막 방어선은 앱 계정 인증(계정 3종 분리, rules/04)이다
- **라우팅은 Nginx가 담당** — `/api/v1/streams`·`/api/v1/detection/`은 `ai`, 나머지 `/api`와 `/ws`는 `backend`. 개발 시 Vite dev proxy가 하던 역할을 그대로 옮긴 것이라 **두 곳의 규칙이 어긋나면 안 된다**
- **GPU: 기본 CPU, GPU 서버 배포(B)는 임베딩만 GPU** (2026-08-25 #88 재검토, 99_DECISIONS 참조). 애초 CPU 실측(탐지 9.4ms/프레임·앙상블 임베딩 167ms/얼굴, 검출 초당 1회)으로 소규모엔 충분했고 **기본 빌드는 지금도 CPU**다. GPU 서버에선 배정 GPU(3번)가 유휴라, ai 이미지를 `--build-arg TARGET=gpu`로 굽고(`onnxruntime-gpu` + CUDA/cuDNN 휠, `requirements-gpu.txt`) `AUTOGT_USE_GPU=1`로 **앙상블 임베딩(무거운 부분)만** GPU에 올린다. **탐지(YuNet=OpenCV)는 CPU 유지**(가볍다). `docker-compose.gpu.yml`이 이 빌드+GPU 예약을 담당하고, **Mac/로컬 빌드는 TARGET 기본 cpu라 무변경**(nvidia 휠은 amd64 전용이라 arm64에서 설치 자체가 안 되는데, 애초에 설치를 건너뛴다). ONNX 서빙이라 전환이 가벼웠다(#88이 예견한 대로)
- ⚠ **GPU를 쓸 때는 3번을 쓴다 — 우리 조 배정이다** (2026-08-14 학원 확인). 학원 서버는 **L40S 4장을 여러 조가 나눠 쓴다.** 지정하지 않으면 0번을 잡는데 그건 남의 몫이고, 상대가 쓰는 중이면 **서로 메모리를 밀어내며 둘 다 죽는다**
  - 파인튜닝: `export CUDA_VISIBLE_DEVICES=3` (`tools/finetune/README.md`) — 프로세스에는 3번만 보이고 `cuda:0`으로 잡히므로 코드 수정이 필요 없다
  - compose(ollama): `.env`의 `OLLAMA_GPU_DEVICE=3`
  - compose(ai, B 구성): `.env`의 `AI_GPU_DEVICE=3` + `docker-compose.gpu.yml`이 GPU 예약과 `AUTOGT_USE_GPU=1`을 건다. **ollama와 같은 3번 — "전부 3번"**. GPU 이미지가 아니거나(`TARGET≠gpu`) `AUTOGT_USE_GPU`가 꺼져 있으면 CPUExecutionProvider로 조용히 폴백한다
  - ⚠ 실측(2026-08-25): 0번은 **다른 조 vLLM이 40GB 상주** 중이었다. `OLLAMA_GPU_DEVICE`/`AI_GPU_DEVICE` 미설정 시 기본 0번을 잡아 겹친다 — 반드시 `.env`에 3을 박을 것
  - 확인: `nvidia-smi --query-gpu=index,memory.used --format=csv,noheader` 에서 **3번에만** 우리 메모리가 잡혀야 한다. GPU별 프로세스는 `nvidia-smi --query-compute-apps=gpu_uuid,pid,process_name,used_memory --format=csv`
- ⚠ **식별 모델 두 개는 이미지에 굽지 않고 볼륨으로 넣는다** (`./webapp/models:/app/models`). 합쳐 500MB이고 자동 다운로드가 불가능한 자료라서다(권한 자료·export 산출물). **없으면 기동에 실패한다** — 조용히 열화된 백엔드로 도는 것보다 낫다고 판단했다. 배치 방법은 `webapp/README.md` "앙상블 모델 배치"
  - **배치된 파일이 맞는 파일인지는 기동 시 지문 대조로 확인한다** (이슈 #114): `webapp/models/CHECKSUMS.txt`(커밋됨 — 해시일 뿐 가중치가 아니다)의 sha256과 다르면 **ERROR 로그**를 찍는다. 기동은 막지 않는다(PR #112와 같은 정책). 확인: `docker compose logs ai | grep -E "모델|지문"` — **아무것도 안 나오면 일치**다(정상 경로는 INFO라 기본 레벨에서 안 보인다)
  - 파인튜닝 산출물 등으로 **의도적으로 교체**할 때는 CHECKSUMS.txt를 같은 PR에서 갱신한다(`cd webapp/models && shasum -a 256 glintr100.onnx adaface_ir101.onnx > CHECKSUMS.txt`) — **그 갱신 커밋 자체가 "모델을 바꿨다"는 기록**이다. 갱신 없이 파일만 바꾸면 다음 기동부터 ERROR가 계속 찍힌다

### 카메라 연결 — Jetson(엣지) → FastAPI

현행은 **서버가 스트림을 당겨온다**(엣지 탐지 API-019는 설계 단계). 그래서 엣지는 영상을 HTTP로 내보내기만 하면 되고, 서버는 `CAMERA_SOURCES`에 URL을 한 줄 더할 뿐 코드 변경이 없다.

⚠⚠ **카메라(엣지 기기)는 반드시 GPU 서버의 공유기 망에 물린다** (2026-08-14 실측으로 확정):
- 학원에는 **같은 192.168.0.x 대역을 쓰는 라우터가 두 개다** — 메인 LAN(와이파이 포함)과 서버실 공유기(116.42.115.24). 번호가 같아서 같은 망처럼 보이지만 서로 못 본다. 대역이 겹쳐서 **라우팅 우회도 성립하지 않는다** — 젯슨이 메인 LAN에 물려 있던 날, 맥북(같은 메인 LAN)에서는 스트림이 멀쩡한데 서버에서는 `curl` 타임아웃이 났다
- **판정은 눈이 아니라 명령으로**: 서버에서 `curl -m 3 -s http://<엣지IP>:8080/health` — 실패하면 랜선을 서버 공유기로 옮기고 엣지의 **새 IP**로 다시 등록한다 (망이 바뀌면 IP도 바뀐다)
- 어느 망에 있는지 헷갈리면 그 기기에서 `arp -a`로 대조 — 상대가 `(incomplete)`면 다른 망이다
- VPN(Tailscale) 우회는 불가 — 서버의 tailscale이 **다른 사용자 계정**으로 로그인돼 있고, 공용 서버에서 남의 VPN 설정을 건드리지 않는다 (반별 포트 충돌 사고와 같은 종류의 침범이다)
- 발표 체크리스트: **시연 장소에서 이 curl부터** — 화면이 아니라 망 배치가 시연 성패를 가른다

```
CAMERA_SOURCES=cam_jetson=http://<젯슨IP>:8080/video;cam_pc=http://<팀원PC IP>:8080/video
```

**여러 대는 세미콜론으로 잇는다** — 대수가 늘어도 코드는 그대로다.

**엣지 쪽 송출은 `webapp/tools/stream_camera.py`로 한다** (2026-08-14 추가). 그 전에는 기기마다 즉석 스크립트를 만들어 썼는데, 카메라를 한 대 늘릴 때마다 같은 걸 다시 짜야 했다.

```bash
python tools/stream_camera.py --camera-id cam_pc          # 기본 0번 카메라, :8080
PYTHONPATH= python tools/stream_camera.py --camera-id cam_jetson   # ⚠ Jetson은 이 접두어 필수
```

- 의존성은 `opencv-python` 하나뿐이다 — HTTP는 표준 라이브러리로 처리한다. 엣지에 Flask·FastAPI를 깔게 만들면 Jetson처럼 파이썬 환경이 까다로운 기기에서 설치부터 막힌다
- 기동하면 **서버 `.env`에 붙여넣을 `CAMERA_SOURCES=` 줄을 그대로 출력**한다 (자기 LAN IP를 찾아서 넣어 준다)
- `GET /health`로 송출 프레임 수·읽기 실패 수를 볼 수 있다 — "영상이 안 보인다"가 카메라 문제인지 네트워크 문제인지 여기서 갈린다
- Windows는 자동으로 DirectShow 백엔드를 쓴다. 기본 MSMF는 장치에 따라 첫 프레임까지 수 초가 걸리거나 해상도 설정이 무시된다
- macOS에서 열리지 않으면 **터미널 앱의 카메라 권한**을 확인할 것 — OpenCV 로그에 `not authorized to capture video`만 뜨고 원인이 안 보인다
- 형식은 **MJPEG over HTTP**(`multipart/x-mixed-replace`)다. `cv2.VideoCapture(url)`가 그대로 읽고, `tools/capture.py`가 다루던 IP Webcam 형식과 같다
- ⚠ **캡처 스레드는 하나만 둔다.** 클라이언트마다 `cap.read()`를 하면 프레임이 서로 섞여 양쪽 다 깨진다 (서버의 `CameraStream`도 같은 구조)
- ⚠ **Jetson Nano(JetPack 4.x)의 시스템 Python은 OpenCV 4.1.1이라 `FaceDetectorYN`이 없다**(4.5.4+ 필요). JetPack을 새로 구워도 번들 OpenCV 버전은 같으니 **OS 재설치로는 해결되지 않는다.** Miniforge로 별도 환경(Python 3.10 + `opencv-python-headless`)을 만들어 우회한다 — 시스템 Python을 건드리면 JetPack 도구가 깨진다
  - `deadsnakes` PPA는 **arm64를 지원하지 않는다** (2026-08-13에 실제로 막혔다). aarch64에서는 Miniforge를 쓸 것
  - 실행 시 `PYTHONPATH=` 를 앞에 붙인다 — JetPack이 걸어 둔 `PYTHONPATH`에 시스템 OpenCV가 있어 새 환경이 그쪽을 먼저 집는다
  - ⚠ 단 **송출만** 할 때는 이 우회가 필요 없다 — `stream_camera.py`는 `VideoCapture`·`imencode`만 쓰므로 `FaceDetectorYN` 부재와 무관하다. 위 Miniforge 절차는 엣지에서 **탐지까지** 돌릴 때(API-019)의 이야기다
- ⚠ **맨 우분투 젯슨(JetPack 환경 없음)도 있다** (2026-08-14 실측 — woori 젯슨): 시스템 Python 3.6에 cv2 자체가 없었다. 이 경우 `sudo apt-get install -y python3-opencv`(OpenCV 3.2, arm64)면 송출에 충분하다. `stream_camera.py`는 **Python 3.6·OpenCV 3.2 호환을 유지한다**(#159 — `__future__ annotations`·`X | Y` 주석·`ThreadingHTTPServer`·`VideoCapture` 2인자가 전부 그날 밟은 지뢰였다). 스크립트에 새 문법을 넣기 전에 모듈 머리말의 호환 규칙을 볼 것
- 실전 확인(2026-08-13): 웹캠 1대 기준 서버 수신 **fps 18.7 / 드롭 0**, 탐지·식별까지 정상 (rules/05 "첫 실전 관측")
- 실전 확인(2026-08-14): **2대 동시** — 젯슨(aarch64) + 팀원 PC(Windows). 양쪽 **fps 14.9 / 드롭 0 / 재연결 0**, 대시보드 라이브 뷰에 둘 다 표시. 화면 코드는 한 줄도 안 고쳤다 — `LiveView`가 `/api/v1/streams/status` 목록을 그대로 그리므로 `.env`에 한 줄 더한 것이 전부다 (rules/05 "카메라 2대 동시 관측")
  - ⚠ **`.env` 변경은 `restart`가 아니라 `docker compose up -d`로 반영한다.** `restart`는 기존 환경변수를 그대로 들고 다시 뜬다 — 컨테이너가 `Recreated`가 아니라 `Running`으로 나오면 반영이 안 된 것이다
  - ⚠ **명령을 붙여넣을 때 `--`가 `–`(대시)로 바뀌는 일이 있다.** 2026-08-14에 팀원 PC에서 `--camera-id`가 그렇게 깨져 `unrecognized arguments`가 났다. usage에는 그 옵션이 멀쩡히 보이는데 에러가 나면 이걸 의심할 것 — 하이픈 키를 두 번 직접 누르면 된다
  - `--camera-id`는 **화면에 찍히는 안내 문구용일 뿐**이라 송출에 영향이 없다. 카메라 이름은 서버 `.env`에서 정한다 — 막히면 그냥 빼고 실행해도 된다
- **모델 파일은 이미지에 굽는다** — 런타임 자동 다운로드에 의존하면 발표장 네트워크에 시연이 걸린다
- 전 컨테이너에 `TZ=Asia/Seoul` — 기본 UTC라 로그 시각이 9시간 어긋나 퇴실 시간대 디버깅이 어려워진다
- 개발 중 "DB만 띄우고 앱은 IDE에서"는 `docker compose up -d mongodb`로 한다 — 팀원이 IDE에서 앱을 띄우는 기존 워크플로를 깨지 않는다.
  (예전의 `database/docker-compose.yml`은 이 파일로 합쳐지면서 없어졌다 — 99_DECISIONS 참조)

### 라즈베리파이 4B 엣지 설정 (팀원 실행 가이드, 2026-08-19)

엣지 기기를 Jetson Nano → **Raspberry Pi 4B (8GB)**로 전환한다. **현재 엣지 역할은 송출뿐**(YuNet 탐지는 서버가 한다 — API-019는 설계 단계)이라, Pi는 `stream_camera.py`만 돌리면 되고 **서버 쪽은 `.env` 한 줄 외에 아무것도 안 바뀐다.** Pi가 젯슨보다 쉬운 이유: Pi OS는 모던 Python이라 젯슨의 지뢰(시스템 Python 3.6·OpenCV 4.1.1·arm64 deadsnakes 막힘·Miniforge 우회)가 전부 사라진다.

⚠⚠ **젯슨 SD카드를 Pi에 그대로 꽂으면 부팅 안 된다.** 아키텍처는 같은 arm64지만 부트로더·커널·OS가 다르다(젯슨=JetPack/L4T, Pi=Raspberry Pi OS). "카드를 옮긴다"가 아니라 **Pi용 OS를 새로 굽는다**가 맞다. 굽기 전 젯슨 카드에 살릴 것(개인정보·설정)이 있으면 먼저 백업하고, **새 microSD를 쓰면 젯슨을 보존**할 수 있다(권장).

**A. SD카드 굽기** (SD슬롯 있는 PC + Raspberry Pi Imager, `brew install --cask raspberry-pi-imager`)
- CHOOSE DEVICE → **Raspberry Pi 4** / OS → **Raspberry Pi OS (64-bit)** ⚠ 반드시 64-bit / STORAGE → SD카드 (⚠ 맥 내장·외장 디스크 오선택 주의)
- **"설정 편집(EDIT SETTINGS)" 필수** — 헤드리스 접속의 핵심:
  - 일반: 호스트명(예 `pi-autogt`) · 사용자명·비밀번호 · **Wi-Fi SSID·비밀번호**(엣지가 붙을 그 망) · 국가 `KR`
  - 서비스: **SSH 활성화(비밀번호 인증)**

**B. 첫 부팅 + 접속** — 카드 꽂고 전원, 1~2분(파일시스템 확장) 후 같은 망에서 `ssh <사용자>@<호스트명>.local` 또는 IP

**C. 송출 환경** — ⚠ 최신 Pi OS(Bookworm)는 PEP 668로 `pip install`이 막힌다. **apt로 설치**가 가장 깔끔하다(맨 우분투 젯슨과 같은 처방 — `stream_camera.py`는 `VideoCapture`·`imencode`만 쓴다):
```bash
sudo apt update && sudo apt install -y python3-opencv
```
`stream_camera.py`를 Pi로 옮긴다(리포에서 `curl`로 raw 파일을 받거나 scp). USB 웹캠은 `cv2.VideoCapture(0)` 그대로라 **코드 수정 0**:
```bash
python3 stream_camera.py --camera-id cam_pi          # 동작 확인용 수동 실행 — 상시 운용은 E(systemd)로
```
⚠ USB 웹캠이 GStreamer 백엔드에서 0x0으로 열리며 실패하면(ABKO APC480 실측 2026-08-19) V4L2로 강제한다: `OPENCV_VIDEOIO_PRIORITY_GSTREAMER=0 python3 stream_camera.py ...`

**D. 서버 반영** — `.env`의 `CAMERA_SOURCES`에 Pi 주소 한 줄(젯슨 자리를 대체하거나 추가):
```
CAMERA_SOURCES=cam_pi=http://<PiIP>:8080/video
```
→ 서버에서 `docker compose up -d ai` (⚠ `restart` 아님 — 위 젯슨 절 참조). 확인은 서버에서 `curl -m 3 http://<PiIP>:8080/health` → `{"frames":…,"read_failures":0}`

- ⚠ **망 배치는 젯슨과 똑같은 제약**이다 — Pi도 **서버가 닿는 망**에 있어야 한다. 학원 이중 라우터(위 카메라 절) 문제가 그대로 적용되니, Pi를 서버 공유기 쪽에 두거나 같은 망 확인 필수
- 카메라를 **Pi 카메라 모듈(CSI)**로 바꾸면 최신 Pi OS는 libcamera라 `cv2.VideoCapture`로 못 잡는다 — `picamera2` 캡처 경로가 필요(별도 작업). USB 웹캠은 이 문제 없음
- 엣지에서 **탐지(YuNet)까지** 돌릴 때(API-019)는 Pi에 GPU가 없어 CPU 추론(저프레임)이고 `opencv-python` 4.5.4+가 필요하다. **임베딩(ArcFace/AdaFace)은 절대 Pi에 올리지 않는다 — 서버 몫.** 지금 단계에선 불필요

**E. 부팅 자동 송출 — systemd** (2026-08-19 설정·재부팅 검증 완료). 시연·상시 운용의 기본 형태다: 전원만 꽂으면 송출이 시작되고, 프로세스가 죽으면 5초 뒤 재시작한다. ⚠ **nohup을 쓰지 않는다** — nohup은 "쉘에서 띄운 프로세스를 로그아웃에서 살리는" 도구일 뿐, 부팅 시작·자동 재시작·로그 로테이션이 없다. systemd가 셋 다 대체한다(로그는 journald).

Pi에서 유닛 파일 생성 (`stream_camera.py` 경로·사용자명이 다르면 맞출 것):
```bash
sudo tee /etc/systemd/system/autogt-camera.service > /dev/null <<'EOF'
[Unit]
Description=AutoGT edge camera stream (cam_pi, MJPEG :8080)
After=network-online.target
Wants=network-online.target

[Service]
User=autogt
WorkingDirectory=/home/autogt
# GStreamer 백엔드가 USB 웹캠에서 0x0 실패 — V4L2로 강제 (위 C 절 경고와 동일)
Environment=OPENCV_VIDEOIO_PRIORITY_GSTREAMER=0
ExecStart=/usr/bin/python3 /home/autogt/stream_camera.py --camera-id cam_pi --device 0
# 카메라 미인식·프로세스 사망 시 5초 후 재시도 — 부팅 직후 USB 초기화가 늦어도 알아서 붙는다
Restart=always
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF
sudo systemctl daemon-reload && sudo systemctl enable --now autogt-camera
```

확인 3종: `systemctl status autogt-camera`가 `active (running)` / Pi에서 `curl -s localhost:8080/health` / 서버 화면·API-001에서 `cam_pi` connected. 로그는 `journalctl -u autogt-camera -f`.

- ⚠⚠ **손으로 띄워 둔 stream_camera가 있으면 먼저 죽인다** (`pkill -f stream_camera.py`) — 옛 프로세스가 8080과 카메라를 잡고 있으면 서비스가 `activating (auto-restart)`로 무한 재시도한다. 이때 health가 응답해도 **옛 프로세스가 응답하는 것**이라 "된 것처럼" 보인다(2026-08-19 실측 — `frames` 카운터가 크면 옛 프로세스, 작으면 새 프로세스). pkill은 systemd 프로세스도 함께 죽이지만 5초 뒤 자동 재시작되며 이번엔 포트를 잡는다
- **최종 검증은 재부팅이다**: `sudo reboot` 후 무개입으로 1~2분 내 서버에서 `cam_pi` connected 복귀 확인(2026-08-19 통과 — 서버 `reconnect_count` 증가가 재접속의 흔적으로 남는다)

## 2. 환경 분리

### 환경 구성
- 상태: ✅ 결정
- 선택지: local만 / local + prod / local + dev + prod
- 결정: local + 학원 demo 2단계
- 결정일: 2026-08-05

### 환경별 설정 관리
- 상태: ✅ 결정
- 결정: 02_BACKEND.md 시크릿 관리와 동일 — `.env` + gitignore(FastAPI·프론트), 프로파일 분리(Spring). DB IP·패스워드는 `.env`로 팀 내 공유(커밋 금지, `.env.example`로 키 목록만)
- 결정일: 2026-08-05

## 3. CI/CD

### CI (빌드/테스트 자동화)
- 상태: ✅ 결정 (2026-08-13 **부분 도입**으로 갱신)
- 선택지: 없음(수동) / GitHub Actions / GitLab CI
- 결정(2026-08-05): 자동화 도구 없음 — **GitHub 브랜치 전략으로 관리** (배포용 브랜치 분리 운영). 브랜치 전략 상세는 08_COLLABORATION.md
- **갱신(2026-08-13 CTO): 도커 이미지 빌드 확인만 GitHub Actions로 돌린다** — `.github/workflows/build.yml`
  - 도는 것: `docker compose build` (backend·ai·frontend) 하나뿐. **테스트는 여전히 로컬 게이트다** (rules/07의 "PR 전 로컬 전체 통과" 규칙 유지)
  - 이유: 테스트는 각자 PC에서 돌려도 거의 같은 결과가 나오지만 **도커 빌드는 그렇지 않다.** 로컬에 이미 깔린 패키지가 이미지엔 없고, 지운 심볼을 Dockerfile이 아직 참조한다 — 로컬 테스트로는 **원리상 못 잡는** 종류다
  - 근거가 된 실제 사고 (2026-08-13): SFace 제거 후 Dockerfile이 `SFACE_FILE`을 참조한 채 남아 빌드가 깨졌는데, 그걸 모르고 **옛 이미지가 계속 돌았다** (`backend: sface` 응답을 보고서야 발견)
  - ~~이 CI가 못 잡는 것: `tools/requirements.txt`의 의존성 누락~~ — **2026-08-14 해소 (이슈 #113)**.
    별도 job(`tools-deps`)이 `tools/requirements.txt`를 클린 환경에 설치하고 대표 도구를
    `--help`로 띄워 확인한다. 무거운 `tools/requirements-finetune.txt`(torch 등)는 CI에
    안 넣는다 — 매 PR마다 수 분이 걸려 "가벼운 확인"이라는 목적에 안 맞고, 그쪽 검증은
    GPU 서버 실행 시점의 `tools/finetune/README.md` 절차(`verify_onnx_conversion.py`)가
    담당한다
  - 언제 도는가: **PR 하나당 2번** — PR을 열거나 커밋을 추가할 때(`pull_request`), dev에 머지될 때(`push`). 런당 약 2분
  - 비용 관리: 문서만 고친 PR은 돌지 않고(`paths-ignore`), 같은 PR에 새 커밋이 오면 이전 실행을 취소한다(`concurrency`). 비공개 리포라 실행 시간이 유한하다
  - ⚠ **취소는 PR에만 적용한다** (2026-08-14 수정). dev 머지 런까지 취소하면 검증 구멍이 생긴다 — PR 런은 "그 PR을 base에 얹은 결과"를 **그 시점 기준으로** 빌드하므로, 그 뒤 dev가 움직이면 실제로 머지된 조합은 아무도 빌드해 본 적이 없게 된다. 그걸 잡는 유일한 실행이 dev push 런인데 연달아 머지하면 그것마저 서로 취소한다
    - 실제로 났다: #122의 PR 런에 21초 뒤 머지된 #121이 없었고, #120의 dev 런은 #121 머지에 취소됐다 — 두 변경이 합쳐진 dev를 검증한 실행이 하나도 없었다
    - `cancel-in-progress: ${{ github.event_name == 'pull_request' }}` — 실행 횟수는 그대로고, 취소되던 것이 끝까지 돌 뿐이다
  - `.env`는 커밋하지 않으므로 워크플로가 `.env.example`을 복사해 쓴다 — 빌드 단계는 시크릿을 쓰지 않는다(값은 컨테이너 기동 시 읽힌다). 덤으로 **`.env.example`이 compose를 만족하는지도 함께 확인**된다
- 결정일: 2026-08-05 / 2026-08-13(빌드 확인 도입)

### CD (배포 자동화)
- 상태: ✅ 결정 (2026-08-20 **배포 절차 스크립트화**로 갱신)
- 선택지: 수동 배포 / push 시 자동 배포 / 태그 기반 배포
- 결정: 수동 배포 — 배포 브랜치에서 직접 실행 환경에 반영
- **갱신(2026-08-20, 이슈 #119): `ops/deploy.sh`로 그 "수동"을 한 줄로 묶었다.** push 훅
  자동 배포는 여전히 도입하지 않는다(수동 유지) — 다만 로컬 우선 전환 뒤 배포가 CTO 맥
  한 대에서 매일 반복되므로, `git pull --ff-only` → `docker compose up -d --build`(ollama
  제외) → **헬스가 초록불이 될 때까지 `docker inspect`로 대기** → `docker compose ps` 요약을
  한 번에 밟는다. "Recreated 떴으니 됐겠지"로 끝내지 않고 실제 기동을 확인하는 것이 핵심
  (그게 `.env` 반영 실패·콜드스타트 지연을 놓치던 자리다). `down -v` 같은 파괴적 명령은
  쓰지 않는다 — mongo 볼륨(등록·기록)은 건드리지 않는다. `--no-pull`로 로컬 수정만 재빌드 가능
- **회귀 게이트 자동화(이슈 #115)는 CI가 아니라 로컬 게이트로 확정** — 모델·갤러리·`MATCH_*`·
  정렬 코드를 바꾸는 PR은 `webapp/tools/regression_gate.py`를 **로컬(데이터 있는 CTO 맥)에서**
  돌려 결과 블록을 PR에 붙인다. CI에 못 넣는 이유는 모델 500MB + 얼굴 사진(PII)이라서다(도구
  docstring). 강제 장치는 `.github/pull_request_template.md`의 조건부 체크리스트 (99_DECISIONS #115)
- 결정일: 2026-08-05 / 2026-08-20(배포 스크립트화·회귀 게이트 로컬 확정)

## 4. 도메인 / 네트워크

### 도메인 및 HTTPS
- 상태: ➖ 해당 없음
- 결정: 내부망 실행 — 도메인·HTTPS 불필요. (⬜ 참고: 교실과 외부 IP 체계가 달라 8/7~8/10 IP 통과·영상 스트리밍 테스트 예정 — 회의 액션 아이템)
- 결정일: 2026-08-05
- ⚠ **예외 하나 (2026-08-12, SCR-004 얼굴 등록)**: 브라우저 카메라(`getUserMedia`)는 **보안 컨텍스트(`https` 또는 `localhost`)에서만** 열린다. 다른 화면은 `http://<서버IP>:8090`으로 잘 돌지만 **얼굴 등록 화면만은 카메라가 통째로 막힌다**(화면이 그 상태를 감지해 안내한다). 현재 대응: **등록은 서버가 도는 그 PC의 `localhost`로 열어서** 한다. 등록을 다른 PC에서 해야 하면 그때 HTTPS(자체 서명 인증서 + 브라우저 신뢰 등록)를 붙인다 — 지금 붙이지 않는 이유는 등록이 데스크 한 대에서 이뤄지는 작업이라서다

## 5. 운영

### 모니터링 / 로그 수집
- 상태: ✅ 결정 (2026-08-14 **헬스체크·장애 알림 추가**로 갱신)
- 선택지: 없음 / 플랫폼 기본 로그 / Sentry(에러 추적) / 자체 구축
- 결정: 앱 기본 로그만 (02 로깅 규칙 준수 — 개인정보 마스킹 필수)
- **갱신(2026-08-14, 이슈 #116)**: 로그는 "누가 보러 갈 때만" 쓸모 있는데, 1차 모니터링이
  "행정실이 상시 띄워 둔 화면"이라 아무도 안 보는 시간(새벽·주말·점심)엔 죽어도 아무도
  모른다는 문제가 있었다. 두 가지를 더했다 — Sentry 같은 별도 자체 구축까지는 아니다:
  - **compose healthcheck** — `backend`·`ai`에 `curl -fsS <상태 엔드포인트>` 추가
    (`docker-compose.yml`). `restart: unless-stopped`는 healthcheck가 없으면 "멈춘 채
    살아 있는" 프로세스를 재시작하지 않는다. `backend`(`eclipse-temurin:21-jre`)엔
    curl이 기본으로 없어 `backend/Dockerfile`에 설치를 추가했다
  - **`ops/healthwatch.sh`** — **CTO 맥북 crontab 5분 주기** (2026-08-20 로컬 우선 전환으로
    GPU 서버 → 맥으로 이동). **상태가 바뀔 때만**(플랩 방지) 배치 실패 알림(#117)과 같은
    Slack 채널로 발송한다. 개인정보는 넣지 않는다(서비스명·상태·시각까지)
  - ⚠ **판정은 `curl`이 아니라 `docker inspect`로 한다** (2026-08-20 변경, 99_DECISIONS #116):
    로컬 전환 뒤 backend·ai는 **호스트 포트를 안 열어**(nginx만 창구) 호스트에서 `curl
    localhost:8080`이 항상 실패해 오탐한다. 대신 compose가 각 컨테이너에 심어 둔 healthcheck
    결과를 `docker inspect --format '{{.State.Health.Status}}'`로 읽는다 — `healthy`만 up.
    포트 개방이 불필요하고, 컨테이너 내부에서 실제 엔드포인트를 찌른 결과라 더 정확하다
  - ⚠ 그 compose healthcheck가 `/health`가 아니라 **`/api/v1/health`(backend)·
    `/api/v1/detection/status`(ai)**를 쓰는 이유 — Nginx가 `/health`를 SPA 폴백으로 넘겨
    ai가 죽어 있어도 200 HTML을 돌려준다(specs/13 §2에 이미 경고된 함정)
  - 설치: `crontab -e`에 `*/5 * * * * /path/to/AutoGT/ops/healthwatch.sh >> /tmp/autogt-healthwatch.log 2>&1`
    — cron PATH에 Docker Desktop 경로(`/usr/local/bin`·`/opt/homebrew/bin`)가 없으면 `docker`를
    못 찾으니 crontab 맨 위에 `PATH=...` 한 줄을 더한다. 맥이 잠들면 cron도 멈춘다(`caffeinate`)
- 결정일: 2026-08-05 / 2026-08-14(헬스체크·장애 알림 추가) / 2026-08-20(로컬 전환 — docker inspect·맥 crontab)

### 백업 정책
- 상태: ✅ 결정
- 결정: **개인정보 컬렉션 백업 금지** — 퇴실 즉시 삭제·TTL 정책이 백업본으로 무력화되는 것 방지(04 참조). 익명 통계(`daily_stats`)만 필요 시 덤프
- 결정일: 2026-08-05
