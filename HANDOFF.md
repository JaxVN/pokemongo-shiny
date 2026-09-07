# HANDOFF — Shiny / Lucky Dex (`docs/`)

Last updated 2026-09-07. Written for a Claude Code session that clones this repo (e.g. in the cloud) and has **no access to the author's local machine**. Read "What you cannot see" before changing anything under `docs/`.

## 1. This repo holds two unrelated apps

| Path | What it is | Published? |
| --- | --- | --- |
| repo root (`index.html`, `src/`, `vite.config.js`, `package.json`) | the upstream `Rplus/pokemongo-shiny` Svelte app, inherited by the fork | no |
| `docs/` | **Shiny / Lucky Dex** — a standalone static app, the thing actually being worked on | yes |

`docs/` is what GitHub Pages serves. The Svelte app at the root is untouched legacy; don't build it, don't wire it into anything, and don't assume `package.json` describes `docs/`.

## 2. The `docs/` app

Static, vanilla JS, no build step, no dependencies to install.

```
docs/index.html                 the whole app: markup + logic in one file
docs/support.js                 runtime that renders index.html — do not edit
docs/data/dex.json              1,609 entries (species, regional forms, Mega/GMax/Origin, costumes)
docs/data/dexjourney-map.json   pid -> DexJourney variant key (see §5)
docs/assets/*.png               4 in-game badge icons (shiny, lucky, shadow, purified)
docs/.nojekyll                  required, or Pages hides files starting with _
```

Pokémon sprites are fetched at runtime from the PokeMiners CDN; no sprite files are committed.

`index.html` has two parts that `support.js` requires: an `<x-dc>` template (inline styles only, `{{ dotted.path }}` holes, `<sc-for>`/`<sc-if>`; **no expressions inside holes**) and a `<script data-dc-script>` holding `class Component extends DCLogic`. All computation lives in `renderVals()`, which returns flat values, styles and handlers. Two constraints learned the hard way: never put a `{{ }}` hole in an `<img src>` inside an `<sc-if>`, and handlers must be flat values on the `renderVals()` return, not nested on an object that can be null.

**Storage:** `localStorage['shinydex.profiles.v1']` is the profile registry; `localStorage['shinydex.marks.v2.<profileId>']` holds `{pid: bitmask}` per profile. Bits: Pokémon 1, Shiny 2, Lucky 4, XXL 8, XXS 16, G-MAX 32, MEGA 64, Shadow 128, Purified 256, \*100% 512. Migration from the older `shinydex.marks.v1` / `.v2` keys runs on first load — keep it.

## 3. Deploy

GitHub Pages is configured as **Deploy from a branch → `main` → `/docs`**, live at <https://jaxvn.github.io/pokemongo-shiny/>.

Pushing to `main` is the whole deploy. **Do not add a Pages Actions workflow** (`.github/workflows/*` with `actions/deploy-pages`) — the two source modes are mutually exclusive and adding one broke an earlier deploy with "Pages is not configured with GitHub Actions as the source". There is currently no `.github/` directory; keep it that way unless the Pages source setting is changed first.

After pushing, expect a cache lag: a browser may serve the old `index.html` for a minute or two even though Pages has rebuilt. Verify with a cache-busting fetch (`fetch('index.html?cb='+Date.now(), {cache:'no-store'})`) before concluding a deploy failed.

## 4. What you cannot see (important)

`docs/` is **not authored in this repo**. It is a copy of a bundle produced by Claude Design and delivered to a local folder outside the repo:

```
C:\Dev\Pokemon\Pokémon GO ShinyLucky Dex\design_handoff_shiny_lucky_dex\
  site/     the built app  -> copied verbatim into docs/
  source/   Shiny Lucky Dex.dc.html — the editable Design Component
```

A cloud session has none of that. Consequences:

- You may edit `docs/` directly — that is the only copy you have, and it deploys correctly.
- **Say so explicitly in your final message.** Any edit you make to `docs/` must be mirrored by the author back into `site/index.html` (byte-identical to `docs/index.html`) *and* `source/Shiny Lucky Dex.dc.html` (same body, different `<head>` — patch it, do not copy over it), or the next Design handoff drop will silently overwrite your work.
- Do not "fix" `docs/` to look generated, and do not add a build step for it.

