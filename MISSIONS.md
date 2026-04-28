# Missions — 미션 시스템 전체 흐름

> 9개 미션이 어떻게 등록·번호화·도장 처리·트립 완료까지 가는지.
> 미션 추가/수정 전 [추가 체크리스트](#미션-추가-체크리스트) 먼저 봐.

## 9개 미션 일람 (1~9)

`getAllMissions()` 가 PAGE_IDS 순서 (prep → d1 → d2 → d3 → d4) 로 정렬한 결과:

| # | ID | 페이지 | 라벨 | 카드 타입 |
|---|---|---|---|---|
| 1 | `prep-ready` | prep | Pre-Trip · 여행 준비 완료! | `.mission-card` |
| 2 | `d1-boarding` | d1 | Departure · 인천 → 오키나와 비행기 ✈️ | `.mission-card.boarding` |
| 3 | `d1-hotel` | d1 | Check-in 15:00 · Hotel Orion Motobu — 도착! | `.mission-card` |
| 4 | `d1-family-photo` | d1 | Family Photo · 가족 첫 사진 — 찍었어! | `.mission-card.photo-mission` |
| 5 | `d2-churaumi` | d2 | 고래상어 — 만났어! | `.opt[data-mission]` (Today's Main) |
| 6 | `d2-kouri-bridge` | d2 | 코우리 대교 — 건넜어! | `.opt[data-mission]` |
| 7 | `d3-vessel` | d3 | Check-in 13:00 · Vessel Campana — 도착! | `.mission-card` |
| 8 | `d3-chatan-sunset` | d3 | 차탄 선셋 — 봤어! | `.opt[data-mission]` |
| 9 | `d4-departure` | d4 | Going Home · 오키나와 → 인천 ✈️ | `.mission-card.boarding.final` |

## 두 종류 미션 카드

### 1. 표준 `.mission-card[data-mission]` (5개: 1, 2, 3, 4, 7, 9)

```html
<div class="mission-card" data-mission="d1-hotel">
  <span class="mission-tag">Mission · Check-in 15:00</span>  <!-- assignMissionNumbers 가 'Mission 3 · ...' 로 prefix -->
  <h3 class="mission-title">Hotel Orion Motobu — 도착!</h3>
  <p class="mission-desc">...</p>
  <div class="mission-action-row">
    <button type="button" class="mission-stamp-btn">
      <span class="ms-icon">✓</span><span class="ms-label">호텔 도착!</span>
    </button>
  </div>
  <div class="mission-timestamp"></div>
</div>
```

- 클릭하면 `state.missions[id] = { done: true, at: timestamp }` 저장
- `card.classList.add('completed')` + 골드 도장 시각 효과
- 가족 사진 미션 (#4) 은 사진 1장 업로드 시 자동 done

### 2. 인라인 `.opt[data-mission]` (3개: 5, 6, 8)

```html
<div class="opt highlight" data-mission="d2-churaumi" data-mission-label="고래상어 — 만났어!">
  <span class="opt-badge amber">Today's Main ✦</span>
  <span class="mission-num-pill">Mission 5 · 고래상어 — 만났어!</span>  <!-- assignMissionNumbers 가 주입 -->
  <div class="opt-title">세계 최대급 수족관</div>
  ...
</div>
```

- `.opt-select` 체크 버튼이 chosen + mission done 양쪽 동시 처리
- `data-mission-label` = 도장 그리드/평점 모음에 표시되는 라벨
- 옵션이라 `chosen[slotKey]` 에도 들어감 (3-namespace 분리 적용)

## 자동 번호 매기기 (`assignMissionNumbers`)

DOM 로드 후 + buildStampGrid 후 1회 실행. 결과:

| 영역 | 표시 방식 |
|---|---|
| `.mission-card .mission-tag` | 텍스트 prefix: `Mission · X` → `Mission 3 · X` (원본은 dataset.origText 캐시) |
| `.opt[data-mission]` | `.mission-num-pill` 별도 주입 (opt-badge 바로 다음 위치) |
| 마무리 페이지 `.stamp-item` | `.stamp-num-badge` 좌상단 작은 원 (1~9, collected 시 navy + amber) |

번호는 `getAllMissions()` 순서 = single source of truth. 미션 추가/순서 변경 시 자동 재매핑.

## 트립 완료 흐름

### `updateMissionCounter()`

매 미션 클릭 후 호출:
```js
done = state.missions 의 done:true 개수
total = getAllMissions().length
allDone = (done === total && total > 0)

if (allDone) {
  // back 페이지 default intro 숨김, trip-conquer-card·visited-list 노출
  // updateTripStats() → 통계 grid 갱신 (도장·평균만족도·사진·방문곳)
}
```

### 마지막 미션 클릭 시 — `celebrateMission()`

```js
allDone = ?
if (allDone) fireTripComplete() → fireMissionComplete() 대신 풀스크린 trip-complete (1.8s 대신 3.2s, 60 confetti, sound chime)
else fireMissionComplete()
```

`navToBackAndReveal()` 가 마무리 페이지 자동 이동 + `.just-revealed` 클래스로 trip-conquer-card 화려한 등장.

## stamp grid (마무리 페이지)

`buildStampGrid()` — `getAllMissions()` 순회하며 각 미션을 `.stamp-item` 으로 렌더:

```html
<button class="stamp-item" data-mission="d1-hotel">
  <span class="stamp-num-badge">3</span>  <!-- assignMissionNumbers 주입 -->
  <span class="stamp-icon-large">✓</span>
  <span class="stamp-day-tag">D1</span>
  <span class="stamp-name">Hotel Orion Motobu — 도착!</span>
</button>
```

클릭 → 해당 페이지로 이동 + 미션 카드로 스크롤.

## trip-conquer-card

마무리 페이지 통계 카드. `updateTripStats()` 이 채움:

### ⚠️ UI ↔ Canvas Parity Rule (필수)

`trip-conquer-card` 의 화면 UI 와 canvas 다운로드 이미지는 **반드시 시각적으로 일치**해야 함.
가족 단톡 공유본 (canvas PNG) 이 화면과 다르면 → "내가 본 거랑 다른데?" 혼란.

**변경 시 동시 갱신 필수**:
- 화면 SVG 변경 (`.td-japan-map`, `.td-okinawa-zoom`, `.td-stats`, `.td-conquer-*`) → `downloadConquerImage()` 도 동기 변경
- 새 요소 추가 (예: 줌 SVG, 화살표, 라벨) → canvas 에도 같은 요소 그려야 함
- 색상·폰트·여백 변경 → canvas drawing 의 `fillStyle`, `font`, position 도 갱신
- 박스 사이즈·레이아웃 변경 → canvas Y 좌표 + `H` (canvas 높이) 도 갱신

**parity 위반 사례 (2026-04-28)**:
- 화면에 오키나와 줌 SVG 추가했는데 `downloadConquerImage()` 안 고침 → 다운로드 이미지에 오키나와 부분 누락
- 학습: 시각 컴포넌트 추가는 **양쪽 동시 작업** = 한 commit 안에 둘 다 수정

**`downloadConquerImage()` 위치** (검색 키워드):
- 함수: `async function downloadConquerImage()`
- 헬퍼: `drawJapanMapFromSvg`, `drawOkinawaZoomToCanvas`
- 캔버스 크기: `const W = 1080, H = 1560`
- 변경 시 VERIFICATION.md 자동 검증 [P] (canvas parity 항목) 도 갱신

**검증 절차**:
1. 화면에서 trip-conquer-card 캡처
2. 다운로드 버튼으로 PNG 저장
3. 두 이미지 좌우 비교 — 요소·위치·색상·폰트 모두 일치하는지



| stat key | 계산 |
|---|---|
| `missions-frac` | `${done}/${total}` 형식 |
| `rating` | 별점 평균 (state.ratings 의 finite 값들 평균, `4.3★` 형식) |
| `photos` | 모든 photos 배열 길이 합 |
| `visited` | `getVisitedPlaces().items.length` 합 (Day 그룹 픽 + 호텔/공항 미션 합산) |

**일본 지도 + 오키나와 강조** — `japan-map.svg` (Adobe Illustrator export, Ozean 제거) 임베드 + 좌표 (381, 535) 에 골드 ★ 강조.

## visited-list (방문한 곳)

```js
function getVisitedPlaces() {
  // PAGE_IDS 순회 + DOM 순서로 미션 + 픽 수집
  // 1. .mission-card[data-mission] with state.missions[id].done:true
  // 2. .opt with chosen + .opt[data-mission] 중복 제거
  return [{dayId, items: [{time, title}]}]
}
```

`renderVisitedList(visited)` — Day 별 그룹 + 시간 순서. 다운로드 가능 (canvas, 1080px).

## 미션 추가 체크리스트

새 미션 (예: `d2-aquarium-photo`) 추가 시:

- [ ] `index.html` 에 `.mission-card` 또는 `.opt[data-mission]` 추가
- [ ] ID 가 `MISSION_ID_RE` (`^[a-z0-9-]+$`) 를 통과하는지 확인
- [ ] `data-mission-label` 추가 (`.opt[data-mission]` 인 경우)
- [ ] `assignMissionNumbers()` 가 자동 인식하는지 — DOM 위치가 PAGE_IDS 순서대로면 OK
- [ ] `getAllMissions()` 결과에 들어가는지 console 확인
- [ ] `MISSIONS.md` 일람표 업데이트 (#번호도)
- [ ] `VERIFICATION.md` 자동 검증 [G] 항목에 미션 카운트 체크 업데이트
- [ ] `trip-conquer-card` 의 `[data-stat="missions-frac"]` 분모가 자동 갱신되는지 (getAllMissions().length 기반)
- [ ] 테스트: 추가한 미션 클릭 → `state.missions[id].done = true` + counter 갱신 + stamp grid 표시
- [ ] 테스트: 모든 미션 done → trip-complete 정상 발사

## 미션 ID 명명 규칙

```
{pageId}-{slug}
e.g.  d1-hotel, d2-churaumi, d3-vessel, prep-ready
```

영숫자 + 하이픈만, 64자 이하. `__proto__` / `constructor` / `prototype` 차단됨 (sanitizer).

---

## Hidden Mission System (Bonus 6 — 트리거: d1-hotel 도장)

> **별도 namespace** 로 메인 9개와 완전 분리 운영. 11살 조카 도파민 설계.

### 흐름 (d1-hotel ✓ 클릭 시)

```
[click ✓ 호텔 도착!]
     ↓
[0~1.8s] MISSION COMPLETE 풀스크린 (앰버, 일반 미션 효과)
     ↓
[2.1s]   🎫 TICKET ISSUED 풀스크린 cinematic
         · 30개 그린 confetti 폭발
         · 부드러운 sine chime 도-미-솔 (놀람 X)
         · "남건우님, 시크릿 레벨 풀렸다!"
     ↓
[3.8s+]  YES/NO 모달
         · "남건우님께서 Hidden Mission을 활성화시키셨습니다"
         · 🗝 YES — 지금 풀게  /  NO — 나중에 풀게
     ↓
   YES → state.hiddenUnlocked = true → Hidden 탭 단계 3 (라벨+NEW 배지)
   NO  → 마무리 페이지 CTA "🎫 히든 미션 풀기" 상시 노출 (안전망)
```

### Hidden 탭 3단계 가시성 (단계 enum)

| 상태 | d1-hotel.done | hiddenUnlocked | 탭 노출 | 클릭 결과 |
|---|---|---|---|---|
| 단계 1 | false | (anything) | display:none | N/A |
| 단계 2 | true | false | 자물쇠 🔒 + 반짝 | unlock 모달 등장 |
| 단계 3 | true | true | "Hidden" 라벨 + NEW 배지 | Hidden 페이지로 이동 |

**state machine 룰**:
- 단계 1 → 2: `state.missions['d1-hotel'].done = true` 시점 (자동)
- 단계 2 → 3: 사용자가 모달에서 YES 또는 마무리 CTA 클릭 (사용자 선택)
- 단계 3 → 2 → 1: d1-hotel 도장 취소 시 (`isD1Hotel && state.hiddenUnlocked → reset`)

### TICKET PENDING CTA 박스 (단계 2 안전망)

마무리 페이지(`back`) 안에 노출되는 그린 dashed 박스. **도장 박스(stamp-collection) 바로 위 ~2cm (76px) 간격** 으로 위치.

**노출 조건** (`updateHiddenUnlockUI()`):
- d1-hotel done O **AND** !hiddenUnlocked → CTA 노출 (단계 2)
- 그 외 단계 (1 / 3) → CTA hidden

**왜 도장 박스 위?**:
- 사용자가 NO 누른 후 마무리 페이지 진입 시 가장 먼저 보이는 위치
- 도장 박스보다 위에 있어서 "히든 티켓이 아직 살아있다" 라는 인상 강조
- ~2cm 간격으로 떠있어서 도장 박스와 시각적 분리 (별도 시스템임을 명확히)

**NO 모달 → 자동 스크롤 흐름** (`#hum-no` 클릭 시):
```js
hideHiddenUnlockModal();
showPage('back');                                    // 마무리 페이지로 이동
setTimeout(() => {
  const cta = document.getElementById('hidden-unlock-cta');
  if (cta && !cta.hidden) {
    cta.scrollIntoView({behavior: 'smooth', block: 'center'});
  }
}, 380);                                             // 페이지 전환 애니 종료 대기
```

**사용자 시나리오**:
- d1-hotel ✓ → MISSION COMPLETE → TICKET ISSUED → YES/NO 모달
- NO 클릭 → 모달 닫힘 → 마무리 페이지 자동 이동 → CTA 박스로 부드럽게 스크롤
- 사용자: "아 나중에 풀게" 의도였어도 CTA 위치 확인 → 언제든 다시 풀 수 있음

**회귀 방지 룰**:
- CTA 의 DOM 위치 변경 시 → 단계 2/3 가시성 (`updateHiddenUnlockUI`) 회귀 검증
- `id="hidden-unlock-cta"` 정확히 1개 (페이지 끝에 잔여 없는지 grep)
- `selector` 변경 X — id/class 그대로 유지 (selector 끊기면 click 핸들러 + visibility 모두 깨짐)

### state 영속성 함정 (회귀 사례 2026-04-28)

**문제**: 개발 테스트 시 한 번 YES 누르면 `state.hiddenUnlocked = true` 영구 저장 → 다음 d1-hotel ✓ 클릭에서 cinematic 재발화 X

**해결 (커밋 c2f6d47 이후)**:
- d1-hotel 도장 취소(`mission-undo-btn`) 시 `state.hiddenUnlocked = false` 자동 리셋
- 사용자한테는 confirm 메시지 명시: "도장을 취소하면 Hidden Mission 잠금도 함께 초기화됩니다"
- 다음 d1-hotel ✓ 시 cinematic 재진입 가능

**왜 이렇게 설계?**:
- 실수로 도장 누른 사용자 = 도장 취소 + 다시 도장으로 cinematic 재경험
- 개발 테스트 = 도장 취소만으로 fresh 상태 복귀 (localStorage clear 불필요)
- 11살 조카 본인 사용 시 = 한 번 클릭하면 영구 unlock — 정상

### 6개 Hidden Mission 일람 (data-mission-type="bonus")

| ID | 카테고리 | 라벨 |
|---|---|---|
| `hidden-h1-massage` | 가족 | 아빠 어깨 5분 안마 |
| `hidden-h2-bag` | 가족 | 엄마 짐 들어주기 |
| `hidden-h3-shisa` | 오키나와 문화 | 류큐 시사상 1개 발견 + 사진 |
| `hidden-h4-fish` | 추라우미 학습 | 좋아하는 물고기 그림 + 가족 설명 |
| `hidden-h5-japanese` | 일본어 인사 | ありがとう / こんにちは / おいしい 3개 사용 |
| `hidden-h6-order` | BOSS — 도전 | 현지 상점에서 직접 주문 1번 |

### 보상 (메인 9 vs 히든 6 분리)

| | 메인 9개 | 히든 6개 |
|---|---|---|
| 색감 | 골드 + 다크 네이비 | 그린 + 다크 네이비 |
| 풀스크린 라벨 | "TRIP CONQUERED · 오키나와 정복!" | "FAMILY LEGEND · 패밀리 레전드!" |
| 풀스크린 시간 | 3.2s | 1.8s |
| 사운드 | 도-미-솔-도 (장조) | 미-솔-시-미 (장조, 다른 톤) |
| Counter | 9/9 COMPLETE (메인 페이지) | 6/6 FAMILY LEGEND (Hidden 페이지) |
| 인증 카드 | trip-conquer-card (마무리 페이지) | family-master-msg (Hidden 페이지) |
| Stamp Grid | 마무리 페이지 stamp-grid (메인만) | 별도 (Hidden 페이지 카드 자체) |

### Mission 3 (d1-hotel) 시각 강조 — `.mission-special` 클래스

다른 미션과 명확히 구분 — Hidden Mission 트리거임을 11살 시선에 강하게 어필:
- Border: gradient amber → green → amber
- 좌상단 리본: "🎫 SECRET TRIGGER" (그린, 펄스)
- 큰 티저 박스: "호텔 도착한 다음 — 예상 못한 일이 생길지도?!?" (그린 fill + 광택 shine)
- 이중 펄스 애니: shimmer + glow

### 새 Hidden Mission 추가 시 체크리스트

- [ ] `<div class="mission-card hidden-mission" data-mission="hidden-hN-..." data-mission-type="bonus">`
- [ ] `data-mission-type="bonus"` 빠뜨리면 메인 9 카운터에 들어감 (회귀 위험)
- [ ] Hidden 페이지 6/6 카운터 갱신 위해 `getHiddenMissions().length` 자동 적용
- [ ] `mission-tag-hidden` 클래스 (그린 톤)
- [ ] `MISSIONS.md` 일람표 업데이트
- [ ] `VERIFICATION.md` [Q] 항목에 케이스 추가

## 관련 문서

- 인덱스 → `DOC-MAP.md` (모든 문서 위계 + 변경 영향 매트릭스)
- 위로 → `PRD.md` F-02 (미션 시스템 요구사항)
- 옆 도메인 → `DESIGN.md` § 3-namespace 분리 — chosen / chosenWrites / chosenExtras
- 동선 연계 → `ROUTE.md` § afterMission — 동선 mid-stop 위치 결정에 미션 ID 활용
- 운영 룰 → `CLAUDE.md` § state 구조 — 전체 localStorage 스키마
- 검증 → `VERIFICATION.md` — 회귀 검증 항목 [G] 9개 미션 카운트, [Q] Hidden 3단계 가시성
- 회고 → `learning.md` § 3 미션 시스템 + § 9 Hidden Mission 도파민 설계
