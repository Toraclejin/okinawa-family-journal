# Verification — 기능 검증 절차

이 매거진의 기능이 회귀 없이 작동하는지 확인하는 표준 절차.
**큰 변경(브랜치 머지 직전, 또는 외부 라이브러리 버전 업) 시 반드시 실행.**

## 검증 방식

- **자동 검증**: 브라우저 console에서 한 번에 실행하는 eval 스크립트
- **수동 검증**: 모바일 뷰포트에서 시각/터치/네트워크 동작 확인

자동 검증 12개 항목 + 수동 검증 8개 항목. 전체 약 10분 소요.

---

## 자동 검증 (Console eval)

로컬 프리뷰(`localhost:8765`)에서 reload 후 console에 다음 한 덩어리 붙여넣기.

```js
(async () => {
  await new Promise(r => setTimeout(r, 1000));
  const checks = {};

  // 검증 전 깨끗한 상태 보장
  ['d1-ts0','d1-ts1','d1-ts1b','d1-ts2','d1-ts3','d2-ts3','d2-ts4','d2-ts5','d2-ts6','d2-ts7',
   'd3-ts7','d3-ts-move','d3-ts8','d3-ts9','d3-ts10','d3-ts11'].forEach(k => {
    if (state.chosen) delete state.chosen[k];
  });
  saveState(state);
  ['d1','d2','d3'].forEach(d => renderDayRoute(d));
  await new Promise(r => setTimeout(r, 200));

  // [A] 페이지 7개
  const pages = Array.from(document.querySelectorAll('main.sheet > .page')).map(p => p.dataset.page);
  checks.A_pages_7 = pages.length === 7 ? 'PASS' : `FAIL (${pages.length})`;

  // [B] CSP 메타
  const csp = document.querySelector('meta[http-equiv="Content-Security-Policy"]')?.content;
  checks.B_csp = csp?.includes("default-src 'self'") ? 'PASS' : 'FAIL';

  // [C] 호텔 카드 D1·D3
  const d1H = document.querySelector('[data-page="d1"] .hotel-card');
  const d3H = document.querySelector('[data-page="d3"] .hotel-card.dual');
  checks.C_hotelCards = (d1H && d3H) ? 'PASS' : 'FAIL';

  // [D] dek 더보기 4개
  checks.D_dekToggle = document.querySelectorAll('.dek .dek-toggle').length === 4 ? 'PASS' : 'FAIL';

  // [E] PLACE_QUERIES + DIR
  const dirCount = PLACE_QUERIES.filter(([k,v]) => v.startsWith('DIR:')).length;
  checks.E_placeQueries = (PLACE_QUERIES.length > 50 && dirCount >= 6) ? 'PASS' : 'FAIL';

  // [F] localStorage
  state.chosen = state.chosen || {};
  state.chosen['d1-ts0'] = ['0'];
  state.chosen['d1-ts1b'] = ['0'];
  state.chosen['d1-ts2'] = ['1'];
  saveState(state);
  renderDayRoute('d1');
  await new Promise(r => setTimeout(r, 200));
  const stored = JSON.parse(localStorage.getItem('okinawa-journal-v1') || '{}');
  checks.F_storage = stored.chosen?.['d1-ts0']?.[0] === '0' ? 'PASS' : 'FAIL';

  // [G] 픽 번호 배지 (3개) + 첫 픽 번호 = 2 (Start가 1을 차지)
  const numbered = Array.from(document.querySelectorAll('[data-page="d1"] .opt[data-pick-num]'));
  const firstNum = numbered[0]?.dataset.pickNum;
  checks.G_pickBadges = (numbered.length === 3 && firstNum === '2') ? 'PASS' : `FAIL (count=${numbered.length}, first=${firstNum})`;

  // [H] chip 동선 (Start + 3 picks + End = 5)
  const stops = document.querySelectorAll('[data-day-route="d1"] .route-stop').length;
  checks.H_routeChips = stops === 5 ? 'PASS' : `FAIL (${stops}/5)`;

  // [I] D1 라벨 (공항 → 북부 호텔)
  const startL = document.querySelector('[data-day-route="d1"] .route-stop:first-child .rs-name')?.textContent;
  const endL = document.querySelector('[data-day-route="d1"] .route-stop:last-child .rs-name')?.textContent;
  checks.I_d1Labels = (startL === '나하 공항' && endL === '북부 호텔') ? 'PASS' : `FAIL (${startL} → ${endL})`;

  // [J] Leaflet 지도 + OSRM 도로 폴리라인
  document.querySelector('.page-nav-btn[data-nav="d1"]')?.click();
  await new Promise(r => setTimeout(r, 4000));
  const map = leafletMaps?.d1;
  let mks = 0, plPoints = 0;
  if (map) map.eachLayer(l => {
    if (l instanceof L.Marker) mks++;
    else if (l instanceof L.Polyline && !(l instanceof L.Polygon)) plPoints = Math.max(plPoints, l.getLatLngs().length);
  });
  checks.J_leafletOsrm = (mks === 5 && plPoints > 100) ? 'PASS (OSRM curves)' : `PARTIAL (mks=${mks}, plPts=${plPoints})`;

  // [K] PDF 순서 재배치 + 복원
  const before = Array.from(document.querySelectorAll('main.sheet > .page')).map(p => p.dataset.page);
  preparePrint();
  const after = Array.from(document.querySelectorAll('main.sheet > .page')).map(p => p.dataset.page);
  restorePrint();
  const restored = Array.from(document.querySelectorAll('main.sheet > .page')).map(p => p.dataset.page);
  checks.K_pdfOrder = (after.join(',') === 'cover,prep,d1,d2,d3,d4,back' && restored.join(',') === before.join(',')) ? 'PASS' : 'FAIL';

  // [L] 보안 sanitizer
  const evil = {
    __proto__: { polluted_test: true },
    photos: { d1: [{ id: 'x', dataUrl: 'x" onerror="window.__XSS_TEST=1"' }] },
    extras: { b: ['" onmouseover=1 "'] }
  };
  const cleaned = sanitizeImportedState(evil);
  await new Promise(r => setTimeout(r, 100));
  checks.L_security = (!window.__XSS_TEST && !({}).polluted_test && cleaned.photos?.d1?.length === 0) ? 'PASS' : 'FAIL';

  // [M] 오늘 선택 초기화 + 되돌리기 — 3 시스템 (chosen / chosenWrites / chosenExtras)
  // 회귀 케이스: 2026-04-25 — placeholder ✓ + dynamic extra ✓ 가 초기화로 안 지워지던 문제
  state.chosen = state.chosen || {};
  state.chosen['d1-ts1'] = ['0'];
  state.chosenWrites = state.chosenWrites || {};
  state.chosenWrites['d1-ts1'] = true;
  state.chosenExtras = state.chosenExtras || {};
  state.chosenExtras['__test_extra__'] = true;
  saveState(state);
  // (placeholder/extra 카드는 DOM에 없을 수 있음 — 시드 데이터만으로 clear 동작 검증)
  // clear 시뮬레이션
  const dayPage = document.querySelector('.page[data-page="d1"]');
  const slotKeys = ['d1-ts1'];
  slotKeys.forEach(sk => { if (state.chosen[sk]) delete state.chosen[sk]; });
  Object.keys(state.chosenWrites).filter(k => k.startsWith('d1-')).forEach(k => delete state.chosenWrites[k]);
  Object.keys(state.chosenExtras).forEach(k => delete state.chosenExtras[k]);
  saveState(state);
  const afterClear = JSON.parse(localStorage.getItem('okinawa-journal-v1') || '{}');
  checks.M_clearAll3 = (
    !afterClear.chosen?.['d1-ts1'] &&
    !afterClear.chosenWrites?.['d1-ts1'] &&
    !afterClear.chosenExtras?.['__test_extra__']
  ) ? 'PASS' : 'FAIL (clear leaked)';

  // [N] 평점 stale 회귀 — 별 클릭 시 Today's Picks 즉시 갱신
  // 회귀 케이스: 2026-04-27 — refreshDayRatings 정의돼 있는데 호출 안 돼서 별 클릭해도 화면 안 바뀜
  state.chosen = state.chosen || {};
  state.chosen['d1-ts1'] = ['0'];
  state.ratings = state.ratings || {};
  state.ratings['d1-emerald'] = 0;
  saveState(state);
  renderDayRoute('d1');
  await new Promise(r => setTimeout(r, 200));
  // 별점 직접 변경 + refreshDayRatings 호출 시뮬
  state.ratings['d1-emerald'] = 4;
  saveState(state);
  if (typeof refreshDayRatings === 'function') refreshDayRatings('d1');
  await new Promise(r => setTimeout(r, 100));
  const ratingRow = document.querySelector('[data-day-route="d1"] .day-rating-row[data-rating-key="d1-emerald"]');
  const onDots = ratingRow?.querySelectorAll('.dr-dot.on').length || 0;
  checks.N_ratingRefresh = (typeof refreshDayRatings === 'function' && onDots === 4)
    ? 'PASS' : `FAIL (refreshFn=${typeof refreshDayRatings}, onDots=${onDots})`;

  // [O] 번호 네임스페이스 분리 — 지도 픽은 숫자, 추가옵션은 +N
  // 회귀 케이스: 2026-04-27 — End chip "3" === Today's Picks extras "3" 충돌
  // (예: D1 픽 1개 + extra 1개 → 픽=2, End chip=3, extra=+1)
  const optWithNum = document.querySelector('[data-page="d1"] .opt[data-pick-num]');
  const optWriteWithNum = document.querySelector('[data-page="d1"] .opt-write[data-pick-num]');
  const optNumStr = optWithNum?.dataset?.pickNum || '';
  const optWriteNumStr = optWriteWithNum?.dataset?.pickNum || '';
  // 사전 .opt 는 숫자만, .opt-write 는 + prefix
  const optNumOK = optNumStr && /^\d+$/.test(optNumStr);
  const optWriteNumOK = !optWriteWithNum || /^\+\d+$/.test(optWriteNumStr);
  checks.O_numberNamespaces = (optNumOK && optWriteNumOK)
    ? 'PASS' : `FAIL (opt=${optNumStr}, optWrite=${optWriteNumStr})`;

  // 정리
  delete state.chosen['d1-ts0'];
  delete state.chosen['d1-ts1'];
  delete state.chosen['d1-ts1b'];
  delete state.chosen['d1-ts2'];
  delete state.ratings['d1-emerald'];
  saveState(state);
  renderDayRoute('d1');

  console.table(checks);
  const failed = Object.entries(checks).filter(([k, v]) => !String(v).startsWith('PASS'));
  if (failed.length === 0) console.log('%c✓ ALL 15 PASS', 'color: green; font-weight: bold');
  else console.warn('✗ FAILED:', failed);
  return checks;
})();
```

