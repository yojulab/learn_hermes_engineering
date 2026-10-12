# 의뢰자 소통용 UI 일련번호 태깅 & 인스펙션 뱃지 시스템 규격
## (Client-Agent UI Tagging & Inspection System)

- **적용 대상**: `frontend_user/`, `frontend_admin/`, `mobileApp/` 및 모든 프로토타입/와이어프레임 화면
- **적용 목적**: 의뢰자(클라이언트/기획자)와 AI 에이전트/개발자 간의 피드백 소통 시, 모호한 묘사 대신 **화면(Page)과 UI 요소(Tag/Element)의 고유 일련번호**를 기반으로 100% 오차 없는 정밀한 커뮤니케이션을 실현함.

---

## 1. 일련번호 부여 규칙 (Serial Numbering Convention)

모든 페이지 및 대화형 UI 요소는 아래 표준 포맷에 따라 유일한 식별 코드를 부여합니다.

### 1-1. 전체 식별자 구조
```text
[페이지코드]-[섹션코드]-[태그/요소코드]
예시: P01-SEC03-BTN02
```

### 1-2. 코드 구성 규칙

| 단위 | 코드 체계 | 규칙 및 예시 |
| :--- | :--- | :--- |
| **페이지 (Page)** | `P{01~99}-{도메인}` | - `P01-USER-HOME`: 수강생 메인 홈<br>- `P02-USER-COURSE-DETAIL`: 16대 질환 상세 및 4단계 수강<br>- `P03-USER-EXAM-CBT`: 디지털 OMR CBT 시험 화면<br>- `P04-ADMIN-CALLCENTER`: 콜센터 3분할 원스크린 포털<br>- `P05-ADMIN-CERT-REGISTRY`: 민간자격증 대장 및 직능원 보고 |
| **섹션 (Section)** | `SEC{01~99}-{영역}` | - `SEC01-HEADER`: 상단 글로벌 헤더<br>- `SEC02-HERO`: 메인 비주얼 배너<br>- `SEC03-DISEASE-GRID`: 16대 질환군 카드 그리드<br>- `SEC04-OMR-PANEL`: 우측 종이 감성 OMR 카드 영역<br>- `SEC05-CTI-PANEL`: 콜센터 좌측 1열 CTI 고객정보 패널 |
| **요소/태그 (Tag)** | `{타입약어}{01~99}` | - `BTN01`: 버튼 (Button)<br>- `INP01`: 입력창 (Input Text)<br>- `SEL01`: 선택박스 (Select / Dropdown)<br>- `CRD01`: 카드 컴포넌트 (Card)<br>- `MOD01`: 모달/팝업 (Modal / Dialog)<br>- `BDG01`: 뱃지/라벨 (Badge)<br>- `TAB01`: 탭 요소 (Tab Item) |

---

## 2. 단축키 인스펙션 뱃지 기능 규격

모든 프론트엔드 빌드 시 아래 인스펙션 오버레이 메커니즘을 내장합니다.

### 2-1. 단축키 토글 (Toggle Shortcut)
- **전역 단축키**: **`Alt + I`** (Inspection의 I) 또는 **`Ctrl + Shift + T`**
- **동작**:
  - 기본 상태: 비활성화 (운영/학습자 뷰에 방해되지 않도록 숨김).
  - 단축키 입력 시: 화면의 모든 태그에 부여된 시각적 뱃지가 부드러운 페이드인 애니메이션과 함께 즉시 표시됨. 재입력 시 숨김.

### 2-2. 뱃지 시각 스타일 (Badge Styling)
- **위치**: 해당 태그 요소의 좌상단(Top-Left)에 떠 있는 플로팅 핀 형태.
- **디자인**: 고대비 다크 바이올렛/네온 블루 배경, 흰색 텍스트, 11px 폰트, 둥근 모서리(border-radius: 4px).
- **마우스 호버**: 스케일 1.1배 확대 및 `"클릭 시 ID 복사"` 툴팁 노출.

### 2-3. 원클릭 클립보드 복사 (Click to Copy)
- **동작**:
  1. 의뢰자가 수정하고 싶은 요소의 일련번호 뱃지를 마우스로 클릭.
  2. 클립보드에 해당 고유 ID(예: `P01-SEC03-BTN02`)가 즉시 복사됨 (`navigator.clipboard.writeText`).
  3. 뱃지 라벨이 1.2초간 녹색 **`[Copied! ✓]`**로 바뀌며 하단에 미니 토스트 알림 발생.

---

## 3. 의뢰자-에이전트 협업 소통 시나리오

```
[의뢰자 (클라이언트)]
  1. Alt + I 를 눌러 화면의 인스펙션 뱃지를 켠다.
  2. 수정하고 싶은 버튼의 뱃지를 클릭한다 -> 'P02-SEC02-BTN01' 클립보드 자동 복사.
  3. 채팅창에 붙여넣기(Ctrl+V)하여 요청:
     "P02-SEC02-BTN01 버튼의 문구를 '자격시험 바로 응시'로 변경하고 배경색을 에메랄드로 바꿔주세요."

[AI 에이전트]
  1. 'P02-SEC02-BTN01' 태그 식별자를 grep_search로 즉시 추적.
  2. 오차 없이 정확한 파일의 해당 엘리먼트를 특정하여 0.1초 만에 수정 완료.
```

