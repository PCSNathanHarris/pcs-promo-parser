# Output CSV schemas

All four files use `utf-8-sig` encoding, CRLF line endings, comma
delimiter, standard CSV quoting. See `conventions.md` for general rules
(dates, prices, empty cells).

The schemas below match exactly what `kb/anglera.py`,
`kb/deck_parser/audit.py`, and `kb/nlp_sheet.py` produce in the
current app. The AI-stripped fork of the app reads these files as the
Stage 1 inputs to Stage 2/3/4 — column order matters.

---

## promo_list.csv

Source: `kb/kit_builder.py::write_promo_list_csv` / `PROMO_LIST_HEADERS`

**27 columns**, in this exact order:

| # | Column | Source / format |
|---|--------|-----------------|
| 1 | `Promo Name` | Free-text deal name + bracketed PCE/PCR if present, e.g. `"Mx Fuel Kit Get One Free [PCE 252601]"` |
| 2 | `Start Date` | `M/D/YYYY` non-padded |
| 3 | `End Date` | `M/D/YYYY` non-padded |
| 4 | `Item SKU 1` | First slot SKU |
| 5 | `Item Qty 1` | Integer, default `1` |
| 6 | `Item Price 1` | 2-decimal fixed (`199.00`) or `0.00` for free good |
| 7 | `Item Credit 1` | **Almost always blank** |
| 8–11 | `Item SKU 2` / `Qty 2` / `Price 2` / `Credit 2` | Same shape |
| 12–15 | `Item SKU 3` / ... / `Credit 3` | Same shape |
| 16–19 | `Item SKU 4` / ... / `Credit 4` | Same shape |
| 20–23 | `Item SKU 5` / ... / `Credit 5` | Same shape |
| 24–27 | `Item SKU 6` / ... / `Credit 6` | Same shape |

**Empty slots emit 4 empty cells** (SKU, Qty, Price, Credit all blank).

### Sample row — typical B1G1 (1 paid + 1 free)

```csv
Promo Name,Start Date,End Date,Item SKU 1,Item Qty 1,Item Price 1,Item Credit 1,Item SKU 2,Item Qty 2,Item Price 2,Item Credit 2,Item SKU 3,Item Qty 3,Item Price 3,Item Credit 3,Item SKU 4,Item Qty 4,Item Price 4,Item Credit 4,Item SKU 5,Item Qty 5,Item Price 5,Item Credit 5,Item SKU 6,Item Qty 6,Item Price 6,Item Credit 6
Mx Fuel Kit Get One Free [PCE 252601],5/4/2026,8/2/2026,2962-22,1,499.00,,48-11-1850,1,0.00,,,,,,,,,,,,,,,,,,,
```

### Sample row — multi-paid bundle (3 paid, no free)

```csv
86 Pc. Mix Drive Socket Set Bundle,3/1/2026,5/31/2026,81230P,1,99.99,,80950T,1,,,80551,1,,,,,,,,,,,,,,,,
```

