# HANDOFF — PoGo Trade Manager (Phase 1)

Written 2026-10-05 for a Claude Code session on the web that has **only this repo** (no local machine, no Drive). Supersedes the earlier planning brief, which assumed a Svelte app that is not what is deployed. Read `HANDOFF.md` too — it covers the `docs/` app internals and the "what you cannot see" caveats; this file does not repeat them.

## 1. Goal

Grow the **Shiny / Lucky Dex** (`docs/`, live at <https://jaxvn.github.io/pokemongo-shiny/>) from a checklist into a **personal multi-account trade manager**. The owner (Jax) runs 4–6 Pokémon GO accounts and trades in Facebook groups: today he posts a screenshot, gets offers, and eyeballs inventories. The app should make that manageable. A community platform is Phase 2 — out of scope.

## 2. Ground truth (verified against the repo)

- **The deployed app is `docs/`** — vanilla JS, single `docs/index.html`, no build step, no dependencies. Pages source is *Deploy from a branch → `main` → `/docs`*. Do **not** add a Pages Actions workflow or a `.github/` directory.
- **The root Svelte/Vite app is unpublished legacy.** Do not build `.svelte` components there; they would deploy nothing. Port any idea into `docs/` as plain JS. Preserve the `<x-dc>` template + `class Component extends DCLogic` structure (constraints in `HANDOFF.md` §2). `docs/support.js` — do not edit.
- **Data model today:** `docs/data/dex.json` has 1,609 entries `{d, n, p, g, f, c, b, t, r, dt, sf}` (`d` dex no., `n` name, `p` pid like `pm1`, `pm3.fMEGA`, `pm1.cJAN_2020_NOEVOLVE`, `f` form, `c` costume, `b` ?, `sf` suffix). Per-profile marks live in `localStorage['shinydex.marks.v2.<profileId>']` as `{pid: bitmask}`; the profile registry is `shinydex.profiles.v1`. Bits: Pokémon 1, Shiny 2, Lucky 4, XXL 8, XXS 16, G-MAX 32, MEGA 64, Shadow 128, Purified 256, \*100% 512. Keep the migration from older keys.
- **Multi-profile already exists** (profile registry + per-profile marks). The "AccountSwitcher" in the old backlog is largely done — extend it, don't rebuild it.
- **The data model is bitmask-per-pid, not rows with CP.** The old brief's CSV schema (`name,cp,shiny,lucky,form,gender,note`) describes *individual Pokémon*; the app stores *which dex entries are ticked*. Reconciling these is the central design question (§5).
- Existing CSV **export** (shiny-lucky-dex.csv and the DexJourney shape) is in `docs/index.html` near `doExport`; `docs/data/dexjourney-map.json` maps pid → DexJourney slot. There is **no CSV import** yet.
- License: `LICENSE` has only `Copyright (c) 2024 Rplus`. The owner intends dual copyright (Rplus 2024 + JaxVN 2026), MIT. **Ask before editing** — see §6.

## 3. Locked decisions (do not re-litigate)

| Decision | Choice |
| --- | --- |
| Hosting | Static site, all logic client-side, no secrets in repo |
| Client storage | `localStorage` (each visitor owns their data; no backend, no login) |
| Sync / share | Google Sheet or CSV (export/import), later |
| Image recognition | **BYO-AI** — the app ships a copy-paste prompt; the user runs it in ChatGPT/Gemini/Claude on their phone and pastes the CSV back. **No built-in vision, no API keys.** |
| Multi-account | 4–6 accounts switchable in one app |
| Trade model | FT (For Trade) / LF (Looking For) dual list — the PokeXperience / 9db.jp convention |
| UI language | Bilingual VI/EN for user-facing copy |
| Source of truth for schema | the CSV parser; prompt + UI must stay in sync with it |

## 4. Missing assets — read this first

The previous brief said three files were "already built". **They are not in this repo** (not in the tree, any branch, or any commit) and not on Drive. Check the repo root / `docs/` for them first — the owner may have committed them since:

- `csvParser.js` · `pogo-ai-extraction-prompt.md` · `pogo-guide-ai-import.md`

If they are **present**: use them as-is; apply the one known cleanup (in `toCsv()`, replace the redundant `h === 'shiny' || h === 'lucky' ? r[h] : r[h]` with `EXPECTED_HEADERS.map((h) => esc(r[h]))`).

If they are **still absent**: stop and ask the owner to commit them, or get explicit OK to rewrite from this contract (the rewrites are new and untested — say so in the PR):

```
Columns (header row required):  name,cp,shiny,lucky,form,gender,note
name    official English species name, apostrophes kept (Farfetch'd, Mr. Mime). REQUIRED
cp      integer or blank
shiny   true/false      lucky  true/false
form    Alolan|Galarian|Hisuian|Paldean|Mega|Gigantamax|Dynamax|Costume|Shadow|Purified,
        costumes as "Costume: <desc>", else blank
gender  male|female|blank
note    free text; the word "verify" flags an AI-uncertain row
Prompt output contract: strict 7-column CSV + a trailing "Rows: N" line.

parsePokemonCsv(raw)            -> {rows, errors, warnings, count}
                                   strips ```csv fences and "Rows: N" lines
splitDuplicates(incoming, existing) -> {unique, dupes}   key = name|cp|form
toCsv(rows)                     -> string
```

Put the parser at `docs/lib/csvParser.js` (plain ES module or classic script — `docs/index.html` has no bundler, so confirm how you load it; if `support.js` blocks module imports, inline it).

## 5. Open decisions — ask the owner, don't guess

1. **Row model vs bitmask model.** CSV rows are individual Pokémon (with CP); the app stores ticks per dex entry. Options: (a) import maps each row onto a pid and sets Shiny/Lucky bits (lossy: CP and duplicates dropped, trivial to build); (b) add a second per-profile store of individual Pokémon rows (needed for real trading, since you trade a specific 3★ Mewtwo, not "a shiny Mewtwo"). **Recommend (b) for FT/LF, with (a) as an optional "also tick the dex" action.** Needs the owner's call before the Import UI is built.
2. **`account` column.** The schema has none; the plan assigns the account in the UI at import. Decide before Sheet sync whether one CSV carries all accounts (then add `account` to both the prompt and the parser).
3. **Name+form → pid mapping.** Imported `name` + `form` must resolve to `dex.json` pids (e.g. `Alolan` → the entry with `f` Alolan; `Costume: Party Hat` → `c`). Decide on fuzzy-match behaviour and what to do with unmatched rows (recommend: keep them, flagged, in the preview).
4. **LICENSE** dual-copyright line — confirm, then add `Copyright (c) 2026 JaxVN`; keep Rplus 2024.

## 6. Build order (each a separate PR, branch from `main`)

1. **Import panel** (`docs/index.html`) — paste CSV → `parsePokemonCsv` → editable preview (fix shiny/cp/form inline, `needsVerify` rows highlighted) → `splitDuplicates` against the active profile → choose profile → Confirm → persist. Closes one full loop and exercises the parser. Needs §5.1 and §5.3 answered.
2. **Account switcher polish** — create/rename/switch up to 6; reuse the existing profile registry.
3. **Guide page** — render the bilingual guide and a **Copy Prompt** button (copies the prompt block).
4. **FT/LF marking** — per-Pokémon `status`: owned / for_trade / looking_for / traded, plus filter.
5. **Trade-list export** — shareable image (For Trade + Looking For grids) and Discord-ready text.
6. **Sheet sync** — CSV/Google Sheet round trip (depends on §5.2).
7. **Trade offers + matcher** — import a partner's list into a separate bucket, compare with own inventory, surface matches.
8. **Trade pipeline** — pending → matched → scheduled → completed, with a history log.

Phase 2 references (don't build): pokexperience.com/trade (FT/LF, variant-exact grid, matching), 9db.jp/pokego (image generator), pokemongotrading.com/tools.

## 7. Guardrails

- Static-site-friendly; no server, no secrets, no API keys, no built-in vision.
- Editing `docs/` here means the author must mirror changes into the local Design source (`site/index.html` byte-identical, and `source/Shiny Lucky Dex.dc.html` patched, not overwritten). **Say this explicitly in every final message**, per `HANDOFF.md` §4.
- Never overwrite stored marks; imports are additive/merge. Never touch DexJourney account-level columns on export.
- Write bilingual (VI/EN) copy for anything user-facing.
- Develop on the branch you are given; open a PR only if asked.
- Verify by actually loading `docs/` in a browser (Chromium + Playwright are preinstalled; serve with `python3 -m http.server -d docs`) and check the console is clean — don't rely on reading the code.
