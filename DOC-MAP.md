# Doc Map — Okinawa Journal

> **모든 문서의 마스터 인덱스 + 위계 + 참조 관계 + 영향 매트릭스.**
> 새 문서 만들면 여기 등록 필수. 안 그러면 미아 (orphan).
> 코드 만지기 전 반드시 § 영향 매트릭스 확인.

---

## 1. 위계 다이어그램 (5 Layers + Cross-Cutting)

```
╔══════════════════════════════════════════════════════════════════════════╗
║  LEVEL 0 · VISION  (왜 만드나)                                            ║
║  ─────────────────────────────────────                                    ║
║      ┌─────────────────┐         ┌─────────────────┐                     ║
║      │   CONCEPT.md    │  ────▶  │     PRD.md      │                     ║
║      │  기획서·페르소나  │         │  요구사항·NFR    │                     ║
║      │  Fixed Facts    │         │  리스크 표        │                     ║
║      └─────────────────┘         └─────────────────┘                     ║
║                                                                           ║
║                     비전을 운영 룰화                                       ║
║                          ▼                                                ║
╠══════════════════════════════════════════════════════════════════════════╣
║  LEVEL 1 · PROJECT GUIDE  (모든 작업 진입점)                              ║
║  ─────────────────────────────────────                                    ║
║                  ┌───────────────────────┐                                ║
║                  │      CLAUDE.md        │  ← Claude 세션 시작 시 항상 읽음 ║
║                  │  운영 룰 + 빠른 룰북    │                                ║
║                  │  · 절대 금지 룰        │                                ║
║                  │  · 자주 하는 작업      │                                ║
║                  │  · 장소 검증 프로세스   │                                ║
║                  │  · init-path TDZ 룰   │                                ║
║                  └───────────────────────┘                                ║
║                                                                           ║
║                     도메인별 구체화                                        ║
║                          ▼                                                ║
╠══════════════════════════════════════════════════════════════════════════╣
║  LEVEL 2 · IMPLEMENTATION  (도메인 명세)                                  ║
║  ─────────────────────────────────────                                    ║
║   ┌─────────────┐  ┌─────────────┐  ┌─────────────────┐                  ║
║   │ DESIGN.md   │  │  ROUTE.md   │  │  MISSIONS.md    │                  ║
║   │ 아키텍처      │  │  동선 시스템  │  │  미션 9 + 6      │                  ║
║   │ 상태 모델    │  │  3-call-    │  │  Hidden Mission │                  ║
║   │ 디자인 토큰   │  │  sites rule │  │  state machine  │                  ║
║   │ 보안·CSP    │  │             │  │  UI↔Canvas      │                  ║
║   │ Dual-render │  │             │  │  Parity Rule    │                  ║
║   └─────────────┘  └─────────────┘  └─────────────────┘                  ║
║                                                                           ║
║                     구현 → 검증 항목으로                                   ║
║                          ▼                                                ║
╠══════════════════════════════════════════════════════════════════════════╣
║  LEVEL 3 · OPERATIONS  (운영·검증)                                        ║
║  ─────────────────────────────────────                                    ║
║   ┌───────────────────┐  ┌───────────┐                                   ║
║   │  VERIFICATION.md  │  │  TODO.md  │                                   ║
║   │  · [Z] 스모크 자동  │  │  L0~L4    │                                   ║
║   │  · [Q] Hidden     │  │  Sub      │                                   ║
║   │  · [P] Canvas     │  │  Sections │                                   ║
║   │  · [G] 동선        │  │           │                                   ║
║   │  · 매뉴얼 모바일    │  │           │                                   ║
║   └───────────────────┘  └───────────┘                                   ║
║                                                                           ║
║                     실행 결과 누적                                         ║
║                          ▼                                                ║
╠══════════════════════════════════════════════════════════════════════════╣
║  LEVEL 4 · KNOWLEDGE  (종합 회고 + 다음 시즌 인풋)                         ║
║  ─────────────────────────────────────                                    ║
║                  ┌───────────────────────┐                                ║
║                  │     learning.md       │                                ║
║                  │  § 1~12 작성됨        │                                ║
║                  │  § 14 SNS 발췌 패키지   │                                ║
║                  │  비전공자 친화 비유      │                                ║
║                  └───────────────────────┘                                ║
║                                                                           ║
╚══════════════════════════════════════════════════════════════════════════╝

   ┌────────────────────────────────────────────────────────────────────┐
   │  CROSS-CUTTING (모든 Layer 횡단)                                    │
   │  ─────────────────────────────────                                  │
   │   ┌─────────────────┐                                              │
   │   │   DOC-MAP.md    │  ★ 이 문서 — 마스터 인덱스 + 영향 매트릭스      │
   │   └─────────────────┘                                              │
   │                                                                    │
   │   ┌──────────────────────────────────┐                             │
   │   │  POSTMORTEM-2026-04-28.md        │  심각 회귀 사고 보고서        │
   │   │  (TDZ 사고 — Severity Critical)   │  → CLAUDE.md 룰화 + § 9     │
   │   └──────────────────────────────────┘                             │
   │                                                                    │
   │   ┌──────────────────────────────────┐                             │
   │   │  TEST-REPORT-2026-04-28.md        │  전수 테스트 결과 67/67 PASS  │
   │   │  (output of VERIFICATION 자동)     │                             │
   │   └──────────────────────────────────┘                             │
   └────────────────────────────────────────────────────────────────────┘

   ┌────────────────────────────────────────────────────────────────────┐
   │  SOURCE OF TRUTH (코드 — 모든 docs 가 이 두 파일을 묘사)              │
   │  ─────────────────────────────────                                  │
   │   ┌─────────────────┐    ┌──────────────────────┐                   │
   │   │   index.html    │    │  japan-map.svg       │                   │
   │   │   ~13,500 줄    │    │  okinawa-mainland.svg│                   │
   │   │   (HTML+CSS+JS) │    │  (지도 SVG asset)     │                   │
   │   └─────────────────┘    └──────────────────────┘                   │
   └────────────────────────────────────────────────────────────────────┘
```

