# PCS Tools Plugin Marketplace

Two Claude Code plugins for the PCS Kit Builder and monthly MAP pricing workflows.

---

## pcs-promo-parser

Parses vendor promotional decks into the four Stage 1 CSVs for the PCS Kit Builder pipeline.

## Supported Vendors
Milwaukee, DeWalt, Makita, Bosch, EGO, Flex, GearWrench, Crescent

## Output Files
All files are named `<Vendor>-<Q#>-<YYYY>-<Type>.csv`, e.g.:
- `DeWalt-Q2-2026-Promo-List.csv`
- `DeWalt-Q2-2026-Non-Included.csv`
- `DeWalt-Q2-2026-NLP-Sheet.csv`
- `DeWalt-Q2-2026-Parser-Audit.csv`

## Exclusion Types
Handles: brick-and-mortar, SPIFF, RSA, spend-to-earn, POS Redemption, BMSM, promo-code-only, ARP, NLP/Special Buy, killed/strikethrough, missing-price.

## Additional Outputs (v1.0.0)
- `<Vendor>-<Q#>-<YYYY>-RSA-Kits.csv` — RSA kit promos with credit amounts
- `<Vendor>-<Q#>-<YYYY>-RSA-NLP.csv` — RSA NLP/price-drop promos with credit amounts
- `<Vendor>-<Q#>-<YYYY>-Needs-Pricing.csv` — Makita rows missing price (for manual fill-in)
- `<Vendor>-<Q#>-<YYYY>-Parser-Audit.csv` — Run summary counts

---

## pcs-map-updater

Merges a vendor MAP pricing sheet with a NetSuite ERP export, flags exceptions (missing ERP match, MAP below purchase price, promo MAP below purchase price, keep-price review), and exports a review-ready Excel workbook.

**Skill:** `/map-price-update`
**Inputs:** Manufacturer MAP sheet (CSV/XLS/XLSX) + NetSuite ERP export (CSV/XLS/XLSX)
**Output:** Excel workbook with MAP Update Review, Exceptions, All Manufacturer Rows, and Summary sheets.
**Source:** `plugins/map-price-updater/`

---

## Team Installation
See [docs/Team-Install-SOP.md](docs/Team-Install-SOP.md) for full step-by-step setup instructions.

## Changelog
- **1.0.0** — RSA promos route to dedicated RSA-Kits/RSA-NLP outputs with credit amounts. New-product launches skipped. Makita missing-price rows route to Needs-Pricing.csv. `Get N` / `Choice of N` promos emit C(M,N) combination rows. Price-label fallback rule applied across all vendors.
- **0.2.0** — Added RSA, POS Redemption, ARP exclusion types. Added brand-quarter-year file naming convention. Auto-detects quarter from filename or deck content.
- **0.1.0** — Initial release.
