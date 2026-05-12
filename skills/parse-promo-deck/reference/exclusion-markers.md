# Exclusion markers — exact phrases and traps

Each marker triggers one specific exclusion reason or routing. The
phrases below are the patterns the rule parser uses in
`kb/deck_parser/patterns.py`. As a vision-aware AI, you can be slightly
more flexible (e.g. spot a phrase in an image badge that isn't in the
text layer), but you should never be **less** strict.

Several markers have **false-positive traps** documented after the
phrase lists. Read those carefully before deciding.

---

## SPECIAL_BUY_MARKER → NLP routing

(Routes to `nlp_sheet.csv`, NOT `non_included.csv`)

### Phrases (case-insensitive)

- `Special Buy` (also `Special Pricing`)
- `EDLP`, `Everyday Low Price`
- `New Lower Price`, `NLP`
- `Price Reduction`, `Price Drop`, `Price Cut`
- `Permanent Price Change`
- `Clearance`
- `(Get) X% Off` where X is 1–2 digits, e.g. `20% Off`, `Get 15% Off`
- `Promote At X%` where X is a digit

### Real-world examples that should match

- "Get 20% Off Heavy-Duty Pricing, Promote At 15% Off to User"
- "Permanent Price Change — see Online Execution"
- "% Off" badge on a price box

### Traps

- Bare `% Off` with no digit (e.g. "% Off Select Items" without a
  specific percent) is too vague — don't trigger on this alone.
- "Off" in narrative copy ("buy this off the rack") obviously doesn't count.

---

## NLP_MARKER → NLP routing

Subset of SPECIAL_BUY_MARKER specifically for the `NLP` acronym and
"New Lower Price" phrase. When NLP_MARKER hits, set `source_marker =
"nlp"` on the emitted NLPRow; for other Special-Buy variants set
`source_marker = "special-buy"`.

### Phrases

- `NLP` (bare acronym, word-boundary)
- `New Lower Price`

---

## BM_MARKER → `non_included` reason `brick-and-mortar`

### Phrases (case-insensitive)

- `Brick & Mortar`, `Brick and Mortar`, `Brick-Mortar`
- `In-Store Only`, `In Store Execution Only`, `In-Store Exclusive`
- `Not Available Online`, `Not For Online`
- `Branches Only`, `Branch Only`
- `Club Only`
- `Direct Ship Only`
- `B&M` (with optional spaces around the `&`)

### Traps

- "in store" as part of a sentence ("available in store now") may or
  may not be B&M-only — look for accompanying language confirming
  exclusion ("only", "exclusive", "not available online").

---

## SPIFF_MARKER → `non_included` reason `spiff`

### Phrases

- `SPIFF` followed by an optional `$N` amount: `SPIFF $20`, `SPIFF $100`
- `Sales SPIFF`
- `Rep Incentive`

### Traps

- "Spiffy" or "spiff up" in narrative copy isn't a SPIFF program.
  Trigger only on the all-caps acronym or the explicit phrases.

---

## RSA_MARKER → `non_included` reason `rsa`

RSA (Retail Sales Associate) incentive programs reward store associates
for selling specific products. These are associate-facing, not
customer-facing promos — do not kit them.

### Phrases (case-insensitive)

- `RSA` (standalone word-boundary, not embedded in a model number)
- `Retail Sales Associate`
- `RSA Reward`, `RSA Incentive`, `RSA Program`
- `Associate Reward`, `Associate Incentive`
- `Store Associate Incentive`
- `RSA Deck`, `RSA Program Page`, `RSA Offer`

### Real-world examples that should match

- "RSA Reward: Sell 3 units, earn $15"
- "Retail Sales Associate Incentive Program"
- "RSA: Earn $10 per kit sold"

### Traps

- `RSA` embedded inside a model number or SKU (e.g. `RSAB123`) is NOT
  this marker. Match only when `RSA` appears as a standalone label,
  section header, or badge.