---

## 2. 문서 일람 표

| Level | 문서 | 줄 수 | 역할 | 주 독자 |
|---|---|---|---|---|
| 0 | `CONCEPT.md` | 89 | 기획서 — vision, audience, fixed facts, success criteria | 사람 (메인테이너) |
| 0 | `PRD.md` | 185 | 제품 요구사항 (functional + non-functional + risks) | 사람 + Claude |
| 1 | `CLAUDE.md` | 285 | Claude 운영 가이드 (룰·자주 하는 작업·검증) | Claude |
| 2 | `DESIGN.md` | 180 | 아키텍처·상태·디자인 토큰·Dual-render·보안 | Claude (코드 작성 시) |
| 2 | `ROUTE.md` | 181 | 동선 시스템 (DAY_HOTEL·midHotel·3-call-sites) | Claude (동선 수정 시) |
| 2 | `MISSIONS.md` | 291 | 미션 9 + Hidden 6, state machine, UI↔Canvas | Claude (미션 수정 시) |
| 3 | `VERIFICATION.md` | 415 | 회귀 검증 절차 ([Z][Q][P][G] + 매뉴얼) | Claude + 사람 (머지 직전) |
| 3 | `TODO.md` | 93 | 백로그 (Level 별 분류) | 사람 |
| 4 | `learning.md` | ~1300 | 종합 회고 § 1~12 + § 14 SNS 패키지 | 사람 (다음 프로젝트) |
| ✕ | `DOC-MAP.md` | (이 문서) | 마스터 인덱스 + 영향 매트릭스 | Claude + 사람 |
| ✕ | `POSTMORTEM-2026-04-28.md` | 186 | TDZ 사고 보고서 (Critical) | Claude + 사람 (사고 후) |
| ✕ | `TEST-REPORT-2026-04-28.md` | 125 | 전수 테스트 결과 67/67 PASS | 사람 (배포 직전) |

---

## 3. 위계 룰 — 정보의 흐름

### 3.1 Top-down (새 기능 추가 시)

```
[L0] CONCEPT.md 페르소나 부합 확인
   ↓
[L0] PRD.md F-XX 섹션에 기능 명세
   ↓
[L1] DOC-MAP.md § 영향 매트릭스 → 어느 L2 docs 같이 만질지
   ↓
[L2] DESIGN/ROUTE/MISSIONS 도메인 명세
   ↓
[CODE] index.html 수정
   ↓
[L3] VERIFICATION.md 검증 항목 추가
   ↓
[L4] learning.md 회고 (회귀 발생 시)
```

### 3.2 Bottom-up (회귀 발견 시)

