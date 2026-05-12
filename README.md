# PCS Promo Parser

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

## Changelog
- **0.2.0** — Added RSA, POS Redemption, ARP exclusion types. Added brand-quarter-year file naming convention. Auto-detects quarter from filename or deck content.
- **0.1.0** — Initial release.
