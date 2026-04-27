# Doc Map — Okinawa Journal

> **모든 문서의 마스터 인덱스 + 위계 + 참조 관계.**
> 새 문서 만들면 여기 등록 필수. 안 그러면 미아.

## 위계 (Hierarchy)

```
┌──────────────────────────────────────────────────────────────┐
│  LEVEL 0 · VISION (왜 만드나)                                 │
│  ─────────────────────────────────────────                    │
│  CONCEPT.md  ─ 기획서 (audience, success criteria, fixed facts)│
│  PRD.md      ─ 제품 요구사항 (functional + non-functional)     │
└──────────────┬───────────────────────────────────────────────┘
               │ 비전을 운영 가이드로
               ▼
┌──────────────────────────────────────────────────────────────┐
│  LEVEL 1 · PROJECT GUIDE (작업 진입점)                         │
│  ─────────────────────────────────────────                    │
│  CLAUDE.md   ─ 모든 Claude 세션이 시작점, 빠른 룰북            │
└──────────────┬───────────────────────────────────────────────┘
               │ 도메인별 구체 룰
               ▼
┌──────────────────────────────────────────────────────────────┐
│  LEVEL 2 · IMPLEMENTATION (도메인 명세)                        │
│  ─────────────────────────────────────────                    │
│  DESIGN.md   ─ 아키텍처·상태 모델·디자인 토큰·Auth·Export      │
│  ROUTE.md    ─ 동선 시스템 (DAY_HOTEL, midHotel)              │
│  MISSIONS.md ─ 미션 시스템 (9개, 번호, stamp grid)            │
└──────────────┬───────────────────────────────────────────────┘
               │ 구현이 깨졌나 / 뭘 해야 하나
               ▼
┌──────────────────────────────────────────────────────────────┐
│  LEVEL 3 · OPERATIONS (운영·검증)                              │
│  ─────────────────────────────────────────                    │
│  VERIFICATION.md ─ 회귀 검증 절차 (자동 console eval + 수동)   │
│  TODO.md         ─ 백로그 (Level 1~2 모든 도메인 참조)         │
└──────────────┬───────────────────────────────────────────────┘
               │ 끝나고 무엇을 배웠나
               ▼
┌──────────────────────────────────────────────────────────────┐
│  LEVEL 4 · KNOWLEDGE (종합 회고 — 다음 프로젝트에 전수)         │
│  ─────────────────────────────────────────                    │
│  learning.md ─ Level 0~3 모든 docs + 코드의 통합 회고           │
└──────────────────────────────────────────────────────────────┘

         ◀━━━ ALL LEVELS ━━━━━━━━━━━━━━━━━━━━━━━━▶
                       ▼
              ┌─────────────────┐
              │  index.html     │  ← single source of truth (코드)
              │  japan-map.svg  │
              └─────────────────┘
```

## 문서 일람 (계층 순)

| Level | 문서 | 역할 | 주 독자 |
|---|---|---|---|
| 0 | `CONCEPT.md` | 기획서 — vision, audience, fixed facts | 사람 (메인테이너) |
| 0 | `PRD.md` | 제품 요구사항 (기능·플로우·NFR·리스크) | 사람 + Claude |
| 1 | `CLAUDE.md` | Claude 운영 가이드 (룰·자주 하는 작업) | Claude |
| 2 | `DESIGN.md` | 아키텍처·상태 모델·디자인 토큰 | Claude (코드 작성 시) |
| 2 | `ROUTE.md` | 동선 시스템 (3-call-sites consistency) | Claude (동선 수정 시) |
| 2 | `MISSIONS.md` | 미션 시스템 (9개·번호·stamp grid) | Claude (미션 수정 시) |
| 3 | `VERIFICATION.md` | 회귀 검증 절차 | Claude + 사람 (머지 직전) |
| 3 | `TODO.md` | 백로그 (각 Level 별 분류) | 사람 |
| 4 | `learning.md` | 종합 회고 (Level 0~3 통합) | 사람 (다음 프로젝트) |
| - | `DOC-MAP.md` | **이 문서** | Claude + 사람 |

## 위계 룰

### 위에서 아래로 (Top-down)
- **Level 0** 결정 (CONCEPT/PRD) → **Level 1** (CLAUDE.md) 가 받아 운영 룰화
- **Level 1** 룰 → **Level 2** (DESIGN/ROUTE/MISSIONS) 가 도메인별로 구체화
- **Level 2** 구현 → **Level 3** (VERIFICATION/TODO) 가 검증·작업 항목으로 변환
- **Level 3** 작업 결과 → **Level 4** (learning.md) 에 회고로 누적

### 아래에서 위로 (Bottom-up)
- 코드(`index.html`) 변경 → 영향 받는 Level 2 문서 갱신
- Level 2 변경 → Level 1 (CLAUDE.md) 의 빠른 참조 갱신
- 큰 결정 변경 → Level 0 (CONCEPT/PRD) 까지 거슬러 올라가 검토

### 위계 간 참조 (Reference)
- 모든 Level 2~3 문서 끝에 "참조" 섹션 — 위로/아래로 link
- `learning.md` 는 모든 Level 의 sub-section 참조 가능 (통합)

## 변경 영향 매트릭스 (이거 만지면 저거 같이 봐야 함)

