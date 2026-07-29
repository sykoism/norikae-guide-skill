# Hermes Agent — Norikae Guide Skill Installation Guide

This document is written for **Hermes agent** to read and follow in order to install and activate the `norikae-guide` skill for Japan transit route planning.

---

## Prerequisites

Before installing, verify the following are available in your execution environment:

```bash
python3 --version   # Must be 3.10 or higher
python3 -c "import urllib.request, json, re, argparse; print('OK')"
```

If Python 3.10+ is not available, ask the user to install it before proceeding.

---

## Installation

### Method 1 — Skills CLI (Recommended)

Use the `skills` CLI to install directly from GitHub:

```bash
npx -y skills@latest add sykoism/norikae-guide-skill --skill norikae-guide --yes --global
```

> If the above fails (e.g. npm not available), use Method 2.

### Method 2 — Manual from GitHub (zip)

1. Download the skill zip from the repository:

```bash
curl -L -o norikae-guide-skill.zip \
  https://github.com/sykoism/norikae-guide-skill/archive/refs/heads/main.zip
```

2. Unzip and locate the skill files:

```bash
unzip norikae-guide-skill.zip
# Files will be in: norikae-guide-skill-main/
```

3. Copy the required files to your skills directory (adjust path to your agent's skills folder):

```
norikae-guide-skill-main/
  SKILL.md                                  ← main agent instructions
  HERMES_INSTALL.md                         ← this file
  agents/openai.yaml                        ← agent interface config
  scripts/fetch_norikae_routes.py           ← route search tool
  scripts/fetch_timetable.py               ← timetable lookup tool
  references/natural-language-examples.md  ← NL parsing reference
  references/yahoo-transit-params.md       ← parameter mapping reference
```

### Method 3 — Clone directly

```bash
git clone https://github.com/sykoism/norikae-guide-skill.git
```

---

## Activation

After installation, ensure `SKILL.md` is loaded in your skill context. The skill is activated whenever a user asks about:
- Japan train routes
- Station timetables
- Transit options between Japanese cities or stations

---

## Verification

Run the following tests to confirm the skill is working correctly. All three are benchmark queries from the primary user base (Traditional Chinese / Cantonese speakers).

### Test 1 — Last limited express, Sapporo → Hakodate, Oct 24

```bash
python3 scripts/fetch_norikae_routes.py \
  --from 札幌 --to 函館 \
  --month 10 --day 24 \
  --time-type last_train \
  --show-url
```

**Expected:** Returns at least one route showing the last 特急 (limited express) departing Sapporo for Hakodate on October 24, with departure/arrival times and fare.

---

### Test 2 — Fastest route, Shinjuku → Kawaguchiko

```bash
python3 scripts/fetch_norikae_routes.py \
  --from 新宿 --to 河口湖 \
  --sort-by time \
  --show-url
```

**Expected:** Returns multiple routes sorted by travel time. Typically involves JR Chuo Line to Otsuki then Fujikyu Railway.

---

### Test 3 — Fewest transfers, Shin-Chitose Airport → Otaru, Oct 25 (Sunday)

```bash
python3 scripts/fetch_norikae_routes.py \
  --from 新千歳空港 --to 小樽 \
  --month 10 --day 25 \
  --sort-by transfer \
  --show-url
```

**Expected:** Returns routes sorted by number of transfers. The best route is typically a direct JR Airport Rapid service with 0 or 1 transfer.

---

### Test 4 — Timetable station search

```bash
python3 scripts/fetch_timetable.py search 渋谷 --station-only
```

**Expected:** Returns `渋谷` with its station code (e.g. `code=22715, 東京`).

---

### Test 5 — Build distributable zip

```bash
python3 scripts/build_skill_zip.py
```

**Expected:** Creates `norikae-guide-skill.zip` containing all skill files.

---

## Known Limitations

| Issue | Workaround |
| --- | --- |
| `UnicodeEncodeError` on Windows when printing Japanese output | The scripts auto-configure stdout to UTF-8 on startup. If the error still occurs, run: `set PYTHONIOENCODING=utf-8` before executing scripts |
| Yahoo! 乗換案内 may throttle rapid successive requests | Add a 1–2 second delay between queries if running multiple searches in batch |
| The `--url` flag only accepts Yahoo transit URLs from `transit.yahoo.co.jp` | Do not pass URLs from other transit services |
| Timetable `__NEXT_DATA__` format may change if Yahoo updates their Next.js app | If `fetch_timetable.py` errors with "Could not find __NEXT_DATA__ in page", the site structure may have changed — report to the skill maintainer |
| Maximum 3 via stations supported | Yahoo! 乗換案内 API restriction |

---

## Reference

- **Skill instructions:** `SKILL.md`
- **Parameter mapping:** `references/yahoo-transit-params.md`
- **Language examples:** `references/natural-language-examples.md`
- **Repository:** https://github.com/sykoism/norikae-guide-skill
