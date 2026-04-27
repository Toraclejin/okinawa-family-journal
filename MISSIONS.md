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

## 관련 문서

- 인덱스 → `DOC-MAP.md` (모든 문서 위계 + 변경 영향 매트릭스)
- 위로 → `PRD.md` F-02 (미션 시스템 요구사항)
- 옆 도메인 → `DESIGN.md` § 3-namespace 분리 — chosen / chosenWrites / chosenExtras
- 동선 연계 → `ROUTE.md` § afterMission — 동선 mid-stop 위치 결정에 미션 ID 활용
- 운영 룰 → `CLAUDE.md` § state 구조 — 전체 localStorage 스키마
- 검증 → `VERIFICATION.md` — 회귀 검증 항목 [G] 9개 미션 카운트
- 회고 → `learning.md` § 3 미션 시스템