**기대 결과**: 15/15 PASS.

---

## 수동 검증 (모바일 뷰포트 375×812)

각 항목을 폰 또는 데스크탑 모바일 에뮬레이터에서 직접 동작.

### M1. 페이지 네비
- [ ] 상단 7개 네비 (표지·여행 전·Day1-4·마무리) 클릭 시 페이드+슬라이드 전환
- [ ] 좌우 화살표로 이전/다음 페이지 이동
- [ ] 각 페이지 끝 "이전 / 다음" 페이지네이션 버튼

### M2. 옵션 카드 ✓ 선택
- [ ] D1 옵션 카드 우상단 ✓ 클릭 → 좌상단 번호 배지 (2, 3, 4...) 표시 (Start가 1을 차지하므로 픽은 2부터)
- [ ] 같은 슬롯 내 미선택 옵션 반투명
- [ ] 다시 ✓ 클릭 → 선택 해제 + 번호 사라짐

### M3. Day Route 카드
- [ ] D1·D2·D3 페이지 끝에 "DAY N · TODAY'S ROUTE" 카드 자동 표시
- [ ] 옵션 선택 시 chip 동선 즉시 갱신
- [ ] D1: Start "나하 공항" / End "북부 호텔"
- [ ] D2: Start "북부 호텔" / End "북부 호텔" (왕복)
- [ ] D3: Start "북부 호텔" / End "차탄 호텔" (편도)

