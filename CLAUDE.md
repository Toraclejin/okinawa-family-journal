# Okinawa Journal — Claude Code Guide

가족 공유용 여행 매거진 (오키나와 3박 4일). 단일 정적 HTML, 가족들이 여행 전/중/후 읽고 기록하는 용도.

## 여행 개요

- **일정**: 2026-05-10 ~ 05-13 (3박 4일)
- **항공**: 도착 2026-05-10 11:05 / 귀국 2026-05-13 오전, 공항 도착 08:30 목표
- **숙소 이동 있음** — 북부 2박 → 차탄 1박:
  - D1 (5/10) ~ D2 (5/11): Hotel Orion Motobu Resort & Spa (북부 비세)
  - D3 (5/12): Vessel Hotel Campana Okinawa (차탄 아메리칸 빌리지)
- **렌터카 필수** — 북부는 대중교통 거의 불가
- **매거진 확정 전제** — 출발 후 콘텐츠 수정 불가. 모든 장소·시간·링크는 출발 전 이중 검증.

## 스택

- 순수 vanilla HTML + CSS + JavaScript (프레임워크·빌드 도구 **없음**)
- 단일 파일 `index.html`
- 상태 관리: `localStorage` (JSON, key = `okinawa-journal-v1`)
- 호스팅: GitHub Pages (`main` 브랜치 push → 자동 배포, 1~2분)
- 외부 의존: Google Fonts (Archivo Black / Fraunces / Space Grotesk), Pretendard CDN, Unsplash 이미지

## 파일 구조

```
index.html          단일 파일 (HTML + CSS + JS 한 덩어리)
japan-map.svg       마무리 페이지 일본 지도 누끼 (Ozean 제거 버전)

DOC-MAP.md          ★ 마스터 인덱스 — 모든 문서 + 참조 관계 + 변경 영향 매트릭스
DESIGN.md           아키텍처·상태 모델·디자인 토큰·페이지 구조·Auth·Export/Import
ROUTE.md            구글맵 동선 시스템 (DAY_HOTEL, midHotel, computeMidInsertIdx, 3-call-sites rule)
MISSIONS.md         미션 시스템 (9개 + 자동 번호 + stamp grid + trip-conquer-card)
VERIFICATION.md     회귀 검증 절차 (자동 console eval + 수동 모바일 체크)
TODO.md             미완료 작업 백로그
learning.md         개발 과정 회고 (사용자용)
CLAUDE.md           이 파일 (Claude 세션용 가이드 — 빠른 참조)

.claude/launch.json npx serve 로 로컬 프리뷰 (포트 8765)
```

**문서 룰**:
- **코드 수정 전** → `DOC-MAP.md` 변경 영향 매트릭스 확인 → 어떤 *.md 같이 봐야 하는지 파악
- **코드 수정 후** → 영향 받은 *.md 같이 갱신 (스토리·표·예시 코드)
- **새 문서 추가 시** → `DOC-MAP.md` 표 + 다이어그램에 등록 (안 등록하면 미아)
- 새 미션·새 픽·새 페이지·새 state 필드 → 각 영역별 체크리스트 (`MISSIONS.md`, `ROUTE.md`, `DESIGN.md`)

## index.html 레이아웃 (상단→하단)

1. `<style>` (L~2700) — 모든 CSS 인라인
2. 상단 네비 (`.page-nav`) + 좌우 화살표 (`.page-arrow`)
3. 툴바 (`.toolbar`) — 맨 위/아래, import/export/print/help
4. 구글맵 모달 + Info 모달
5. `<main class="sheet">` 내부에 7개 `.page` 컨테이너
6. `<script>` (L~4200부터) — 상태·네비·지도·선택·애니메이션 로직

## 7개 페이지 (data-page 값)

1. `cover` — 표지 + 목차
2. `prep` — Packing list + Things I Want (여행 전)
3. `d1` — Day 1 Arrival
4. `d2` — Day 2 Whale Shark
5. `d3` — Day 3 Chatan
6. `d4` — Day 4 Farewell
7. `back` — Enjoy the sea (마무리)

PAGES 배열(`PAGE_IDS`)에 순서대로 정의. 네비/좌우화살표/하단 페이지네이션 모두 이걸 참조.

## 주요 인터랙션

