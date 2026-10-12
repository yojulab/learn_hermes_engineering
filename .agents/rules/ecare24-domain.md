# eCare24 이러닝 플랫폼 도메인 엔지니어링 가이드

- **문서 참조**: `TASK_SPECIFICATION_ECARE24.md` (문서 번호: `ECARE24-SOW-2026-001 Rev. 1.2`)
- **프로젝트 명칭**: ecare24 환자 질환별 전문간병사 자격과정 웹·앱 통합 플랫폼
- **프로젝트 핵심 원칙**:
  > **※ 본 단계는 기획 및 상세 아키텍처 수립 단계입니다.**
  > **본 단계에서는 실제 소스코드 개발을 진행하지 않으며**, 개발팀, 기획팀, 인프라팀이 상호 합의하고 즉각 스프린트에 착수할 수 있도록 구체적인 스펙, 인터페이스 정의, 스키마, 작업 분할 구조(WBS) 및 검증 아키텍처를 정의합니다.
  > **모든 유의미한 업무 단위 완료 시에는 반드시 Git Commit을 수행합니다.**

---

## 1. 4대 타겟 멀티 프로젝트 디렉토리 구조 및 기술 스택

플랫폼은 역할과 대상 사용자에 따라 루트 기준 4개 개별 프로젝트 및 테스트 슈트로 엄격히 분리됩니다:

```text
ecare24/
├── .env.example                           # 루트 공통 환경 변수 템플릿
├── .gitignore                             # 각 폴더별 .env 파일 커밋 방지
├── .vscode/                               # 4대 타겟 통합 디버그 프로파일 (launch.json)
├── frontend_user/                         # [Web-User] 수강생 학습 포털 (Port 3000)
├── frontend_admin/                        # [Web-Admin] 통합 관리자 & 콜센터 CS 포털 (Port 3001)
├── backend/                               # [API] NestJS / TypeScript 엔터프라이즈 백엔드 (Port 4000)
├── mobileApp/                             # [Mobile] React Native 기반 스마트러닝 모바일 앱
├── tests/                                 # [E2E & Automation] Playwright Headless 캡처 슈트
└── docs/                                  # TASK_SPECIFICATION_ECARE24.md 등 프로젝트 산출물
```

### 1-1. 프로젝트별 기술 스택 규격

| 타겟 구분 | 권장 기술 스택 | 핵심 역할 및 아키텍처 규격 |
| :--- | :--- | :--- |
| **frontend_user** | React, TypeScript, Next.js / Vite, Tailwind CSS, TanStack Query | **수강생 학습 포털**. 50~60대 중장년 대상 **실버 친화적 초직관 UX (WCAG AAA)**, 고대비 모드, 글자크기 조절기, 4단계 원클릭 패스, 종이 감성 OMR CBT. |
| **frontend_admin** | React, TypeScript, Vite / Next.js, Ant Design / Shadcn UI | **통합 관리자 및 콜센터 CS 포털**. **3분할 원스크린 레이아웃**(고객정보 30%, 수강진도 40%, 상담메모 30%), CTI 연동 팝업 프리, 키보드 단축키(Alt+1~3, Alt+S). |
| **backend** | Node.js, NestJS, TypeScript, Clean Architecture | **엔터프라이즈 REST API**. 8대 핵심 도메인 모듈(auth, users, diseases, courses, exams, certificates, media-tv24, counseling, payments), PostgreSQL 16+ 트랜잭션. |
| **mobileApp** | React Native, TypeScript (Expo / RN CLI) | **스마트러닝 모바일 앱 (iOS/Android)**. 48dp 터치 타깃, 오프라인 교안 캐싱, FCM 푸시 알림, 세로형 숏폼 스와이프 플레이어, 모바일 자격증 지갑. |
| **tests** | Playwright (Headless Mode), TypeScript | **자동화 검증 & 화면 스냅샷 캡처 슈트**. 16개 질환군 상세, 실버 UX 폰트 조절, OMR CBT, 자격증, 콜센터 3분할 화면 무인 캡처 및 직능원 심사용 증빙 추출. |

