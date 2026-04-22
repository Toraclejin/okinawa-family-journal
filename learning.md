# Learning — Okinawa Journal 개발 회고

이 프로젝트를 만들며 알게 된 것들. 다음에 비슷한 매거진·가족 공유 정적 사이트 만들 때 같은 실수 피하기용.

## 시작 지점

- 단일 HTML 파일(약 4500줄)로 이미 짜여 있던 매거진 레이아웃을 받음.
- 목표: GitHub Pages 배포 → 가족이 여행 전·중·후 같이 보면서 별점·선택·메모·사진 기록.
- 처음엔 "그냥 올리기만 하면 되겠네" 였지만, 모바일로 열어보면서 문제가 쏟아졌음.

## 의사결정 로그 (시간순)

1. **퍼블릭 저장소 + noreply 이메일** — 가족 공유용이라 로그인 없이 바로 접속 가능해야 함. GitHub Pages 무료 플랜이 public 필수. 커밋 이메일은 `{id}+{username}@users.noreply.github.com` 써서 개인 이메일 노출 방지.
2. **이미지 교체** — 원본이 몰디브 오버워터 방갈로였음 (오키나와랑 전혀 무관). Unsplash Okinawa 태그에서 실제 오키나와 사진 찾아서 교체. 백그라운드 이미지로만 쓰여서 `loading="lazy"` 적용 불가 → 나중에 `content-visibility: auto` 로 대체.
3. **페이지 분할** — 초반엔 세로로 쭉 스크롤하는 단일 페이지였는데, 가족들이 "너무 길어서 어디가 어딘지 모르겠다" 피드백. `display:none/block` 토글로 7개 페이지 분할 + 상단 네비 + 좌우 화살표 + 하단 이전/다음 버튼 추가.
4. **지도 연결** — `.opt-place` 및 mood card 클릭 → 구글맵 딥링크. Google Maps Platform API (유료) 아니고 그냥 `?api=1&query=` URL 형식 → 0원, 키 불필요.
5. **옵션 선택** — 처음엔 single-select (라디오)로 만들었는데 "짧게 들르는 곳 복수 선택 가능하면 좋겠다" → multi-select로 변경. state 구조만 문자열 → 배열로 확장, 구 포맷 자동 마이그레이션.

## 모바일에서 배운 것 (최대 시간 소비 구간)

### 1. 하드코딩된 대형 폰트

원본이 데스크탑 우선 설계라 `.cover h1 { font-size: clamp(100px, 19vw, 280px) }` 같은 값이 곳곳에 박혀있음. 아이폰(375px)에선 OKINAWA가 화면 바깥으로 튀어나감.

**교훈**: 매거진 타이포 쓸 때 `clamp()` 의 **하한을 모바일 기준으로** 잡아라. 19vw는 vw는 반응형이지만 min이 100px이라 하한이 커서 모바일에선 항상 100px 이상. 실제론 `clamp(40px, 12vw, 200px)` 정도가 안전.

### 2. `grid-template-columns: 1fr 1fr` 의 함정

`1fr` 은 사실 `minmax(auto, 1fr)` 과 같아서, 자식 콘텐츠의 min-content 크기만큼 컬럼이 부풀어오름. mood card big에 `font-size: 56px` "SUNSET" 텍스트가 있으면 그 min-content가 컬럼을 밀어냄 → 1fr 1fr 인데 컬럼이 2.something:1 비율로 깨짐.

**교훈**: 그리드 컬럼이 예상대로 안 나뉘면 `minmax(0, 1fr)` 로 바꿔봐라. 이거 하나로 99% 해결됨.

### 3. grid-area 는 **직접 grid item** 에만

HTML 구조가:
```html
<div class="mood-grid">
  <div class="mood-card-wrap">  <- 실제 grid item
    <div class="mood-card big"> <- grid-area: big 이 여기 걸려있었음
```

grid-area가 inner 엘리먼트에만 있어서 outer wrap은 자동 흐름으로 배치 → 데스크탑에선 우연히 그럴싸, 모바일에선 완전히 깨짐.

**교훈**: CSS Grid 에서 `grid-area`, `grid-column`, `grid-row` 는 **직접 자식**에만 작동. 래퍼가 있다면 래퍼에 걸어라. `:nth-child()` 로 할당하는 것도 방법.

### 4. 색상 통일화의 역설

프로젝트 초반에 누군가 `--coral: #0047ff` 로 "통일"해뒀음. 디자인 팔레트가 "블루+앰버+민트" 로 단순화된 줄 알았는데, 원본 CSS는 여전히 `var(--coral)` 을 **파란 배경 위에 따로 보여지는 원**으로 쓰고 있어서 → 파랑 배경 + 파란 원 = 원 안 보임. "AT THE CONVEYOR BELT" 앰버 글자 위에 앰버 원 겹쳐서 글자 안 보이는 문제도.

**교훈**: 레거시 별칭으로 색을 통일할 땐 해당 색이 **어떤 배경 위에서** 쓰이는지 전부 확인. 시맨틱(용도별)으로 토큰을 나누면 이런 실수 방지. `--accent-warm`, `--accent-cool`, `--surface-main`, `--surface-contrast` 같이.

### 5. iOS Safari 캐시

