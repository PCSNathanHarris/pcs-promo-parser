# Makita

## Brand keywords

- `makita`
- `lxt` (18V LXT platform)
- `xgt` (40V XGT platform)
- `cxt` (12V CXT platform)

## SKU pattern

Regex: `\b[A-Z]{2,4}\d{2,4}[A-Z0-9-]*\b`

Examples:
- `XAD02Z`, `XPH14Z`, `GT200D` (cordless tools)
- `BL1850B`, `BL4040` (batteries)
- `GMH04PLX` (XGT kits)

The pattern is broader than DeWalt's — Makita uses many letter
prefixes (`X`, `B`, `G`, `D`, `R`, etc.). Be conservative: only emit
SKUs that match this shape AND appear in/near a price-bearing row.

## Promo code pattern

Regex: `\b(XGT|LXT)[\s-]*([A-Z0-9.\s]+?\d+(?:\.\d+)?)` (case-insensitive)

Examples:
- `XGT Q2 1.03.0`
- `LXT 2025 Q3`

Makita codes are bottom-right corner identifiers; often absent entirely.
When present, append in brackets to `promo_name`:

- `"FX01-Z XGT Hammer Drill Promo [XGT Q2 1.03.0]"`

If no code is visible, the deal title alone is `promo_name`.

## Price label priority

1. `PMAP` (Promo MAP)
2. `MAP`
3. `HPP` (rarely used; some legacy decks)
4. `Special Price`
5. `Promo Price`

## Non-price labels (CRITICAL — NEVER treat as price)

- **`Dealer`** — wholesale cost
- **`M1`**, **`M2`**, **`M3`** — purchase tiers (NOT prices). Common
  trap. "M1" looks like a price column but it's a buying-volume tier.
- `List Price` — sometimes shown but always lower priority than MAP
- `Description`
- `QTY`

## Header signatures

Most common headers:
- `Model Description Dealer M1 MAP` (most common — 129+ pages in audit)
- `Model Description Dealer M1 MAP PMAP`
- `MODEL # LIST PRICE DEALER M1 MAP`

Signature tokens:
- `Model`, `Model #`
- `Description`
- `Dealer`
- `M1`
- `MAP`
- `PMAP`
- `HPP`
- `List Price`
- `Special Price`
- `Promo Price`
- `QTY`
- `Total Price`

## Quirks

- **`M1` is a purchase tier, NOT a price**. NEVER pick the `M1` column
  as the customer price. The customer price is `MAP` (or `PMAP` when
  shown).
- **`receive M1`** in promo title text refers to the M1 purchase tier,
  not a SKU named "M1" — ignore.
- **`Recommended Accessory` lines** are NOT part of the deal — exclude
  any SKUs on those lines.
- **Side-by-side tables** (Makita Q4 layouts): two parallel SKU+price
  tables on one page. See `edge-cases.md#side-by-side-tables-makita-q4`.
- **Date label**: `Promo Execution`.
- **XGT vs LXT vs CXT**: separate Makita platforms. A page may carry
  only XGT-prefix codes; treat platform name as a non-blocking hint —
  the SKUs and price labels are the same Makita conventions.