- A page that contains BOTH an RSA reward section AND a valid B1G1 kit
  table should be classified as a **kit page** (priority #9), not `rsa`.
  The kit deal is customer-facing and still needs to be extracted. Emit
  a separate `non_included` row for the RSA section only if there are
  RSA-specific SKUs that are not part of the B1G1 deal.
- An entire page dedicated exclusively to RSA programming (no B1G1
  panel, no customer promo price table) → `non_included` reason `rsa`.

---

## SPEND_TO_EARN_MARKER → `non_included` reason `spend-to-earn`

### Phrases

- `Spend $N (to) (earn|get|receive) ...` — e.g. "Spend $500 to earn a $50 rebate"
- `Earn $N ...`
- `$N rebate`, `$N in promo bucks`, `$N in bucks`
- **`Buy $N in <category>`** where `N` is 2+ digits and `<category>` is
  a category word (concrete, diamond, accessories, products, items).
  Real-world examples this catches:
  - "Buy $800 in Concrete Accessories - Get (1) of Your Choice Free"
  - "Buy $1000 in Diamond Cutting Accessories - Get (1) M18 Fuel Grinder Free"

### Traps

- The `Buy $N in/of/worth` pattern requires **2+ digits** in the
  dollar amount. This is deliberate so a legitimate kit titled "Buy a
  $99 tool, get a battery free" doesn't trip it. If you see `Buy $9`
  or `Buy $99` *in narrative*, that's not spend-to-earn.
- "Earn rewards" / "earn miles" in a credit-card aside isn't a vendor
  spend-to-earn program.

---

## POS_REDEMPTION_MARKER → `non_included` reason `pos-redemption`

Point-of-sale redemption programs, mail-in rebates, and similar
redemption mechanisms where the customer must submit something (a form,
card, or receipt) to receive the benefit. There is no fixed free-good
SKU paired at the kit level — the deal is processed through a
separate redemption channel.

### Phrases (case-insensitive)

- `POS Redemption`, `Point of Sale Redemption`
- `Redeem at POS`, `Redeem at Register`
- `In-Store Redemption`, `Redemption at Checkout`
- `Mail-In Rebate`, `Mail In Rebate`, `MIR`
- `Rebate Form`, `Rebate Card`
- `Instant Rebate` (when no fixed free-good SKU is paired alongside it)
- `Savings Card`, `Redemption Card`
- `Redeem Online`, `Online Redemption` (when no accompanying B1G1 table)

### Real-world examples that should match

- "Mail-In Rebate: Receive $50 back by mail"
- "POS Redemption — present this page at checkout"
- "Instant Rebate: $30 off at register" (standalone, no free good)

### Traps

- **Spend-to-earn has higher priority** (decision tree step 4 beats
  step 4a). If a page has both a dollar-threshold spend trigger AND a
  POS redemption mechanism, classify as `spend-to-earn`.
- A page with a POS redemption badge **plus** a full B1G1 price table
  with paired free-good SKUs → classify as a **kit page** (priority #9).
  The kit structure is what matters; the POS badge is just the delivery
  mechanism.
- `Instant Rebate` paired with a full MAP/SKU price column may route to
  **NLP** (priority #7) instead of `pos-redemption` if a customer-facing
  shelf price is clearly extractable. Prefer NLP routing when in doubt
  and a price is visible.

---

## BMSM_MARKER → `non_included` reason `buy-more-save-more`

### Phrases

- `Buy More Save More`, `Buy More, Save More`
- `BMSM`
- `Volume Discount`, `Volume Pricing`, `Volume Savings`
- `Tiered Discount`, `Tiered Pricing`, `Tiered Savings`
- `The More You Buy ...`
- `Buy N+ (and) (get|save) ...` (e.g. "Buy 5+ save 10%", "Buy 3 save $20")

### Traps

- "Buy more of these tools" in marketing copy isn't BMSM unless tied
  to a tiered structure.

---

## PROMO_CODE_MARKER → `non_included` reason `promo-code-only`

### Phrases (case-insensitive)

- `Discount Code`
- `Coupon Code`
- `Checkout Code`
- `Use Code XYZ123` (with an actual code following)
- `Enter Code XYZ123` (with an actual code following)
- `Apply Code At Checkout`

### CRITICAL TRAP — FLEX deal identifiers

**`PROMO CODE: SOT2514219` is NOT a checkout discount code.** FLEX
uses `PROMO CODE: <identifier>` as the deal's tracking identifier
label on EVERY kit page in their decks. If your only signal is the
literal phrase `PROMO CODE:` followed by a code, that's NOT enough to
classify the page as promo-code-only.

The marker requires one of:
- The literal word "discount" / "coupon" / "checkout" preceding "code"
- An imperative verb ("use", "enter", "apply") preceding the code
- "Apply code at checkout" phrasing

A bare `PROMO CODE: SOT...` on a FLEX page should be treated as a
deal identifier — extract it as the PCE-equivalent identifier in
`promo_name` (the same way you'd handle Milwaukee PCE).

---

## ARP_MARKER → `non_included` reason `arp`

ARP (Authorized Retailer Program / Authorized Retail Program) deals
are channel-restricted — they are available only to specific authorized
retailer accounts and cannot be offered through general online
channels. Do not kit these.

### Phrases (case-insensitive)

- `ARP` (standalone word-boundary, not embedded in a model number or SKU)
- `Authorized Retail Program`, `Authorized Retailer Program`
- `Authorized Retail Only`, `Authorized Retailer Only`
- `ARP Only`, `ARP Price`, `ARP Exclusive`, `ARP Promo`
- `Authorized Dealer Only`, `Authorized Dealer Program`
- `Dealer Exclusive` (when tied to an ARP label or authorized-channel language)

### Real-world examples that should match

- "ARP: Authorized Retail Program — Available to ARP accounts only"
- "ARP Exclusive Pricing — Not for general online sale"
- "Authorized Retailer Only — See your rep for details"

### Traps

- `ARP` embedded inside a model number (e.g. `DCARP204`) is NOT this
  marker. Match only when `ARP` appears as a standalone program label,
  section header, or badge.
- Emit **one `non_included` row per page** (with the page's primary SKU
  if extractable, blank otherwise). Do NOT emit kit rows for ARP pages
  even if a price table is present — the deal is channel-gated.
- A page marked ARP that also carries an NLP / Special Buy price column
  → still route to `non_included` reason `arp`. Channel restriction
  takes precedence over NLP routing for ARP pages.

---

## Strikethrough detection

The rule parser uses pdfplumber to detect horizontal rules overlapping
character bounding boxes. As an AI, you detect strikethrough
**visually**:

- A SKU with a horizontal line passing through the middle of the text
  is struck.
- A SKU rendered in light gray with a red "X" or "CANCELLED" badge
  near it counts as killed.
- Whole panels covered with a "DEAL CANCELLED" or large diagonal strike
  count as page-level kill (priority #1 in classification).

Routing:
- **Per-SKU strikethrough** in an active panel → `non_included` reason
  `strikethrough`; the rest of the page parses normally.
- **Whole-panel strikethrough** → `non_included` reason `killed` (page
  classified as case #1; no other rows emitted from that page).

---

## Recommended-accessory exclusion

Lines marked `Recommended Accessory` or `Sold Separately` (case-
insensitive) are NOT part of the deal — they're related-product
suggestions. Exclude any SKUs on those lines from extraction.

This is common on Makita pages: the main promo lists a tool, then
shows a "Recommended Accessory" line with a chuck or blade that's
NOT free with the deal.