---

## 2. 16개 질환군 분류 체계 및 4단계 심화 학습 편성

### 2-1. 16개 표준 질환군
1. **신경계 질환군** (뇌졸중/중풍, 파킨슨병, 뇌손상, 척수손상 등)
2. **호흡기계 질환군** (COPD, 만성기관지염, 기관절개관(T-tube) 관리, 산소요법)
3. **심혈관계 질환군** (심부전, 협심증, 심근경색 후유증, 혈관질환 케어)
4. **근골격계 및 재활 질환군** (인공관절 치환술 후 간병, 고관절 골절, 척추재활)
5. **소화기·간담도 질환군** (경관유동식, 위루관(PEG) 관리, 간경변, 인공항문 케어)
6. **비뇨기·신장 질환군** (만성신부전, 혈액/복막투석, 유치도뇨관(Foley) 관리)
7. **내분비·대사 질환군** (인슐린 의존성 당뇨, 당뇨발 예방/소독 케어)
8. **종양·암 질환군** (항암화학요법 부작용 케어, 방사선치료 간병, 면역관리)
9. **인지·정신·치매 질환군** (알츠하이머, 혈관성 치매, 섬망, BPSD 행동심리 대처)
10. **감염성 질환군** (CRE/VRE 격리환자 간병, 결핵, 옴/욕창 감염병 예방격리)
11. **중환자·집중케어 질환군** (ICU 퇴원환자 간병, 가정용 인공호흡기 Home-Vent 케어)
12. **외상·중증 화상 질환군** (교통사고/낙상 중증외상, 피부이식 및 흉터 드레싱)
13. **노인성 복합만성 질환군** (다약제 복용 관리, 노인증후군, 와상환자 토탈케어)
14. **호스피스·완화의료 질환군** (임종기 돌봄, 통증 조절 보조, 영적/정서적 지지)
15. **소아·청소년 희귀난치 질환군** (소아마비, 발달장애, 희귀 대사질환 특수간병)
16. **응급기도·연하장애 질환군** (석션 Suction 흡인 보조, 삼킴장애 연하곤란 식이보조)

### 2-2. 질환별 4단계 필수 이수 커리큘럼
- **Step 1. 이론 심화 교안 (E-Book / PDF 뷰어)**:
  - 병태생리, 활력징후 이상 판별, 위험 징후(Red Flag) 감지 및 응급 프로토콜.
  - 전용 뷰어 (워터마크, 목차 이동, 오디오 TTS 음성 듣기 모드 탑재).
- **Step 2. 실기·실습 심화 강의 (VOD / Live 스트리밍)**:
  - 4K 실습 영상, 1.5배속 제한, 80% 진도율 스킵 방지 트래킹, 실습일지 사진 첨부.
- **Step 3. 단원별 & 종합 모의고사 (Mock Exam)**:
  - 디지털 OMR 카드 CBT 엔진, 문제은행 랜덤 출제, 3초 주기 자동 임시저장, 즉시 오답노트.
- **Step 4. 정규 자격시험 응시 (Certification Exam)**:
  - 강의 진도율 80% 이상 + 실습일지 승인 시 응시 자격 자동 부여, 과락(40점) 및 평균 60점 합격 판정.

---

## 3. 사용자 경험(UX) 및 인터페이스 설계 지침

### 3-1. 수강생 실버 UX 핵심 원칙 (`frontend_user`, `mobileApp`)
1. **WCAG AAA 수준의 실버 접근성**:
   - 본문 텍스트 최소 16~18px, 헤드라인 20~24px, Pretendard 시스템 폰트.
   - 상단 고정 **[글자 크기 조절 (가- / 가 / 가+)]** 토글 버튼 영구 제공.
   - 눈의 피로를 낮추는 **고대비 모드(High Contrast)** 1클릭 전환.
   - 모바일 터치 타깃 최소 48x48dp, 인접 버튼 간 여백 12px 이상.
