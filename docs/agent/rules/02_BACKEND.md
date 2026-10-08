
# 02. 백엔드 설계 규칙

> 문서 상태: ✅ 완료 (2026-08-04)
> 백엔드는 투트랙 — FastAPI(AI·영상)와 Spring Boot(업무). 역할 경계는 `01_TECH_STACK.md` 참고.

## 1. 아키텍처

### 아키텍처 스타일
- 상태: ✅ 결정
- 선택지:
  - **레이어드** — 단순, 혼자·소규모에 적합
  - **헥사고날(포트&어댑터)** — 협업·테스트 용이, 초기 비용 있음
  - **모듈러 모놀리스** — 도메인별 모듈 분리, 확장 대비
  - **멀티모듈** — 협업 시 모듈 단위 분담 가능
- 결정: **Spring = 도메인별 패키지 + 도메인 내부 헥사고날(포트&어댑터)** / **FastAPI = FastAPI 표준 구조**(routers·schemas·services — 2026-08-05 강사 지침으로 변경, 99_DECISIONS 기록)
- 이유: Spring은 도메인 단위로 팀원 분담 가능(5명 협업)·헥사고날로 테스트 용이성 확보(사용자 제안 구조). FastAPI는 강사 지침에 따라 프레임워크 표준 관례 준수
- 결정일: 2026-08-04 (FastAPI 부분 2026-08-05 변경)

### 패키지 / 모듈 구조
- 상태: ✅ 결정
- 결정: 도메인별 폴더 → 각 도메인 내부에 domain/application/adapter 계층
```
[Spring Boot — 업무] (루트 패키지는 임시 com.team.project3 — 서비스명 확정 시 변경)
com/team/project3/
├── attendance/            # 출결 판정·예외처리 도메인 (LMS 태그=기준, 카메라=보조 / 문자는 "퇴실 미체크 AND 퇴실 시간대 영상 식별 포착" 시에만 — 2026-08-05 CTO 플로우 확정, 99 참조)
│   ├── domain/            # 엔티티·도메인 규칙·exception/(커스텀 예외 + ErrorCode enum) — 프레임워크 무관
│   ├── application/       # port/in · port/out · service (유스케이스)
│   └── adapter/           # in/web (REST 컨트롤러) · out/persistence (Spring Data MongoDB)
├── detection/             # 탐지 이벤트 수신·저장 도메인 (동일 계층 구조)
├── notification/          # Slack 알림 + LMS 웹발신 API 문자 도메인 (동일 계층 구조)
├── common/                # 공통 응답 포맷·ErrorCode 공통 인터페이스·유틸
└── global/                # 설정·Swagger·전역 예외 핸들러(ErrorCode → 응답 포맷 변환기 역할만)

[FastAPI — AI 서버 (webapp/)] ※ 2026-08-05 강사 지침: 헥사고날이 아닌 FastAPI 표준 구조 (99_DECISIONS 기록)
app/
├── main.py                # 앱 생성·라우터 등록
├── config.py              # 설정 (리포 루트 .env 로드)
├── database.py            # MongoDB(Motor) 연결
├── routers/               # 엔드포인트 (health·detections 등)
├── schemas/               # Pydantic 모델 (공통 응답·탐지 이벤트)
└── services/              # 비즈니스 로직 (video 수집·detection 추론)
```
- 결정일: 2026-08-04

### 공유 코드 수정 정책 (common·global)
- 상태: ✅ 결정
- 결정: **`common/`·`global/`은 수정 금지, 사용만** — 도메인 개발(attendance·detection·notification)은 이 폴더들을 import해서 쓰되 직접 변경하지 않는다. 변경이 필요하면 **CTO에게 요청 → 별도 PR + CTO 리뷰**로만 반영
- 이유: 두 폴더는 전 도메인이 공유하는 계약(응답 포맷·ErrorCode 인터페이스·전역 예외 변환기)이라, 한 도메인이 임의 수정하면 다른 도메인 전체가 흔들린다. 도메인별 분담 개발(2026-08-07~)에서 응답 일관성을 지키는 최소 장치
- 도메인 개발자가 할 일: ① 자기 도메인 `domain/exception/`에 `ErrorCode` 구현 enum 정의 ② `BusinessException`에 담아 던지기 ③ 정상 응답은 `ApiResponse.ok()` — 이 3가지면 포맷 통일이 자동으로 보장된다
- 결정일: 2026-08-06

### 서버 간 통신 (FastAPI → Spring Boot)
- 상태: ✅ 결정
- 선택지: REST 내부 API / Redis Pub·Sub / RabbitMQ·Kafka
- 결정: **REST 내부 API** — FastAPI가 탐지 이벤트를 Spring 내부 API로 HTTP POST
- 이유: 가장 단순하고 디버깅 쉬움. CCTV 3대·이벤트 빈도 수준이면 충분. 실패 시 재시도 로직만 추가. (이벤트 규모가 커지면 Redis 도입 재검토 — 99_DECISIONS에 기록)
- 결정일: 2026-08-04

## 2. API 설계 규칙

### API 스타일
- 상태: ✅ 결정
- 선택지: REST / GraphQL / gRPC / 혼합
- 결정: REST (학원 체크리스트 RestAPI CRUD 요건 + Swagger 문서화)
- 이유: 표준 조합, 팀 익숙도
- 결정일: 2026-08-04

### URL / 네이밍 규칙
- 상태: ✅ 결정
- 결정: `/api/v1/{복수형 리소스}` — 케밥케이스, v1 버저닝. 내부 이벤트 수신은 `/api/v1/internal/...` 프리픽스
- 결정일: 2026-08-04