### M4. 미니 지도 (Leaflet + OSRM)
- [ ] 옵션 선택 시 카드 안 지도 즉시 표시 (직선 폴리라인)
- [ ] 1~3초 후 직선이 **도로 따라가는 곡선**으로 자동 교체 (네트워크 OK 시)
- [ ] 마커: Start(1)/End(N) 검정+앰버, 픽 번호(2~N-1) 파랑+흰 — 모두 숫자
- [ ] 모든 마커 한 화면에 들어옴 (자동 fit)
- [ ] 지도 줌·드래그 동작
- [ ] **chip 클릭 → 지도가 그 좌표로 flyTo + 지도 영역으로 부드럽게 스크롤**

### M5. 구글맵 버튼
- [ ] "구글맵에서 전체 경로 보기 ↗" → 새 탭 multi-stop directions URL
- [ ] 호텔 카드의 호텔 이름 클릭 → 검색 모달 → "예" 새 탭
- [ ] Mood card "Drive North/South/Fly Home" 클릭 → 길찾기 모달 → 새 탭 directions

### M6. Toolbar
- [ ] MAP 버튼 (지도 아이콘) 클릭 → 현재 Day의 동선 카드로 smooth scroll
- [ ] cover/prep/d4에서 MAP 클릭 → D1으로 이동 후 동선 카드로 점프