2. **원클릭 패스 학습 동선**:
   - 4단계 가로형 스텝 바(Step Indicator), "내가 마지막으로 공부하던 강의" **원클릭 이어보기 플로팅 바**.
   - 교안을 음성으로 읽어주는 **오디오북 TTS 모드**.
3. **CBT 스트레스 제로 인터페이스**:
   - 실제 종이 시험 감성의 우측 디지털 OMR 카드, 미마킹 문항 수 붉은색 카운트다운.

### 3-2. 관리자 및 콜센터 CS 포털 UX 핵심 원칙 (`frontend_admin`)
1. **3분할 원스크린 레이아웃**:
   - **1열 (30%)**: 고객/환자 통합 프로필, CTI 전화 인입 0.5초 내 자동 매칭.
   - **2열 (40%)**: 16대 질환군 수강 현황, 4단계 진도율 바, 자격증 취득 및 보수교육 주기.
   - **3열 (30%)**: 실시간 상담 티켓 작성, 원클릭 템플릿, 알림톡 재발송.
2. **상담 단축키 체계**:
   - `Alt + 1` (수강료 환불 안내), `Alt + 2` (응시자격 미달 안내), `Alt + 3` (QR 자격증 알림톡 재발송), `Alt + S` (상담 저장 및 대기 전환).

