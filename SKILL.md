---
name: norikae-guide
description: Search Japan train routes and station timetables via Yahoo! 乗換案内. Use when users ask about train routing, schedules, or timetables in Japan.
---

# Norikae Guide

Search Japan train routes and look up station timetables using Yahoo! 乗換案内.

Supports English, Japanese, Simplified Chinese, **Traditional Chinese, and Cantonese** input.
Always convert station names to Japanese before querying (e.g. Shibuya → 渋谷, 澀谷/涩谷 → 渋谷, 新宿 → 新宿).
See `references/yahoo-transit-params.md` for the full station name normalization table.

## Route Search

```bash
python3 scripts/fetch_norikae_routes.py --from <出発駅> --to <到着駅> --show-url [options]
```

`--show-url` must always be included.

Options:
- `--via <駅>` (repeatable, max 3)
- `--year/--month/--day/--hour/--minute` — resolve relative dates to absolute values; defaults to now
- `--time-type`: `departure` (default), `arrival`, `first_train`, `last_train`, `unspecified`
- `--sort-by`: `time` (default), `fare`, `transfer`
- `--ticket`: `ic` (default), `cash`
- `--seat-preference`: `non_reserved` (default), `reserved`, `green`
- `--walk-speed`: `slightly_slow` (default), `fast`, `slightly_fast`, `slow`
- `--no-use-shinkansen`, `--no-use-express`, `--no-use-airline`, `--no-use-highway-bus`, `--no-use-local-bus`, `--no-use-ferry`

You can also fetch from an existing URL:
```bash
python3 scripts/fetch_norikae_routes.py --url '<full-yahoo-url>' --show-url
```

### Intent Mapping

| User intent | CLI flags |
| --- | --- |
| cheapest / 最便宜 / 最平 | `--sort-by fare` |
| fastest / 最快 / 搭咩車最快 | `--sort-by time` |
| fewest transfers / 换乘最少 / 最少轉車 / 轉車最少 | `--sort-by transfer` |
| no shinkansen / 不要新干线 / 唔搭新幹線 | `--no-use-shinkansen` |
| local trains only / 在来線だけ / 普通車 | `--no-use-shinkansen --no-use-express` |
| no buses / 不要巴士 / 唔搭巴士 | `--no-use-highway-bus --no-use-local-bus` |
| cash fare / 现金票价 / 現金票 | `--ticket cash` |
| reserved seat / 指定席 / 劃位 | `--seat-preference reserved` |
| Green Car / 绿车 / 頭等車廂 | `--seat-preference green` |
| arrive by / XX点前到 / XX點前到 | `--time-type arrival` |
| first train / 始発 / 首班車 | `--time-type first_train` |
| last train / 終電 / 尾班車 / 最後一班 | `--time-type last_train` |

## Timetable Lookup

Follow these steps in order:

1. **Search for the station code:**
   ```bash
   python3 scripts/fetch_timetable.py search <station-name-in-japanese> --station-only
   ```
   Use `--station-only` to exclude bus stops. Pick the matching station code from the results.

2. **List lines and directions at the station:**
   ```bash
   python3 scripts/fetch_timetable.py lines <station-code>
   ```
   This returns available lines with their `gid` (group ID). If the user specified a line, match it; otherwise ask which line/direction they want. If there is only one line, skip asking and proceed.

3. **Show the departure timetable:**
   ```bash
   python3 scripts/fetch_timetable.py timetable <station-code> <gid> [--kind 1|2|4]
   ```
   `--kind`: `1` = weekday, `2` = saturday, `4` = holiday/sunday. Defaults to today's schedule.

   **Day-of-week inference rule:** When the user specifies a date with an explicit weekday or holiday indicator, resolve `--kind` automatically:
   - 星期一〜五 / Monday–Friday / 平日 → `--kind 1`
   - 星期六 / Saturday / 土曜 → `--kind 2`
   - 星期日 / Sunday / 祝日 / 公眾假期 / holiday → `--kind 4`

## Clarification Rules

Ask one concise clarification when:
- Station name is ambiguous (multiple matches)
- Departure station is missing and cannot be inferred
- User gives conflicting priorities (e.g. "fastest" and "cheapest") without tie-break
- Station has multiple lines and user didn't specify which one