iOS의 모든 브라우저(크롬 포함)는 WebKit 엔진 강제 → Safari랑 동일. CSS 캐시 공격적으로 붙잡아서, 새 배포 반영이 안 됨. `<meta http-equiv="Cache-Control" content="no-cache, no-store, must-revalidate">` 로 부분 해결. 사용자한텐 pull-to-refresh 안내.

### 6. 고정 툴바 vs 페이지 콘텐츠 충돌

하단 고정 툴바가 페이지 맨 아래 요소(요약 바, 페이지네이션)를 가림. 처음엔 하단 여백 늘리기로 대응했는데 근본 해결 아님. `IntersectionObserver` 로 페이지네이션이 뷰에 들어오면 툴바 자동 숨김 → 해결.

**교훈**: fixed position UI는 항상 **콘텐츠와 어떻게 간섭하는지** 고려. `IntersectionObserver` 가 가장 우아한 답.

## 콘텐츠 검증의 중요성

처음 지도 링크 만들 때 "Sunset Beach Okinawa" 로 검색어 보냈는데 구글맵이 "SUNSET 호텔" 리턴. 전 세계에 "Sunset Beach" 가 너무 많아서 그냥 비치가 아닌 호텔이 매칭됨.

**교훈**:
1. **검색어는 공식 명칭으로** — "Chatan Park Sunset Beach" (진짜 이름). 구글맵 검색어 결과를 코드에서 바로 확인할 수 있으니 믿지 말고 검증.
2. **"근처 소바집" 같은 모호한 표현 금지** — 구체적 이름 없으면 링크도 무의미. "Kishimoto Shokudo 1905년 창업" 처럼 실제 검증 가능한 상호로.
3. **주소는 건물명 + 블록번호까지**. "Depot Island Seaside 3F" vs "미하마 9-21 depoisland seaside 3F". 후자가 정확.
4. 장소 정보 작성 후 **공식 사이트 최소 한 번 재확인**. 예: Vessel Hotel Campana 루프탑 풀 존재/요금/운영기간 — 사용자가 의심했을 때 공식 사이트에서 바로 확인 가능했음.

## 아키텍처 결정: "왜 React 안 썼냐"

SPA 프레임워크(React/Next) 쓰면 좋았을까? **아니, 이 규모엔 과함**.

| 요건 | vanilla | React |
|---|---|---|
| 첫 로드 후 오프라인 작동 (호텔 Wi-Fi 불안정) | ✓ 한 파일 캐시 | 빌드된 번들 + 런타임 |
| 가족이 수정 실수로 깨뜨릴 여지 | 거의 없음 | package.json 건드리면 끝 |
| 빌드 단계 | 없음 (즉시 반영) | 있음 |
| 개발 시간 | 짧음 | 초기 세팅 비용 |
| 확장성 (여러 여행지로) | 템플릿 복사 | 컴포넌트화 |

규모가 "여행지 10개 이상 + 공용 컴포넌트 재활용" 되면 React 가 이득. 그 전엔 vanilla 가 압승.

## 코드 구조 교훈

- **단일 HTML은 4500줄까지도 버틸 만함**. 섹션별로 주석으로 구분해두면 네비게이션 가능. 다만 5000줄 넘으면 CSS만이라도 별도 파일로 분리 고려.
- **상태는 localStorage JSON 하나로 충분**. Redux, Zustand 같은 거 필요 없음. `saveState(state)` 함수 하나만 있으면 됨.
- **기능 추가 시 JS 마지막에 붙이기**. IIFE 안 쓰고 그냥 `</script>` 직전에 새 기능 append. 의존성 순서만 맞으면 됨.

## 다음 프로젝트에서 다르게 할 것

1. **처음부터 모바일 뷰포트로 테스트** — 데스크탑에서 만들고 마지막에 모바일 확인하면 패치가 패치를 부름.
2. **색상 토큰 **의미**로 나누기** — `--blue-500`, `--amber-500` 이런 것보다 `--surface-primary`, `--text-emphasis`, `--accent-warm` 식으로.
3. **장소 데이터를 JS 객체로 분리** — 지금은 HTML 안에 하드코딩돼있어서 장소 수정할 때 HTML + JS 두 군데 손봐야 함. 다음 버전은 `PLACES` 객체 하나에 모두 넣고 HTML 은 JS 로 렌더.
4. **Cloudflare Pages 도 고려** — GitHub Pages 는 해시 베이스 URL, Pages 는 Cloudflare Worker로 동적 라우팅 가능. 여전히 정적 사이트 무료.
5. **프리뷰 테스트를 더 일찍** — Claude Preview 로 375×812 모바일 뷰 확인하면 배포 전 발견 가능. 기본 습관화.

## 사용자가 가장 많이 말한 피드백

> "모바일로 다 잘리는데?"

→ 이게 반복될 때마다 "아, 내가 또 모바일 확인 안 했구나" 깨달음. **모든 UI 변경 후 모바일 스크린샷** 찍는 걸 루틴으로.

> "이거 정확한 거 맞아?"

→ 장소 정보 검증 필수. Tripadvisor/공식사이트 최소 한 번은 열어본다.

> "너무 길어"

→ 모바일에서 긴 본문은 자동 접기 + 더보기 버튼. 3줄 초과는 감춰라.