```
사용자 ("이거 왜 안 돼?")
   ↓
재현 + 원인 추적
   ↓
[CODE] 즉시 fix (같은 commit 에 docs 갱신)
   ↓
심각 회귀 (Severity Critical) → POSTMORTEM 작성
   ↓
[L1] CLAUDE.md 새 룰 (재발 방지)
   ↓
[L3] VERIFICATION.md 자동 검증 항목 추가 (다음에 자동 catch)
   ↓
[L4] learning.md 새 섹션 (비유 + 신호 + 다음 시즌 룰)
   ↓
[CROSS] DOC-MAP.md § 영향 매트릭스 행 추가 (이 패턴 문서화)
```

### 3.3 위계 간 참조 룰

- 모든 L2~L4 문서 끝에 **"참조" 섹션** 표준화
  - **위 link** (L0/L1) — "이 결정의 출처"
  - **옆 link** (같은 L) — 관련 도메인
  - **아래 link** (L3/L4) — 검증·회고
  - **인덱스 link** — `DOC-MAP.md`
- `learning.md` 는 모든 layer 의 sub-section 참조 가능 (통합 회고이므로)

---

## 4. 변경 영향 매트릭스 (이거 만지면 저거 같이 봐야)

> 코드 만지기 전 항상 이 표 확인. 매트릭스에 케이스 없으면 새 행 추가가 첫 작업.

### 4.1 미션 시스템

| 만지는 것 | 같이 봐야 할 것 |
|---|---|
| 새 메인 미션 추가 | `MISSIONS.md` § 추가 체크리스트 → `VERIFICATION.md` § [G] → `PRD.md` F-02 → `learning.md` § 3 |
| 새 Hidden Mission 추가 | `MISSIONS.md` § Hidden Mission System + `data-mission-type="bonus"` 필수 → `VERIFICATION.md` § [Q] → `learning.md` § 9 |
| Hidden 탭 가시성 변경 | `MISSIONS.md` § 3단계 enum + `updateHiddenUnlockUI()` → `learning.md` § 9 |
| state.hiddenUnlocked 트리거 | `DESIGN.md` § state.hiddenUnlocked → `MISSIONS.md` § 영속성 함정 → `sanitizeImportedState` |
| celebrateMission 분기 | `MISSIONS.md` § 메인/히든 독립 분기 → `VERIFICATION.md` [E] → `learning.md` § 9 |
| 미션 카운트 위젯 | `MISSIONS.md` § context-aware → `DESIGN.md` (페이지 전환 시 갱신) |

### 4.2 동선 시스템

| 만지는 것 | 같이 봐야 할 것 |
|---|---|
| 새 픽/장소 추가 | `CLAUDE.md` § 장소 검증 → `ROUTE.md` § 픽 좌표 fallback → `PLACE_QUERIES`/`PLACE_COORDS` |
| midHotel 위치 변경 | `ROUTE.md` § midHotel + § 3-call-sites → `MISSIONS.md` (afterMission ID 일치) |
| midHotel 로직 (3 함수) | `ROUTE.md` § 3-call-sites consistency rule (renderDayRoute / renderDayMap / assignPickNumbers) |
| Day 픽 순서 변경 | DOM 순서가 시간 순서이므로 → HTML 직접 swap → `ROUTE.md` § DOM = 시간 |

### 4.3 상태 / Persistence

| 만지는 것 | 같이 봐야 할 것 |
|---|---|
| state 새 필드 | `DESIGN.md` § State Model → `sanitizeImportedState` → `VERIFICATION.md` § [F] |
| 3-namespace (chosen / chosenWrites / chosenExtras) | `CLAUDE.md` § 3-namespace 동시 점검 룰 → 6 곳 코드 위치 모두 |
| sanitizer 보안 | `DESIGN.md` § 보안 → `learning.md` § 보안 → CSP meta → import 4 겹 방어 |

### 4.4 UI ↔ Canvas Parity

| 만지는 것 | 같이 봐야 할 것 |
|---|---|
| trip-conquer-card UI | `MISSIONS.md` § UI↔Canvas Parity Rule → `downloadConquerImage()` canvas 동시 갱신 → `learning.md` § 7 |
| legend-card (PRO TRAVELER) UI | `DESIGN.md` § Dual-render → `downloadLegendImage()` → `drawOkinawaSilhouetteToCanvas` → `learning.md` § 11 |
| visited-list UI | `downloadVisitedListImage()` → CSS .vl-item.mission ↔ Canvas 별 그리기 동시 → `learning.md` § 10 |
| 새 dual-render 컴포넌트 추가 | `DESIGN.md` § Dual-render 표에 행 추가 → `VERIFICATION.md` [P] 시각 비교 항목 |

