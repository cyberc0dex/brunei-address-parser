# Brunei Address Parser

A simple browser-based tool that extracts structured fields from messy Brunei address strings. No server, no dependencies, no installation — just open the HTML file and use it.

---

## What It Does

Addresses typed into forms are often inconsistent. This tool takes a raw address string like:

```
No 7 Spg 123-45-67 Jln Gadong Kg Sungai Akar BE1234 Negara Brunei Darussalam
```

And parses it into clean, structured fields:

| Field | Value |
|---|---|
| House Number | 7 |
| Simpang | 123-45-67 |
| Jalan | GADONG |
| Kampong | SUNGAI AKAR |
| Postcode | BE1234 |
| District | _(empty)_ |
| Country | NEGARA BRUNEI DARUSSALAM |

---

## How to Use

1. Open the link [ https://cyberc0dex.github.io/brunei-address-parser ]
2. Paste or type an address into the text box
3. Click **Parse**

A debug log is available below the results to show you step-by-step what the parser did to the input string. Click **Show debug log** to expand it.

---

## How It Works

The parser runs in four phases:

1. **Sanitation** — Converts input to uppercase and removes punctuation noise
2. **Snipers** — Pulls out predictable fields first: Country, District, and Postcode using exact string and regex matching
3. **Alias Mapping** — Standardises abbreviations (e.g. `SPG`, `JLN`, `KG`) into internal marker tokens
4. **State Machine** — Reads the remaining words one by one; each marker token switches the active "bucket", and all following words fall into that bucket until the next marker

---

## Supported Abbreviations

| You can write... | Parsed as |
|---|---|
| NO, NOMBOR, NUMBER, UNIT, BLOCK, BLK | House Number marker |
| SIMPANG, SPG | Simpang marker |
| JALAN, JLN | Jalan marker |
| KAMPONG, KAMPUNG, KG, KPG | Kampong marker |

---

## Configuration

All configurable values are at the **top of the `<script>` block**, clearly labelled. You do not need to touch the parser logic to make changes.

### Adding a place name to the whitelist

The whitelist protects known place names that contain marker words inside them (e.g. `KAMPONG AYER`) from being incorrectly split. To add a new entry:

```js
const WHITELIST = [
  "KAMPONG AYER",
  "YOUR NEW ENTRY HERE",  // ← add here
];
```

### Adding a new alias

```js
const ALIAS_MAP = {
  "SPG"    : "MARKER_SPG",
  "LORONG" : "MARKER_JLN",  // ← example: treat LORONG same as JALAN
};
```

---

## Considerations

- **Single address input only** — batch processing is not yet implemented, though the parser function is written to make this straightforward to add later
- **Ambiguous marker words in street names** — e.g. a street literally named "Jalan Kampong Ayer" could confuse the state machine; mitigated by the whitelist
- **No marker fallback** — if an address has no recognisable markers at all (e.g. `7 Gadong Belait`), all words default into the House Number field

---

## Future Upgrades

- [ ] Batch mode (paste multiple addresses or upload a CSV)
- [ ] Export parsed results as CSV

---

## Note

* Free to use, modify, and distribute.
* Although I have reviewed the design process and code flow, the majority of this was vibe coded with Gemini + Claude. Consider this as a test application.
