# FRLG Master RNG

HTML + JavaScript tool for Pokémon Fire Red / Leaf Green RNG manipulation: find initial seed and advance for a target IV spread and nature, confirm which seed you hit via the Explorer, and calibrate timing. No build step—open `index.html` in a browser.

---

## How to run

Open **index.html** in a modern browser (Chrome, Firefox, Edge). No server or install required.

---

## Tabs overview

### Seed Marathon

- Enter **TID**, **SID**, **target nature**, and **IVs** (or derive IVs from Level 100 stats + species + nature via **Auto-Extract IVs**).
- Set **min** and **max advance**, then run **Execute Global Scan**.
- Results list seed (hex), advance, method (M1/M2/M4), IV spread, and shiny.
- **Initial Seed Consistency Log** groups hits by seed.

### Seed Explorer

- Enter a **seed** (hex, e.g. `0xBF6`), set **scan depth**, then **Map Advancements**.
- Table shows advance, nature, IV spread, and shiny for Method 1 shinies only (for seed confirmation).
- **Verify seed @ advance:** enter seed and advance (e.g. from PokeFinder) and click **Verify** to see the nature/IVs this tool computes (PokeFinder sync check).

### Calibration

- Enter **target frame** and **actual hit** to compute latency/offset (ms).

---

## RNG and advance semantics (for debugging)

- **LCRNG:** `next(s) = (s * 0x41C64E6D + 0x6073) >>> 0` (Gen 3).
- **Advance:** “Advance N” means the RNG has been advanced N times from the initial seed; the encounter uses the *next* four (Method 1) or five/six (Method 2/4) states for PID and IVs. Logic matches the “calculadora seed” style: advance state at the start of each frame, then use `next(state)`, `next(next(state))`, … for PID and IVs.
- **PID:** Built from the first two 16-bit outputs (high 16 bits of those states): `PID = (2nd << 16) | 1st`; nature = `PID % 25`.
- **IVs (Method 1):** Two 16-bit words.
  - First word: HP = bits 0–4, Atk = 5–9, Def = 10–14.
  - Second word: SpA = bits 5–9, SpD = 10–14, Spe = bits 0–4. (Order matters; wrong order was a past bug.)
- **Methods:** M1 = gift/static (e.g. Eevee); M2/M4 = wild (different RNG call order; one call discarded). All three are searched in Marathon.
- **Shiny:** `(TID ^ SID ^ PID_low ^ PID_high) < 8`.

---

## IV from stats

Level 100 stats + species + nature → **Auto-Extract IVs** fills the IV fields. If multiple IVs give the same stat (nature rounding), a warning is shown; prefer exact IVs from a known spread when possible.

---

## References

- Smogon: “The Process of PID and IV Creation of Non-Bred Pokemon”.
- PokeFinder (advance 0-based, same “advance then output” semantics).
- Aligned with logic in “calculadora seed”–style tools for seed/advance finding.

---

## Changelog / fixes applied

- IV second word order fixed (SpA/SpD/Spe from bits 5–9, 10–14, 0–4).
- Method 2 and 4 use correct RNG calls (v4/v5 and v3/v5 etc.).
- RNG “advance then output” and marathon loop aligned with calculadora (advance at start of frame, report advance = frame + 1).
- Verify seed @ advance added for PokeFinder cross-check.