- **페이지 전환**: `.page.active` 만 `display: block`, 나머지 hidden. 전환 시 페이드+슬라이드.
- **URL 해시 연동**: `/#d2` 로 직접 접근 가능, 브라우저 뒤로/앞으로 지원.
- **localStorage 지속**: 마지막 페이지, 별점, 선택된 옵션, 체크박스, 메모, 업로드한 사진까지.
- **장소 → 구글맵**: `.opt-place` 또는 `.mood-card-caption` 클릭 → 확인 모달 → Google Maps 딥링크 (API 키 불필요).
- **옵션 선택**: `.opt` 우측 상단 체크 버튼 → 복수 선택 가능, 같은 슬롯 내 미선택 옵션은 반투명.
- **더보기/접기**: `.opt-body` 모바일 한정, 3줄 초과 시 JS가 버튼 자동 주입.
- **스크롤 리빌**: IntersectionObserver로 `.reveal` 요소 페이드+슬라이드 등장.

## PLACE_QUERIES (장소→구글맵 검색어 매핑)

JS 안 `PLACE_QUERIES` 배열. `[['키 substring', 'Google Maps 정확한 검색어']]` 형태.

**장소 추가 순서**:
1. `.opt` 카드 HTML 작성 (title, place, body 등)
2. `PLACE_QUERIES`에 `['브랜드명 또는 고유 키워드', 'Exact Search Query']` 추가
3. `opt-title`이 브랜드명을 포함하면 자동 매칭됨 (텍스트 앞쪽 위치 기준)

**매칭 규칙**: 텍스트 내 등장 위치가 빠른 키 우선, 동점이면 긴 키 우선. `SKIP_MAP_PATTERNS`에 매칭되면 모달 자체를 안 염 (예: "돌아오는 비행기", "창가 자리").

**다국어 키 전략** — 매칭은 lowercase 대소문자 무관이지만 한국어/일본어/영어는 별개 문자열이므로, opt-place 텍스트에 나올 수 있는 **모든 표기**를 키로 등록해야 함:
- 공식 일본어 표기: 예 `'備瀬のフクギ並木'`, `'万座毛'`, `'今帰仁城跡'`
- 한국어 음차: 예 `'비세 후쿠기'`, `'만자모'`, `'나키진'`
- 영어 표기: 예 `'fukugi'`, `'manzamo'`, `'nakijin'`

옵션 중 하나라도 빠지면 해당 표기로 쓴 opt-place에서 맵 모달이 안 열림. 신규 장소 추가 후 반드시 각 표기로 매칭 테스트.

## 장소 팩트 검증 프로세스 (필수)

"현지 수정 불가" 전제 때문에 신규 장소 도입·기존 장소 수정 시 **무조건 공식 출처 재확인**. 팩트가 틀리면 가족이 엉뚱한 곳으로 가거나 문 닫힌 식당 앞에서 당황함.

**검증 항목 (각각 공식 URL 확인)**
1. 정식 명칭 (한/일/영 세 버전)
2. 주소 (번지까지)
3. 영업시간 — 여행 날짜 요일 기준 (예: 수요일 휴무 가게를 수요일에 가라고 쓰지 말 것)
4. 계절 운영 (수영 시즌, 풀 시즌, 꽃 시기)
5. 요금 — 최신 연도 기준
6. 호텔 기준 거리 (도보/차로 몇 분) — Google Maps 경로로 직접 측정
7. Google Maps 검색 쿼리 — 실제로 Google Maps에서 쳐서 정확한 핀이 나오는지 확인

**출처 강도 순**
- 🟢 공식 홈페이지 (hotel, restaurant, park 공식) — 우선
- 🟡 Tabelog, Tripadvisor (복수 교차 확인 시)
- 🔴 개인 블로그 — 보조만, 단독 근거 ❌

**불확실 시 표기 규칙**
- 공식 사이트가 다운돼서 못 본 항목 → opt-body에 `⚠️ 공식 사이트 재확인 권장` 명시
- 요금이 복수 출처에서 다르면 보수적(비싼 쪽)으로 쓰고 "변동 가능" 멘트
- 절대 "보수적으로 추정"하지 말 것 — 모르면 옵션에서 빼기

**대표 검증 실패 사례 (2026-04)**
- Blue Seal Ice Park: 2024년 폐업됐는데 코드에 "공장 견학" 멘트 남아있음 → 제거
- Blue Seal 영업시간: 실제 10:00~22:00인데 09:00~24:00으로 잘못 표기 → 수정
- "Orion Beer Bar"라는 venue: 공식 사이트에 없음 → 제거
- 물소수레: 토/일만 운영인데 평일 옵션으로 들어감 → 요일 확인 후 반영
- 추라우미 "After 4pm 할인": 2023년 3월 종료됐는데 코드에 잔존 → 제거
- 海邦丸 좌표 [26.6833, 127.8700] (북쪽 3km off): 실제 ハナサキマルシェ 안 [26.6552, 127.8848]로 수정
- **許田IC 沖縄自動車道 좌표 [26.5878, 127.9558] (서쪽 바다 위)**: 道の駅許田 인근 [26.5783, 127.9786]로 수정. 좌표 검증 룰:
  - 신규 좌표 입력 후 반드시 `https://www.google.com/maps?q=lat,lng` 로 직접 열어 핀이 land 위에 있는지 확인
  - 비교 기준: 인근 검증된 장소 (e.g., 海邦丸 26.6552 vs Orion 호텔 26.6917)와 같은 위도대인지

