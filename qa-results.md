# Rehab Cost Estimator QA Results

**Date:** 2026-04-01
**Test Config:** 1500 SF, 3 bed, 2 bath, 1 story, National Average, 10% contingency, no tax

## Summary

**77/77 checks passed** (after fixes applied)

### Fixes Applied to `rehab-cost-estimator.html`

1. **Match string fixes (6 orphan patterns):**
   - `backsplash — subway tile` -> `backsplash — ceramic subway` (missing "ceramic" in pattern)
   - `window — vinyl double-hung` -> `window replacement — vinyl double-hung` (missing "replacement")
   - `garage door (single)` -> `garage door (single,` (closing paren didn't match ", 8x7)")
   - `garage door (double)` -> `garage door (double,` (same issue with ", 16x7)")
   - `attic (blown-in)` -> `attic insulation` (item name is "Attic insulation (blown-in)")
   - `wall (blown-in` -> `wall insulation` (item name is "Wall insulation (blown-in, existing)")
   - `stain-blocking primer` -> `primer (stain` (item name is "Primer (stain-blocking)")

2. **Auto-quantity fix:**
   - Added `getAutoQty` rule for `Landscaping & Exterior Cleanup` per-SF items (sod installation)
   - Previously defaulted to qty=1 instead of living SF

## Preset Totals

| Preset | Items | Subtotal | + 10% Contingency | $/SF |
|--------|-------|----------|-------------------|------|
| Cosmetic | 7 | $16,440 | $18,084 | $12 |
| Light | 24 | $24,029 | $26,432 | $18 |
| Medium | 48 | $96,854 | $106,539 | $71 |
| Heavy | 68 | $181,495 | $199,645 | $133 |
| Gut | 83 | $366,803 | $403,483 | $269 |

## Check Categories

### 1. Preset Item Selection (12/12 passed)
- [x] All 5 presets: every match pattern finds a COST_DATA item (no orphans)
- [x] No duplicate matches causing unexpected multi-item selection
- [x] Progressive item counts: 7 < 24 < 48 < 68 < 83
- [x] Cosmetic items are a subset of Light items
- [x] Light items are roughly subset of Medium (3 expected tier changes: cabinet refacing -> semi-custom, laminate counters -> quartz, fiberglass tub -> tile surround)

### 2. Quantity Logic (39/39 passed)

**Electrical:**
- [x] Light fixtures = rooms (bed+bath+kitchen+living) = 7
- [x] Recessed lighting = rooms*2 = 14 (heavy/gut only)
- [x] Recessed lighting NOT in cosmetic/light
- [x] Ceiling fans = bedrooms = 3 (medium+ only)
- [x] Smoke/CO detectors = rooms = 7
- [x] GFCI outlets = bath+1 = 3

**Plumbing:**
- [x] Faucets = bath+1 = 3
- [x] Toilets = bathrooms = 2
- [x] Garbage disposal = 1
- [x] Exhaust fans = bathrooms = 2

**Interior:**
- [x] Interior doors = bed*2+bath+2 = 10
- [x] Cabinet painting = SF*0.015 = 23 LF
- [x] Interior paint (walls+trim+ceilings) uses paintable SF = 2553 (not 1500)
- [x] Bathroom mirrors = bathrooms = 2
- [x] Vanity single = bathrooms = 2

**Flooring:**
- [x] Cosmetic: only LVP (no other flooring types)
- [x] Light: LVP + bathroom tile only
- [x] Bathroom floor tile = bathrooms = 2
- [x] No preset includes both LVP and hardwood (mutually exclusive)

**Kitchen:**
- [x] Appliance package = 1
- [x] Range hood = 1
- [x] Kitchen sink = 1
- [x] Countertop uses per-SF pricing

**Bathroom:**
- [x] Tub/shower = bathrooms = 2
- [x] Full bathroom remodel NOT in any preset (users can add manually)

**Roofing:**
- [x] Roof tear-off = Roof SF = 1725 (1500*1.15/1)
- [x] Shingles = Roof SF = 1725

**Exterior:**
- [x] Exterior paint = wall SF = 1053
- [x] Siding = wall SF = 1053

**Landscaping:**
- [x] Yard cleanup = 1 (lump sum)
- [x] Sod = living SF = 1500
- [x] Tree removal = 2 (gut only)

**Misc:**
- [x] Cosmetic: only general cleaning
- [x] Light: cleaning + dumpster
- [x] Medium: cleaning + termite
- [x] Heavy: cleaning + termite
- [x] Gut: cleaning + termite + mold

**Admin:**
- [x] Medium+: permits, insurance, holding costs, appraisal
- [x] NOT in cosmetic or light

### 3. Tier Consistency (10/10 passed)
- [x] Cosmetic: all items request 'contractor' tier
- [x] Light: all items request 'contractor' tier
- [x] Medium: all items request 'mid_range' tier
- [x] Heavy: all items request 'mid_range' tier
- [x] Gut: all items request 'high_end' tier
- [x] All fallbacks valid when preferred tier is null

### 4. Cost Sanity (9/9 passed)
- [x] Cosmetic: $12/SF (range: $10-20)
- [x] Light: $18/SF (range: $15-35)
- [x] Medium: $71/SF (range: $50-85)
- [x] Heavy: $133/SF (range: $100-150)
- [x] Gut: $269/SF (range: $200-300)
- [x] Each level total > previous level

### 5. Edge Cases (5/5 passed)
- [x] 800 SF studio (0 bed, 1 bath): no division by zero, all calculations valid
- [x] 4000 SF (5 bed, 4 bath, 2 story): light fixtures = 11 rooms
- [x] 2-story: roof SF halves = 2300 (4000*1.15/2)
- [x] 5 bed/4 bath: interior doors = 16 (5*2+4+2)
- [x] Studio (0 bed, 1 bath): interior doors = 3 (0*2+1+2)

## Calculated Values Reference (1500 SF, 3/2, 1 story)
- Rooms: 7 (3 bed + 2 bath + kitchen + living)
- Roof SF: 1725 (1500 * 1.15 / 1)
- Floor SF per story: 1500
- Perimeter: ~155 ft (4 * sqrt(1500))
- Wall SF: 1053 (perim * 8ft * 1 story * 0.85)
- Paintable SF: 2553 (wall 1053 + ceiling 1500)
