# 10. API 명세서

> 문서 상태: 산출물 문서 (개발하며 갱신)
> **규칙**: 새 API를 구현하기 전에 반드시 여기에 명세를 먼저 작성하고 사용자 확인을 받은 후 구현한다.
> 공통 응답 포맷·에러 규약은 `rules/02_BACKEND.md`를 따른다. 여기에는 개별 API만 기록한다.

## 0. 문서 원본(source of truth) 규칙

이 문서의 역할은 **구현 전 설계 계약**이다. 문서-코드 불일치를 막기 위해 다음을 따른다.

### 자동 생성 문서 도입 여부
- 상태: ❓ 미결정
- 선택지: Springdoc(Swagger UI) / Spring REST Docs / NestJS Swagger 모듈 / FastAPI 기본 제공 / 도입 안 함
- 결정:
- 결정일:

### 도입 후 운영 방식
- 자동 생성 문서를 도입한 시점부터 **상세 명세(파라미터·응답 필드)의 원본은 자동 생성 문서(코드)**다.
- 이 문서에는 **1장 API 목록과 API별 설계 의도·정책**(인증, 페이징, 멱등성 등)만 유지하고, 2장 상세 명세는 "구현 완료 후 자동 문서 참조"로 표기한다.
- 자동 생성 문서를 도입하지 않는 프로젝트는 이 문서가 끝까지 원본이며, API 변경 시 같은 작업 안에서 갱신한다.

## 1. API 목록

> 상태: 📝 설계 / 🔄 개발중 / ✅ 완료 / 🗑 폐기

| ID | 메서드 | URL | 이름 | 서버 | 인증 | 상태 |
|---|---|---|---|---|---|---|
| API-001 | GET | `/api/v1/streams/status` | 스트림 수집 상태 조회 | FastAPI(:8000) | 불필요 | ✅ |
| API-002 | GET | `/api/v1/streams/{camera_id}/live` | 실시간 화면 재송출 (MJPEG) | FastAPI(:8000) | 불필요 | ✅ |
| API-003 | GET | `/api/v1/attendance/today` | 당일 출결 요약·미퇴실 목록 | Spring(:8080) | 불필요 | ✅ |
| API-004 | GET | `/api/v1/detections` | 미태그 포착 알람 목록 (필터·페이징) | Spring(:8080) | 불필요 | ✅ |
| API-005 | POST | `/api/v1/exceptions` | 예외 등록 | Spring(:8080) | 불필요 | ✅ |
| API-006 | GET | `/api/v1/exceptions` | 당일 예외 목록 | Spring(:8080) | 불필요 | ✅ |
| API-007 | DELETE | `/api/v1/exceptions/{id}` | 예외 삭제 | Spring(:8080) | 불필요 | ✅ |
| API-008 | WS | `/ws` | 실시간 이벤트 (포착·발송·퇴실) | Spring(:8080) | 불필요 | ✅ |
| API-009 | POST | `/api/v1/internal/detections` | 탐지 이벤트 수신 (FastAPI→Spring) | Spring(:8080) | 내부 | ✅ |
| API-010 | GET | `/api/v1/attendance/students` | 예외 등록용 학생 검색 (LMS 실시간·미저장) | Spring(:8080) | 불필요 | ✅ |
| API-011 | GET | `/api/v1/detection/status` | 탐지 파이프라인 상태 조회 | FastAPI(:8000) | 불필요 | ✅ |
| API-012 | POST | `/api/v1/detection/registry/reload` | 등록 임베딩 리로드 | FastAPI(:8000) | 불필요 | ✅ |
| API-013 | GET | `/api/v1/internal/detection-window` | 오늘의 퇴실(=탐지) 창 조회 (Spring→FastAPI) | Spring(:8080) | 내부 | ✅ |
| API-014 | GET | `/api/v1/detection/records` | 탐지 기록 검색 (날짜·시각·학생·카메라) | FastAPI(:8000) | 불필요 | ✅ |
| API-014b | GET | `/api/v1/detection/records/{id}/photo` | 탐지 사진 — **서명 URL 필수** | FastAPI(:8000) | 서명 | ✅ |
| API-015 | GET | `/api/v1/attendance/history` | 누적 출결 이력 조회 (날짜·학생) | Spring(:8080) | 불필요 | ✅ |
| API-016 | GET | `/api/v1/enroll/registered` | 등록 완료 학생 목록 (임베딩 제외) | FastAPI(:8000) | 불필요 | ✅ |
| API-017 | POST | `/api/v1/enroll/sessions` | 가이드 촬영 세션 시작 | FastAPI(:8000) | 불필요 | ✅ |
| API-017b | POST | `/api/v1/enroll/sessions/{id}/frames` | 프레임 1장 자세·품질 판정 | FastAPI(:8000) | 불필요 | ✅ |
| API-017c | POST | `/api/v1/enroll/sessions/{id}/commit` | 등록 확정 (임베딩 저장 + 등록부 리로드) | FastAPI(:8000) | 불필요 | ✅ |
| API-017d | DELETE | `/api/v1/enroll/sessions/{id}` | 촬영 취소 (임베딩 즉시 폐기) | FastAPI(:8000) | 불필요 | ✅ |
| API-018 | DELETE | `/api/v1/enroll/registered/{student_id}` | 등록 얼굴 삭제 (동의 철회·오등록) | FastAPI(:8000) | 불필요 | ✅ |
| API-019 | POST | `/api/v1/internal/detections/faces` | 엣지 탐지 결과 수신 (Jetson→FastAPI) | FastAPI(:8000) | 내부 | 📝 |
| API-019b | GET | `/api/v1/internal/edge/config` | 엣지 촬영 설정 조회 (Jetson→FastAPI, 폴링) | FastAPI(:8000) | 내부 | 📝 |
| API-021 | POST | `/api/v1/briefing/run` | 일일 브리핑 수동 실행 (PDF 생성 → 메일 발송) | FastAPI(:8000) | 불필요 | ✅ |
| API-021b | GET | `/api/v1/briefing/status` | 브리핑 설정·발송 준비 상태 조회 | FastAPI(:8000) | 불필요 | ✅ |
| API-020 | POST | `/api/v1/internal/nudge-message` | 넛지 동적 메시지 본문 생성 (Spring→FastAPI, LLM-004) | FastAPI(:8000) | 내부 | ✅ |