### 4.5 페이지 / 네비

| 만지는 것 | 같이 봐야 할 것 |
|---|---|
| 페이지 추가/삭제 | `PAGE_IDS` → `<nav>` → `DESIGN.md` § Page Structure → `PRD.md` F-01 |
| Hidden 탭 nav 차단 | `getActivePageIds()` → 좌우 화살표 + 하단 페이지네이션 + capture phase click 핸들러 |
| 페이지네이션 spacing | `.page-pagination` 전역 — 모든 페이지 영향 → `learning.md` § 11 |

### 4.6 디자인 / 톤

| 만지는 것 | 같이 봐야 할 것 |
|---|---|
| 디자인 토큰 변경 | `DESIGN.md` § Design Tokens (alias 추적!) → `learning.md` § 1 (--coral → --blue 회귀) |
| 어휘 변경 (PRO TRAVELER 등) | 모든 등장 위치 grep → `CONCEPT.md` § 페르소나 부합 → `learning.md` § 12 |
| 사운드 변경 | `learning.md` § 9·12 (띠링 vs 삐이익) → 11살 톤 검수 |
| 강조 스타일 (highlighter 등) | `learning.md` § 11·12 |

### 4.7 보안 / 인프라

| 만지는 것 | 같이 봐야 할 것 |
|---|---|
| 새 외부 의존 | `PRD.md` NFR (의존 표) → CSP meta → `DESIGN.md` § 보안 |
| 비번 / 사용자 이름 | `CONCEPT.md` § 여행 사실 → `DESIGN.md` § Auth → `getTravelerName()` fallback |
| init-path 함수 호출 | `CLAUDE.md` § 절대 호출 금지 룰 (init-path) → `POSTMORTEM-2026-04-28.md` → try/catch 패턴 |

### 4.8 텍스트 콘텐츠

| 만지는 것 | 같이 봐야 할 것 |
|---|---|
| 핵심 멘트 (보상·축하·CTA) | 모든 등장 위치 grep → `CONCEPT.md` 페르소나 톤 부합 → 11살 어휘 검수 |
| 줄바꿈 / 호흡 | `learning.md` § 11·12 (한국어 자연 어미, word-break: keep-all) |

---

## 5. index.html 내부 영역 매핑 (키워드 검색용)

> 라인 번호는 변동 — **키워드 검색 기반** (grep / 에디터 search).

| 영역 | 키워드 | 위계 문서 |
|---|---|---|
| `<style>` 디자인 토큰 | `:root {`, `.page-nav`, `.day` | DESIGN.md § Design Tokens |
| `<head>` 보안 | `Content-Security-Policy` | DESIGN.md § 보안, CLAUDE.md |
| 페이지 8개 | `<div class="page" data-page="..."` | DESIGN.md § Page Structure, PRD.md F-01 |
| 미션 카드 | `data-mission=`, `data-mission-type="bonus"` | MISSIONS.md |
| Hidden Mission system | `state.hiddenUnlocked`, `getActivePageIds`, `celebrateMission` | MISSIONS.md § Hidden Mission |
| 동선 시스템 | `DAY_HOTEL =`, `computeMidInsertIdx`, `renderDayRoute`, `renderDayMap`, `assignPickNumbers` | ROUTE.md (3-call-sites) |
| 인증/Auth | `AUTH_HASH_HEX`, `auth-gate` | DESIGN.md § Auth |
| Export/Import | `btn-export`, `sanitizeImportedState` | DESIGN.md § State Persistence |
| trip-conquer-card | `id="trip-conquer-card"`, `updateTripStats`, `downloadConquerImage` | MISSIONS.md § Trip Complete |
| legend-card (PRO TRAVELER) | `id="legend-card"`, `updateLegendStats`, `downloadLegendImage`, `drawOkinawaSilhouetteToCanvas` | MISSIONS.md § Hidden Reward |
| stamp-grid | `id="stamp-grid"`, `buildStampGrid`, `assignMissionNumbers` | MISSIONS.md § Stamp Grid |
| visited-list | `<details class="visited-list">`, `getVisitedPlaces`, `renderVisitedList`, `downloadVisitedListImage` | MISSIONS.md § UI↔Canvas |
| 일본 지도 | `td-japan-map`, `japan-map.svg`, `drawJapanMapFromSvg` | MISSIONS.md § Trip Complete |
| 오키나와 본도 SVG | `okinawa-mainland.svg`, `lc-okinawa-svg`, `drawOkinawaZoomToCanvas`, `drawOkinawaSilhouetteToCanvas` | MISSIONS.md, learning.md § 11 |