## state 구조 (localStorage)

```js
{
  ratings: { 'd1-icecream': 3, ... },         // 별점 (0-5)
  texts: { 'traveler-name': '...', ... },     // 입력 필드
  packing: { 'pack-item-0': true, ... },      // 체크박스
  photos: { 'day1': [{id, dataUrl}, ...] },   // 사진 (base64)
  chosen: { 'd1-ts0': ['0', '2'], ... },      // 사전 .opt 선택 (복수, 문자열 idx 배열)
  chosenWrites: { 'd1-ts0': true, ... },      // placeholder ✓ (slotKey 기반)
  chosenExtras: { 'abc123': true, ... },      // 동적 .opt-write.extra ✓ (extraId 기반)
  extras: { 'd1-ts0': ['abc123'], ... },      // 동적 추가된 카드 list (slot별)
}
```

### 3-namespace 동시 점검 룰 (필수)
선택 상태는 **3개 분리 namespace**로 관리됨:
- `state.chosen` — 사전 정의 옵션 (.opt)
- `state.chosenWrites` — placeholder (.opt-write:not(.extra))
- `state.chosenExtras` — 사용자 동적 추가 (.opt-write.extra)

**한 곳을 건드리는 변경은 다른 두 곳도 영향 받는지 확인**. 영향 받는 6개 코드 위치:
1. `clearBtn` (오늘 선택 초기화)
2. `undoBtn` (되돌리기)
3. `getDayPicks` / `getDayExtras` (Today's Picks 데이터)
4. `assignPickNumbers` (옵션 카드 좌상단 번호)
5. 평점·chip·지도 마커 렌더링
6. `sanitizeImportedState` (import 검증)

체크리스트 누락 시 발생한 회귀: 2026-04-25 placeholder/extras가 초기화로 안 지워짐.

## 번호 시스템 — 두 개 분리 네임스페이스 (필수 룰)

지도 chip·옵션 카드 번호 배지·Today's Picks 평점 시퀀스는 **두 개 분리된 네임스페이스**:

| 네임스페이스 | 멤버 | 표기 | 위치 |
|---|---|---|---|
| **지도** | Start, [Mid], 사전 .opt 픽, End | `1, 2, 3, ...` | chip + Leaflet 마커 + .opt 카드 좌상단 |
| **추가옵션** | placeholder, 동적 extras | `+1, +2, +3, ...` | Today's Picks 평점 + .opt-write 카드 좌상단 |

**왜 분리하나**: 둘 다 `1, 2, 3...` 쓰면 "End chip = 3" 과 "TEST extras = 3" 이 충돌해 의미 모호. 사용자가 "extras도 지도에 들어가는 픽" 으로 오해.

**구현 위치** (`index.html` 검색):
- `assignPickNumbers()` — `n` (지도) + `extraN` (추가옵션) 두 카운터
- `buildRatingRow(num, ...)` — picks=숫자, extras=`'+' + i`
- CSS `.opt-write[data-pick-num]::before` — 점선 + 흰 배경 (지도 픽은 실선 + 파랑)

회귀 사례 (2026-04-27): 두 네임스페이스가 합쳐져 있어 #3 = End chip = TEST extra 충돌.

## 디자인 토큰 (CSS 변수)

- `--frame` 민트 #7ce0d3 (페이지 바깥 프레임)
- `--page` 흰색 / `--page-tint` 연블루 틴트
- `--blue` #0047ff (메인 코발트, 헤드라인·포인트)
- `--blue-deep` #001f66 (본문), `--blue-ink` #000a33 (최강조)
- `--amber` #ffc400 (별점·액센트), `--accent-soft` #ffeba3
- `--coral` / `--lime` = 레거시, 각각 blue/amber로 통일됨

폰트:
- Archivo Black — 대형 디스플레이 (OKINAWA, DAY ONE)
- Fraunces — 이탤릭 세리프 (강조·부제)
- Space Grotesk — 영문 UI (라벨·버튼)
- Pretendard — 한글 본문·네비

## 자주 하는 작업

### 장소 추가

1. 옵션이 들어갈 `.options` 블록 안에 `.opt` 카드 복사·수정
2. `opt-title` / `opt-place` / `opt-body` / `opt-menu` 또는 `opt-features` / `opt-rating` 채우기
3. `opt-rating` 의 `data-key` 는 고유값으로 (예: `d2-kishimoto`)
4. `PLACE_QUERIES`에 검색어 매핑 추가

### 페이지 추가

1. `<main class="sheet">` 안에 `<div class="page" data-page="신규ID"> ... </div>` 삽입
2. 상단 네비 `<nav class="page-nav">` 안에 버튼 추가
3. JS 의 `PAGES` 배열에 `{id, label, short}` 추가

### 새로운 여행지 버전 만들기

1. 전체 복사 → 새 저장소
2. 색상 토큰, 커버 타이틀, Day 구조, 장소 데이터 교체
3. `PLACE_QUERIES` 지역에 맞게 재작성
4. GitHub Pages 배포

## 보안 (Import 검증)

localStorage 기반 정적 HTML의 유일한 실질 공격 경로는 **악성 JSON을 Import 버튼으로 가져오게 하는 것**. 다음 4가지 방어선이 index.html에 들어있음 — **신규 버전 복제 시 반드시 그대로 옮길 것**.

1. **`sanitizeImportedState()`** (L~5295) — import된 JSON을 화이트리스트 방식으로 재구성:
   - 허용된 최상위 키(`ratings`, `texts`, `packing`, `photos`, `chosen`, `extras`, `extraOptions`)만 통과
   - `__proto__` / `constructor` / `prototype` 키 차단 (프로토타입 오염 방지)
   - 각 값의 타입·길이·포맷 검증
   - 결과는 `Object.create(null)` 기반

2. **`DATA_IMG` 정규식** — 사진 dataUrl은 `^data:image/(jpeg|jpg|png|webp|gif);base64,...$` 만 허용, 5MB 상한.

3. **사진 DOM은 createElement** (`makePhotoItem` L~5589) — `innerHTML` 대신 `img.src = dataUrl` 로 속성 세터 사용. 속성 탈출 원천 차단.

4. **extraId 가드** (`createExtraCard` L~5518) — 영숫자 외 문자열 거부. localStorage 직접 변조 대비 심층 방어.

**새 state 필드 추가 시 규칙**
- sanitizer에 **대응 블록 추가 필수** (누락되면 해당 필드가 import 경로에서 통째로 사라짐)
- DOM 주입 시 `innerHTML` 템플릿 안에 `${변수}` 보간은 **정적 값만**. 사용자·저장소 유래 값은 `createElement + .textContent / .setAttribute / .src=` 로 써야 함
- 이미지 업로드 추가 시 `file.type.startsWith('image/')` 가드 유지

**5. CSP meta 태그 (추가 완료)** — `<head>`에 `Content-Security-Policy` + `X-Content-Type-Options: nosniff` + `Referrer-Policy: strict-origin-when-cross-origin` 추가. default-src self, script/style self + Google Fonts·jsDelivr 허용, img self + data: + unsplash, frame/object 금지. Google Fonts·Pretendard CDN 모두 정상 동작 확인.

**의도적으로 생략한 것** (새 버전에서도 굳이 추가 안 해도 됨)
- CDN SRI 해시 — Google Fonts URL이 브라우저별 동적이라 적용 난이도 ↑
- `.opt-body` / `makeExtraCardHTML` / `nav.innerHTML` — 저자 하드코딩 템플릿, 현재 안전

**CSP가 허용한 것**
- `style-src 'unsafe-inline'` — 페이지 내 `<style>` 덩어리가 있어서 필수
- `script-src 'unsafe-inline'` — 페이지 내 `<script>` 덩어리가 있어서 필수. nonce/hash 적용은 빌드 툴이 없어 현실성 낮음
- `img-src data: blob:` — 사진 업로드가 dataURL로 저장되므로 필수

## 로컬 프리뷰

`.claude/launch.json` 이 `npx serve -l 8765 .` 실행. Claude Preview 도구에서 `preview_start` → `okinawa-journal`.

## 확인 없이 절대 하면 안 되는 것

- `git push --force` / `git reset --hard` (명시 요청 없이)
- 전체 섹션 삭제 (거의 쓰지 않는 것처럼 보여도 다른 링크가 걸려있을 수 있음)
- CSS 변수 값 변경 (레거시 alias 까지 추적 필요 — 예: `--coral` → `--blue` 통일 때 가독성 붕괴된 전례)
