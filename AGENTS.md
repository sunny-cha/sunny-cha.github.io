# Tesla Subsidy PWA — Project Context

## Overview
PWA at `index.html` that displays Tesla EV subsidy amounts by model and region, sourced from 무공해차 누리집 (ev.or.kr) via teslacharger.co.kr scraping.

## Data Source
- **Official site**: ev.or.kr (JavaScript-rendered, not directly scrapable)
- **Proxy data**: teslacharger.co.kr/subsidy (scrapes ev.or.kr and displays machine-readable tables)
- **Verification site**: ev-vs.com (cross-reference)
- **Tesla Korea prices**: tesla.com/ko_kr

## Car Data (`carData`)
Key: `rwd`, `longrange`, `model_y_l`, `model3_rwd`, `model3_longrange`, `model3_perf`

Each entry: `{ name, price (만원), natCurr, convNat, natLast }`

| Model | Price | natCurr | convNat | natLast |
|-------|-------|---------|---------|---------|
| Y RWD | 4999 | 170 | 34 | 211 |
| Y LR AWD | 6699 | 210 | 42 | 240 |
| Y L 6인승 | 7299 | 210 | 42 | 230 |
| 3 RWD | 4699 | 168 | 34 | 200 |
| 3 LR | 5999 | 210 | 42 | 230 |
| 3 Perf | 6999 | 200 | 40 | 220 |

### Rules
- **natCurr**: ev.or.kr published values (already includes 50% reduction for cars > 5,300만원)
- **convNat**: 20% of natCurr (confirmed: 34/170 = 0.20)
- **natLast**: previous year value for comparison

## Region Data (`regionsData`)
Each province has:
- `status`: `{ text, badge, class }` for budget exhaustion display
- `districts`: map of districtKey → `{ name, localRate, convLocal }`

### localRate
Proportion of natCurr paid as local subsidy. **Always** used as: `localCurr = Math.round(natCurr * localRate)`

Source: ev.or.kr data via teslacharger scrapes. Example: Seoul rate=0.30 (confirmed: 51/170=0.30), Seongnam rate=0.447 (confirmed: 76/170=0.447).

### convLocal
Uniform 10만원 across all districts.

## UI Structure (result display order)
1. **기본 보조금 (blue)** — ev.or.kr 기준: 국고 + 지자체 2열 grid
2. **작년 비교 (gray)** — 작년 총 보조금 및 증감
3. **전환지원금 (purple)** — 국비+지방 별도, 전환 포함 합계/실구매가
4. **지자체 소진 현황 (amber)** — 예산 잔여 상태 배지
5. **자격 진단 (emerald)** — 체크리스트

## Calculation Logic (`calculateSubsidy()`)
```
localCurr = Math.round(natCurr * localRate)
convTotal = convNat + convLocal

// Base (ev.or.kr 기준)
totalBase = natCurr + localCurr
finalBase = car.price - totalBase

// All-in (전환 지원금 포함)
totalAll = natCurr + localCurr + convTotal
finalAll = car.price - totalAll
```

## Price Update History
- **2026-07-01**: All model prices updated to post-July Tesla Korea official prices
  - Y RWD: 4999만원 (동결)
  - Y LR AWD: 6699만원 (+300)
  - Y L 6인승: 7299만원 (+300)
  - 3 RWD: 4699만원 (+500)
  - 3 LR: 5999만원 (+700)
  - 3 Perf: 6999만원 (+500)

## Known Issues
- ev.or.kr `initLocalCarPirceAction.do` returns empty (JavaScript form); data sourced from teslacharger.co.kr scrapes instead
- Price bracket (5,300만원 cutoff) changes are reflected in natCurr values directly (not computed)
- 전환지원금 is NOT included in ev.or.kr's displayed total (shown separately in PWA)
- Model names differ between Tesla Korea site and ev.or.kr (e.g., "Model Y Premium RWD" vs "Model Y RWD")

## Common Tasks
- **Verify subsidy**: Open browser at `index.html`, compare against ev.or.kr for same region/model
- **Update localRate**: Edit `regionsData` district entries — compute as `(지방비 / 국비)` from ev.or.kr data
- **Add model**: Add entry to `carData`, add `<option>` to `#modelSelect`
- **Add district**: Add entry under province's `districts` map