> API-003~010은 SCR-001~003 화면 설계(specs/11)에서 도출된 목록 — 전부 상세 명세 완료 (문서 우선 원칙). 구현은 도메인별 담당자 몫
> API-011~012는 탐지 파이프라인 운영 API (FastAPI) — SCR-001 "퇴실 감시 중" 뱃지·등록부 무중단 갱신용
> API-009·013은 화면이 부르지 않는 **서버 간 내부 API**다 (FastAPI↔Spring). 각각 탐지 이벤트 발행·탐지 창 동기화
> **API-019·019b는 엣지(Jetson)↔FastAPI 내부 API**다 — 탐지를 카메라 쪽으로 옮길 때의 계약. ⚠ **아직 설계 단계이고 현행은 서버가 스트림에서 직접 탐지한다.** 계약을 미리 잡아 두는 이유는, 그 형태에 맞춰 서버를 리팩터링해 두면 전환이 "입력 소스 교체"로 끝나기 때문이다 (API-019 비고 마지막 항목)
> **API-016·017은 얼굴 등록(SCR-004)**이다 — CLI(`tools/enroll_capture.py`+`register_faces.py`)로만 가능하던 등록을 행정실 브라우저로 옮겼다. **판정 로직은 CLI와 한 벌**(`app/services/enroll.py`)이고, 화면은 프레임만 보낸다. ⚠ **미등록자 목록(당일 명단 ∖ 등록부)은 서버가 아니라 화면이 합친다** — 명단은 Spring(API-010), 등록부는 FastAPI(API-016)로 소유가 갈리고(rules/04), 한쪽이 다른 쪽을 대신 조회하면 LMS 장애가 등록 화면까지 끌고 내려간다. 화면에서 합치면 명단을 못 받아도 학번·이름 직접 입력으로 등록할 수 있다
> **API-021·021b는 일일 브리핑(LLM-002, #224)의 수동 입구**다 — 자동 실행은 webapp 안의 1분 tick(`BRIEFING_SEND_HOUR`, 기본 09시)이고 이 API는 그것과 **별개**다. 개발·시연에서 09시를 기다릴 수 없고, 앱이 그 시각 이후에 떠서 건너뛴 날을 사람이 다시 만들 방법도 있어야 한다. ⚠ **발송은 두 스위치가 모두 켜져야 나간다** — 요청의 `dry_run: false`와 환경의 `BRIEFING_ENABLED=true`. 설정을 잊었을 때 실수로 메일이 나가는 쪽이 아니라 아무것도 안 나가는 쪽으로 실패한다. ⚠ `/status`는 **수신 주소·비밀번호를 내보내지 않는다** — 이 서버엔 인증이 없어(rules/02) 상태에 실린 주소는 곧 공개된 주소다. 상세 계약은 specs/14 LLM-002
> **API-014·015는 보존기간 전환(rules/04 2026-08-12)의 소비자**다 — 다음주 검색 화면과 LLM 도구가 **같은 엔드포인트**를 쓴다(화면용·LLM용을 따로 만들면 두 번 관리하게 된다). 둘로 나뉜 이유는 컬렉션 소유권이다: 사진이 붙은 탐지 기록은 FastAPI, 출결 이력은 Spring (rules/04)

## 2. API 상세 명세

---

### API-001. 스트림 수집 상태 조회

- **메서드 / URL**: `GET /api/v1/streams/status`
- **설명**: 카메라(폰/Jetson)별 영상 수집 파이프라인의 연결 상태·FPS·드롭 수를 반환한다. "영상이 서버까지 넘어오는지"를 증명하는 최소 모니터링 엔드포인트 (강사 요구 4번 대응)
- **인증**: 불필요 (02 — 인증 없음)
- **연관 화면**: SCR-001 (카메라 상태 위젯·헤더 ●●●)
- **상태**: ✅ 완료 (webapp `routers/streams.py` 구현)

#### 요청

**Path / Query 파라미터**: 없음

**Request Body**: 없음

#### 응답

**성공 (200)** — 공통 포맷 `{success, data, error}`

```json
{
  "success": true,
  "data": {
    "streams": [
      {
        "camera_id": "cam_12f_1",
        "connected": true,
        "last_frame_at": "2026-08-11T17:42:03+09:00",
        "fps": 14.2,
        "dropped_frames": 3,
        "reconnect_count": 1,
        "uptime_seconds": 1824
      }
    ]
  },
  "error": null
}
```

- `connected=false`인 카메라는 `last_frame_at`이 마지막 수신 시각으로 남는다 (재접속 시도는 백그라운드 자동)

**에러**

| HTTP | 에러 코드 | 발생 조건 |
|---|---|---|
| 500 | `STREAM_500` | 파이프라인 매니저 초기화 실패 |

#### 비고
- 폴링 주기 권장 3~5초 (프론트). 카메라 목록은 `.env`의 `CAMERA_SOURCES`(camera_id=URL 목록)로 정의 — 폰(MJPEG)→Jetson(RTSP) 전환 시 URL만 교체

---

### API-002. 실시간 화면 재송출 (MJPEG)

- **메서드 / URL**: `GET /api/v1/streams/{camera_id}/live`
- **설명**: 수집 중인 최신 프레임을 MJPEG(`multipart/x-mixed-replace`)로 재송출한다. 대시보드에서 `<img src>`로 바로 표시 (강사 요구 5번 대응). 추후 탐지 박스 오버레이 프레임으로 교체 예정
- **인증**: 불필요
- **연관 화면**: SCR-001 (라이브 뷰 3분할)
- **상태**: ✅ 완료 (webapp `routers/streams.py` 구현)

#### 요청

**Path / Query 파라미터**

| 이름 | 위치 | 타입 | 필수 | 설명 |
|---|---|---|---|---|
| camera_id | path | string | YES | 카메라 ID (`cam_12f_1` / `cam_12f_2` / `cam_1f`) |
| max_fps | query | int | NO | 송출 FPS 상한 (기본 10 — 대역폭 절약) |
| width | query | int | NO | 리사이즈 폭 px (기본 원본. 지정 시 비율 유지 축소) |

**Request Body**: 없음

#### 응답

**성공 (200)** — ⚠ **공통 JSON 포맷 예외**: 스트리밍 응답이므로 `{success,data,error}` 래핑 없음

```
Content-Type: multipart/x-mixed-replace; boundary=frame

--frame
Content-Type: image/jpeg

<JPEG 바이너리> (반복)
```

**에러** (스트림 시작 전 검증 실패 시에만 JSON 공통 포맷)

| HTTP | 에러 코드 | 발생 조건 |
|---|---|---|
| 404 | `STREAM_001` | 존재하지 않는 camera_id |
| 503 | `STREAM_002` | 카메라 미연결 상태 (수집 파이프라인 끊김) |

#### 비고
- 동시 시청 제한 3 커넥션/카메라 (데모용 — 초과 시 503 `STREAM_003`)
- 개인정보: 라이브 뷰는 행정실 대시보드 전용. 프레임 저장은 하지 않음(탐지 이벤트만 DB 기록 — 04 참조)
- 에러 코드 소유: `STREAM_*`은 FastAPI `services/video.py` 소유 (02 예외 처리 전략)

---

### API-003. 당일 출결 요약 조회

- **메서드 / URL**: `GET /api/v1/attendance/today`
- **설명**: 당일(`Asia/Seoul` 기준) 출결 요약·강의별 집계·미퇴실자 명단을 반환한다. 대시보드(SCR-001)의 요약 카드·강의별 출석 칩·미퇴실자 테이블이 이 API 하나로 동작한다. **데이터 원천은 우리 DB뿐** — `daily_roster`(명단)·`daily_courses`(정원)·`attendance_exceptions`(예외 표시)를 읽고 **LMS를 직접 조회하지 않는다** (적재는 BAT-001 폴링 담당, specs/12). LMS 장애 시에도 마지막 동기화 상태로 화면이 유지되는 것이 이 설계의 목적
- **인증**: 불필요 (02 — 인증 없음, 내부망 전제)
- **연관 화면**: SCR-001(진입 시)
- **상태**: ✅ 완료 (2026-08-07 — `attendance/adapter/in/web/AttendanceController` + `application/service/AttendanceQueryService`)
  - `exception` 필드는 `attendance_exceptions` **조회만** 구현됐다 — 등록·삭제는 API-005·007과 함께 붙인다
  - `lms_sync`는 BAT-001의 메모리 상태에서 온다 (단일 인스턴스 전제) — 재시작 시 초기화되고 다음 tick에 채워진다
  - ⚠ MongoDB 실연동 미검증 — 라우팅·에러 계약(DB 다운 시 `ATTENDANCE_500`)만 확인했다

#### 요청

**Path / Query 파라미터**: 없음 (당일 고정 — 과거 조회는 범위 외, 필요 시 별도 API로 설계)

**Request Body**: 없음

#### 응답

**성공 (200)** — 공통 포맷 `{success, data, error}`

```json
{
  "success": true,
  "data": {
    "date": "2026-08-11",
    "summary": {
      "checked_in_count": 32,
      "not_checked_out_count": 5,
      "checked_out_count": 27,
      "sms_sent_count": 2
    },
    "courses": [
      {
        "course_id": "AIG20240000498258",
        "course_name": "AI기반 지능형 솔루션 개발과정",
        "capacity": 20,
        "checked_in_count": 18
      }
    ],
    "students": [
      {
        "student_id": "100061869350",
        "student_name": "김OO",
        "course_name": "AI기반 지능형 솔루션 개발과정",
        "check_in_time": "2026-08-11T08:52:00+09:00",
        "exception": null
      },
      {
        "student_id": "100061869351",
        "student_name": "이OO",
        "course_name": "AI기반 지능형 솔루션 개발과정",
        "check_in_time": "2026-08-11T09:01:00+09:00",
        "exception": { "type": "OUTING", "start_time": "15:00", "end_time": "16:00" }
      }
    ],
    "lms_sync": {
      "last_synced_at": "2026-08-11T17:41:00+09:00",
      "healthy": true
    }
  },
  "error": null
}
```

- `students`는 **현재 `daily_roster`에 남아 있는 학생 = 미퇴실자**다. `daily_roster`에는 **입실자만 저장**되므로(rules/04 행 최소화·BAT-001 처리 4) `check_in_time`은 항상 non-null이고, **미출석자는 이 목록에 없다**
  - 대시보드 미퇴실자 테이블 = `students` 그대로 (프론트 추가 필터 불필요)
  - 퇴실 태그된 학생은 즉각 삭제 정책(rules/04)으로 목록에서 사라지고 `summary.checked_out_count` 집계로만 남는다
  - **SCR-003 학생 검색은 이 API가 아니라 API-010을 쓴다** — 병결 등 미출석자 대상 예외 등록이 필요하기 때문
- `check_in_time` 유무가 출석 이진 판정 그 자체 (2026-08-06 결정, 99 참조) — "roster에 없다 = 미출석"이 판정이다. 경과 시간은 프론트가 `check_in_time`으로 계산
- `exception.type`: `OUTING`(외출) / `EARLY_LEAVE`(조퇴) / `SICK_LEAVE`(병결) / `MORNING_CLASS`(오전반) / `AFTERNOON_CLASS`(오후반) — 저장·전송은 영문 enum, 한글 라벨은 프론트 매핑 (rules/09 경계면 규칙)
- `courses`: 강의별 칩(SCR-001)용 — **원천은 `daily_courses` 하나**(`capacity`·`checked_in_count` 둘 다 그 문서에 있다). **칩 표시는 `checked_in_count` / `capacity`**(예: 18/20). 결석자만큼 차이 나는 것이 정상이므로 이 둘의 차이는 경고가 아니다 — 수집 이상 감지는 BAT-001이 LMS 응답 자체에서 검증하고 `lms_sync.healthy`로 내려준다
  - ⚠ **`checked_in_count`를 `daily_roster`에서 세면 안 된다** — roster는 퇴실 시 삭제되므로 전원 퇴실한 강의가 "입실 0"으로 보인다. `daily_courses.checked_in_count`에 누적으로 유지한다 (2026-08-07 시드 작업 중 발견)
- `student_id`는 LMS `trneeCstmrId`(12자리 문자열) 그대로 (specs/12 실스펙)
- `lms_sync.healthy=false`면 대시보드 상단 배너 "LMS 연동 이상 — 마지막 동기화 HH:mm" 표시 (BAT-001 연속 5회 실패 기준)

**에러**

| HTTP | 에러 코드 | 발생 조건 |
|---|---|---|
| 500 | `ATTENDANCE_500` | 조회 실패 (DB 접근 불가 등) |

#### 비고
- 명단 미수신(폴링 전·주말 등)은 에러가 아님 — `students: []` + `summary` 0값으로 200 응답 (SCR-001 "데이터 없음" 상태 대응)
- 미출석자 수는 이 API로 내려가지 않는다 — 개인정보를 저장하지 않는 대신 익명 집계(`daily_stats.absent_count`)와 강의별 입실 마감 Slack 알림(BAT-002)으로 다룬다
- 폴링 주기: 진입 시 1회 + WebSocket(API-008) 이벤트 수신 시 TanStack Query 캐시 무효화로 재조회 — 주기 폴링 없음
- 응답 규모: 수강생 수십 명 × 5필드 수준이라 페이징 불필요
- 에러 코드 소유: `ATTENDANCE_*`는 Spring `attendance/domain/exception/` ErrorCode enum 소유 (rules/02 예외 처리 전략)
- 개인정보: 연락처는 응답에 포함하지 않는다 (SCR-001 비고 — 이름만 표시)

---

### API-004. 미태그 포착 알람 목록

- **메서드 / URL**: `GET /api/v1/detections`
- **설명**: API-009로 수신한 탐지 이벤트의 **판정 결과**(퇴실 미체크 잔존 여부·문자 발송 상태)를 필터·페이징으로 조회한다. 데이터 원천은 **Spring 소유 `detection_alerts` 컬렉션**(rules/04) — FastAPI 소유 `detection_events`(탐지 원본)와 구분되며, 이 API는 판정·발송 상태가 붙은 Spring 쪽 문서만 반환한다
- **인증**: 불필요
- **연관 화면**: SCR-002 (진입/필터 변경 시)
- **상태**: ✅ 완료 (backend `detection` — DetectionQueryService)

#### 요청

**Path / Query 파라미터**

| 이름 | 위치 | 타입 | 필수 | 설명 |
|---|---|---|---|---|
| date | query | string(YYYY-MM-DD) | NO | 기본 오늘(Asia/Seoul). TTL 보존 기간 내 날짜만 의미 있음 (비고) |
| camera_id | query | string | NO | `cam_12f_1` / `cam_12f_2` / `cam_1f` |
| sms_status | query | string | NO | `PENDING` / `SENT` / `FAILED` / `NOT_TARGET` (SCR-002 발송상태 필터) |
| q | query | string | NO | 학생명 부분 일치 검색 (미매칭 건은 이름이 없어 검색 결과에서 제외됨) |
| page | query | int | NO | 0부터 시작 (기본 0 — Spring Pageable 관례) |
| size | query | int | NO | 페이지 크기 (기본 20, 최대 100) |

- 정렬: `detected_at` 내림차순 고정 (정렬 파라미터 없음 — 신규 알람은 WS로 최상단 삽입되므로 서버 정렬과 일치)

**Request Body**: 없음

#### 응답

**성공 (200)** — 공통 포맷 `{success, data, error}`

```json
{
  "success": true,
  "data": {
    "alerts": [
      {
        "alert_id": "66b41a...",
        "event_id": "66b3f2...",
        "camera_id": "cam_12f_1",
        "detected_at": "2026-08-11T17:42:03+09:00",
        "matched_student_id": "100061869350",
        "student_name": "김OO",
        "similarity": 0.87,
        "bbox": [412, 180, 96, 96],
        "roster_status": "REMAINING",
        "sms_status": "SENT",
        "sms_sent_at": "2026-08-11T17:42:41+09:00"
      }
    ],
    "page": { "number": 0, "size": 20, "total_elements": 43, "total_pages": 3 }
  },
  "error": null
}
```

- `roster_status` — 수신 시점 판정 근거 (SCR-002 상세 패널의 "판정 근거"):
  - `REMAINING`: 퇴실 미체크로 daily_roster 잔존 → **문자 플로우 대상** (단, 당일 예외 등록자는 제외 — API-005)
  - `NOT_IN_ROSTER`: 이미 퇴실 태그됨(roster에서 삭제) 또는 명단 밖 인물
  - `UNMATCHED`: 얼굴 미매칭 — `matched_student_id`·`student_name`·`sms_sent_at`은 `null`
- `sms_status`: `PENDING` / `SENT` / `FAILED` / `NOT_TARGET` — 발송 대상은 "`REMAINING` AND 당일 예외 없음"뿐, 그 외는 즉시 `NOT_TARGET`
  - `PENDING` = 발송 대상으로 판정됐고 아직 결과가 오지 않은 구간. 판정 직후부터 발송 확정까지 항상 지나가며(수 초~수십 초), 대조 폴링이 늦어지면 더 길어진다. **null이 아니다** — 2026-08-09까지 이 자리가 null이라 화면이 매핑할 값이 없었다(뱃지 빈칸)
- 미매칭(`UNMATCHED`) 건도 목록에 포함한다 (SCR-002 — 회색 표시, 판정 투명성·05 안전장치)

**에러**

| HTTP | 에러 코드 | 발생 조건 |
|---|---|---|
| 400 | `COMMON_400` | date·page·size 형식 오류, 알 수 없는 enum 값 |
| 500 | `DETECTION_500` | 조회 실패 |

#### 비고
- 알람 문서는 `student_name`(PII)을 포함하므로 **TTL 24시간**(rules/04) — 날짜 필터는 사실상 오늘·어제 조회용이다. 장기 이력은 익명 통계(`daily_stats`)로만 남는다 (04 개인정보 라이프사이클)
- `event_id`로 `detection_events` 원본 역추적 가능 (원본도 TTL 24h — webapp 소유)
- 스냅샷 이미지는 없음 — 프레임 미저장 정책 (API-002 비고). 상세 패널은 메타데이터만 표시
- 에러 코드 소유: `DETECTION_*`은 Spring detection 도메인 (rules/02)

---

### API-005. 예외 등록

- **메서드 / URL**: `POST /api/v1/exceptions`
- **설명**: 당일 예외(외출·조퇴·병결·오전반·오후반)를 등록한다. 등록 즉시 **BAT-002 미출석 알림 대상**과 **퇴실 문자 플로우 대상**에서 제외된다 — 제외는 저장 시점 플래그가 아니라 **판정 시점에 `attendance_exceptions`를 조회**하는 방식이라 등록 순서와 무관하게 반영된다. 저장 컬렉션은 Spring 소유 `attendance_exceptions` (rules/04)
- **인증**: 불필요
- **연관 화면**: SCR-003 ([등록] 클릭)
- **상태**: ✅ 완료 (backend `attendance` — ExceptionService)

#### 요청

**Path / Query 파라미터**: 없음

**Request Body**

```json
{
  "student_id": "100061869351",
  "type": "OUTING",
  "start_time": "15:00",
  "end_time": "16:00",
  "reason": "병원 방문"
}
```

| 필드 | 타입 | 필수 | 설명 |
|---|---|---|---|
| student_id | string | YES | **당일 LMS 수강생이면 등록 가능** — 미출석자(병결 대상)도 포함하므로 `daily_roster` 존재 여부로 검증하지 않는다. 검증·검색 원천은 API-010 |
| type | string | YES | `OUTING` / `EARLY_LEAVE` / `SICK_LEAVE` / `MORNING_CLASS` / `AFTERNOON_CLASS` (API-003과 동일 enum) |
| start_time | string(HH:mm) | 조건부 | `OUTING`·`EARLY_LEAVE`·`SICK_LEAVE`는 필수 (SCR-003 입력 규칙) |
| end_time | string(HH:mm) | 조건부 | `OUTING`만 필수 — `start_time`보다 뒤여야 함. 그 외 유형은 보내도 무시 |
| reason | string | NO | 100자 이하 (SCR-003) |

- `MORNING_CLASS`·`AFTERNOON_CLASS`는 시간 입력 없음 — 유형 자체가 시간대를 내포 (SCR-003 입력란 비활성)

#### 응답

**성공 (201)** — 공통 포맷. `data` = 생성된 예외 문서

```json
{
  "success": true,
  "data": {
    "exception_id": "66b4c1...",
    "date": "2026-08-11",
    "student_id": "100061869351",
    "student_name": "이OO",
    "type": "OUTING",
    "start_time": "15:00",
    "end_time": "16:00",
    "reason": "병원 방문",
    "source": "MANUAL",
    "created_at": "2026-08-11T14:20:11+09:00"
  },
  "error": null
}
```

- `source`: `MANUAL`(이 API로 행정실이 등록) / `LMS_AUTO`(BAT-003이 LMS `checkoutStatus="조퇴"`를 보고 자동 등록 — specs/12). **이 API로 생성되는 문서는 항상 `MANUAL`** (요청 본문으로 받지 않는다)

**에러**

| HTTP | 에러 코드 | 발생 조건 |
|---|---|---|
| 400 | `COMMON_400` | 필수 누락·형식 오류·유형별 시간 규칙 위반(end_time ≤ start_time 포함) |
| 404 | `ATTENDANCE_001` | 당일 LMS 수강생 명단에 없는 학생 (API-010 원천 기준 — 미출석자는 정상 등록 대상이므로 404가 아니다) |
| 409 | `ATTENDANCE_002` | 동일 학생 + 동일 유형 당일 중복 등록 (SCR-003 비고) |
| 502 | `ATTENDANCE_005` | LMS 조회 실패 — 학생 검증 원천이 API-010이라 LMS가 죽으면 등록도 불가하다. "학생이 없다"(404)와 구분되어야 행정실이 재시도할지 이름을 다시 확인할지 판단할 수 있다 |
| 500 | `ATTENDANCE_500` | 저장 실패 |

- ⚠ **409는 이 API(수동 등록)에만 적용된다** — BAT-003의 자동 등록은 같은 unique 키에 대해 멱등 upsert로 처리하고, 이미 `MANUAL` 문서가 있으면 덮어쓰지 않고 skip한다 (행정실이 적은 사유 보존)

#### 비고
- `student_name`은 등록 시점 스냅샷으로 저장 — 학생이 퇴실해 roster에서 삭제된 뒤에도, 그리고 **애초에 미출석이라 roster에 없던 경우에도** 당일 예외 목록에 이름 표시 가능 (문서 자체가 TTL로 익일 소멸하므로 개인정보 정책과 상충 없음, rules/04)
- **미출석자 대상 등록(주로 `SICK_LEAVE` 병결)이 정상 경로다** — 이 경우 `attendance_exceptions`가 그 학생의 이름이 우리 DB에 남는 유일한 곳이며, BAT-002 미출석 알림 제외의 근거가 된다
- 등록 성공 시 WS `exception_changed` 이벤트 발행 (API-008) — 대시보드 예외 컬럼·예외 목록 실시간 갱신
- 중복 판정 키: `(date, student_id, type)` unique 인덱스 — 409의 근거
- 에러 코드 소유: `ATTENDANCE_*`는 Spring attendance 도메인 (예외는 출결 판정의 일부)

---

### API-006. 당일 예외 목록

- **메서드 / URL**: `GET /api/v1/exceptions`
- **설명**: 당일(Asia/Seoul) 등록된 예외 전체를 반환한다. 당일 고정 — 과거 조회는 범위 외 (API-003과 동일 원칙, 문서 자체가 TTL로 익일 소멸)
- **인증**: 불필요
- **연관 화면**: SCR-003 (진입 시)
- **상태**: ✅ 완료 (backend `attendance` — ExceptionService)

#### 요청

**Path / Query 파라미터**: 없음

**Request Body**: 없음

#### 응답

**성공 (200)** — 공통 포맷. `exceptions`는 API-005 응답 `data`와 동일 구조의 배열, `created_at` 내림차순

```json
{
  "success": true,
  "data": {
    "date": "2026-08-11",
    "exceptions": [
      {
        "exception_id": "66b4c1...",
        "student_id": "100061869351",
        "student_name": "이OO",
        "type": "OUTING",
        "start_time": "15:00",
        "end_time": "16:00",
        "reason": "병원 방문",
        "source": "MANUAL",
        "created_at": "2026-08-11T14:20:11+09:00"
      }
    ]
  },
  "error": null
}
```

- 등록된 예외가 없으면 `exceptions: []`로 200 응답 (에러 아님 — SCR-003 "등록된 예외가 없습니다")
- `source="LMS_AUTO"` 항목은 **프론트에서 삭제 버튼을 표시하지 않는다** — 원천이 LMS 태그라 지워도 다음 배치에 다시 생성됨 (API-007 비고)

**에러**

| HTTP | 에러 코드 | 발생 조건 |
|---|---|---|
| 500 | `ATTENDANCE_500` | 조회 실패 |

#### 비고
- 하루 수십 건 이하 규모라 페이징 없음
- 한글 라벨(외출·조퇴 등)은 프론트 매핑 — 저장·전송은 영문 enum (rules/09 경계면 규칙)

---

### API-007. 예외 삭제

- **메서드 / URL**: `DELETE /api/v1/exceptions/{id}`
- **설명**: 등록된 예외를 삭제한다. **삭제 즉시 해당 학생은 판정·문자 대상으로 복귀**하므로 프론트는 확인 다이얼로그를 필수로 거친다 (SCR-003)
- **인증**: 불필요
- **연관 화면**: SCR-003 ([삭제] 클릭)
- **상태**: ✅ 완료 (backend `attendance` — ExceptionService)

#### 요청

**Path / Query 파라미터**

| 이름 | 위치 | 타입 | 필수 | 설명 |
|---|---|---|---|---|
| id | path | string | YES | `exception_id` (ObjectId 문자열) |

**Request Body**: 없음

#### 응답

**성공 (200)** — 공통 포맷. `data`: `{ "deleted": true }`

**에러**

| HTTP | 에러 코드 | 발생 조건 |
|---|---|---|
| 404 | `ATTENDANCE_003` | 존재하지 않는 예외 (이미 삭제됨·TTL 소멸 포함) |
| 409 | `ATTENDANCE_004` | `source="LMS_AUTO"` 예외 삭제 시도 — LMS 태그가 원천이라 삭제 대상이 아님 |
| 500 | `ATTENDANCE_500` | 삭제 실패 |

#### 비고
- 같은 id 재삭제는 404 — 프론트는 404를 "이미 삭제됨"으로 안내하고 목록만 갱신하면 됨
- 삭제 성공 시 WS `exception_changed` 이벤트 발행 (API-008)
- **`MANUAL` 예외만 삭제 가능** — `LMS_AUTO`(BAT-003 자동 등록)는 지워도 다음 실행에 재생성되므로 서버가 409로 막고, 프론트는 애초에 삭제 버튼을 감춘다 (API-006 참조)

---

### API-008. 실시간 이벤트 (WebSocket)

- **메서드 / URL**: `WS /ws` (HTTP GET 업그레이드)
- **설명**: 서버→클라이언트 **단방향 브로드캐스트** 채널. 포착·문자 발송·퇴실 태그 등 실시간 이벤트를 모든 연결 클라이언트에 동일하게 전달한다. 이벤트는 "피드 표시용 최소 데이터 + 갱신 신호"이며, 정합 데이터는 이벤트 수신 시 API-003/004/006 재조회(TanStack Query 캐시 무효화)로 가져온다
- **프로토콜 결정**: **순수 WebSocket + JSON 텍스트 프레임** — STOMP·SockJS 미사용. 구독 토픽이 필요 없는 소규모 단방향 브로드캐스트이고, 최신 Chrome·Edge 전제(rules/03)라 폴백도 불필요 (결정일 2026-08-06)
- **인증**: 불필요
- **연관 화면**: SCR-001(이벤트 피드·카드 갱신), SCR-002(신규 알람 삽입·발송상태 갱신), SCR-003(예외 목록 갱신)
- **상태**: ✅ 구현 (2026-08-07 — `global/websocket` 핸들러·브로드캐스터, 프론트 수신부)

#### 메시지 형식

⚠ **공통 JSON 포맷 예외**: `{success,data,error}` 래핑 없음 — 아래 이벤트 봉투 사용 (필드는 snake_case, rules/09)

```json
{
  "type": "detection_alert",
  "occurred_at": "2026-08-11T17:42:03+09:00",
  "data": { }
}
```

#### 이벤트 타입

| type | 발행 시점 (발행 도메인) | data 필드 | 소비 화면 |
|---|---|---|---|
| `roster_synced` | BAT-001 폴링 반영 시 — 변경 있을 때만 (attendance) | `{ "changed_count": 3, "last_synced_at": "..." }` | SCR-001 카드·칩 재조회 신호 |
| `check_out` | 퇴실 태그 확인·roster 삭제 시 (attendance) | `{ "student_id", "student_name", "course_name", "checked_out_at" }` | SCR-001 피드 "퇴실 태그" 항목 + 카드 갱신 |
| `detection_alert` | API-009 수신·판정 직후 (detection) | API-004 알람 1건과 동일 구조 (`alert_id`~`sms_status`) | SCR-001 피드 "포착" 항목, SCR-002 최상단 삽입+하이라이트 |
| `sms_result` | 문자(RPA) 발송 성공/실패 확정 시 (notification) | `{ "alert_id", "student_id", "student_name", "status": "SENT\|FAILED", "sent_at" }` | SCR-001 피드 "문자 발송" 항목 + 문자 카드, SCR-002 발송상태 뱃지 갱신 |
| `exception_changed` | API-005 등록 / API-007 삭제 성공 시 (attendance) | `{ "action": "CREATED\|DELETED", "exception_id", "student_id" }` | SCR-001 테이블 예외 컬럼, SCR-003 목록 재조회 |
| `lms_sync_status` | BAT-001 연속 5회 실패 경고 / 복구 시 (attendance) | `{ "healthy": false, "last_synced_at": "..." }` | SCR-001 상단 배너 (specs/12 BAT-001 알림) |

#### 동작 규칙

- **수신 전용**: 클라이언트→서버 메시지는 없다 — 서버는 수신 텍스트를 무시한다
- **구독 개념 없음**: 연결된 모든 클라이언트가 동일 스트림 수신 (행정실 소수 클라이언트 전제 — 연결 수 제한 없음)
- **재전송 없음**: 서버는 무상태 브로드캐스트 — 연결 끊김 중 이벤트는 재전송하지 않는다. 클라이언트는 **재연결 시 API-003/004/006 전체 재조회**로 복구 (rules/03 재연결 로직, 헤더 ⚠ 표시)
- **keepalive**: 서버가 30초 간격 ping 프레임 발송 (브라우저 자동 pong) — 프록시 유휴 타임아웃 방지
- 핸드셰이크 실패는 표준 HTTP 응답 (JSON 공통 포맷 아님)

#### 비고
- SCR-001 이벤트 피드는 클라이언트에서 최대 50건 유지 (11 참조) — 서버는 히스토리를 제공하지 않음
- 이벤트 계약(이 표)만 이 문서가 고정한다 — WS 세션 관리·브로드캐스터의 구현 배치(각 도메인 vs 공유 인프라)는 구현 팀 협의로 결정. 공유 코드로 두는 경우 rules/02 공유 코드 수정 정책(CTO 리뷰 경유) 적용
- `check_out`·`sms_result`의 이름 필드는 피드 표시용 — 클라이언트 보관은 피드 50건 한도 내 메모리뿐 (개인정보 정책과 상충 없음)

---

### API-009. 탐지 이벤트 수신 (FastAPI→Spring)

- **메서드 / URL**: `POST /api/v1/internal/detections`
- **설명**: 퇴실 시간대에 webapp(FastAPI) 탐지 파이프라인이 얼굴 식별 결과를 Spring으로 발행한다. Spring은 이 이벤트로 "퇴실 미체크(daily_roster 잔존) AND 카메라 식별 포착" 조건을 판정해 복귀 요청 문자 플로우를 태운다 (99 2026-08-05 결정). 이벤트 원본은 `detection_events` 컬렉션(FastAPI 소유)에 이미 저장된 상태로 발행된다
- **인증**: 내부 (내부망 전제 — 별도 토큰 없음, 02)
- **연관 화면**: SCR-001 이벤트 피드·SCR-002 알람 목록 (Spring이 API-008 WS로 중계)
- **상태**: ✅ 구현 (발행측 FastAPI 2026-08-06, 수신측 Spring 2026-08-07)

#### 요청

**Path / Query 파라미터**: 없음

**Request Body**

```json
{
  "event_id": "66b3f2...",
  "camera_id": "cam_12f_1",
  "detected_at": "2026-08-06T17:42:03+09:00",
  "matched_student_id": "100061869350",
  "student_name": "김도윤",
  "similarity": 0.87,
  "bbox": [412, 180, 96, 96]
}
```

| 필드 | 타입 | 필수 | 설명 |
|---|---|---|---|
| event_id | string | YES | `detection_events` 문서 _id — 멱등 처리·역추적 키 |
| camera_id | string | YES | `cam_12f_1` / `cam_12f_2` / `cam_1f` |
| detected_at | string(ISO) | YES | 탐지 시각 (Asia/Seoul 오프셋 포함) |
| matched_student_id | string \| null | YES | 매칭된 등록자 ID — **미매칭(임계값 미달) 시 null** |
| student_name | string \| null | NO | 매칭된 등록자 이름 (`registered_faces.student_name`) — 미매칭 시 null. 2026-08-19 추가: Spring이 daily_roster로 이름을 못 찾는 경우(전원 퇴실 등)에도 `detection_alerts.student_name`이 채워지도록 발행측이 실어 보낸다. **필수 아님(하위 호환)** — 없으면 Spring이 기존 방식대로 조회 |
| similarity | number \| null | YES | 코사인 유사도 (미매칭 시에도 최고 유사도 값, 탐지만 되고 등록자 0명이면 null) |
| bbox | int[4] | YES | 원본 프레임 기준 [x, y, w, h] |

#### 응답

**성공 (200)** — 공통 포맷. `data`: `{ "received": true }`

**에러**

| HTTP | 에러 코드 | 발생 조건 |
|---|---|---|
| 400 | `COMMON_400` | 필수 필드 누락·형식 오류 (Bean Validation) |
| 500 | `DETECTION_500` | 수신 처리 실패 |

#### 비고
- **발행 실패 허용**: Spring 미기동·오류 시 FastAPI는 경고 로그만 남기고 파이프라인을 계속 돌린다 (보조 시스템 — 이벤트 원본은 `detection_events`에 보존, 재발행 없음)
- **중복 억제는 발행측 소유**: 같은 (camera_id, student_id)는 쿨다운(기본 60초, `.env`) 내 재발행하지 않음 — Spring은 받은 이벤트를 신뢰하면 됨. event_id 멱등 체크는 수신측 방어선
- 스냅샷 이미지는 보내지 않음 — 프레임 미저장 정책 (API-002 비고·rules/04)
- **이름 표시 (2026-08-19, 수신측 구현 동일자)**: 수신측(Spring)은 daily_roster 조회 이름을 우선하되, 조회가 이름을 못 주면(전원 퇴실 등) 발행측 `student_name`으로 폴백해 `detection_alerts.student_name`에 저장한다 — daily_roster 기반 조회만으로는 잔존 0명일 때 실패해 SCR-002·개발자 뷰에서 식별된 인물이 "미매칭"으로 보이는 문제가 있었다. NOT_IN_ROSTER 판정과 이름 표시는 별개다
- 에러 코드 소유: `DETECTION_500`은 Spring detection 도메인 (rules/02)

---

### API-010. 예외 등록용 학생 검색

- **메서드 / URL**: `GET /api/v1/attendance/students`
- **설명**: 예외 등록(SCR-003) 화면의 학생 검색 전용. **미출석자에게도 병결·조퇴 예외를 등록해야 하는데 `daily_roster`에는 입실자만 저장**되므로(rules/04 행 최소화), 이 API만 예외적으로 **LMS 스냅샷을 실시간 조회**해 당일 수강생 전원을 반환한다. **조회 결과는 저장하지 않는다** — 응답으로 흘려보낼 뿐이며, 실제 저장은 행정실이 예외를 등록한 소수에 한해 `attendance_exceptions`에만 발생한다
- **인증**: 불필요
- **연관 화면**: SCR-003 (학생 검색)
- **상태**: ✅ 완료 (backend `attendance` — StudentDirectoryService)

#### 요청

**Path / Query 파라미터**

| 이름 | 위치 | 타입 | 필수 | 설명 |
|---|---|---|---|---|
| q | query | string | NO | 이름 부분 일치. 미지정 시 전원 반환 (수십 명 규모) |

**Request Body**: 없음

#### 응답

**성공 (200)** — 공통 포맷 `{success, data, error}`

```json
{
  "success": true,
  "data": {
    "students": [
      {
        "student_id": "100061869350",
        "student_name": "김OO",
        "course_name": "AI기반 지능형 솔루션 개발과정",
        "checked_in": true
      },
      {
        "student_id": "100061869351",
        "student_name": "박OO",
        "course_name": "AI기반 지능형 솔루션 개발과정",
        "checked_in": false
      }
    ]
  },
  "error": null
}
```

- `checked_in`: LMS `checkInTime` 유무 — 프론트가 미출석자를 구분 표시(예: 회색 + "미출석")해 병결 등록 대상을 찾기 쉽게 한다
- 이름·강의명 외 개인정보는 내려보내지 않는다 (`birth`는 애초에 읽지 않음 — specs/12 화이트리스트)

**에러**

| HTTP | 에러 코드 | 발생 조건 |
|---|---|---|
| 502 | `ATTENDANCE_005` | LMS 조회 실패 — 프론트는 "LMS 연결 실패, 잠시 후 재시도" 안내 |
| 500 | `ATTENDANCE_500` | 처리 실패 |

#### 비고
- **이 API는 rules/04 "저장 최소화"의 예외가 아니라 그 결과물**이다 — 미출석자를 저장하지 않기로 했기 때문에 필요한 순간에만 조회하는 구조가 됐다
- LMS 장애 시 502로 실패하지만 대시보드(API-003)는 `daily_roster` 기반이라 영향받지 않는다 — 예외 등록만 일시 불가
- 호출 빈도는 행정실이 예외 등록 화면을 열 때뿐이라 rate limit 영향이 미미하다 (specs/12 호출량 관리)
- 에러 코드 소유: `ATTENDANCE_*`는 Spring attendance 도메인

---

### API-011. 탐지 파이프라인 상태 조회

- **메서드 / URL**: `GET /api/v1/detection/status`
- **설명**: 탐지 파이프라인의 가동 상태를 반환한다. SCR-001 **"퇴실 감시 중" 뱃지의 데이터 소스** — 뱃지 점등 조건은 `monitoring`(= `enabled AND running AND in_window`) 하나만 보면 된다. 스트림 수집 상태(API-001)와 별개로 "탐지·식별이 실제로 돌고 있는지"를 증명하는 모니터링 엔드포인트
- **인증**: 불필요 (02 — 인증 없음)
- **연관 화면**: SCR-001 (헤더 "퇴실 감시 중" 뱃지·카메라 상태 위젯)
- **상태**: ✅ 완료 (webapp `routers/detection.py` 구현)

#### 요청

**Path / Query 파라미터**: 없음

**Request Body**: 없음

#### 응답

**성공 (200)** — 공통 포맷 `{success, data, error}`

```json
{
  "success": true,
  "data": {
    "enabled": true,
    "running": true,
    "in_window": false,
    "monitoring": false,
    "window": "13:00-15:00,17:30-19:10,21:30-23:00",
    "weekdays_only": true,
    "backend": "glintr100.onnx+adaface_ir101.onnx",
    "threshold": 0.4,
    "registered_count": 9,
    "emitted_total": 12,
    "last_event_at": "2026-08-07T18:02:11+09:00"
  },
  "error": null
}
```

| 필드 | 설명 |
|---|---|
| enabled | `.env DETECTION_ENABLED` — 파이프라인 기동 시도 여부 |
| running | 파이프라인 태스크가 실제 살아 있는지. `enabled=true`인데 `running=false`면 기동 실패 상태 (프론트 ⚠ 표시) |
| in_window | 현재 시각이 퇴실 탐지 창(`window`) 안인지 (주말 제외 규칙 포함) |
| monitoring | **뱃지 점등 조건** = enabled AND running AND in_window |
| threshold | 매칭 임계값 — ⚠ 현재 임시값 (2026-08-07 2차 실측에서 무효화, rules/05) |
| registered_count | 로드된 등록 인원 수. 0이면 전부 미매칭 이벤트로만 기록됨 |
| emitted_total | 서버 기동 후 발행한 탐지 이벤트 누적 수 (재시작 시 0) |
| last_event_at | 마지막 탐지 이벤트 발행 시각 (없으면 null) |

- `enabled=false`(개발 기본값)면 `running`·`in_window`·`monitoring`은 false, 카운터는 0 — 200 정상 응답 (에러 아님)

**에러**

| HTTP | 에러 코드 | 발생 조건 |
|---|---|---|
| 500 | `PIPELINE_500` | 처리 실패 |

#### 비고
- 폴링 주기 권장 3~5초 — API-001과 같은 주기로 묶어 호출
- 카운터는 메모리 값 (탐지 원본 조회는 `detection_events`, 판정·알람 조회는 API-004)
- 에러 코드 소유: `PIPELINE_*`은 FastAPI `services/detection.py` (Spring detection 도메인의 `DETECTION_*`과 구분 — rules/02)

#### 보조: `GET /api/v1/detection/live` — 원시 탐지 스냅샷 (2026-08-19, 개발자 뷰 B단계)

SCR-DEV 라이브 영상 위 실시간 bbox 오버레이용. `detection_alert`(60초 쿨다운)로는 얼굴을 따라다니는 박스를 그릴 수 없어, **마지막 탐지 사이클**의 카메라별 원시 결과를 메모리 스냅샷으로 노출한다 (저장 없음 — 개인정보 최소 보유, rules/04).

```json
{ "in_window": true, "interval_seconds": 0.4, "cameras": {
    "cam_pi": { "at": "2026-08-19T19:50:01+09:00", "frame_w": 640, "frame_h": 480,
      "faces": [{ "bbox": [17, 0, 137, 133], "det_score": 0.9,
                  "student_id": "100061120980", "student_name": "황서범", "similarity": 0.77 }] } } }
```

- bbox는 **원본 프레임 좌표** — 화면이 `frame_w/h`로 스케일한다. 미매칭이면 student_* = null
- 2단계 모드(`DETECTION_STAGED=true`)에서는 신원 확정 전이라 bbox·det_score만 온다 (student_* 전부 null)
- 창 밖·정지 시에는 마지막 값이 남는다 — 화면은 `at`으로 신선도를 판단한다 (5초 넘으면 미표시)
- 폴링 주기 권장 300~500ms — 파이프라인 사이클(AIMD 0.2~1초)과 같은 자릿수. 파이프라인 미기동이면 `PIPELINE_001`(503)

---

### API-012. 등록 임베딩 리로드

- **메서드 / URL**: `POST /api/v1/detection/registry/reload`
- **설명**: 등록 임베딩(`registered_faces` 또는 `REGISTRY_FILE` JSON)을 **서버 재시작 없이** 다시 로드한다. `tools/register_faces.py`로 동의자를 추가·갱신한 뒤 이 API를 호출하면 다음 탐지 사이클부터 반영된다
- **인증**: 불필요
- **연관 화면**: 없음 (운영 도구 — curl/스크립트 호출)
- **상태**: ✅ 완료 (webapp `routers/detection.py` 구현)

#### 요청

**Path / Query 파라미터**: 없음

**Request Body**: 없음

#### 응답

**성공 (200)** — 공통 포맷 `{success, data, error}`

```json
{
  "success": true,
  "data": {
    "registered_count": 9,
    "source": "mongo"
  },
  "error": null
}
```

- `source`: `mongo`(registered_faces) / `json`(REGISTRY_FILE 지정 시)

**에러**

| HTTP | 에러 코드 | 발생 조건 |
|---|---|---|
| 503 | `PIPELINE_001` | 탐지 파이프라인 미가동 (`DETECTION_ENABLED=false` 또는 기동 실패) — 리로드할 대상이 없음 |
| 500 | `PIPELINE_002` | 리로드 실패 (JSON 파일 없음·MongoDB 미접속 등) — **기존 등록부는 그대로 유지**되므로 탐지는 계속 돈다 |

#### 비고
- 멱등 — 같은 소스를 다시 읽을 뿐이므로 몇 번을 호출해도 안전. POST인 이유는 서버 메모리 상태를 바꾸기 때문
- 리로드 중에도 탐지 루프는 멈추지 않는다 — 교체는 로드 완료 후 참조 교체 방식이라 부분 적용 상태가 없음
- 개인정보: 요청·응답에 임베딩이나 이름은 오가지 않는다 (인원 수만)

---

### API-013. 오늘의 퇴실(=탐지) 창 조회 (Spring→FastAPI)

- **메서드 / URL**: `GET /api/v1/internal/detection-window`
- **설명**: webapp(FastAPI)이 **오늘 카메라를 돌릴 시간대**를 Spring에게 물어본다. 탐지 창은 스펙상 퇴실 창과 동일한 합집합이고(specs/12), 강의 시간표의 소유자는 attendance라 webapp이 `.env` 고정값으로 자체 계산하지 않는다 — 시간표가 바뀌면 양쪽이 같은 순간에 같은 창을 보게 된다
- **인증**: 내부 (내부망 전제 — 별도 토큰 없음, rules/02)
- **연관 화면**: 없음 (서버 간 동기화). 결과는 API-011의 `window_source`·`window_effective`로 드러난다
- **상태**: ✅ 구현 (2026-08-09)

#### 요청

**Path / Query 파라미터**: 없음 / **Request Body**: 없음

#### 응답

**성공 (200)** — 공통 포맷 `{success, data, error}`

```json
{
  "success": true,
  "data": {
    "date": "2026-08-10",
    "windows": [
      { "start": "13:00", "end": "14:30" },
      { "start": "17:40", "end": "19:10" }
    ]
  },
  "error": null
}
```

| 필드 | 타입 | 설명 |
|---|---|---|
| date | string(YYYY-MM-DD) | 이 창이 적용되는 날짜 (Asia/Seoul) — 소비자가 날짜가 넘어간 캐시를 버리는 데 쓴다 |
| windows | object[] | 강의별 `[T_out−10분, T_out+30분]`. 시작 시각 오름차순 |
| windows[].start / end | string(HH:mm) | **초 단위 없음** — `.env`·webapp 파서 표기와 맞춘다 |

**에러**: 도메인 에러 없음 (조회 실패는 5xx 공통 처리)

#### 비고
- **빈 목록(`windows: []`)은 오류가 아니라 판정이다** — 전 강의 휴강(`classDay=false`)이나 주말이면 그날은 탐지하지 않는다. `.env` 고정값은 이 상태를 표현할 수 없었다
- 겹치는 구간을 병합하지 않고 강의 단위로 그대로 준다 — 소비자 판정이 `any()` 포함 검사라 겹침이 문제되지 않고, 구간이 남아 있어야 어느 강의의 창인지 보인다
- **소비자 동작**: webapp은 주기적으로(기본 300초, `DETECTION_WINDOW_REFRESH_SECONDS`) 이 API를 물어 쓰고, 응답을 못 받으면 `.env`의 `DETECTION_WINDOW`로 후퇴한다. 즉 **`DETECTION_WINDOW`는 폴백값이지 정상 경로가 아니다**
- 주말 가드(`DETECTION_WEEKDAYS_ONLY`)는 2차 방어선으로 남긴다 — Spring이 응답하지 못한 날에도 주말이면 안 돌게 하는 용도

---

### API-014. 탐지 기록 검색 + 사진 서빙

- **메서드 / URL**: `GET /api/v1/detection/records` · `GET /api/v1/detection/records/{record_id}/photo`
- **설명**: "몇 년 몇 월 며칠 몇 시에 누가 잡혔나". 보존기간을 과정 기간으로 늘린(rules/04) 결과를 실제로 쓰는 창구다 — 24시간 TTL이던 시절엔 어제를 물어볼 수 없었다
- **인증**: 검색은 불필요 / **사진은 서명 URL 필수**
- **연관 화면**: (다음주 신설) 검색 화면 · LLM 검색·알리바이 도구 (specs/14 ②)
- **상태**: ✅ 구현 (2026-08-12)

#### 요청 — 검색

| 이름 | 위치 | 타입 | 필수 | 설명 |
|---|---|---|---|---|
| date | query | string(YYYY-MM-DD) | ✅ | Asia/Seoul 기준 날짜 |
| student_id | query | string | | 특정 학생만 |
| camera_id | query | string | | 특정 카메라만 |
| from_time / to_time | query | string(HH:mm) | | 그날의 시각 범위. `to_time`은 **미만**(exclusive) |
| page / size | query | int | | 기본 0 / 20, size 상한 100 |

#### 응답

**성공 (200)** — 공통 포맷 `{success, data, error}`

```json
{
  "success": true,
  "data": {
    "total": 2, "page": 0, "size": 20,
    "items": [{
      "record_id": "…", "student_id": "900000000201", "date": "2026-08-11",
      "camera_id": "cam_12f_1", "detected_at": "2026-08-11T18:05:00+09:00",
      "similarity": 0.87, "event_id": "…",
      "photo_url": "/api/v1/detection/records/…/photo?exp=1786459819&sig=c16a…"
    }]
  },
  "error": null
}
```

**에러**

| HTTP | 에러 코드 | 발생 조건 |
|---|---|---|
| 400 | RECORD_004 | `date`·시각 형식 오류 |
| 403 | RECORD_002 | 사진 서명이 없거나·틀리거나·만료됨 |
| 404 | RECORD_001 | 기록 없음 (만료됐을 수 있다) / 저장 루트 밖 경로 |
| 404 | RECORD_003 | 문서는 있는데 사진 파일이 없음 (정리 루프가 먼저 지운 경우) |

#### 비고
- **결과는 미퇴실 대상만이다.** 사진이 남는 범위가 그것뿐이다(08-11 회의: "경찰 관제 로그식 전량 저장" 폐기). *"그 시각에 있던 전원"이 아니라 "그 시각에 미퇴실 상태로 포착된 사람"*으로 읽어야 한다 — 화면 문구에도 이 구분이 필요하다
- **`photo_path`는 응답에 담지 않는다** — 서버 파일 구조를 드러낼 이유가 없다 (rules/04)
- **사진은 서명 URL로만 나간다.** 이 시스템엔 인증이 없어, record_id만으로 열면 URL을 아는 사람 누구나 학생 얼굴을 본다. 서명은 `HMAC-SHA256(record_id:exp)`이며 **record_id를 포함**하므로 남의 링크로 다른 사진을 열 수 없다. 키는 `PHOTO_URL_SECRET`, 유효 시간은 `PHOTO_URL_TTL_SECONDS`(기본 300초)
  - ⚠ 한계: "링크를 받은 사람이 잠깐 볼 수 있다"까지만 보장한다. 누가 봤는지 추적하지 못한다. 제대로 된 인증이 붙으면 이 층은 그 아래에 그대로 둔다
  - 응답에 `Cache-Control: no-store`를 붙인다 — 만료 뒤에도 브라우저·프록시가 계속 내주면 유효 시간이 무의미해진다
- 시각 범위는 문자열 `detected_at`이 아니라 **`created_at`(BSON date)** 으로 건다. 문자열 비교는 타임존 표기가 한 글자만 달라도 조용히 빗나간다
- 링크는 조회할 때마다 새로 서명한다 — 저장해 두고 돌려쓰면 만료가 의미를 잃는다

---

### API-015. 누적 출결 이력 조회

- **메서드 / URL**: `GET /api/v1/attendance/history`
- **설명**: "그날 누가 출석했고 언제 나갔나". `daily_roster`는 당일 작업 테이블이라(삭제가 곧 판정) 과정 기간 조회의 원천이 될 수 없어, 지워지는 값을 `attendance_history`로 옮겨 두고 여기서 읽는다
- **인증**: 불필요
- **연관 화면**: (다음주 신설) 검색 화면 · LLM 도구
- **상태**: ✅ 구현 (2026-08-12)

#### 요청

| 이름 | 위치 | 타입 | 필수 | 설명 |
|---|---|---|---|---|
| date | query | string(YYYY-MM-DD) | | 생략 시 오늘 (서버가 채운다) |
| student_id | query | string | | 특정 학생만. 공백은 전원으로 본다 |

#### 응답

**성공 (200)**

```json
{
  "success": true,
  "data": {
    "date": "2026-08-11",
    "history": [{
      "student_id": "900000000201", "student_name": "서다은",
      "course_id": "AIG…", "course_name": "심화_AI기반 지능형 솔루션 개발 과정",
      "check_in_time": "2026-08-11T08:39:00+09:00",
      "checked_out_at": "2026-08-11T18:01:00+09:00"
    }]
  },
  "error": null
}
```

**에러**: 도메인 에러 없음

#### 비고
- **비어 있어도 200이다** — "그날은 아무도 퇴실하지 않았다"도 정상 상태다. 과정 기간 밖 날짜는 자연히 빈 목록이 되는데(TTL이 이미 지웠다), 화면 입장에서 "없다"와 "만료됐다"의 처리가 같아 404로 구분하지 않는다
- **연락처가 없는 것은 실수가 아니다** — 이력에 애초에 담지 않는다 (rules/04 저장 최소화). `phone`은 명단과 함께 퇴실 즉시 사라지는 값이다
- `checked_out_at`은 LMS 태그 시각이 아니라 **BAT-001이 명단에서 지운 시점**이다. 폴링 주기(최대 1분)만큼 늦을 수 있다
- ⚠ **쿼리 파라미터 이름을 `@RequestParam(name=…)`으로 명시해야 한다** — 전역 SNAKE_CASE는 Jackson(본문·응답) 전용이라, 생략하면 `student_id`가 조용히 무시되고 **필터가 안 걸린 전원 조회가 200으로 나간다** (2026-08-12 실측으로 발견)
- 정렬은 `checked_out_at` 내림차순. MongoTemplate 정렬 키는 **자바 프로퍼티명**(`checkedOutAt`)으로 준다 — DB 필드명을 직접 적으면 매핑을 타지 않는다

---

### API-016. 등록 완료 학생 목록

- **메서드 / URL**: `GET /api/v1/enroll/registered`
- **설명**: `registered_faces`에 등록된 학생 목록. **화면이 당일 명단(API-010)에서 이 목록을 빼서 "미등록자"를 만든다** — 대조를 서버에서 하지 않는 이유는 위 목록의 비고 참조. `REGISTRY_FILE`이 설정돼 있으면 그 JSON을 읽는다(DB 없는 개발·시연 경로, `source="json"`)
- **인증**: 불필요
- **연관 화면**: SCR-004 (미등록자 목록)
- **상태**: ✅ 완료 (FastAPI `routers/enroll.py`)

#### 응답

**성공 (200)** — 공통 포맷

```json
{
  "success": true,
  "data": {
    "source": "mongo",
    "students": [
      {
        "student_id": "100061869350",
        "student_name": "김OO",
        "embedding_count": 10,
        "course_id": "course_ai_01",
        "updated_at": "2026-08-12T18:20:11+09:00",
        "expire_at": "2026-09-08T00:00:00+09:00"
      }
    ]
  },
  "error": null
}
```

#### 비고
- ⚠ **임베딩은 내려보내지 않는다** — 생체정보에 준한다(rules/04). 화면은 "누가 등록됐나"만 알면 되므로 장수만 준다
- `expire_at`은 그 학생 과정의 종료일 다음날. 일일 스윕이 LMS 기간 변경을 따라 갱신한다

---

### API-017. 가이드 촬영 세션 (시작·프레임·확정·취소)

- **메서드 / URL**: `POST /api/v1/enroll/sessions` · `POST .../{id}/frames` · `POST .../{id}/commit` · `DELETE .../{id}`
- **설명**: 5방향(정면·좌·우·상·하) 가이드 촬영. 브라우저가 프레임을 보내면 서버가 **CLI 도구와 같은 로직**(`app/services/enroll.py`)으로 자세·품질을 판정하고, 통과한 프레임의 임베딩만 세션에 모은다. 확정 시 `registered_faces`에 저장하고 등록부를 리로드한다
- **인증**: 불필요
- **연관 화면**: SCR-004 (촬영 가이드·5방향 진행·등록 완료)
- **상태**: ✅ 완료 (FastAPI `routers/enroll.py`)

#### 요청

**세션 시작** — `consent: true`가 없으면 400 (rules/04 동의자만)

```json
{
  "student_id": "100061869350", "student_name": "홍길동", "consent": true,
  "course_name": "심화_AI기반 지능형 솔루션 개발 과정",
  "append": false, "save_photos": false, "shots": 6, "relax": 1.0
}
```

- `course_name`: 만료를 그 학생 과정에 맞추기 위한 값 (rules/04 — 없으면 저녁 등록에서 강의를 못 찾는다)
- `append`: **true면 기존 등록에 더한다**(추가 촬영). false(기본)면 덮어쓴다. 인당 상한 60개이고, 넘치면 기존을 등간격으로 솎아 옛 세션의 자리를 남긴다
- `save_photos`: **true면 임베딩과 함께 사진(크롭)도 보관**한다(기본 false). 저장 위치·구조는 CLI(`enroll_capture.py`)와 같아 `register_faces.py`가 그대로 읽는다 — 아래 비고 참조
- `shots`: 방향당 장수. 화면은 6(= 인당 30개)을 보낸다

**프레임** — `image`는 JPEG base64(data URL 접두사 허용). `manual: true`는 화면의 [지금 촬영](=CLI의 SPACE)

```json
{ "image": "data:image/jpeg;base64,...", "manual": false }
```

#### 응답

**프레임 판정 (200)** — 화면은 이 값으로 안내 문구·가이드 색·진행률을 갱신한다

```json
{
  "success": true,
  "data": {
    "step_key": "left", "step_label": "왼쪽", "step_index": 1, "total_steps": 5,
    "captured": 1, "shots": 3,
    "satisfied": false, "quality_ok": true,
    "message": "고개를 왼쪽으로 돌려주세요",
    "took_shot": false, "completed": false, "total_captured": 4,
    "base_pitch": 0.42,
    "pose": { "yaw": 0.05, "pitch": 0.44 },
    "face_box": [0.36, 0.21, 0.27, 0.48],
    "steps": [{ "key": "front", "label": "정면", "hint": "…", "state": "done", "captured": 3 }]
  },
  "error": null
}
```

**등록 확정 (200)**

```json
{
  "success": true,
  "data": {
    "student_id": "100061869350", "student_name": "홍길동",
    "embedding_count": 60, "added_count": 30, "sessions": 2,
    "course_id": "course_ai_01", "expire_at": "2026-09-08T00:00:00+09:00",
    "registered_count": 24, "registry_reloaded": true
  },
  "error": null
}
```

**에러**

| HTTP | 에러 코드 | 발생 조건 |
|---|---|---|
| 404 | `ENROLL_001` | 세션 없음 — 만료(TTL 15분)됐거나 이미 확정·취소됨 |
| 409 | `ENROLL_002` | 5방향을 다 찍지 않고 확정 시도 |
| 422 | `ENROLL_003` | 유효 임베딩 2장 미만 (register_faces.py와 같은 하한) |
| 400 | `ENROLL_004` | 프레임 디코드 실패 |
| 400 | `ENROLL_005` | 동의 확인 없이 세션 시작 |
| 500 | `ENROLL_006` | 등록 저장 실패 — **세션은 유지된다**(촬영본을 잃지 않고 재시도) |

#### 비고
- **`base_pitch`는 정면 단계가 끝나면 바뀐다** — 그 사람의 중립 자세를 정면에서 실측해 상/하 기준으로 쓴다(rules/05). 고정값은 CCTV 크롭 값이라 눈높이 웹캠에서 상/하가 통과되지 않는다
- **단계가 넘어간 프레임의 응답은 이미 다음 단계를 가리킨다** — 방금 끝난 단계를 그대로 주면 화면이 한 박자 늦고 사용자가 이미 채운 방향을 계속 유지한다
- **사진은 기본적으로 저장하지 않는다** — 임베딩만 남는다(SCR-004·rules/04)
- **`save_photos`를 켜면 사진도 남는다** (2026-08-13 추가). 켜는 이유는 하나다: **임베딩은 모델에 종속**이라 앙상블로 바뀌면 전부 무효가 되는데, 사진이 없으면 **학생을 다시 불러 5방향을 재촬영**해야 한다. 사진이 있으면 `register_faces.py <ENROLL_PHOTO_DIR>` 한 줄로 임베딩만 다시 뽑는다
  - 저장 경로: `ENROLL_PHOTO_DIR/<이름>=<학번>/{방향}_{n}.jpg` — 폴더명 규약은 `register_faces.py`의 `partition("=")`에 맞춘 것
  - 저장되는 것은 **대상자 크롭(1.8배)**뿐이다 — 프레임 전체를 남기면 배경의 비동의자 얼굴이 함께 저장된다(2026-08-12 실기기에서 실제 발생)
  - `append=true`면 번호를 이어서 쌓고, 재촬영이면 폴더를 비우고 새로 쓴다 (임베딩 병합 규칙과 같은 원칙)
  - **사진 보관 실패는 등록을 되돌리지 않는다** — 임베딩은 이미 DB에 있다. 응답 `photos_saved: 0`으로 알린다
  - ⚠ 사진은 임베딩보다 개인정보 무게가 크다. 기본 꺼짐이고, 켠 세션만 남으며, 보존기간·파기 규칙은 CLI 경로와 같다(동의자만·비커밋·과정 종료 시 파기)
- 프레임 처리는 CPU 블로킹이라 **스레드로 넘기고 세션 단위로 직렬화**한다 — 같은 프로세스가 재송출하는 MJPEG(API-002)이 끊기지 않게 하기 위함
- 확정은 저장 + 등록부 리로드(API-012)를 **한 번에** 한다. 리로드만 실패하면 `registry_reloaded=false`로 알리고 등록 자체는 되돌리지 않는다
- ⚠ 세션은 프로세스 메모리에 있다 — **uvicorn 워커를 늘리면 고정 세션·공유 저장소가 필요하다**(현재 단일 워커)
- ⚠ 브라우저 카메라(`getUserMedia`)는 `https`/`localhost`에서만 열린다. 서버 IP로 접속하면 이 API가 멀쩡해도 촬영이 불가능하다 (rules/06)
- **추가 촬영은 모델이 같을 때만 잇는다** — 임베딩 공간이 다르면 유사도가 무의미해진다. 모델이 다르면(`ensemble` 메타 대조) 이어붙이지 않고 새로 시작한다(WARN 로그). 두 필드는 **같은 인덱스로 함께** 솎이므로 짝이 유지된다 — 따로 솎으면 장수는 맞는데 다른 장의 두 얼굴이 한 장으로 묶인다
- **날짜는 KST로 내려간다** — Mongo는 UTC로 보관하므로 그대로 직렬화하면 화면에 **하루 빠른 만료일**이 뜬다(만료는 KST 자정 = UTC 전날 15:00). 행정실이 파기일을 하루 당겨 읽으면 멀쩡한 등록을 다시 찍게 된다 (2026-08-12 발견)

---

### API-018. 등록 얼굴 삭제

- **메서드 / URL**: `DELETE /api/v1/enroll/registered/{student_id}`
- **설명**: `registered_faces`에서 그 학생의 임베딩을 지우고 탐지 등록부를 리로드한다.
- **인증**: 불필요
- **연관 화면**: SCR-004 (등록됨 탭 → [삭제])
- **상태**: ✅ 완료 (FastAPI `routers/enroll.py`)

#### 응답

**성공 (200)**

```json
{
  "success": true,
  "data": {
    "student_id": "100061869350", "deleted": true,
    "registered_count": 23, "registry_reloaded": true
  },
  "error": null
}
```

**에러**

| HTTP | 에러 코드 | 발생 조건 |
|---|---|---|
| 404 | `ENROLL_007` | 등록된 얼굴이 없음 — 이미 지워졌는데 200을 주면 화면은 "지웠다"고 표시하고 실제로는 아무 일도 없다 |

#### 비고
- ⚠ **재촬영에는 이 API가 필요 없다** — 같은 학번으로 다시 등록하면 upsert로 덮인다. 삭제는 **동의 철회**(개인정보는 요구 즉시 파기해야 하며 period를 기다리지 않는다 — rules/04)·**오등록**(그 학번에 남의 얼굴이 붙어 있으면 오식별이 아니라 오탐지다)·**중도 퇴원**을 위한 것이다
- **임베딩과 보관 사진을 함께 지운다** (2026-08-13 수정). 사진 보관 옵션이 08-12에 들어왔는데 삭제는 임베딩만 지우고 있었다 — **동의 철회를 이행해도 얼굴 사진이 디스크에 남는** 상태였고, 조회에서 안 잡히니 고쳐진 것처럼 보이는 게 특히 나빴다. 사진은 학번으로 찾는다(`<이름>=<학번>` 폴더 — 개명·오타로 이름이 달라져도 학번이 같으면 같은 사람)
- **되돌릴 수 없다** — 복구하려면 5방향을 다시 촬영해야 한다. 화면이 확인 창을 한 번 띄운다
- 삭제도 등록과 같이 **등록부 리로드까지** 한다 — "지웠다"가 아니라 "다음 탐지부터 이 사람이 식별되지 않는다"까지 가야 끝난 것이다

---

### API-019. 엣지 탐지 결과 수신 (Jetson→FastAPI)

- **메서드 / URL**: `POST /api/v1/internal/detections/faces`
- **설명**: Jetson Nano가 YuNet으로 탐지한 **얼굴 확장 크롭 + 랜드마크**를 서버로 올린다. 서버는 받은 크롭으로 정렬·임베딩·매칭(앙상블)을 하고, 같은 크롭을 `detection_records` 사진으로 저장한다. **원본 프레임은 올라오지 않는다.**
- **인증**: 내부 (내부망 전제 — 별도 토큰 없음, rules/02)
- **연관 화면**: SCR-001 이벤트 피드 (식별 결과가 API-009로 이어짐)
- **상태**: 📝 설계 — 구현은 ② 카메라·영상 담당(팀원 B) 일정에 맞춘다. **그전까지는 서버가 스트림에서 직접 탐지하는 현행 경로를 유지한다**

#### 요청

**Request Body**

```json
{
  "camera_id": "cam_12f_1",
  "captured_at": "2026-08-13T18:42:11+09:00",
  "faces": [
    {
      "crop": "<base64 JPEG>",
      "landmarks": [[62.1, 70.4], [98.3, 69.8], [80.2, 91.0], [66.5, 112.7], [95.1, 112.2]],
      "bbox_in_frame": [412, 180, 96, 96],
      "det_score": 0.97
    }
  ]
}
```

| 필드 | 타입 | 필수 | 설명 |
|---|---|---|---|
| camera_id | string | YES | `cam_12f_1` / `cam_12f_2` / `cam_1f` — `.env` `CAMERA_SOURCES` 키와 같은 체계 |
| captured_at | string(ISO) | YES | **Jetson이 찍는 촬영 시각**(KST 오프셋 포함). 서버 도착 시각이 아니다 — 알리바이 기록의 기준 시각이므로 NTP 동기화가 전제 |
| faces[].crop | string(base64) | YES | **bbox 2.5배 확장 크롭** JPEG. 배율은 서버 `RECORD_CROP_SCALE`과 같은 값이며 API-019b로 내려준다 |
| faces[].landmarks | number[5][2] | YES | YuNet 5점(우눈·좌눈·코·우입·좌입), **크롭 좌표계**. 정렬은 서버가 한다 |
| faces[].bbox_in_frame | int[4] | YES | 원본 프레임 기준 [x,y,w,h] — 화면 내 위치 로그·중복 판단용 |
| faces[].det_score | number | YES | YuNet 신뢰도 |

#### 응답

**성공 (200)** — 공통 포맷. `data`: `{ "accepted": 2, "skipped": 1 }` (쿨다운·창 밖 등으로 버린 수)

**에러**

| HTTP | 에러 코드 | 발생 조건 |
|---|---|---|
| 400 | `EDGE_001` | 크롭 디코드 실패·랜드마크 형식 오류 |
| 413 | `EDGE_002` | 페이로드 초과 (프레임 전체를 보낸 경우 — 크롭만 허용) |
| 503 | `EDGE_003` | 탐지 창 밖이라 수신하지 않음 — 엣지는 API-019b를 다시 읽고 촬영을 멈춘다 |

#### 비고
- **왜 "확장 크롭 + 랜드마크"인가** (2026-08-12 설계):
  - ① **이미 우리가 저장하는 형식과 같다** — `detection_records` 사진이 bbox 2.5배 확장 크롭이다(rules/04). 받은 걸 그대로 저장하고 그대로 식별에 쓴다. 서버가 다시 크롭을 뜨지 않는다
  - ② **원본 프레임이 네트워크·서버를 거치지 않는다** — 비동의자가 찍힌 프레임이 Jetson에서 끝난다. "확장 크롭만 남긴다"는 규칙을 저장 단계가 아니라 **수집 단계에서** 지키게 된다
  - ③ **정렬을 서버가 소유한다** — "정렬된 112×112"를 보내면 정렬 방식이 바뀔 때(MediaPipe 468점·TP-GAN 정면화 예정) Jetson 3대를 재배포해야 한다. 크롭+랜드마크면 서버 배포 한 번으로 끝난다
- **임베딩까지 엣지에서 만들지 않는다** — Nano는 앙상블을 지지 못한다 (rules/05 역할 분리)
- **라이브 뷰(API-002)는 별도 경로로 남는다** — 엣지 탐지로 옮겨도 SCR-001 화면은 영상이 필요하다. 다만 상시 당기지 말고 **관제 화면을 열었을 때만** 스트림을 여는 쪽으로 바꾸면 평소 대역폭이 0에 수렴한다
- **중복 억제는 서버가 계속 소유한다** — 쿨다운(같은 camera_id+student_id 60초)은 식별 이후에만 판단할 수 있다. 엣지는 "같은 자리의 연속 프레임"만 가볍게 줄이고, 최종 판단은 서버가 한다
- 수신 후 흐름은 기존과 같다: 식별 → `detection_events` 저장 → API-009로 Spring 발행 → 미퇴실 대상이면 `detection_records`에 크롭 저장
- ✅ **선행 리팩터링 완료 (2026-08-13)**: `FaceEngine`이 `detect()`(위치·랜드마크만) / `embed()`(후보 하나 → 벡터)로 나뉘었다. `detect()`가 돌려주는 **`FaceCandidate`가 이 API의 페이로드와 같은 모양**(bbox·landmarks·det_score)이라, 엣지 전환은 **"그 객체를 로컬 탐지로 만드느냐 HTTP로 받느냐"**의 차이만 남는다. 식별 이후 코드는 그대로다
  - 같은 리팩터링이 앙상블 배선에도 필요했다(`identification.EnsembleIdentifier`가 프레임+랜드마크를 요구) — **두 작업이 하나였고, 한 번에 해결됐다**
  - 부수 효과: 등록 화면이 랜드마크 때문에 탐지기를 따로 열어 **같은 프레임을 두 번 탐지하던 것**이 사라졌다. 자세를 판정한 얼굴과 임베딩을 뽑는 얼굴이 같은 객체임도 보장된다

---

### API-019b. 엣지 촬영 설정 조회 (Jetson→FastAPI)

- **메서드 / URL**: `GET /api/v1/internal/edge/config`
- **설명**: Jetson이 주기적으로 물어보는 촬영 설정. **지금 찍어야 하는지(탐지 창)**와 촬영 파라미터를 서버가 중앙에서 내려준다.
- **인증**: 내부
- **상태**: 📝 설계

#### 응답

```json
{
  "success": true,
  "data": {
    "capture": true,
    "window": "13:00-15:00,16:30-19:20,21:30-23:00",
    "interval_seconds": 1.0,
    "crop_scale": 2.5,
    "jpeg_quality": 85,
    "poll_seconds": 60
  },
  "error": null
}
```

#### 비고
- **왜 푸시가 아니라 폴링인가** — 엣지 장비에 인바운드 포트를 열지 않아도 되고, 재부팅·네트워크 단절 후 자동으로 제자리를 찾는다. 3대 규모에 브로커(MQTT)를 얹을 이유가 없다
- **탐지 창의 주인은 여전히 Spring**이다 — FastAPI가 API-013으로 받아 둔 창을 그대로 내려준다. Jetson이 Spring을 직접 부르지 않는 이유는 시간표 판단이 여러 곳으로 복제되면 안 되기 때문이다 (`SpringSyncedWindow` 주석과 같은 원칙)
- **`crop_scale`·`jpeg_quality`를 서버가 내려주는 이유**: 크롭 배율이 바뀌면(증빙 품질 조정) Jetson 3대를 다시 배포하지 않고 서버 설정만 고치면 된다
- ⚠ **Jetson NTP 동기화가 전제** — `captured_at`이 틀어지면 알리바이 기록 전체가 어긋난다. 팀원 B 셋업 체크리스트 항목

---

### API-020. 넛지 동적 메시지 본문 생성 (Spring→FastAPI)

- **메서드 / URL**: `POST /api/v1/internal/nudge-message`
- **설명**: 미퇴실 복귀 요청 문자의 본문을 만든다. 골격(기관명·요청 행동)은 Spring이 고정으로
  들고 있고, 이 API는 그 사이에 들어갈 어투 변주 한 구절만 생성·검증해서 돌려준다
  (specs/14 LLM-004, 이슈 #225). 검증 실패·LLM 미기동·타임아웃은 전부 `variant_used: false`로
  응답하며, 이 경우 호출측(Spring)은 이 API가 준 문구를 쓰지 않고 자체 고정 템플릿으로 되돌아간다
  — 폴백 문구의 출처를 Spring 한 곳으로 묶어 두기 위함
- **인증**: 내부 (내부망 전제 — 별도 토큰 없음, rules/02). `frontend/nginx.conf.template`이 `/api/v1/internal/`을
  외부 화이트리스트에 포함하지 않아 바깥에서 닿지 않는다
- **연관 화면**: 없음 (서버 간 호출, RPA 발송 직전)
- **상태**: ✅ 구현 (2026-08-25, `feat/nudge-message` 계열 브랜치)

#### 요청

**Request Body**

```json
{ "test_mode": true, "resend": false }
```

| 필드 | 타입 | 필수 | 설명 |
|---|---|---|---|
| test_mode | boolean | YES | `SMS_MODE=test` 여부. webapp이 바이트 예산에서 Spring이 붙일 `[TEST] ` 접두어를 미리 뺀다 |
| resend | boolean | YES | 오늘 이미 확정 발송(SENT)된 문자가 있는가 — 어투(1차/재안내)만 가른다 |

#### 응답

**성공(200)** — 공통 포맷 `{success, data, error}`. HTTP 상태는 항상 200이다 — 이 API는
"실패"를 모른다(routers/nudge.py 주석). 발송 자체를 막지 않기 위해서다

```json
{
  "success": true,
  "data": {
    "message": "우리인재개발원입니다. 퇴실 체크가 아직 확인되지 않았습니다. 퇴실 부탁드립니다.",
    "variant_used": true,
    "reason": null
  },
  "error": null
}
```

| 필드 | 타입 | 설명 |
|---|---|---|
| message | string | 최종 본문. `variant_used=false`여도 채워지지만 **Spring은 이 값을 쓰지 않는다** |
| variant_used | boolean | true면 LLM 변주가 검증을 통과해 반영됨. false면 무언가 걸려 고정 템플릿으로 갔다는 뜻 |
| reason | string\|null | 폴백 사유(`llm_disabled`·`llm_unavailable`·`validation_failed`·`internal_error`) — 로그·운영 확인용, Spring 로직은 이 값을 안 본다 |

**에러**: 도메인 에러 없음 — 이 API 자체가 이미 "실패=폴백"으로 흡수한다

#### 비고
- **왜 골격을 여기서 안 만드는가** — 기관명·요청 행동 문구는 학생에게 나가는 문자의 절대
  사실이라 LLM이 손대면 안 된다(#225 설계 원칙 "골격 고정 + 변주 + 폴백"). Spring이 조립한다
- **요금 제약이 이 API의 핵심 계약이다** — 단문(SMS) 한도 90바이트(EUC-KR, 한글 45자)를
  넘으면 장문(LMS)으로 전환돼 건당 18원→45원(2.5배)이 된다. `test_mode`를 안 넘기면 검증을
  통과한 문구가 Spring의 `[TEST] ` 접두어 때문에 뒤늦게 장문이 될 수 있다
- **Spring이 최종 관문을 한 번 더 둔다** — 이 API의 바이트 검증과 별개로,
  `NudgeMessageClient.fitsInSingleSms()`가 `[TEST] ` 접두어까지 붙인 실제 발송 문자열을
  다시 검사한다. webapp과 Spring의 버전이 어긋나도 요금 사고로 새지 않는 이중 안전장치
- **호출측 타임아웃(연결 2초·읽기 6초)과 실패 쿨다운(30초)은 Spring 쪽 관심사** —
  BAT-004가 미퇴실 후보를 순차 처리하므로, 이 API가 느려지면(다운이 아니라 지연) 한
  tick의 처리 시간이 후보 수만큼 늘어날 수 있어 Spring이 방어한다 (`NudgeMessageClient` 주석)

---

### 템플릿 (새 API 추가 시 복사)

### API-XXX. (API 이름)

- **메서드 / URL**: `GET /api/v1/...`
- **설명**:
- **인증**: 필요 / 불필요
- **연관 화면**: (11_SCREEN_SPEC.md의 화면 ID)
- **상태**: 📝 설계

#### 요청

**Path / Query 파라미터**

| 이름 | 위치 | 타입 | 필수 | 설명 |
|---|---|---|---|---|
|  | path / query |  |  |  |

**Request Body** (없으면 "없음")

```json
{
}
```

#### 응답

**성공 (200)**

```json
{
}
```

**에러**

| HTTP | 에러 코드 | 발생 조건 |
|---|---|---|
| 400 |  |  |
| 404 |  |  |

#### 비고
- (페이징, 정렬, 캐시, 멱등성 등 특이사항)