---

## 6. 코딩 흐름 — 진입점 결정 트리

```
새 작업 시작
   │
   ├─ "새 기능 추가하고 싶음"
   │      ↓
   │   CONCEPT.md 페르소나 부합 확인
   │      ↓
   │   PRD.md F-XX 명세
   │      ↓
   │   DOC-MAP.md § 영향 매트릭스 확인 ★
   │      ↓
   │   영향 받는 docs 같이 갱신 + 코드 수정
   │
   ├─ "버그 / 회귀 발견"
   │      ↓
   │   재현 → 원인 추적
   │      ↓
   │   심각도 분류
   │      ├─ Critical (사용자 기능 마비) → POSTMORTEM 작성
   │      └─ Normal → 즉시 fix + 회고
   │      ↓
   │   VERIFICATION.md 자동 검증 추가 (다음에 자동 catch)
   │      ↓
   │   learning.md 새 섹션 (비유 + 신호)
   │
   ├─ "코드 어디 만져야 할지 모르겠음"
   │      ↓
   │   DOC-MAP.md § index.html 영역 매핑 (키워드 검색)
   │      ↓
   │   해당 영역 위계 문서 (L2 도메인) 정독
   │
   ├─ "문서 어디 적어야 할지 모르겠음"
   │      ↓
   │   분류:
   │      ├─ 결정의 출처 (왜) → L0 (CONCEPT/PRD)
   │      ├─ 작업 룰 (어떻게) → L1 (CLAUDE.md)
   │      ├─ 도메인 명세 (구체) → L2
   │      ├─ 검증/백로그 → L3
   │      └─ 회고/다음 시즌 → L4 (learning.md)
   │      ↓
   │   DOC-MAP.md § 문서 일람 표 + 영향 매트릭스 갱신
   │
   └─ "사용자 피드백 모호 / 톤 불확실"
          ↓
       CONCEPT.md § 3 페르소나 (1순위 11살, 2순위 운전자) 재확인
          ↓
       learning.md § 12 11살 톤 7 원칙 참조
```

---

## 7. 위험 신호 ↔ 진단표

> 사용자 / 자기 검수에서 이런 말이 나오면 거의 항상 어디가 깨졌나.

| 피드백 신호 | 거의 항상 원인 | 진입점 |
|---|---|---|
| "기본적인 게 왜 안 돼?" | dual-render parity 깨짐 (UI/Canvas 한 쪽만 갱신) | DESIGN.md § Dual-render → MISSIONS.md § Parity Rule |
| "갑자기 이상하게 보여" | 모바일 캐시 또는 state namespace 누락 | learning.md § 1·2 |
| "위계가 박살났다" | DOC-MAP 영향 매트릭스 갭 | **DOC-MAP.md (이 문서) 갱신부터** |
| "이거 정확한 거 맞아?" | 장소 검증 갭 | CLAUDE.md § 장소 검증 |
| "두 번째 같은 사고" | L1 룰 부족 | POSTMORTEM 작성 → CLAUDE.md 룰 |
| "이 톤 어색해" | CONCEPT 페르소나 (11살) 미반영 | CONCEPT.md § 3 + learning.md § 12 |
| "왜 자꾸 안 뜨지?" (히든 unlock) | state 영속 플래그 (hiddenUnlocked) 미리셋 | MISSIONS.md § 영속성 함정 |
| "어 이상하다, 이거 옆에 있는 거랑 어긋나 보여" | 폰트 메트릭 의존 (left bearing) | learning.md § 10 (flex centering) |
| "다운로드 받았는데 뭔가 빠진 느낌" | UI↔Canvas Parity 깨짐 | learning.md § 7·11 |

---

## 8. 문서 작성 규칙

1. **새 문서** → 위계 결정 (Level 0~4) → DOC-MAP.md 표 + 다이어그램에 등록 → 끝부분에 "참조" 섹션
2. **문서 수정** → § 4 영향 매트릭스 보고 같이 갱신할 곳 확인
3. **코드 수정 후** → Level 2 도메인 문서 + Level 3 VERIFICATION 갱신
4. **stale 방지** → 분기 1회 (또는 큰 변경 후) `VERIFICATION.md` 의 [doc 동기 점검] 실행
5. **번호 / 라인 hard-coding 금지** — 키워드 검색 기반으로 작성
6. **새 회귀 패턴** → 학습 가능 → learning.md § N 신설 (비유 포함)
7. **심각 회귀 (사용자 기능 마비)** → POSTMORTEM-YYYY-MM-DD.md 별도 파일

