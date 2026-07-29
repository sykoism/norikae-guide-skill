# Natural Language to Parameter Examples

Use these examples when translating user requests into canonical fields before querying Yahoo! 乗換案内.

## Basic Route

| User Request | Canonical Fields |
| --- | --- |
| 东京到新宿怎么走 | `from=東京`, `to=新宿` |
| Route from Shinjuku to Yokohama | `from=新宿`, `to=横浜` |
| 東京から九段下まで | `from=東京`, `to=九段下` |

## Via Stations

| User Request | Canonical Fields |
| --- | --- |
| 東京から新宿まで、表参道経由で | `from=東京`, `to=新宿`, `via=["表参道"]` |
| Shibuya to Ikebukuro via Harajuku and Shinjuku | `from=渋谷`, `to=池袋`, `via=["原宿","新宿"]` |
| 东京到上野，经过秋叶原和神田 | `from=東京`, `to=上野`, `via=["秋葉原","神田"]` |

## Time Intent

| User Request | Canonical Fields |
| --- | --- |
| 10:30出发 | `hour=10`, `minute=30`, `timeType=departure` |
| I need to arrive by 18:00 | `hour=18`, `minute=0`, `timeType=arrival` |
| 始発で行きたい | `timeType=first_train` |
| 最终电车回去 | `timeType=last_train` |

## Sort Preference

| User Request | Canonical Fields |
| --- | --- |
| 一番安いルート | `sortBy=fare` |
| 最快路线 | `sortBy=time` |
| 换乘最少 | `sortBy=transfer` |

## Fare, Seat, Walk

| User Request | Canonical Fields |
| --- | --- |
| きっぷ運賃で | `ticket=cash` |
| 指定席で | `seatPreference=reserved` |
| グリーン車で | `seatPreference=green` |
| 急いでるので早歩き | `walkSpeed=fast` |
| ゆっくり歩きたい | `walkSpeed=slow` |

## Transport Exclusion

| User Request | Canonical Fields |
| --- | --- |
| 新幹線なしで | `useShinkansen=false` |
| 在来線だけで | `useShinkansen=false`, `useExpress=false` |
| No buses please | `useHighwayBus=false`, `useLocalBus=false` |
| 飞机也不要 | `useAirline=false` |

## Combined Constraints

### Example 1

User request:

`明天早上10点前到大阪，不要新干线，最便宜。`

Canonical fields:

- `from=東京` (if implied departure from current context)
- `to=大阪`
- `timeType=arrival`
- `hour=10`
- `useShinkansen=false`
- `sortBy=fare`

### Example 2

User request:

`来週金曜18時までに品川着、新宿発、飛行機なし、乗換少なめ。`

Canonical fields:

- `from=新宿`
- `to=品川`
- `timeType=arrival`
- `year/month/day` resolved from "来週金曜"
- `hour=18`
- `useAirline=false`
- `sortBy=transfer`

## Timetable Queries