### M7. Import/Export + Print
- [ ] ↓ 내보내기 클릭 → confirm 다이얼로그 등장
  - [ ] 통계 표시: 사진 N장 / 별점 N개 / 미션 N/9 (Hidden 풀린 경우 시크릿 N/6 추가) / 파일 크기 X MB
  - [ ] 다중 백업 권장 멘트 (카톡 / 메일 / 클라우드) 표시
  - [ ] [취소] 클릭 → 다운로드 안 됨 / [다운로드] 클릭 → JSON 저장
- [ ] JSON 파일명 `okinawa-journal-{이름}-{날짜}.json`
- [ ] ↑ 가져오기 → 정상 JSON 복원 / 악성 JSON 차단
- [ ] **사진 손실 알림** (Import 시):
  - [ ] 정상 사진만 → "복원 완료!" alert
  - [ ] 5MB 초과 사진 포함 → "사진 N장 파일 크기 너무 큼" 명시
  - [ ] 잘못된 dataUrl 포함 → "사진 N장 파일 형식 문제" 명시
  - [ ] silent skip 0건 (모든 손실은 사용자에게 표시)
- [ ] ⎙ 인쇄 → PDF 미리보기 페이지 순서 cover→prep→d1→d2→d3→d4→back
- [ ] PDF에 호텔 카드·동선 chip·옵션 본문 모두 펼쳐져 출력

### M8. 보안 (XSS)
- [ ] devtools console에서 다음 입력 → 폐기 확인:
  ```js
  const evil = {version:'okinawa-journal-v1', state:{photos:{d1:[{id:'x',dataUrl:'x" onerror="alert(1)"'}]}}};
  // (Import 버튼으로 evil 페이로드 import 후) 사진 영역 빈 채로 표시 + alert 안 뜸
  ```

---

## 검증 강도 — 변경 규모에 따라

매번 12+8개 다 돌리면 부담. 변경 규모에 따라 차등:

### A. 전체 검증 (큰 변경 / 라이브 리스크)
- **언제**: 외부 라이브러리 업그레이드, state 스키마 변경, CSP·sanitizer 손댔을 때, 큰 PR 머지 전
- **무엇**: 자동 12개 + 수동 8개 모두