### 3-3. 의뢰자 소통용 UI 일련번호 태깅 및 단축키 인스펙션 뱃지 시스템
- **참조 규격**: [`.agents/rules/ui-tagging-inspection.md`](file:///apps/project_ecare24_elearning/.agents/rules/ui-tagging-inspection.md)
- **일련번호 체계**: 모든 화면과 주요 태그 요소에 유일한 식별 코드(`[페이지]-[섹션]-[태그]`, 예: `P01-SEC03-BTN02`) 부여.
- **단축키 토글 (`Alt + I`)**:
  - 기본 숨김 상태에서 `Alt + I` 입력 시 전 화면의 요소별 일련번호 뱃지가 일괄 표시/숨김 토글.
- **원클릭 클립보드 복사 (`Click to Copy`)**:
  - 표시된 뱃지를 클릭하면 해당 ID가 즉시 클립보드에 복사되어(`[Copied! ✓]`), 의뢰자가 피드백 작성 시 오차 없이 타깃 요소를 지목할 수 있도록 지원.

---

## 4. 민간자격 등록 및 '간병24TV' 미디어 연계 규정

### 4-1. 민간자격 관리규정 (한국직업능력연구원 pqi.or.kr)
- **자격 종목 명칭**: "환자질환별전문간병사 (16개 분야별 각 종목)"
- **자격증 발급 인프라**: 고유 등록번호(`EC24-2026-NEURO-XXXXX`), HMAC-SHA256 2D QR 코드(스캔 시 실시간 진위 검증), 모바일 자격증 지갑 카드 및 PDF 인증서, 2년 주기 보수교육 관리.

### 4-2. '간병24TV' 미디어 융합 퍼널
- YouTube Data API v3 연동: 최신 영상 3시간 주기 자동 동기화.
- 1분 케어 쇼츠 상하 스와이프 플레이어 및 "심화 교안 무료 받기" 1초 유입 퍼널 연동.

### 4-3. 간병 실무 필수 전문 용어 사전 준수 규정
- **기준 문서**: [`docs/caregiver_terminology_dictionary.md`](file:///apps/project_ecare24_elearning/docs/caregiver_terminology_dictionary.md) 및 [`.agents/rules/caregiver-terminology.md`](file:///apps/project_ecare24_elearning/.agents/rules/caregiver-terminology.md)
- **적용 원칙**:
  - 교안, 동영상 시나리오, CBT 문제은행, 콜센터 템플릿 작성 시 간병 실무 필수 전문 용어(피딩, L-tube, PEG, 폴리, 유린백, 체변, 앙와위/측위, 아스피레이션 등) 및 **"현장 압축 10대 골든 룰"**을 철저히 준수합니다.
  - 환자 안전과 직결된 임상 수칙(유린백 방광 하단 거치, 탈건착환 공식, 석션 1회 15초 이내, 피딩 시 반좌위 유지 등)을 왜곡하거나 누락하는 행위를 엄격히 금지합니다.

---

## 5. 데이터베이스(PostgreSQL) 핵심 테이블 스키마 규격

백엔드는 아래 7대 핵심 DDL 구조를 기본 데이터 모델로 채택합니다:
1. `disease_categories`: 16대 질환군 마스터 (id, code, name_ko, icon_url, display_order)
2. `courses`: 교육 과정 마스터 (id, category_id, title, target_disease, required_hours, qualification_name)
3. `curriculum_steps`: 4단계 커리큘럼 (id, course_id, step_number, step_type, content_url, audio_tts_url)
4. `exam_questions`: CBT 문제은행 (id, category_id, question_type, title, explanation, points, options JSONB, correct_answer)
5. `certificates`: 민간자격 발급 대장 (id, certificate_no, registry_no, user_id, course_id, qr_verify_token, pdf_file_url)
6. `counseling_tickets`: 콜센터 상담 티켓 (id, user_id, agent_id, inbound_caller_number, category, status, subject)
7. `counseling_logs`: 상담 통화 메모 및 녹취 (id, ticket_id, agent_id, content, call_duration_seconds, recording_file_url)

---

## 6. eCare24 하네스 전담 에이전트 팀 및 역할 (Roles)

| 에이전트 롤 | 모델 티어 | 전담 업무 범위 | 산출물 기준 |
| :--- | :--- | :--- | :--- |
| **System Architect** | Gemini Pro | 전체 4대 멀티 프로젝트 구조, DB DDL 스키마, .env 규격, .vscode 통합 디버그 환경 설계 | `docs/architecture/`, DDL 스크립트, 인터페이스 정의 |
| **Silver UX Specialist** | Gemini Pro/Flash | 수강생 포털(`frontend_user`) 및 모바일앱 실버 접근성(폰트 조절, 고대비, OMR CBT, 이어보기) 명세 | 와이어프레임 스펙, 컴포넌트 인터페이스 명세 |
| **Callcenter CS Specialist** | Gemini Pro/Flash | 관리자 및 콜센터 포털(`frontend_admin`) 3분할 원스크린, CTI 팝업 연동, 단축키 체계 설계 | 콜센터 대시보드 스펙, CTI 프로토콜 명세 |
| **Caregiver Cert Specialist** | Gemini Pro | 16대 질환군 4단계 커리큘럼, 직능원 민간자격 관리규정, CBT 출제 및 과락 판정 엔진 설계 | 커리큘럼 편성표, 자격검정 출제 가이드 |
| **Media TV24 Specialist** | Gemini Flash | 간병24TV 유튜브 API 연동, 1분 숏폼 스와이프 플레이어 및 무료 교안 유입 퍼널 설계 | 미디어 파이프라인 명세, 쇼츠 연동 규격 |
| **Quality & E2E Assurer** | Gemini Pro | Playwright Headless 16개 질환/콜센터 자동 캡처 시나리오, 보안/개인정보 암호화 감사 | `tests/specs/` 시나리오 명세, 심사 증빙 추출 규격 |

---

## 7. 업무 종료 시 Git Commit 필수 수칙

- **의무 사항**: 모든 에이전트는 기획/아키텍처 문서 생성, 스펙 갱신, 검증 완료 등 유의미한 작업 단위를 완수했을 때 **반드시 원자적 Git Commit을 수행**해야 합니다.
- **컨벤션**: `[태그]: 상세 설명 (PRD 연계)` 형식 준수 (예: `docs: 16개 질환군 커리큘럼 스펙 수립`, `arch: 콜센터 3분할 CS 인터페이스 설계`).
- **보안 검증**: 커밋 전 `.env` 등 민감 정보가 스테이징되지 않았는지 `git status`로 철저히 검증합니다.