| User Request | Action |
| --- | --- |
| 渋谷駅の時刻表を見せて | `search 渋谷` → `lines <code>` → ask which line/direction → `timetable <code> <gid>` |
| Shibuya Yamanote line schedule | `search 渋谷` → `lines <code>` → find 山手線 gid → `timetable <code> <gid>` |
| 東京駅の東海道新幹線、平日の時刻表 | `search 東京` → `lines <code>` → find 東海道新幹線 gid → `timetable <code> <gid> --kind 1` |
| 新宿から中央線で何時に電車がある？ | `search 新宿` → `lines <code>` → find 中央線 gid → `timetable <code> <gid>` |
| 渋谷有哪些线路？ | `search 渋谷` → `lines <code>` (show all lines, don't proceed to timetable) |

When the user asks for a timetable:
- If the station has only one line/direction, skip the `lines` step and go directly to `timetable`.
- If the station has multiple lines, either match the user's stated line or ask which line/direction they want.
- If the user asks about a specific day type (weekday/saturday/holiday), pass `--kind`.

---

## Traditional Chinese & Cantonese Examples

The primary user base communicates in **Traditional Chinese (繁體中文)** and **Cantonese (廣東話)**.
The following examples illustrate how to parse colloquial Cantonese phrasing into canonical fields.

### Benchmark Queries

#### Query 1 — Last limited express, specific date

> **「10月24號札幌去函館最後一班特急係幾點?」**

Parsing:
- `from=札幌` (Traditional Chinese: 札幌 → Japanese: 札幌)
- `to=函館` (Traditional Chinese: 函館 → Japanese: 函館)
- `month=10`, `day=24`
- `timeType=last_train` ("最後一班")
- `useExpress=true` (特急 = limited express; keep express enabled)

Command:
```bash
python3 scripts/fetch_norikae_routes.py --from 札幌 --to 函館 --month 10 --day 24 --time-type last_train --show-url
```

---

#### Query 2 — Fastest route, any transport

> **「新宿去河口湖搭咩車最快?」**

Parsing:
- `from=新宿`
- `to=河口湖`
- `sortBy=time` ("最快" = fastest)
- "搭咩車" = which train/transport — no constraint, use defaults

Command:
```bash
python3 scripts/fetch_norikae_routes.py --from 新宿 --to 河口湖 --sort-by time --show-url
```

---

#### Query 3 — Fewest transfers, specific date, explicit weekday (Sunday)

> **「10月25號星期日新千歲去小樽, 最少轉車次數點搭?」**

Parsing:
- `from=新千歳空港` (新千歲 → 新千歳空港; nearest train station at New Chitose Airport)
- `to=小樽`
- `month=10`, `day=25`
- `sortBy=transfer` ("最少轉車次數" = fewest transfers)
- "星期日" = Sunday → if looking up timetable, use `--kind 4`

Command:
```bash
python3 scripts/fetch_norikae_routes.py --from 新千歳空港 --to 小樽 --month 10 --day 25 --sort-by transfer --show-url
```

---

### Additional Traditional Chinese / Cantonese Examples

| User Request | Canonical Fields / Command hint |
| --- | --- |
| 東京去大阪最平係幾錢? | `from=東京`, `to=大阪`, `sortBy=fare` |
| 我想坐頭班車由新宿去橫濱 | `from=新宿`, `to=横浜`, `timeType=first_train` |
| 唔想轉車，由京都去奈良點去? | `from=京都`, `to=奈良`, `sortBy=transfer` |
| 澀谷去池袋，唔搭巴士，最快 | `from=渋谷`, `to=池袋`, `useHighwayBus=false`, `useLocalBus=false`, `sortBy=time` |
| 11月3號(公眾假期)新大阪時刻表 | `search 新大阪` → `lines <code>` → `timetable <code> <gid> --kind 4` |
| 想搭指定席由東京去名古屋 | `from=東京`, `to=名古屋`, `seatPreference=reserved` |
| 由大阪去廣島，唔搭新幹線，點搭? | `from=大阪`, `to=広島`, `useShinkansen=false` |

### Station Name Normalization — Traditional Chinese to Japanese

| Traditional Chinese Input | Cantonese Reading | Japanese Station Name |
| --- | --- | --- |
| 澀谷 / 澀谷 | Sap Guk | 渋谷 |
| 新宿 | San Zuk | 新宿 |
| 東京 | Dung Ging | 東京 |
| 橫濱 | Waang Ban | 横浜 |
| 大阪 | Daai Baan | 大阪 |
| 京都 | Ging Dou | 京都 |
| 名古屋 | Ming Gu Uk | 名古屋 |
| 札幌 | Jaap Pou | 札幌 |
| 函館 | Ham Gun | 函館 |
| 小樽 | Siu Taan | 小樽 |
| 新千歲 | San Cin Seoi | 新千歳空港 |
| 河口湖 | Ho Hau Wu | 河口湖 |
| 奈良 | Naai Loeng | 奈良 |
| 廣島 | Gwong Dou | 広島 |
| 神戶 | San Woo | 神戸 |

---

## Clarification Rules

Ask one concise clarification when:

- station name is ambiguous
- departure station is missing and cannot be inferred from context
- user gives conflicting priorities (`fastest` and `cheapest`) without tie-break
- station has multiple lines and user didn't specify which one