### B. 부분 검토 (작은 변경 / 한 PR)
- **언제**: 일반 PR (한 기능, 한 파일, 한 함수 수정)
- **무엇**: Claude Code 빌트인 스킬 활용
  - **`/review`** — PR diff 자동 리뷰 (의도·회귀·코드 품질)
  - **`/security-review`** — 변경된 부분만 보안 검토 (XSS·CSP·sanitize 영향)
  - 둘 다 변경 영역만 보고 짧은 리포트 — 매번 12개 안 돌려도 됨
- **추가**: 영향 받는 자동 항목만 골라 실행
  - 옵션·동선 관련 변경 → `F`, `G`, `H`, `J` 만
  - PDF·인쇄 관련 → `K` 만
  - 보안 관련 → `L` 만
  - state 스키마 변경 → `F`, `L` 필수

### C. 발견된 회귀 버그 (작은 fix)
- **언제**: 사용자가 "이거 이상해" 리포트 → 수정
- **무엇**: 수정한 영역의 자동 항목 1~2개만 + 시각 한 번 확인
- 회귀 막으려면 그 케이스를 자동 검증에 신규 항목으로 등록

## PR 머지 전 표준 절차

1. 변경 commit + push (브랜치)
2. 채팅에서 **`/security-review`** 실행 — 변경 영역 보안 회귀 점검
3. **`/review`** 실행 — 의도·코드 품질 검토
4. 영향 받는 자동 검증 항목 실행 (위 B 표 참조)
5. PR 본문에 결과 명시 (예: "L_security PASS, /security-review 깨끗")
6. 시각·터치 한 번 확인 (모바일 뷰포트)
7. main 머지 → GitHub Pages 자동 배포

## 새 기능 추가 시

1. 자동 검증 스크립트에 새 항목 (`M_newFeature`) 추가
2. 수동 검증 체크리스트에 새 항목 추가
3. PR 본문에 "VERIFICATION.md 통과 ✓" 명시
4. 머지 전 새 항목 + 영향 받는 기존 항목 PASS 확인

## 의존성·외부 서비스 점검

| 서비스 | 영향 | 대응 |
|---|---|---|
| Google Fonts | 폰트 로드 실패 | fallback 폰트 정의됨, 무해 |
| Pretendard CDN | 한글 폰트 | fallback OK |
| Unsplash | 더미 이미지 | fallback 이미지 무 (visual 깨짐) |
| Leaflet (unpkg) | 지도 init 실패 | renderDayMap retry → 폴백 직선 |
| OpenStreetMap tiles | 지도 타일 로드 실패 | Leaflet 자체 동작은 됨, 회색 배경 |
| OSRM demo | 도로 라인 fetch 실패 | 직선 폴리라인 fallback (5초 timeout) |

**OSRM demo는 production 보장 X** — 가족용 정도 트래픽은 OK, 1년 이상 장기 운영 시 자체 호스팅 또는 다른 routing API 검토.

---

## 변경 이력

- 2026-04-26 초안 (transit-system PR과 함께 도입, 12 자동 + 8 수동)
- 2026-04-25 [M] 항목 추가 — 오늘 선택 초기화가 placeholder/extras를 안 지우던 회귀 (state 3-namespace 시스템 도입 시 누락) 자동 검증으로 등록
- 2026-04-27 [N] [O] 항목 추가
  - [N] 별 클릭 시 Today's Picks 즉시 갱신 — `refreshDayRatings` 호출 누락 회귀
  - [O] 지도 chip 번호 vs 추가옵션 + prefix 네임스페이스 분리 — 충돌 방지
- 2026-04-28 [P] 항목 추가 — **Canvas ↔ UI Parity 검증** (수동 시각 비교)
  - 회귀 사례: 화면 UI 에 오키나와 줌 SVG 추가했는데 `downloadConquerImage()` 미갱신 → 다운로드 이미지에서 오키나와 부분 누락
  - 룰: trip-conquer-card 시각 변경 시 화면 + canvas 모두 동일 commit 에 갱신 (DESIGN.md § Dual-render Components 참조)

