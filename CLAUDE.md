# Shop Sales Dashboard — CLAUDE.md

## Project overview

Single-file static site (`index.html`, ~3200 lines) deployed via GitHub Pages at:
`https://saffron-oneldn.github.io/shop-sales/`

Data is stored in Supabase (project `ljjwssicvvyyueyznmou`). There is **no build step** — edit `index.html`, push to `main`, live in ~30s.

---

## Tech stack

| Layer | Detail |
|---|---|
| Frontend | Vanilla HTML/CSS/JS — single `index.html` |
| Charts | Chart.js 4.4.1 (CDN) |
| PDF parsing | pdf.js 3.11.174 (CDN) |
| CSV parsing | PapaParse 5.4.1 (CDN) |
| Database | Supabase (raw `fetch` — no SDK) |
| Deployment | GitHub Pages (`main` branch → live instantly) |

---

## Supabase tables

| Table | Purpose |
|---|---|
| `shop_sales` | POS records uploaded via WodBoard CSV |
| `shop_product_lookup` | Product master: `product_name_raw`, `display_name`, `category`, `brand`, `sell_price`, `active`, `min_stock_units` |
| `shop_deliveries` | Received stock: `delivery_date`, `product_name_raw`, `quantity_received`, `unit_cost`, `supplier`, `order_no`, `pack_size`, `volume_ml` |
| `shop_consumable_usage` | Weekly consumable usage log (quantity used per product per date) |
| `shop_stock_takes` | Manual stock counts (calibrates estimated stock) |

Supabase client is raw fetch via helpers at lines 897–934:
- `sbGet(table, params)` — GET with query string
- `sbInsert(table, row)` — POST single row
- `sbUpsert(table, rows, onConflict)` — POST with upsert
- `sbDelete(table, id)` — DELETE by id

---

## Tab structure

```
Weekly Summary | Top Sellers | Trends | Consumables | Order Prediction
```

- Tab buttons in HTML at lines ~568–572, `data-tab` attribute drives JS
- Matching `<div class="section" data-tab="...">` panels follow
- Tab switching set up in `init()` — any new `data-tab` button + section works automatically
- `renderAll()` (line ~2350) dispatches to the correct render function based on active tab

---

## Key functions

| Function | Line | Purpose |
|---|---|---|
| `sbGet/sbInsert/sbUpsert/sbDelete` | 897–934 | All Supabase CRUD |
| `loadProductLookup()` | 1331 | Fetches `shop_product_lookup` into `productLookup` object |
| `loadFromSupabase()` | 1341 | Loads sales + deliveries, enriches records |
| `renderAll()` | ~2350 | Dispatches render based on active tab |
| `renderOrderPrediction()` | 2146 | Velocity/stock calc + order flag rendering |
| `renderDeliveryLog()` | 2080 | Fetches + renders delivery history table |
| `autoDetectSupplier(textLines)` | 2749 | Returns `'muscle-finesse'`, `'tapstitch'`, `'enviropack'`, or `'generic'` |
| `parseTapstitchInvoice(lines)` | 2759 | Tapstitch PDF → items array |
| `parseEnviropackInvoice(lines)` | ~2813 | Enviropack PDF → consumables items array |
| `parseAmazonCSV(csvText)` | 2825 | Amazon CSV → cafe + consumable items (routed by ASIN) |
| `handleInvoicePdfs(files)` | 2882 | Orchestrates PDF upload → detect → parse → review modal |
| `renderPdfReviewTable(items, metas)` | 2941 | Renders pre-log review table |
| `extractInvoiceItems(text)` | 2612 | Muscle Finesse line-level parser |
| `fuzzyMatchProduct(name)` | 2517 | Jaro-Winkler match against `productLookup` |
| `renderConsumables()` | — | Loads consumables data + renders all 3 sub-panels |
| `loadConsumablesData()` | — | Fetches deliveries, usage, and stock takes for consumables |
| `calcEstStock(productName)` | — | Stock estimate from last take + deliveries − usage |
| `calcVelocity(productName)` | — | Weekly usage rate over 8-week window |
| `openStockTake()` / `submitStockTake()` | — | Stock take subpage logic |

---

## Suppliers

| Supplier | Format | Parser | Routes to |
|---|---|---|---|
| Muscle Finesse | PDF | `extractInvoiceItems()` | `shop_deliveries` (cafe) |
| Tapstitch | PDF | `parseTapstitchInvoice()` | `shop_deliveries` (merch) |
| Enviropack | PDF | `parseEnviropackInvoice()` | `shop_deliveries` (consumables) |
| Amazon | CSV | `parseAmazonCSV()` | cafe or consumables (ASIN routing) |
| Future Supplies | PDF | `genericParser()` (when invoices arrive) | consumables |
| Out of Eden | PDF | `genericParser()` (when invoices arrive) | consumables |

---

## Consumables architecture

The Consumables tab (5th, before Order Prediction) tracks 23 non-revenue supplies.

**Stock estimation formula:**
- If a stock take exists: `lastTake.actual_count + deliveries_since − usage_since`
- Else: `sum(all deliveries) − sum(all usage)`

**Velocity:** `sum(usage in last 8 weeks) / 8` → units/week

**Subcategory map** is a JS constant (`CONS_SUB`) — not stored in DB.

**ENVIROPACK_SKUS** hardcoded map (SKU → product_name_raw):
`E11060/E11080/E11120/E17314/E20080/E20120`

**AMAZON_CONSUMABLE_ASINS** — ASIN → product_name_raw for consumable items from the Feb–May 2026 Amazon order history.

---

## Consumables product list (22 products)

**Cleaning:** Bin Bags 80L, Jumbo Toilet Rolls, Clinell Disinfectant Wipes, Microfibre Cloths, Nitrile Gloves, Toilet Brushes, Kleenex Tissues
**Bathing:** Flo Period Pads, Flo Tampons, Sure Men Deodorant, Sure Women Deodorant
**Gym Supplies:** Bubble Tea Straws, Plastic Food Bags, Magnesium Chalk, WINALOT Dog Treats
**Cafe Supplies:** Enviropack 6oz Cups, Enviropack 8oz Cups, Enviropack 12oz Cups, Enviropack Lids, Enviropack 16oz Cups, Enviropack 20oz Cups, Wooden Coffee Stirrers

---

## Deployment

```bash
git add index.html
git commit -m "message"
git push origin main
# Live at https://saffron-oneldn.github.io/shop-sales/ in ~30s
```

No CI, no tests, no build step. Open the live URL after push to verify.

---

## Theme tokens

| Token | Value |
|---|---|
| Background | `#0a0a0a` |
| Surface | `#111111` |
| Border | `#1e1e1e` |
| Primary accent (gold) | `#e8c547` |
| Gold hover | `#c9a93a` |
| Text | `#f0f0f0` |
| Muted | `#666666` |
| Green | `#4ade80` |
| Red | `#f87171` |

Buttons: `background: #e8c547; color: #0a0a0a` — hover `background: #c9a93a`
Section margin: 32px. Tab padding: 8px, gap: 8px.
KPI cards: `position:relative; overflow:hidden` + 3px `::before` accent bar.

---

## DO NOT

- Add a build system, bundler, or npm dependencies
- Split into multiple files — intentionally a single `index.html`
- Import the Supabase JS SDK — use the raw fetch helpers already in the file
- Push to `main` without confirming with Saffron if the change affects live data writes