---

## 4. 프론트엔드 공통 인스펙션 스크립트 규격 (`ui-tagger.js`)

개발 단계에서 모든 웹 화면에 주입되는 경량 스크립트 표준:

```javascript
// ui-tagger.js — 의뢰자 소통용 UI 일련번호 인스펙션 엔진
(function() {
  let isInspectMode = false;
  
  // 1. 단축키 리스너 (Alt + I 또는 Ctrl + Shift + T)
  window.addEventListener('keydown', (e) => {
    if ((e.altKey && e.key.toLowerCase() === 'i') || (e.ctrlKey && e.shiftKey && e.key.toLowerCase() === 't')) {
      e.preventDefault();
      toggleInspector();
    }
  });

  function toggleInspector() {
    isInspectMode = !isInspectMode;
    document.body.classList.toggle('ui-inspect-active', isInspectMode);
    showToast(isInspectMode ? '🔍 UI 일련번호 뱃지 활성화 (Alt+I로 닫기)' : 'UI 일련번호 뱃지 숨김');
  }

  // 2. 뱃지 클릭 시 클립보드 복사 이벤트 위임
  document.addEventListener('click', (e) => {
    const badge = e.target.closest('.ui-serial-badge');
    if (!badge) return;
    e.preventDefault();
    e.stopPropagation();

    const serialId = badge.getAttribute('data-serial-id');
    navigator.clipboard.writeText(serialId).then(() => {
      const originalText = badge.innerText;
      badge.innerText = 'Copied! ✓';
      badge.classList.add('copied');
      setTimeout(() => {
        badge.innerText = originalText;
        badge.classList.remove('copied');
      }, 1200);
      showToast(`📋 [${serialId}] 복사 완료! 피드백에 붙여넣으세요.`);
    });
  });

  function showToast(msg) {
    let toast = document.getElementById('ui-tagger-toast');
    if (!toast) {
      toast = document.createElement('div');
      toast.id = 'ui-tagger-toast';
      document.body.appendChild(toast);
    }
    toast.innerText = msg;
    toast.className = 'ui-toast-show';
    setTimeout(() => { toast.className = ''; }, 2000);
  }
})();
```

### 4-1. 전용 스타일시트 규격
```css
/* 기본 상태는 숨김 */
.ui-serial-badge {
  display: none;
  position: absolute;
  top: 2px;
  left: 2px;
  z-index: 99999;
  background: #4f46e5;
  color: #ffffff;
  font-size: 11px;
  font-family: monospace;
  font-weight: 700;
  padding: 2px 6px;
  border-radius: 4px;
  cursor: pointer;
  box-shadow: 0 2px 5px rgba(0,0,0,0.3);
  transition: all 0.2s ease;
}
.ui-serial-badge:hover {
  transform: scale(1.1);
  background: #3730a3;
}
.ui-serial-badge.copied {
  background: #10b981 !important;
}

/* Alt + I 활성화 시 전체 노출 */
body.ui-inspect-active .ui-serial-badge {
  display: inline-flex !important;
  animation: badgeFadeIn 0.25s ease-out;
}
@keyframes badgeFadeIn {
  from { opacity: 0; transform: translateY(-3px); }
  to { opacity: 1; transform: translateY(0); }
}

/* 토스트 알림 */
#ui-tagger-toast {
  position: fixed;
  bottom: 24px;
  left: 50%;
  transform: translateX(-50%);
  background: rgba(17, 24, 39, 0.95);
  color: #fff;
  padding: 10px 20px;
  border-radius: 8px;
  font-size: 13px;
  z-index: 100000;
  pointer-events: none;
  opacity: 0;
  transition: opacity 0.3s ease;
}
#ui-tagger-toast.ui-toast-show {
  opacity: 1;
}
```

---

## 5. 하네스 에이전트 준수 가이드라인

1. **Silver UX Specialist**:
   - `frontend_user/` 및 `mobileApp/` 컴포넌트 마크업 시 모든 주요 태그에 `data-serial-id="P01-SEC01-BTN01"` 및 일련번호 뱃지 마크업을 기본 구조로 반영하도록 스펙을 설계한다.
2. **Callcenter CS Specialist**:
   - `frontend_admin/`의 3분할 패널 및 CTI 단축키 요소에 `data-serial-id="P04-SEC01-INP01"` 등의 일련번호 앵커를 설계한다.
3. **Quality & E2E Assurer**:
   - Playwright 자동화 테스트 작성 시 복잡하고 깨지기 쉬운 CSS 셀렉터 대신 `data-serial-id` 속성을 1순위 테스트 셀렉터로 채택하여 테스트 안정성을 극대화한다.