| 만지는 것 | 같이 봐야 할 것 |
|---|---|
| 새 미션 추가 (메인) | `MISSIONS.md` § 추가 체크리스트 → `VERIFICATION.md` § [G] → `PRD.md` F-02 → `learning.md` § 3 |
| 새 Hidden Mission 추가 | `MISSIONS.md` § Hidden Mission System 일람 + `data-mission-type="bonus"` 필수 → `VERIFICATION.md` § [Q] |
| Hidden 탭 가시성 로직 변경 | `MISSIONS.md` § 3단계 enum + `updateHiddenUnlockUI()` → `learning.md` § 9 |
| state.hiddenUnlocked 트리거 변경 | `DESIGN.md` § state.hiddenUnlocked → `MISSIONS.md` § 영속성 함정 → `sanitizeImportedState` |
| 새 픽/장소 추가 | `CLAUDE.md` § 장소 검증 → `ROUTE.md` § 픽 좌표 fallback → `PLACE_QUERIES`/`PLACE_COORDS` |
| state 새 필드 | `DESIGN.md` § State Model → `sanitizeImportedState` → `VERIFICATION.md` § [F] |
| midHotel 위치 변경 | `ROUTE.md` § midHotel + § 3-call-sites → `MISSIONS.md` (afterMission ID 일치) |
| trip-conquer-card UI 변경 | `MISSIONS.md` § UI↔Canvas Parity Rule → `downloadConquerImage()` canvas 동시 갱신 |
| 페이지 추가/삭제 | `PAGE_IDS` → `<nav>` → `DESIGN.md` § Page Structure → `PRD.md` F-01 |
| 디자인 토큰 변경 | `DESIGN.md` § Design Tokens (alias 추적!) |
| 새 외부 의존 | `PRD.md` NFR (의존 표) → CSP meta → `DESIGN.md` § 보안 |
| 비번 / 사용자 이름 변경 | `CONCEPT.md` § 여행 사실 → `DESIGN.md` § Auth → `getTravelerName()` fallback ('남건우') |

## index.html 내부 영역 매핑 (키워드 검색용)

> 라인 번호는 변동 — **키워드 검색 기반** (grep / 에디터 search).

| 영역 | 키워드 | 위계 문서 |
|---|---|---|
| `<style>` | `:root {`, `.page-nav`, `.day` | DESIGN.md § Design Tokens |
| 헤드 | `Content-Security-Policy` | DESIGN.md § 보안, CLAUDE.md |
| 페이지 7개 | `<div class="page" data-page="..."` | DESIGN.md § Page Structure, PRD.md F-01 |
| 미션 카드 | `data-mission=` | MISSIONS.md |
| 동선 시스템 | `DAY_HOTEL =`, `computeMidInsertIdx`, `renderDayRoute`, `renderDayMap`, `assignPickNumbers` | ROUTE.md |
| 인증/Auth | `AUTH_HASH_HEX`, `auth-gate` | DESIGN.md § Auth |
| Export/Import | `btn-export`, `sanitizeImportedState` | DESIGN.md § State Persistence |
| trip-conquer-card | `id="trip-conquer-card"`, `updateTripStats` | MISSIONS.md § Trip Complete |
| stamp-grid | `id="stamp-grid"`, `buildStampGrid`, `assignMissionNumbers` | MISSIONS.md § Stamp Grid |
| 일본 지도 | `td-japan-map`, `japan-map.svg`, `drawJapanMapFromSvg` | MISSIONS.md § Trip Complete |

## 문서 작성 규칙

1. **새 문서** → 위계 결정 (Level 0~4) → DOC-MAP.md 표 + 다이어그램에 등록 → 끝부분에 "참조" 섹션
2. **문서 수정** → 영향 매트릭스 보고 같이 갱신할 곳 확인
3. **코드 수정 후** → Level 2 도메인 문서 + Level 3 VERIFICATION 갱신
4. **stale 방지** → 분기 1회 (또는 큰 변경 후) `VERIFICATION.md` 의 [doc 동기 점검] 항목 실행
5. **번호 / 라인 hard-coding 금지** — 키워드 검색 기반으로 작성

## ROUTE.md 가 자꾸 어긋나는 이유 + 대응

> 형이 지적한 부분 — 회고

**원인**:
- ROUTE.md 에 코드 스니펫·구체 함수명을 너무 자세히 적어둠 → 코드 변경 시 동시 갱신 필요한데 누락
- 3-call-sites (`renderDayRoute` / `renderDayMap` / `assignPickNumbers`) 가 분리돼 있어 한 곳 고치고 다른 곳 안 고침

**대응 (이번 정리부터)**:
- ROUTE.md 는 **불변 원칙** 위주 — "midHotel 은 afterMission 의 DOM 위치 기준" 같은 invariant
- 코드 디테일은 키워드 검색 hint 로만 (`// 이 함수가 미들 hotel 삽입` 같은 JSDoc 코멘트는 코드에 두고 docs 는 high-level)
- 3-call-sites consistency rule 을 ROUTE.md 첫 부분에 박음 + VERIFICATION.md 에 자동 검증 추가

## 위계가 깨진 패턴 (앞으로 피할 것)

❌ **Level 0 없이 바로 Level 2 파고들기**: "동선 어떻게 만들지" 부터 시작 → 왜 만드는지 잊고 over-engineer
❌ **Level 4 (learning) 가 Level 1 (CLAUDE) 옆에 있음**: 회고가 작업 가이드와 같은 무게로 보여 혼란
❌ **TODO.md 가 단일 카테고리**: 어느 Level 의 작업인지 구분 안 돼서 우선순위 혼란
❌ **계층 간 참조 없음**: "이 결정이 어디서 나왔는지" 추적 불가

✅ **이번 정리 후**:
- 각 문서 끝에 "참조" 섹션 (위로/아래로 link)
- TODO.md 가 Level 별 sub-section
- learning.md 는 통합 회고 (다른 docs 의 핵심 내용 + 학습)
- DOC-MAP.md 가 위계 다이어그램 + 영향 매트릭스로 흐름 시각화