### 공통 응답 포맷
- 상태: ✅ 결정
- 결정:
```json
{
  "success": true,
  "data": {},
  "error": null
}
```
- 실패 시 `success: false`, `error: { "code": "...", "message": "..." }`, `data: null`
- 결정일: 2026-08-04

### 에러 응답 규약
- 상태: ✅ 결정
- 결정: 커스텀 에러코드 체계 + HTTP 상태코드 매핑 (예: 400=검증 실패, 404=리소스 없음, 500=서버 오류). 에러코드는 **각 도메인의 `ErrorCode` enum이 소유**(예: `ATTENDANCE_001`), common/에는 공통 인터페이스만 둔다(2026-08-05 예외 처리 전략 참조). 세부 목록은 개발 시작 시 AI가 초안 제안 → 확인 후 확정
- 결정일: 2026-08-04 (2026-08-05 관리 위치 구체화)

## 3. 인증 / 인가

### 인증 방식
- 상태: ✅ 결정
- 선택지: 없음(개인 도구) / 세션 / JWT / OAuth2 소셜 로그인 / API Key
- 결정: **인증 없음** — 내부 데모용으로 로그인 생략
- 이유: 4주 일정에서 개발 부담 최소화 (2026-08-04 인터뷰)
- ⚠️ **주의 기록**: 출결 데이터는 개인정보 성격이 있음. 학원 내부망 데모 전제로만 유효 — 외부 공개·확장 시 인증 도입 재논의 필수(99_DECISIONS에 기록할 것)
- 결정일: 2026-08-04

### 인가(권한) 모델
- 상태: ➖ 해당 없음
- 결정: 인증 없음에 따라 인가 모델도 생략
- 결정일: 2026-08-04

## 4. 공통 처리 규칙

### 예외 처리 전략
- 상태: ✅ 결정 (2026-08-05 구조 구체화 — CTO 논의 반영)
- 결정: **소유권 분리 원칙** — "무엇이 에러인가"는 도메인이 결정하고, "어떻게 응답하는가"만 global이 담당
  - **각 도메인**: 커스텀 예외 클래스 + `ErrorCode` enum(에러코드·메시지·HTTP 상태)을 `{도메인}/domain/exception/`에 정의 — 도메인이 소유
  - **common/**: `ErrorCode` 공통 인터페이스만 정의 (각 도메인 enum이 구현)
  - **global/**: `GlobalExceptionHandler`(`@RestControllerAdvice`)는 `ErrorCode` 인터페이스에만 의존해 공통 응답 포맷 `{success,data,error}`로 **기계적 변환만** 수행. 검증 실패(Bean Validation)·미분류 예외(500) 등 프레임워크 공통 예외의 폴백도 담당
  - FastAPI도 동일 원칙: 예외·에러코드 정의는 services/(비즈니스 로직) 쪽에, 변환은 전역 exception handler(main.py 등록)로 일원화
- 이유: 핸들러를 도메인마다 두면 응답 포맷 변환이 복제되어 일관성이 깨지고, 설정·Swagger·변환기는 프레임워크 횡단 관심사라 최외곽(global/infrastructure)에 두는 것이 헥사고날 원칙에 부합. 도메인별 예외 관리 요구는 ErrorCode 소유권을 도메인에 두는 것으로 충족 (2026-08-05 CTO 논의)
- 결정일: 2026-08-05

### 로깅 규칙
- 상태: ✅ 결정
- 결정: 로그 레벨 — 운영 INFO, 개발 DEBUG. **개인정보(이름·연락처·출결 기록) 마스킹 필수**. 탐지 이벤트는 이벤트 ID·타임스탬프·카메라 ID 위주로 기록
- 결정일: 2026-08-04

### 환경설정 / 시크릿 관리
- 상태: ✅ 결정 (2026-08-08 변경 — 99_DECISIONS 참고)
- 선택지: .env + gitignore / 프로파일 분리(application-dev.yml 등) / 환경변수
- 결정: **환경변수는 리포 루트 `.env` 한 곳에서만 관리한다.** Slack 토큰·문자 API 키·DB 비밀번호는 절대 커밋 금지, `.env.example`로 키 목록만 공유
  - **FastAPI**: `app/config.py`가 리포 루트 `.env`를 직접 지정해 읽는다
  - **Spring**: `.env`를 자동으로 읽지 않으므로 `application.yml`에 `spring.config.import: optional:file:../.env[.properties]`로 명시 임포트한다. `optional:`이라 파일이 없어도 기동하고, working directory가 `backend/`인 IDE·Gradle 실행이 기준이다. 프로파일 분리(`application-dev.yml`)는 시크릿이 아닌 설정에만 쓴다
  - **Docker Compose**: compose가 루트 `.env`를 읽어 컨테이너에 환경변수로 주입한다. **실제 환경변수가 `.env` 임포트보다 우선**하므로 컨테이너에서는 compose 주입값이 이긴다
  - **셸 스크립트**(`database/apply.sh`·`seed.sh`): 루트 `.env`를 `source`한다
- 이유: 같은 값을 여러 파일에 두면 한쪽만 고쳐 놓고 원인을 못 찾는 사고가 난다 — DB 비밀번호처럼 여러 구성요소가 함께 쓰는 값이 특히 그렇다. (이전: `database/.env`를 따로 두고 Spring은 IDE Run Config에 환경변수 주입)
- 결정일: 2026-08-08 (최초 2026-08-04)

### 유효성 검증 위치
- 상태: ✅ 결정
- 선택지: 컨트롤러(DTO) / 도메인 / 양쪽
- 결정: 컨트롤러 DTO 계층 (Spring: Bean Validation, FastAPI: Pydantic 스키마)
- 결정일: 2026-08-04