(Bundle Retail in the first slot; other slots' Price/Credit blank.)

### Sample row — Cartesian explosion (2 paid × 2 free → 4 rows)

```csv
Buy a Drill, Get a Battery Free [PCE 262776],5/4/2026,8/2/2026,2962-22,1,499.00,,48-11-1850,1,0.00,,,,,,,,,,,,,,,,,,
Buy a Drill, Get a Battery Free [PCE 262776],5/4/2026,8/2/2026,2962-22,1,499.00,,48-11-2440,1,0.00,,,,,,,,,,,,,,,,,,
Buy a Drill, Get a Battery Free [PCE 262776],5/4/2026,8/2/2026,2767-22,1,549.00,,48-11-1850,1,0.00,,,,,,,,,,,,,,,,,,
Buy a Drill, Get a Battery Free [PCE 262776],5/4/2026,8/2/2026,2767-22,1,549.00,,48-11-2440,1,0.00,,,,,,,,,,,,,,,,,,
```

All four rows share the same Promo Name + dates.

---

## non_included.csv

Source: `kb/deck_parser/audit.py::write_non_included` / `NON_INCLUDED_HEADERS`

**5 columns**, in this exact order:

| # | Column | Source / format |
|---|--------|-----------------|
| 1 | `Page` | 1-indexed page number where the exclusion was detected |
| 2 | `Reason` | One of the reason codes below |
| 3 | `SKU` | Affected SKU (blank if page-level exclusion with no extracted SKUs) |
| 4 | `Deal Text` | Short snippet of the promo title / marker text from the page |
| 5 | `Detail` | Free-text detail — e.g. matched marker phrase, error message |

### Valid `Reason` codes

| Code | Cause |
|------|-------|
| `price-change` | NLP / Special Buy / Clearance / EDLP / Price Drop / `% Off` (only when page can't be NLP-routed) |
| `brick-and-mortar` | In-store only, branches only, B&M, not online |
| `spiff` | Sales-rep incentive |
| `rsa` | Retail Sales Associate incentive program |
| `spend-to-earn` | Buy $N in category → get reward; spend-and-earn rebates |
| `pos-redemption` | POS / mail-in / instant rebate or redemption mechanism |
| `buy-more-save-more` | BMSM, volume-tiered pricing, "Buy 5 save 10%" |
| `promo-code-only` | Page IS a coupon code (use code XYZ at checkout) |
| `arp` | Authorized Retailer Program — channel-restricted, not for general online |
| `killed` | Strikethrough on the SKU itself (deal cancelled) |
| `missing-price` | Paid SKU detected but no extractable price |
| `image-only-free-good` | Free SKU visible only in image, no row in price table |
| `strikethrough` | SKU was struck out in the source — exclude |

### Sample rows

```csv
Page,Reason,SKU,Deal Text,Detail
23,price-change,2962-22,Get 20% Off Heavy-Duty Pricing,SPECIAL_BUY_MARKER matched: "20% Off"
45,brick-and-mortar,,In-Store Execution Only - Promote at $99,BM_MARKER matched
67,spend-to-earn,,Buy $800 in Concrete Accessories Get 1 Free,SPEND_TO_EARN_MARKER: "Buy $800"
89,promo-code-only,,Use Code SUMMER25 at Checkout,PROMO_CODE_MARKER: "use code"
12,missing-price,DCS438B,Buy Drill Get Battery Free,price column not found
14,rsa,,RSA Reward Program — Earn $15 per Kit Sold,RSA_MARKER matched
22,pos-redemption,,Mail-In Rebate: $50 back by mail,POS_REDEMPTION_MARKER: "Mail-In Rebate"
31,arp,,ARP Exclusive — Authorized Retailer Only,ARP_MARKER matched
```

---

## nlp_sheet.csv

Source: `kb/nlp_sheet.py::write_nlp_sheet_csv` / `NLP_SHEET_COLUMNS`
(v0.5.9 expands this — see Promo Name column below)

**9 columns**, in this exact order:

| # | Column | Source / format |
|---|--------|-----------------|
| 1 | `Promo Name` | Free-text deal title + bracketed PCE/PCR (e.g. `"PCE 262776"`). Blank if no identifier visible. |
| 2 | `SKU` | One SKU per row |
| 3 | `Promo Price` | 2-decimal fixed price. **Blank if price unextractable** (user fills in manually downstream). |
| 4 | `Online Execution Start` | `M/D/YYYY` non-padded |
| 5 | `Online Execution End` | `M/D/YYYY` non-padded |
| 6 | `Vendor` | Display name, e.g. `Milwaukee` |
| 7 | `Page` | 1-indexed page number |
| 8 | `Price Label` | Which header column was matched (e.g. `Promo IMAP`, `MAP`) |
| 9 | `Source Marker` | `nlp` (NLP_MARKER matched) or `special-buy` (SPECIAL_BUY_MARKER matched) |

### Sample rows

```csv
Promo Name,SKU,Promo Price,Online Execution Start,Online Execution End,Vendor,Page,Price Label,Source Marker
MX FUEL Carry-On Power Supply Special Buy [PCE 262776],MXF002-2XC,1399.00,5/4/2026,8/2/2026,Milwaukee,33,Promo IMAP,special-buy
MX FUEL Carry-On Power Supply Special Buy [PCE 262776],MXF002-1XC,899.00,5/4/2026,8/2/2026,Milwaukee,33,Promo IMAP,special-buy
M18 Drill Driver NLP,2607-20,99.00,5/4/2026,8/2/2026,Milwaukee,67,IMAP,nlp
```

If a SKU has no extractable price, emit it anyway with `Promo Price`
blank — that's a flag for the user to fill in manually.

---

## parser_audit.csv

Source: skill-flavored variant. The original
`kb/deck_parser/audit.py::write_parser_audit` has fields specific to
the rule parser (skipped-page-by-marker breakdown); the skill emits a
simpler, AI-mode audit.

**10 columns**, single row per parse run:

| # | Column | Source |
|---|--------|--------|
| 1 | `Deck` | Path or filename of the parsed deck |
| 2 | `Vendor` | Detected vendor display name (e.g. `Milwaukee`) |
| 3 | `Total Pages` | Page count of the deck |
| 4 | `Kit Pages` | Pages classified as kit-promo |
| 5 | `NLP Pages` | Pages routed to NLP Sheet (NLP_MARKER / SPECIAL_BUY_MARKER) |
| 6 | `Excluded Pages` | Pages that landed in `non_included.csv` (sum of all non-NLP exclusion reasons) |
| 7 | `Promo Rows` | Total rows in `promo_list.csv` |
| 8 | `NLP Rows` | Total rows in `nlp_sheet.csv` |
| 9 | `Non-Included Count` | Total rows in `non_included.csv` |
| 10 | `Run At UTC` | ISO 8601 UTC timestamp (e.g. `2026-05-12T17:23:00Z`) |

### Sample row

```csv
Deck,Vendor,Total Pages,Kit Pages,NLP Pages,Excluded Pages,Promo Rows,NLP Rows,Non-Included Count,Run At UTC
Milwaukee_P2_2026.pdf,Milwaukee,98,28,60,8,412,287,15,2026-05-12T17:23:00Z
```

---

## Empty-file convention

Even when a list is empty, **emit the header-only file**. Downstream
stages of the app expect all four files to exist.

For example, a deck with no excluded pages still writes:

```csv
Page,Reason,SKU,Deal Text,Detail
```

(Just the header row, terminated by `\r\n`.)
