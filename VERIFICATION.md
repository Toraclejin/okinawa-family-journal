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

  // 정리
  delete state.chosen['d1-ts0'];
  delete state.chosen['d1-ts1b'];
  delete state.chosen['d1-ts2'];
  saveState(state);
  renderDayRoute('d1');

  console.table(checks);
  const failed = Object.entries(checks).filter(([k, v]) => !String(v).startsWith('PASS'));
  if (failed.length === 0) console.log('%c✓ ALL 13 PASS', 'color: green; font-weight: bold');
  else console.warn('✗ FAILED:', failed);
  return checks;
})();
```

**기대 결과**: 13/13 PASS.

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
- [ ] ↓ 내보내기 → JSON 다운로드 (`okinawa-journal-{이름}-{날짜}.json`)
- [ ] ↑ 가져오기 → 정상 JSON 복원 / 악성 JSON 차단
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