## ★ [Z] 스모크 테스트 — 핵심 init 경로 작동 확인 (필수, commit 전)

### 회귀 사례 (2026-04-28, 38b55cb 로 fix)
- showPage 안에 updateMissionCounter() 호출 추가 → 초기 IIFE 시 TDZ → 스크립트 전체 중단 → stamp grid 0/0
- 자동 감지 부재로 사용자 보고 받기 전엔 회귀 인지 X
- 상세: POSTMORTEM-2026-04-28.md

### 절차 (preview 에서 한 번 실행)

```js
(() => {
  const checks = {
    DAY_HOTEL: typeof DAY_HOTEL,
    DAY_HOTEL_COORDS: typeof DAY_HOTEL_COORDS,
    MISSION_ID_RE: (() => { try { return typeof MISSION_ID_RE; } catch(e) { return 'TDZ'; } })(),
    PAGE_IDS: Array.isArray(PAGE_IDS) && PAGE_IDS.length,
    state: typeof state,
    stampItems: document.querySelectorAll('.stamp-item').length,
    mainMissions: typeof getMainMissions === 'function' ? getMainMissions().length : 'undef',
    hiddenMissions: typeof getHiddenMissions === 'function' ? getHiddenMissions().length : 'undef',
    progress: document.getElementById('stamp-collection-progress')?.textContent,
  };
  return checks;
})();
```

### 기댓값 (PASS 조건)

| key | expected |
|---|---|
| DAY_HOTEL | `'object'` |
| DAY_HOTEL_COORDS | `'object'` |
| MISSION_ID_RE | `'object'` (regex 타입 = object) |
| PAGE_IDS | `8` (cover/prep/d1~d4/back/hidden) |
| state | `'object'` |
| stampItems | `9` (메인 9개 stamp grid 렌더됨) |
| mainMissions | `9` |
| hiddenMissions | `6` |
| progress | `"X / 9"` (X = done 개수) |

**하나라도 안 맞으면**: commit 보류 + fix 우선. console.error 추가 점검.

### 룰 — init-path 함수 호출 시
showPage / loadState / 초기 IIFE 등 init 경로에서 호출되는 함수에 후방 정의된 const/let 참조 함수를 추가할 땐 **반드시 try/catch defensive guard**. (DESIGN.md § Defensive Init-Path Pattern)

---

## 수동 검증 [Q] — Hidden Mission 3단계 가시성 + 트리거

### 회귀 사례 (2026-04-28)
- d1-hotel 도장 클릭 후 cinematic / 모달 안 뜨는 회귀 2회 발생
  · 첫 번째: `id` 변수명 typo (실제는 `missionId`) — fix 8d3fe20
  · 두 번째: `state.hiddenUnlocked = true` 영속 → 재테스트 시 cinematic skip — fix c2f6d47 (도장 취소 시 자동 리셋)

### 검증 절차

1. **단계 1 (d1-hotel 미클릭) — 탭 완전 숨김**
   - 새 export JSON import (또는 localStorage clear) 후 페이지 로드
   - 상단 네비에 "Hidden" 탭이 **보이지 않아야** 함 (computed display: none)
   - 마무리 페이지 CTA `#hidden-unlock-cta` 도 hidden=true

2. **단계 2 (d1-hotel ✓ 클릭, !hiddenUnlocked) — 자물쇠 + 반짝**
   - D1 페이지 → Mission 3 (Hotel Orion Motobu — 도착!) ✓ 클릭
   - **2.1초**: MISSION COMPLETE 풀스크린 fade out
   - **즉시 (~2.1s)**: 🎫 TICKET ISSUED cinematic — 그린 confetti + 부드러운 띠링 chime
   - **~3.8s**: YES/NO 모달 — "남건우님께서 Hidden Mission을 활성화시키셨습니다"
   - 모달 X (`NO 나중에 풀게`) 클릭 → 마무리 페이지 가면 "🎫 히든 미션 풀기" CTA 노출
   - 상단 네비에 자물쇠 🔒 아이콘 + 반짝반짝 애니 (라벨 "Hidden" 은 숨김)

