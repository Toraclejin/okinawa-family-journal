# Okinawa Journal — Claude Code Guide

가족 공유용 여행 매거진 (오키나와 3박 4일). 단일 정적 HTML, 가족들이 여행 전/중/후 읽고 기록하는 용도.

## 스택

- 순수 vanilla HTML + CSS + JavaScript (프레임워크·빌드 도구 **없음**)
- 단일 파일 `index.html`
- 상태 관리: `localStorage` (JSON, key = `okinawa-journal-v1`)
- 호스팅: GitHub Pages (`main` 브랜치 push → 자동 배포, 1~2분)
- 외부 의존: Google Fonts (Archivo Black / Fraunces / Space Grotesk), Pretendard CDN, Unsplash 이미지

## 파일 구조

```
index.html          단일 파일 (HTML + CSS + JS 한 덩어리)
TODO.md             미완료 작업 백로그
learning.md         개발 과정 회고 (사용자용)
CLAUDE.md           이 파일 (Claude 세션용 가이드)
.claude/launch.json npx serve 로 로컬 프리뷰 (포트 8765)
```

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

## state 구조 (localStorage)

```js
{
  ratings: { 'd1-icecream': 3, ... },        // 별점 (0-5)
  texts: { 'traveler-name': '...', ... },    // 입력 필드
  packing: { 'pack-item-0': true, ... },     // 체크박스
  photos: { 'day1': [{id, dataUrl}, ...] },  // 사진 (base64)
  chosen: { 'd1-ts0': ['0', '2'], ... },     // 옵션 선택 (복수 가능, 문자열 idx 배열)
  extraOptions: { ... }                      // 사용자 추가 옵션
}
```

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

## 로컬 프리뷰

`.claude/launch.json` 이 `npx serve -l 8765 .` 실행. Claude Preview 도구에서 `preview_start` → `okinawa-journal`.

## 확인 없이 절대 하면 안 되는 것

- `git push --force` / `git reset --hard` (명시 요청 없이)
- 전체 섹션 삭제 (거의 쓰지 않는 것처럼 보여도 다른 링크가 걸려있을 수 있음)
- CSS 변수 값 변경 (레거시 alias 까지 추적 필요 — 예: `--coral` → `--blue` 통일 때 가독성 붕괴된 전례)
