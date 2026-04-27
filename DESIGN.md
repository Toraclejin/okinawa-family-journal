# Design — Okinawa Journal 아키텍처

> 코드를 수정하기 전에 한 번 읽어. 왜 이런 모양인지 이해하면 회귀 안 냄.

## 큰 그림

**한 가족이 여행 전·중·후로 같이 보는 디지털 매거진**.
출발 후 콘텐츠 수정 불가 → 모든 데이터는 출발 전에 확정. 사용자는 자기 흔적만 남김 (선택, 별점, 메모, 사진).

## 스택 한 줄 요약

순수 vanilla HTML + CSS + JS (프레임워크·빌드 도구 X). 단일 파일 `index.html`. 상태 = `localStorage` 1개 키. 호스팅 = GitHub Pages.

## 페이지 구조

```
PAGE_IDS = ['cover', 'prep', 'd1', 'd2', 'd3', 'd4', 'back']
```

| ID | 역할 | 핵심 컴포넌트 |
|---|---|---|
| `cover` | 표지 + 목차 | 6장 챕터 링크 |
| `prep` | 출발 전 준비 | Packing list, Things I Want, prep-ready 미션 |
| `d1` | Arrival | hotel-card · 미션 3개 (boarding, hotel, family-photo) · 옵션 슬롯 |
| `d2` | The Whale Shark | Churaumi 메인 옵션(미션) + 코우리 대교(미션) · 평범한 옵션 |
| `d3` | Crossing South | hotel-card.dual (Orion 체크아웃 + Vessel 체크인) · 미션 (vessel) · 옵션 + chatan-sunset(미션) |
| `d4` | Going Home | d4-departure 미션 + 공항 도착 가이드 |
| `back` | 마무리 | trip-conquer-card · stamp-grid · trip-done-msg · visited-list |

## 상태 모델 (localStorage)

키: `okinawa-journal-v1`

```ts
type State = {
  ratings: { [key: string]: number };         // 0–5, key = opt-dots data-key
  texts: { [key: string]: string };           // 텍스트 입력 (traveler-name 포함)
  packing: { [key: string]: boolean };        // 체크박스
  photos: { [target: string]: { id: string, dataUrl: string }[] };  // base64
  chosen: { [slotKey: string]: string[] };    // 사전 정의 옵션 idx (복수)
  chosenWrites: { [slotKey: string]: true };  // placeholder ✓
  chosenExtras: { [extraId: string]: true };  // 동적 추가 ✓
  extras: { [slotKey: string]: string[] };    // 동적 추가 카드 list
  extraOptions: { [extraId: string]: string };// 동적 카드 텍스트 (legacy)
  missions: { [missionId: string]: { done: true, at?: number } };  // 메인 9 + 히든 6 (data-mission-type 으로 분리)
  hiddenUnlocked?: true;                      // d1-hotel done 후 사용자가 YES → Hidden 탭 unlock 영속 플래그
};
```

### state.hiddenUnlocked — Hidden Mission 영속 unlock 플래그

- **기본값**: undefined (= false 취급)
- **true 가 되는 시점**:
  · `d1-hotel` 미션 done 후 unlock modal "🗝 YES" 클릭
  · OR 마무리 페이지 CTA "🎫 히든 미션 풀기" 클릭
- **false 로 리셋되는 시점**:
  · `d1-hotel` 미션 도장 취소 (`mission-undo-btn`) — 회귀 안전망
  · localStorage 전체 clear

**왜 영속 플래그?**:
- 한 번 unlock = Hidden 탭 영구 노출 (단계 3)
- 페이지 새로고침·기기 변경 후에도 유지 (export JSON 에 포함)
- Sanitizer (`sanitizeImportedState`) 통과: `if (raw.hiddenUnlocked === true) clean.hiddenUnlocked = true`

### 3-namespace 분리 (선택 상태)

이게 헷갈리는데 분리한 이유 있음:

- `chosen` — 사전 정의된 `.opt` 카드 (idx 배열)
- `chosenWrites` — placeholder `.opt-write:not(.extra)` (slotKey 단위 boolean)
- `chosenExtras` — 사용자가 추가한 `.opt-write.extra` (extraId 단위 boolean)