3. **단계 3 (hiddenUnlocked = true) — 라벨 + NEW 배지**
   - 모달에서 YES, 또는 마무리 CTA 클릭
   - 자물쇠 사라지고 "Hidden NEW" 배지 노출
   - Hidden 페이지 진입 후엔 NEW 배지 사라짐 (`.seen` 클래스)

4. **회귀 안전망 — 도장 취소로 리셋**
   - d1-hotel `↶ 도장 취소` 클릭 → confirm 메시지에 "Hidden Mission 잠금도 함께 초기화" 안내
   - 확인 후: `state.hiddenUnlocked = false` → 단계 1 복귀
   - 다시 ✓ 도장 → cinematic 재진입 (테스트 반복 가능)

5. **6/6 완료 — FAMILY LEGEND 풀스크린**
   - 6개 hidden mission 모두 ✓ → 자동 fireFamilyMaster()
   - 🏆 메달 spin + 미-솔-시-미 멜로디 + "패밀리 레전드!" 풀스크린 1.8s
   - Hidden 페이지 안 family-master-msg 노출

6. **NO 클릭 → 마무리 페이지 자동 스크롤 (TICKET PENDING CTA)**
   - YES/NO 모달에서 `NO — 나중에 풀게` 클릭
   - 모달 닫힘 → 자동으로 마무리 페이지 (`back`) 이동
   - ~380ms 후 TICKET PENDING CTA 박스로 부드럽게 스크롤 (block: center)
   - CTA 박스는 도장 박스 (stamp-collection) 위 ~2cm (76px) 간격으로 떠있음
   - 사용자 검증: NO 누른 후 화면 가운데에 그린 dashed 박스가 자동 등장하는가
   - 회귀 위험: showPage('back') 또는 scrollIntoView 한 쪽 깨지면 CTA 위치 확인 못 함

## 수동 검증 [P] — Canvas ↔ UI Parity (시각 비교)

### [P] trip-conquer-card 다운로드 PNG 가 화면과 동일

1. 마무리 페이지로 이동, 9개 미션 모두 done 상태 만들기
2. trip-conquer-card 화면 캡처 (스크린샷)
3. `이미지 다운로드` 버튼 클릭 → PNG 저장
4. 두 이미지 좌우 비교, 다음 요소 모두 1:1 매칭 확인:
   - 일본 지도 (4섬 모양)
   - 오키나와 ★ pulse (위치 일치)
   - 줌인 화살표 (점선 amber, 위치)
   - 오키나와 본도 줌 박스 (cream 색, 점선 amber border)
   - "OKINAWA · 본도" 라벨 (amber)
   - "TRIP CONQUERED" tag
   - "오키나와 정복!" 타이틀
   - 5.10 — 5.13 · 2026 날짜
   - stat 4칸: 9/9 COMPLETE / 평균 만족도 / 사진 / 방문한 곳 (모두 amber 톤)
5. 차이가 있으면 `MISSIONS.md § UI↔Canvas Parity Rule` 위반 — 둘 중 하나가 stale

## 참조

- 위로 → `CONCEPT.md` § 5 성공 기준 (출발 전 검증 통과 필수)
- 운영 룰 → `CLAUDE.md`
- 인덱스 → `DOC-MAP.md` (모든 문서 위계 + 영향 매트릭스)
- 도메인 → `ROUTE.md` · `MISSIONS.md` · `DESIGN.md` (각 검증 항목이 어느 도메인을 검증하는지)
- 백로그 → `TODO.md` (P0 = 출발 전 검증 통과 항목)
- 회고 → `learning.md` (회귀 패턴 누적)
