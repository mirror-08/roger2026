# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

뮤지컬 ROGER 2026 관람 기록 PWA. GitHub Pages(`mirror-08/roger2026`)에 배포되며 **단일 파일(`index.html`)** 구조다. 서버/빌드 도구 없음.

## Deployment

변경 후 항상 커밋 + 푸시까지 함께 진행한다. 푸시해야 GitHub Pages에 반영된다.

```bash
git add index.html   # (또는 수정한 파일)
git commit -m "..."
git push
```

서비스 워커(`sw.js`)의 `CACHE_NAME = 'roger2026-vN'` — `index.html`은 네트워크 우선 전략이라 SW 버전 변경 없이도 최신 파일을 받아온다. 이미지·manifest 등 정적 파일 변경 시에만 버전 번호를 올린다.

## Architecture

### 단일 파일 구조
`index.html` 한 파일에 CSS(`<style>`), HTML, JavaScript(`<script>`)가 모두 포함된다.

### 데이터 레이어 (`index.html:900`)
모든 데이터는 `localStorage`에 `roger_` 접두어로 저장. `DB.get/set` 헬퍼 사용.

| localStorage 키 | 내용 |
|---|---|
| `roger_records` | 관람 기록 배열 |
| `roger_stamp_boards` | 도장판 배열 |
| `roger_coupons` | 쿠폰 현황 객체 (`discount40`, `discount50`, `discountPass`, `doubleStamp`, `liveOST`) |
| `roger_schedule` | `schedule.json`에서 fetch한 공연 스케줄 |
| `roger_events` | `events.json`에서 fetch한 이벤트 목록 |
| `roger_settings` | 다크모드 등 앱 설정 |

### 외부 데이터 파일
- `schedule.json` — 공연 스케줄 (`{ date, time, alex, philo }` 배열). `alex`는 스카일러 역, `philo`는 디디 역 배우.
- `events.json` — 이벤트 목록 (`{ startDate, endDate, name, color }` 배열).

두 파일 모두 앱 시작 시 fetch해서 localStorage에 캐시, 이후 캐시 우선으로 동작.

### JS 섹션 구조 (`index.html:900~`)
| 라인 | 섹션 |
|---|---|
| 900 | DATA LAYER — DB 헬퍼, getter/setter, 마이그레이션 |
| 950 | TAB NAVIGATION — `switchTab()` |
| 1088 | ADD / EDIT RECORD — 관람 기록 추가·수정 |
| 1279 | RECORD DETAIL — 기록 상세 보기 |
| 1412 | HOME — `renderHome()` |
| 1549 | CALENDAR — `renderCalendar()`, `renderCalDayPanel()` |
| 1912 | SCHEDULE — 스케줄 탭 렌더링, `fetchSchedule()` |
| 1981 | COUPON / STAMP — 쿠폰·도장판 렌더링 및 로직 |
| 2795 | SETTINGS |
| 2856 | INIT — `init()` 진입점 |

### 쿠폰 계산 로직
쿠폰 잔여 수량은 **세 가지 소스의 합산**: ① 수동 이력(획득/복원 - 사용) ② 도장판 마일스톤 달성(`m.notified === true`) ③ 관람 기록의 할인권 사용. `calcCouponCount()` 참고.

### 도장판 로직
- `board.currentCount`는 `recalcStampCount()`로 항상 `history[].qty` 합산 재계산.
- 마일스톤 달성 시 `m.notified = true`로 표시 — 쿠폰 계산 트리거.
- 도장 추가 방식: `직접 관람`(qty=1), `더블적립`(qty=2, 더블적립권 1장 소모), `트리플적립`(qty=3), `교환`.

### 테마
CSS 변수(`--bg`, `--accent` 등)로 라이트/다크 전환. `applyDark(bool)`로 `data-theme` 속성 토글.

## Key Conventions

- **방식 선택 → 수량 자동 선택**: `updateStampSessionOptions()`에서 더블적립→2개, 트리플적립→3개, 그 외→1개로 자동 지정.
- **쿠폰 카드 레이아웃**: 홈 화면 쿠폰 5개를 위 3개 + 아래 2개(3열 그리드 2행)로 렌더링. `renderHome()` 내 `couponCardHtml` 참고.
- **캘린더 날짜 패널**: `renderCalDayPanel(ds, daySch, doneSet, bookedSet, records, events)` — 날짜 옆에 해당일 이벤트 배지 표시.
