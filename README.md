# Brunei Address Parser

A lightweight, browser-based tool that extracts structured fields from messy Brunei address strings. No server, no frameworks, no installation.

---

## What It Does

Addresses typed into forms are often inconsistent in abbreviation, spacing, and ordering. This tool takes a raw string like:

```
No 7 Spg 5 Jln Gadong Kg Harum BE1234 Brunei Muara
```

And parses it into clean, structured fields:

| Field | Value |
|---|---|
| House Number | NO 7 |
| Simpang | SIMPANG 5 |
| Jalan | JALAN GADONG |
| Kampong | KAMPONG HARUM |
| Mukim | *SENGKURONG* |
| Postcode | BE1234 |
| District | BRUNEI MUARA |
| Country | *BRUNEI DARUSSALAM* |

Fields filled from the postcode map are displayed in italics.

---

## How to Use

1. Open the link [ https://cyberc0dex.github.io/brunei-address-parser ]
2. Paste or type an address into the text box
3. Click **Parse**

A debug log is available below the results showing step-by-step what the parser did. Click **Show debug log** to expand it.

---

## How It Works

The parser runs in six phases:

### Phase 1 — Normalisation
Converts input to uppercase. Strips literal `\n`/`\r` text characters, real line breaks, commas, and periods. Collapses multiple spaces.

### Phase 2 — Snipers
Pulls out predictable fields first, before the state machine runs.

| Sniper | Method |
|---|---|
| Postcode | Regex `/\b[A-Z]{2}\d{4}\b/` |
| District | Exact string match from district list |
| Country | Exact string match from country list |

Both the District and Country snipers include a **street prefix guard** — if the matched name is immediately preceded by a street-type word (e.g. `JALAN`, `SIMPANG`, `KAMPONG`), the snipe is blocked. This prevents `"Jalan Tutong"` from losing `"Tutong"` to the district field.

The Country sniper does **not** stop at the first match — it scans the entire string and the last match wins. This handles cases like `"Malaysia Brunei Darussalam"` correctly outputting `"BRUNEI DARUSSALAM"`.

### Phase 3 — Whitelist Protection
Known compound place names that contain marker words inside them (e.g. `KAMPONG AYER`) are replaced with temporary placeholder tokens so the state machine cannot accidentally split them. Tokens are restored after the loop.

### Phase 4 — Alias Mapping
Abbreviations and synonyms are standardised to a consistent output form before the state machine runs:

| User types | Output |
|---|---|
| NO, NOMBOR, NUMBER | NO |
| UNIT | UNIT |
| BLOCK, BLK | BLOCK |
| SIMPANG, SPG | SIMPANG |
| JALAN, JLN | JALAN |
| KAMPONG, KAMPUNG, KG, KPG | KAMPONG |

### Phase 5 — State Machine
Reads the word array one token at a time. Each marker token switches the active bucket and appends the standardised marker word to that bucket. All following words are collected into the active bucket until the next marker fires. The default starting bucket is `houseNumber` — a numeric fallback for addresses with no explicit house marker.

### Phase 6 — Postcode Enrichment
The extracted postcode is looked up in `POSTCODE_MAP`. Any fields that are still empty after parsing are filled from the map. Fields the user already provided are **never overwritten**. Enriched fields are four: District, Mukim, Kampong, and Country.

---

## Supported Abbreviations

| You can write | Parsed as |
|---|---|
| NO, NOMBOR, NUMBER | House Number marker |
| UNIT, BLOCK, BLK | House Number marker |
| SIMPANG, SPG | Simpang marker |
| JALAN, JLN | Jalan marker |
| KAMPONG, KAMPUNG, KG, KPG | Kampong marker |

---

## Configuration

All configurable values are at the **top of the `<script>` block**, clearly separated from the parser logic. You do not need to touch the parser to make changes.

### Adding a place name to the whitelist

The whitelist protects compound place names that contain marker words inside them from being incorrectly split by the state machine.

```js
const WHITELIST = [
  "KAMPONG AYER",
  "YOUR NEW ENTRY HERE",  // ← add here, UPPERCASE
];
```

### Adding a country

```js
const COUNTRIES = [
  "NEGARA BRUNEI DARUSSALAM",
  "BRUNEI DARUSSALAM",
  "BRUNEI",
  "MALAYSIA",
  "YOUR COUNTRY HERE",  // ← add here, UPPERCASE
  // Important: longer variants must come before shorter ones
  // e.g. "UNITED STATES OF AMERICA" before "UNITED STATES"
];
```

### Adding postcode entries

```js
const POSTCODE_MAP = [
  { "POSTCODE": "BE1234", "DISTRICT": "BRUNEI MUARA", "MUKIM": "GADONG", "KAMPONG": "KAMPONG HARUM", "COUNTRY": "BRUNEI DARUSSALAM" },
  // Add more entries here, one per line
];
```

---

## Postcode Enrichment — Data Privilege Rules

| Scenario | Result |
|---|---|
| User provided District, postcode has a District | Keep user's value |
| User did not provide District, postcode has one | Fill from postcode map |
| Same rule applies to Mukim, Kampong, and Country | Fill-only, never overwrite |
| Postcode not found in map | No enrichment, warning in debug log |
| No postcode extracted | Enrichment phase skipped entirely |

---

## Current Limitations

- **Single address input only** — batch processing is not yet implemented. The core `parseAddress()` function is intentionally isolated so batch mode can be layered on later without restructuring the parser.
- **Ambiguous marker words in place names** — mitigated by the whitelist, but names not on the whitelist containing marker words may still parse incorrectly.
- **No-marker fallback** — addresses with no recognisable markers (e.g. `7 Gadong Belait`) default everything into the House Number field.

---

## Future Improvements

- [ ] Batch mode — paste multiple addresses or upload a CSV
- [ ] Full postcode dataset
- [ ] Copy-to-clipboard for parsed results
- [ ] Export results as CSV

---

## Note
* Free to use, modify, and distribute.
* Although I have reviewed the design process and code flow, the majority of this was vibe coded with Gemini + Claude. Consider this as a test application only.
