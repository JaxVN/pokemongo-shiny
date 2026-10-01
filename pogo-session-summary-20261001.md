# PoGo Project - Session Summary
**Date:** October 1, 2026  
**Focus:** DexJourney Data Import & Variant Catalog Issues  

---

## ✅ Summary
Session focused on data integrity issues when importing Pokemon data from DexJourney into PoGo catalog. Main problem: missing Pokemon entries and variant key mismatches. Identified root causes and documented export workflows.

---

## 🔍 What Was Investigated

### 1. **DexJourney Import Discrepancy**
   - **Issue:** Import from dexjourney.com only brings in a subset of Pokemon
   - **Finding:** Need to check DexJourney integration and UI to ensure all Pokemon are included in initial import
   - **Status:** ⏳ Pending - requires validation of DexJourney data completeness

### 2. **Export CSV Data Mismatch**
   - **Variant 1 (1,609 rows):** Sheets/Excel export - full raw data
   - **Variant 2 (1,025 rows):** DexJourney export - filtered data with specific columns
     - Columns: `dex_id`, `name`, `owned`, `shiny`, `lucky`, `xxl`, `xss`, `mega`, `gmax`, `shadow`, `purified`, `perfect`, `variants_json`
   - **Issue:** Variant key mismatch - variants_json field not parsing correctly in some entries
   - **Status:** 🔄 In Progress - need variant catalog resolution

### 3. **Variant Catalog Issues**
   - **Finding:** Catalog has 591 entries but form parsing breaks for non-costume variants
   - **Problem:** Form keys not resolving from browser - needed manual code-based resolution
   - **Root Cause:** Catalog form key generation relies on specific JSON structure that some variants don't have
   - **Impact:** Can't reliably match Pokemon to their variant forms without fix
   - **Status:** ⏳ Pending fix

### 4. **Checklist Incomplete**
   - Current checklist of 1,025 Pokemon is not representative of full available data
   - Missing Pokemon need to be identified and added to checklist
   - No test case for batch import of missing entries
   - **Status:** ⏳ Needs reconciliation logic

---

## 🔄 What's In Progress

| Item | Current Blocker | Next Step |
|------|-----------------|-----------|
| DexJourney Import | Need to verify all Pokemon are fetched | Check DexJourney API/UI for filtering logic |
| Variant Catalog (591) | Form key parsing doesn't handle non-costume variants | Build variant key catalog with proper resolution |
| Checklist Data | 1,025 entries vs 1,609 available | Merge logic needed - identify & import missing entries |

---

## ⏳ What's Pending

1. **DexJourney Data Validation** 
   - Confirm if DexJourney limits to certain Pokemon subset or if our import logic is incomplete
   - May need to use Sheets/Excel export (1,609) as source of truth instead

2. **Variant Key Resolution**
   - Build proper catalog for variant forms → form key mapping
   - Test all variant types (costume, mega, gmax, shadow, purified, etc.)

3. **Checklist Merge & Sync**
   - Load 1,609 data from Sheets export
   - Identify which 584 Pokemon are missing (1,609 - 1,025)
   - Add missing to checklist with proper variant data

4. **UI/UX Considerations**
   - When importing, need to handle case where user has partial checklist
   - Form selection should work for all variants after catalog fix

---

## 🎯 Key Decisions Made

| Decision | Reasoning | Trade-off |
|----------|-----------|-----------|
| Use DexJourney export for initial import | User-friendly source, filtered data | Limited to 1,025 Pokemon; missing data |
| Supplement with Sheets export (1,609) | More complete data set | Raw format, needs parsing |
| Separate variant catalog from main data | Cleaner code structure, reusable | Extra step to maintain variant mappings |

---

## ⚠️ Known Issues & Risks

- **Data Integrity Risk:** 584 Pokemon in Sheets export not in DexJourney import - could confuse users if they see different totals
  - *Mitigation:* Clearly communicate data source; provide both import options
  
- **Form Key Parsing Bug:** Non-costume variants fail to resolve form keys from browser
  - *Status:* Need code fix to handle missing/edge-case variant structures
  - *Temporary Workaround:* Use static variant catalog instead of browser resolution

- **Import Strategy Unclear:** No clear UX for "merge missing Pokemon into existing checklist"
  - *Next:* Define import modes (full replace vs. merge)

---

## 📋 Next Steps (Priority Order)

1. **IMMEDIATE** 
   - [ ] Decide: Use DexJourney (1,025) or Sheets (1,609) as primary data source?
   - [ ] If Sheets: Build import parser for raw CSV format

2. **SHORT-TERM**
   - [ ] Build variant catalog with proper form key mapping (591 entries)
   - [ ] Test variant parsing for all types (costume, mega, gmax, shadow, purified, perfect)
   - [ ] Identify missing 584 Pokemon and reconcile checklist

3. **MEDIUM-TERM**
   - [ ] Implement merge logic for checklist updates
   - [ ] Add unit tests for variant key resolution
   - [ ] Update DexJourney integration if data source is incomplete

---

## 📂 Files/Resources to Check

- DexJourney integration code (data import)
- Variant catalog (591 entries)
- Sheets/Excel export file (1,609 Pokemon, checklist data)
- Form key parsing logic (currently browser-based, breaking for some variants)

---

## 💬 Open Questions for Next Session

1. What's the canonical source of truth for Pokemon data? (DexJourney vs Sheets)
2. Should variant forms be optional in checklist, or required for completeness tracking?
3. How to handle user expectations when import shows fewer Pokemon than UI promises?

