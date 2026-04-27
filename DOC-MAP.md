# Doc Map — Okinawa Journal

> 이 프로젝트의 문서 구조와 참조 관계.
> **새 문서를 만들면 여기에 추가** — 그래야 서로 끊긴 채로 안 넘어감.

## 문서 일람

| 문서 | 역할 | 누가 보나 |
|---|---|---|
| `CLAUDE.md` | Claude 세션용 가이드 (스택, 파일 구조, 룰, 자주 하는 작업) | Claude 만 |
| `DOC-MAP.md` | **이 문서** — 모든 문서의 마스터 인덱스 | Claude + 사람 |
| `DESIGN.md` | 아키텍처·상태 모델·데이터 흐름·디자인 토큰 | 코드 수정 전 참조 |
| `ROUTE.md` | 구글맵 동선 시스템 (DAY_HOTEL, midHotel, midInsertIdx, computeMidInsertIdx) | 동선·픽 관련 수정 시 |
| `MISSIONS.md` | 미션 시스템 (9개 미션 + 번호 + stamp grid + trip-conquer-card) | 미션 수정 시 |
| `VERIFICATION.md` | 회귀 검증 절차 (자동 console eval + 수동 모바일 체크) | 큰 변경 직전·머지 직전 |
| `TODO.md` | 미완료 작업 백로그 | 사람 |
| `learning.md` | 개발 회고 (왜 이렇게 짰는지, 시행착오) | 사람 |

## 참조 관계

```
                       ┌──────────────────┐
                       │   index.html     │  ← 단일 source of truth (코드)
                       └────────┬─────────┘
                                │ 참조
            ┌───────────────────┼─────────────────────┐
            ▼                   ▼                     ▼
       DESIGN.md           ROUTE.md              MISSIONS.md
       (왜 이런 구조)      (동선 로직)           (미션 흐름)
            │                   │                     │
            └─────┬─────────────┴─────────┬───────────┘
                  ▼                       ▼
              CLAUDE.md            VERIFICATION.md
            (Claude 룰)          (회귀 검증 절차)
                  ▲                       ▲
                  │                       │
              TODO.md               learning.md
            (미완료 작업)          (개발 회고)
```

**규칙**:
- `index.html` 의 코드를 수정하면 → 영향 받는 `DESIGN/ROUTE/MISSIONS.md` 도 같이 업데이트
- 새 검증 항목이 생기면 → `VERIFICATION.md` 자동 검증 스크립트에 케이스 추가
- 새 문서 추가 시 → 이 `DOC-MAP.md` 표·다이어그램 갱신

## index.html 내부 영역 매핑 (라인 번호는 변동 가능, 키워드 검색)

| 영역 | 키워드 | 관련 문서 |
|---|---|---|
| `<style>` | `:root {` (CSS 토큰), `.page-nav`, `.day` | DESIGN.md § Design Tokens |
| 헤드 | `Content-Security-Policy` | CLAUDE.md § 보안 |
| 페이지 7개 | `<div class="page" data-page="..."` | DESIGN.md § Page Structure |
| 미션 카드 | `data-mission=` | MISSIONS.md |
| 동선 시스템 | `DAY_HOTEL =`, `computeMidInsertIdx`, `renderDayRoute`, `renderDayMap` | ROUTE.md |
| 인증/Auth | `AUTH_HASH_HEX`, `auth-gate` | DESIGN.md § Auth |
| Export/Import | `btn-export`, `sanitizeImportedState` | DESIGN.md § State Persistence |
| trip-conquer-card | `id="trip-conquer-card"`, `updateTripStats` | MISSIONS.md § Trip Complete |
| stamp-grid | `id="stamp-grid"`, `buildStampGrid` | MISSIONS.md § Stamp Grid |

## 변경 영향 매트릭스 (이거 만지면 저거 같이 봐야 함)

| 만지는 것 | 같이 봐야 할 것 |
|---|---|
| 새 미션 추가 | MISSIONS.md, VERIFICATION.md, `getAllMissions()`, `assignMissionNumbers()`, stamp grid |
| 새 픽/장소 추가 | CLAUDE.md § 장소 검증, `PLACE_QUERIES`, `PLACE_COORDS`, ROUTE.md |
| state 새 필드 | CLAUDE.md § 보안, `sanitizeImportedState`, VERIFICATION.md, DESIGN.md § State |
| midHotel 위치 변경 | ROUTE.md, `computeMidInsertIdx`, `assignPickNumbers`, `renderDayMap`, `renderDayRoute` |
| 페이지 추가/삭제 | `PAGE_IDS`, `<nav class="page-nav">`, DESIGN.md § Page Structure |
| 디자인 토큰 변경 | DESIGN.md § Design Tokens (alias 추적!) |