한 군데 만지면 다른 두 곳 영향 검토 필수. 대상 코드 6곳:
1. `clearBtn` (오늘 선택 초기화)
2. `undoBtn` (되돌리기)
3. `getDayPicks` / `getDayExtras` (Today's Picks 데이터)
4. `assignPickNumbers` (옵션 카드 좌상단 번호)
5. 평점·chip·지도 마커 렌더링
6. `sanitizeImportedState` (import 검증)

## 번호 시스템 — 두 개 분리 namespace

| Namespace | 멤버 | 표기 | 위치 |
|---|---|---|---|
| **지도** | Start, [Mid], 사전 픽, End | `1, 2, 3, ...` | chip + Leaflet 마커 + .opt 카드 좌상단 |
| **추가옵션** | placeholder, 동적 extras | `+1, +2, +3, ...` | Today's Picks 평점 + .opt-write 카드 좌상단 |

분리 이유: 둘 다 `1, 2, 3...` 쓰면 "End chip = 3" 과 "TEST extras = 3" 이 충돌해 의미 모호.

미션 번호 (1~9, `assignMissionNumbers()`) 는 **세 번째 namespace**:
- 미션 카드 `.mission-tag` 텍스트 (`Mission 3 · Check-in 15:00`)
- `.opt[data-mission]` `.mission-num-pill`
- 마무리 페이지 `stamp-grid` `.stamp-num-badge`

## 디자인 토큰 (CSS 변수)

```css
--frame: #7ce0d3;        /* 민트 — 페이지 바깥 프레임 */
--page: #ffffff;
--page-tint: #f4f8ff;    /* 연한 블루 틴트 */
--blue: #0047ff;         /* 메인 코발트 — 헤드라인·포인트 */
--blue-deep: #001f66;    /* 본문 */
--blue-ink: #000a33;     /* 최강조 */
--amber: #ffc400;        /* 별점·액센트 */
--accent-soft: #ffeba3;
--coral, --lime: 레거시 (각각 blue, amber로 통일)
```

폰트:
- **Archivo Black** — 대형 디스플레이 (OKINAWA, DAY ONE)
- **Fraunces** — 이탤릭 세리프 (강조·부제)
- **Space Grotesk** — 영문 UI (라벨·버튼)
- **Pretendard** — 한글 본문·네비

## Auth Gate

비밀번호 `0430` (가족 4월 30일 의미). SHA-256 해시 hardcoded → 코드 노출돼도 비번 모름.
토큰: `localStorage.okinawa-auth-v1` (성공 시 1년 valid).

## State Persistence — Export / Import

**Export** (btn-export):
- `state` 객체 통째 → JSON
- 파일명: `okinawa-journal-{name}-{date}.json`
- 다운로드 후 `showSaved()` indicator 표시

**Import** (btn-import → file picker):
1. FileReader 로 텍스트 읽음
2. `JSON.parse` 후 `version === 'okinawa-journal-v1'` + `isPlainObject(data.state)` 검증
3. `sanitizeImportedState()` → 화이트리스트 복원
4. `localStorage.setItem` + `location.reload()`

**roundtrip 보장 keys** (sanitizer 통과):
`ratings`, `texts`, `packing`, `photos`, `chosen`, `extras`, `chosenExtras`, `chosenWrites`, `missions`, `extraOptions` — 즉 사용자 상호작용 결과 전부.

## 보안 (Import 검증)

`localStorage` 기반 정적 HTML의 유일한 실질 공격 경로는 **악성 JSON Import**. 4-line defense:

1. **`sanitizeImportedState()`** — 화이트리스트 복원, `__proto__/constructor/prototype` 차단, 각 값 타입·길이·포맷 검증
2. **`DATA_IMG` 정규식** — 사진 dataUrl `^data:image/(jpeg|png|webp|gif);base64,...$` 만, 5MB 상한
3. **사진 DOM은 createElement** — `innerHTML` 절대 X, `img.src = dataUrl` 만
4. **extraId 가드** — `^[a-z0-9]{1,64}$` 만 (localStorage 직접 변조 대비)

**5번 (CSP meta tag)** — `<head>` 의 `Content-Security-Policy` 로 default-src self, frame/object 금지. style/script 인라인 필수라 `unsafe-inline` 허용 (의도적).

## 자주 하는 작업 → 영향 받는 파일 매트릭스

| 작업 | index.html 부분 | 문서 |
|---|---|---|
| 장소 추가 | `.opt` 카드 + `PLACE_QUERIES` + `PLACE_COORDS` | CLAUDE.md § 장소 검증 |
| 미션 추가 | `.mission-card` or `.opt[data-mission]` + 자동 번호 | MISSIONS.md |
| 동선 변경 | `DAY_HOTEL`, `DAY_HOTEL_COORDS` | ROUTE.md |
| state 새 필드 | `sanitizeImportedState` + 3-namespace 점검 | DESIGN.md (이 문서) |
| 페이지 추가 | `<div class="page">` + `PAGES` 배열 + 네비 | DESIGN.md § Page Structure |
| **trip-conquer-card 시각 변경** | 화면 SVG + `downloadConquerImage()` canvas 둘 다 | **MISSIONS.md § UI↔Canvas Parity Rule** |

## ⚠️ Dual-render Components (화면 UI ↔ Canvas 양쪽 그려지는 컴포넌트)

매거진에서 화면 UI 와 canvas PNG 양쪽을 별도로 그리는 컴포넌트들. 한 곳만 고치면 silent regression.

| 컴포넌트 | 화면 (HTML/SVG) | Canvas (downloadXxx) | 동기 룰 |
|---|---|---|---|
| 인증 카드 | `.trip-conquer-card` (DOM + japan-map.svg + okinawa-mainland.svg) | `downloadConquerImage()` + `drawJapanMapFromSvg()` + `drawOkinawaZoomToCanvas()` | MISSIONS.md § Parity Rule |
| 방문한 곳 리스트 | `<details class="visited-list">` (DOM 동적 생성) | `downloadVisitedListImage()` (Canvas Day 별 그룹) | DESIGN.md (이 섹션) |

**룰**:
1. 화면 시각 요소 추가/변경 → canvas drawing 에도 동시 적용
2. 두 코드 위치를 같은 commit 에 묶어 PR (분리 commit 금지)
3. 검증: 화면 캡처 ↔ 다운로드 PNG 좌우 비교 (VERIFICATION.md [P] 항목)
4. 새 dual-render 컴포넌트 추가 시 → 이 표 갱신

## 참조

- 위로 → `CONCEPT.md` (왜 이 매거진을 만드나) · `PRD.md` (어떤 기능을 갖나)
- 룰북 → `CLAUDE.md` (Claude 운영 가이드)
- 인덱스 → `DOC-MAP.md` (모든 문서 위계 + 영향 매트릭스)
- 옆 도메인 → `ROUTE.md` (동선) · `MISSIONS.md` (미션)
- 검증 → `VERIFICATION.md` (회귀 항목 — state 3-namespace, sanitizer)
- 회고 → `learning.md` § 2 (상태 관리 회귀)