---

## 9. 위계가 깨진 패턴 (앞으로 피할 것)

❌ **Level 0 없이 바로 Level 2 파고들기**: "동선 어떻게 만들지" 부터 시작 → 왜 만드는지 잊고 over-engineer
❌ **Level 4 (learning) 가 Level 1 (CLAUDE) 옆에 있음**: 회고가 작업 가이드와 같은 무게로 보여 혼란
❌ **TODO.md 가 단일 카테고리**: 어느 Level 의 작업인지 구분 안 돼서 우선순위 혼란
❌ **계층 간 참조 없음**: "이 결정이 어디서 나왔는지" 추적 불가
❌ **POSTMORTEM 없이 그냥 fix**: 큰 회귀 (사용자 기능 마비) 도 일반 commit 으로 묻혀서 다음 사람이 같은 함정
❌ **DOC-MAP.md 영향 매트릭스 누락**: 새 dual-render 컴포넌트 추가하고 매트릭스에 안 써서 다음 변경 시 한 쪽 까먹음

✅ **현재 상태 (2026-04-28)**:
- 각 문서 끝에 "참조" 섹션 (위/옆/아래/INDEX link)
- TODO.md 가 Level 별 sub-section
- learning.md 가 § 1~12 통합 회고 (다른 docs 의 핵심 + 학습)
- DOC-MAP.md (이 문서) 가 위계 다이어그램 + 영향 매트릭스 + 코딩 흐름 트리 + 위험 신호 진단표
- POSTMORTEM 별도 파일로 심각 회귀 추적
- TEST-REPORT 로 출발 전 검증 결과 누적 (67/67 PASS)

---

## 10. 다음 시즌 (Volume II) 첫 입력

새 여행지 매거진 만들 때 첫 참조 자료 순서:

1. **`learning.md`** (이 프로젝트의 모든 회귀 + 패턴) — § 1~12
2. **`DOC-MAP.md`** (이 문서, 5 layer 구조 그대로 가져가기)
3. **`CONCEPT.md` 템플릿** (페르소나 채우고 fixed facts 갱신)
4. **`CLAUDE.md`** 룰 (동일하게 가져옴, 장소 데이터만 교체)
5. **코드 (`index.html`)** 복사 → 색상 토큰 + 콘텐츠 데이터 교체

**가장 중요한 학습 (learning.md § 14 Quote Card):**
1. "위계 없는 문서는 zip 파일 안에 흩어진 README들이랑 같다."
2. "현지 수정 불가는 제약이 아니라 규율이다."
3. "두 곳에서 따로 그리면 — 한 쪽만 갱신해서 어긋난다."
4. "11살 알림 사운드 = 띠링 (기쁨), 삐이익 (긴급) X."
5. "픽셀 정밀 정렬 시 폰트 메트릭 의존 X — 박스로 가둬라."
6. "왜 React 안 쓰지? 보다 왜 React 를 쓰지? 를 먼저."
7. "코드는 언젠가 복제·공유된다. 지금 10분이 미래의 모든 사본을 살린다."
8. "기본적인 게 왜 안 돼? = dual-render parity 깨짐의 신호."

---

## 11. 참조 (이 문서가 link 거는 곳)

- 비전 → `CONCEPT.md` · `PRD.md`
- 운영 → `CLAUDE.md`
- 도메인 → `DESIGN.md` · `ROUTE.md` · `MISSIONS.md`
- 검증 → `VERIFICATION.md` · `TODO.md`
- 회고 → `learning.md` (§ 1~12 + § 14 SNS 패키지)
- 사고 → `POSTMORTEM-2026-04-28.md`
- 테스트 → `TEST-REPORT-2026-04-28.md`
- 코드 → `index.html` · `japan-map.svg` · `okinawa-mainland.svg`

이 문서를 link 거는 곳:
- `CLAUDE.md` § 작업 시작 전 룰 (1번 항목)
- `learning.md` § 6 위계 정리 + 모든 § 의 "참조" 섹션
- `VERIFICATION.md` § 참조
- 모든 L2 docs (DESIGN/ROUTE/MISSIONS) § 참조
