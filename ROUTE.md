# Route — 구글맵 동선 시스템

> Day 별 동선 chip · Leaflet 미니 지도 · 호텔 mid-stop 자동 삽입 로직.
> **수정 전 반드시 [3-call-sites consistency rule](#3-call-sites-consistency-rule) 읽어.**

## 큰 그림

각 Day 페이지 안에 `[data-day-route="dN"]` 컨테이너가 있고, 그 안에:
1. **chip 동선** — `Start → [Mid?] → 픽들 → End` 가로 흐름 박스
2. **Leaflet 미니지도** — 실제 좌표 + 폴리라인 (도로 따라 OSRM, fallback 직선)
3. **summary** — `N곳 정거장 · Start → End`
4. **평점 모음** — 픽별 별점 평점 (collapsible)

## 데이터 출처

```
DAY_HOTEL[dayId] = { start, end, startLabel, endLabel, midHotel? }
DAY_HOTEL_COORDS[dayId] = { start: [lat,lng], end: [lat,lng], mid?: [lat,lng] }
PLACE_COORDS[query] = [lat, lng]   // 픽 좌표 매핑
PLACE_QUERIES = [[키, '구글맵 검색어' or 'DIR:...']]  // 텍스트 → 쿼리
```

## midHotel — 호텔 체크인을 동선 사이에 끼우는 메커니즘

이동일(D1, D3) 처럼 **하루 중간에 호텔에 체크인**해야 하는 경우, 호텔 chip 을 픽들 **사이**에 자동 삽입.

### 사용 예

**D1 (공항 → 호텔)**:
```
DAY_HOTEL.d1 = {
  start: 那覇空港, end: ホテル オリオンモトブリゾート,
  midHotel: { tag: 'Check-in', label: 'Orion 호텔', afterMission: 'd1-hotel' }
}
DAY_HOTEL_COORDS.d1 = { start: [공항], end: [Orion], mid: [Orion] }
```

흐름:
```
나하 공항 (Start)
  → Blue Seal (12:30 점심 픽, d1-hotel 미션 카드보다 DOM 뒤이므로 mid 앞에 안 옴)
  → Orion 호텔 (Check-in mid)  ← d1-hotel 미션 카드 위치 기준 자동 삽입
  → 일몰/디너 픽 (mid 뒤)
  → Orion 호텔 (End)
```

**D3 (Orion → Vessel)**:
```
DAY_HOTEL.d3 = {
  start: Orion, end: Vessel,
  midHotel: { tag: 'Check-in', label: 'Vessel 호텔', afterMission: 'd3-vessel' }
}
DAY_HOTEL_COORDS.d3 = { start: [Orion], end: [Vessel], mid: [Vessel] }
```

흐름:
```
Orion (Start)
  → 비세 후쿠기 / Emerald Beach / 美ら海の湯 (08:00 모토부 마지막 아침)
  → Okinawa Expressway / Orion Happy Park (이동 픽)
  → Vessel 호텔 (Check-in mid)  ← d3-vessel 미션 위치 기준
  → American Village / Kurasushi (오후·저녁 차탄 픽)
  → Vessel (End)
```

### 자동 삽입 위치 (`computeMidInsertIdx`)

```js
function computeMidInsertIdx(dayId, afterMissionId) {
  // afterMissionId 미션 카드의 DOM 위치 기준으로
  // 그보다 앞에 있는 .options 블록의 chosen 픽 개수 = midInsertIdx
  // (DIR sentinel 픽은 제외)
}
```

**핵심 규칙**:
- 미션 카드보다 **DOM 상 앞**에 있는 .options 블록의 chosen 픽 → mid 앞에 옴
- 미션 카드보다 **DOM 상 뒤**에 있는 .options 블록의 chosen 픽 → mid 뒤에 옴
- DIR sentinel (`PLACE_QUERIES` 에서 `DIR:` prefix) 픽은 카운트 X

따라서 **DOM 순서가 시간 순서와 일치하게** HTML 작성하는 게 가장 중요.
- `d3-vessel` 미션 카드는 D3 페이지에서 *모토부 모닝/이동 .options* 다음, *차탄 .options* 앞에 위치해야 함.

## 3-call-sites consistency rule

`midInsertIdx` 를 쓰는 함수가 **3 곳**에 있음. 한 곳 고치면 다른 두 곳도 동일 로직이어야 함:

| 함수 | 역할 | 위치 (키워드 검색) |
|---|---|---|
| `renderDayRoute(dayId)` | chip 가로 흐름 박스 | `function renderDayRoute` |
| `renderDayMap(dayId)` | Leaflet 폴리라인 + 마커 | `function renderDayMap` |
| `assignPickNumbers(dayId)` | .opt 카드 좌상단 번호 배지 | `function assignPickNumbers` |

3 곳 모두 같은 결과를 내야 사용자가 `chip 5번` ↔ `옵션 카드 5번` ↔ `지도 마커 5번` 매칭 가능.

회귀 사례 (2026-04-27): D3 mid 가 fixed position 2 였을 때 chip / opt 번호가 어긋남 → DOM-based midInsertIdx 로 통일.

## chip 번호 배정 (renderDayRoute)

```
1: Start (Hotel)
2..(midInsertIdx+1): 픽 (mid 이전)
midInsertIdx+2: Mid (Check-in Hotel)
(midInsertIdx+3)..N: 픽 (mid 이후)
N+1: End (Hotel)
```

만약 midHotel 없으면:
```
1: Start
2..N: 픽
N+1: End
```

만약 midInsertIdx === picks.length (모든 픽이 mid 앞):
```
1: Start, 2..N: 픽, N+1: Mid, N+2: End
```

## opt 카드 번호 배정 (assignPickNumbers)

mid 가 들어가는 자리에 번호 한 칸 비우고 이후 픽은 +1 shift.

```js
let n = 1;  // Start = 1
let pickIdx = 0;
let midPlaced = false;
for (each .opt 카드 with chosen) {
  if (midInsertIdx >= 0 && !midPlaced && pickIdx === midInsertIdx) {
    n += 1;  // mid 자리
    midPlaced = true;
  }
  n += 1;
  opt.dataset.pickNum = n;
  pickIdx += 1;
}
```

## Leaflet 마커 (renderDayMap)

```js
route = [start, ...picksBeforeMid, mid?, ...picksAfterMid, end]
isHotelMarker(i) = (i === 0 || i === route.length-1 || i === 1+midInsertIdx)
```

호텔 마커는 둥근 navy 배경 + amber 숫자, 일반 픽은 작은 navy 점 + 흰 숫자.

## 픽 → 좌표 fallback

```js
getCoords(place, fallbackHotelCoords):
  q = resolveMapQuery(place)
  if (q && PLACE_COORDS[q]) return PLACE_COORDS[q]
  if (PLACE_COORDS[place]) return PLACE_COORDS[place]
  return fallbackHotelCoords  // 호텔 활동(별 보기, 객실 휴식 등) → D1만 end, 나머지 start
```

## 폴리라인 — OSRM + 직선 fallback

1. 직선 polyline 즉시 표시 (대시 패턴, opacity 0.6)
2. `fetchRouteGeometry(route)` 비동기 — OSRM demo 서버에 driving route 요청 (timeout 5s, cache by coordsStr)
3. 성공 → 직선 제거 + 도로 폴리라인 (실선, opacity 0.85)
4. 실패 (네트워크/타임아웃) → 직선 유지

## 확장 시 체크리스트

- [ ] 새 Day 추가 시 `DAY_HOTEL[dayN]`, `DAY_HOTEL_COORDS[dayN]`, `PAGE_IDS` 추가
- [ ] midHotel 추가 시 `afterMission` 으로 어떤 미션 카드 위치 기준인지 명시
- [ ] 미션 카드 DOM 위치 변경 시 → 그날 동선이 의도한 시간 순서와 일치하는지 재검증
- [ ] 새 픽 추가 시 `PLACE_QUERIES` + `PLACE_COORDS` 둘 다 등록
- [ ] DIR sentinel 픽이면 동선 chip 에서 자동 제외됨 — 하지만 평점 모음에는 들어감
- [ ] VERIFICATION.md 자동 검증에 새 Day 시나리오 추가

## 관련 문서
- `DESIGN.md` § State Model — chosen / chosenWrites / chosenExtras 분리 이유
- `MISSIONS.md` — afterMission 으로 참조되는 미션 ID 일람
- `CLAUDE.md` § PLACE_QUERIES 매칭 규칙 — DIR sentinel, lowercase, 다국어 키
- `VERIFICATION.md` — 회귀 검증 항목 [F] [G]