Also local-only, not in this repo: `dexjourney-jaxvn-collection.csv` (a real DexJourney export, used as the format reference) and `shiny-lucky-dex.json` (the author's real saved marks, 298 entries, used as a test fixture).

## 5. Last session's change — CSV export (commit `3a34706`)

Two buttons added next to the existing Export JSON, in the block around the `doExport` handler.

**`Export CSV`** → `shiny-lucky-dex.csv`. One row per dex entry: 1,609 rows × 19 columns (`dex, name, form, costume, suffix, gen, shiny_released, debut, pid` + the 10 checklists). Nothing is lost. Written with a UTF-8 BOM so Excel renders `Flabébé` / `Farfetch'd` correctly — `Blob.text()` strips the BOM when you read it back, so verify via `arrayBuffer()` (`EF BB BF`), not `text()`.

**`Export CSV (DexJourney)`** → `shiny-lucky-dex-dexjourney.csv`, the import shape for <https://dexjourney.com/>: `dex_id, name, owned, shiny, lucky, xxl, xxs, mega, gmax, shadow, purified, perfect, variants_json`. One row per species; **only species with at least one mark are emitted**, so importing in their Merge mode is purely additive and never clears anything.

Their import contract, read from their own import dialog: a CSV needs only `dex_id`, `name`, `owned` plus any category fields — not all 89 columns of their export. Their export's **first row also carries account-level payload** (`journey_xp`, `achievement_tiers`, `profile_theme`, `saved_searches`); never populate those columns or you overwrite the user's profile.

### `docs/data/dexjourney-map.json`

584 entries, `pid -> slot`, loaded lazily only when the DexJourney button is clicked:

| Slot | Count | Meaning |
| --- | --- | --- |
| real catalog variant key | 477 | goes into `variants_json` |
| `mega` | 52 | sets the `mega` column |
| `gmax` | 14 | sets the `gmax` column |
| `custom-*` | 41 | no catalog entry exists; their catalog documents `custom-` as supported |

The remaining 1,025 entries of `dex.json` are plain species rows and are absent from the map (a missing key means "this is the species row").

Built from two public, side-effect-free catalogs: `dexjourney.com/variant-data.js` (591 variants, 24 kinds — includes `letter` for Unown, `gender`, `pattern`, `size`) and `dexjourney.com/costume-data.js` (307 costumes). The join is **not** mechanical: DexJourney keys costumes by release event *per species* (Ivysaur's party hat is `costume-jul2025`) while our pids use shared asset codes (`cJAN_2020_NOEVOLVE`), and our data uses emoji suffixes for colours/drives, which defeats label matching. 47 cases needed hand-written overrides. To regenerate after upstream data changes: re-fetch both catalogs, re-run the four-tier join (assetCode prefix → exact label/group → key echoes pid suffix → fuzzy label), and **verify every override still resolves against the catalog** before shipping.

Join on `dex_id`, never on `name`: Farfetch'd and Sirfetch'd differ by apostrophe (`’` in our data, `'` in theirs).

### Verified vs not

Verified end-to-end against the author's real 298-mark fixture on a local server, reading the generated blobs: 1,609/258 rows, uniform column widths, every `variants_json` parses, BOM present, and spot-checks — #89 Muk owned/xxl/perfect with variant `alolan` {shiny, xxl}; #150 Mewtwo `mega=true` folded from Mega X and Mega Y; #201 Unown with all 26 letters plus `exclamation`; #25 Pikachu's two GO Tour costumes mapped to `costume-gotour2024rei`/`akari`. Console clean.

**Not verified: an actual import into dexjourney.com.** That needs a logged-in account and would touch the author's real data. The format matches their documented requirements and the shape of their own export, but the round trip has never been executed. First real import should go to a guest profile.

## 6. Known gaps / candidate next work

- **Mega X / Mega Y collapse** into a single `mega` boolean on export. Their format has one column; nothing to do unless DexJourney adds per-variant Mega keys.
- **41 `custom-*` fallbacks** — mostly Primal Kyogre/Groudon (their catalog excludes temporary power states by design), female gender variants for several species, Keldeo and Dudunsparce formes. Would need DexJourney to extend their catalog.
- **No CSV import**, only export. Reading their CSV back into our bitmasks is the obvious symmetric feature and would reuse the same map inverted.
- Shadow, Purified, XXL, XXS and \*100% are user-marked only — no upstream release list exists, so a Shadow tick is possible on Pokémon that cannot be shadow.
- No PWA manifest or service worker; sprites are CDN-only so offline use needs a local cache.
- No URL state: active checklist and filters are not shareable or restored on reload.
- Only English names are used, though the upstream `name.csv` carries other languages.
