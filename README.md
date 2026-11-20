# Pocket Mortys Save Editor

A single-file, offline save editor for **Pocket Mortys** (`Player.dat` / `CloudContainer.dat`). Open `pocket_mortys_editor.html` in any modern browser. Everything runs locally; nothing is uploaded.

> **Back up your save before using this.** The editor has been checked structurally (see [Status](#status--what-is-and-isnt-verified)) but has never been run against the real game. Test on a copy first.

## Quick start

1. Find your save. On Android it is `Player.dat` (and possibly `CloudContainer.dat`) in the game's `files` folder.
2. Open `pocket_mortys_editor.html`, then drop in the file or click to choose it. Or click **Start with an empty save** (experimental).
3. Edit whatever you want. Edits are counted in the top-right status.
4. Click **Download**. The editor re-reads its own output as a self-check before handing it over.
5. Put the file back (see the **Save & Install** tab). If the game keeps restoring the old data, delete `CloudContainer.dat`. In 2.0.3 that file is re-extracted over the loose `.dat` files on start-up. This is unverified for 2.40.2.

## Features

| Tab | What it does |
| --- | --- |
| **Overview** | Schmeckles, coupons, badges (max 100), storage cap, deck count, dimension count, game time and other root values |
| **Mortys** | Party & Storage list with filters and an inline editor, Decks, Add, and mass modify |
| **Mortydex** | Toggle seen / caught per species |
| **Items** | Edit or add any item quantity |
| **Avatars** | Own and equip avatars |
| **Quests & Story** | Completed and active quests, trophies, story flags |
| **World** | Last-world presets and related fields |
| **Unlock / Reset** | One-click "unlock everything" and "reset everything" with per-option checkboxes |
| **Advanced** | Raw tree editor for every field in the save |
| **Save & Install** | Download and install notes |

### Mortys

- **Edit any Morty:** level, XP, current HP, stats, IVs, EVs, the three seed stat bonuses, shiny flag, dimension caught and moves.
- **Add Morty:** pick a species and set level, IVs, EVs, bonuses and the four moves yourself. A max-EV, max-IV, 1M-attack Morty is possible. Out-of-range values only trigger a warning, never a block.
- **Add one of every species:** adds each missing species at IV 16 and EV 65535 in all stats.
- **Decks:** each deck has a **+ Add Morty** box (adds a level 1 Morty of the chosen species). **Edit** jumps to Party & Storage and opens that Morty. Party & Storage has an **Add morty** button that jumps to the Add tab.
- **Mass modify:** apply level, IV, EV, seed bonus, shiny or heal changes to selected or filtered Mortys.
- **Fix unnatural stats:** detects the defence bug, where a Morty's defence stat is missing its base stat and EV. It fixes only defence by default. Tick **recompute all** to recalculate every stat from base, IV, EV and level.

### Unlock / Reset

- **Unlock everything:** adds every missing species (level 100, IV 16, EV 65535), sets badges to 100, owns all decks and marks tutorials complete. It only adds; it never removes.
- **Reset everything:** clears coins, dex, items, quests, trophies and more. **Delete all Mortys** keeps only the first Morty of the active deck.
- **Start with an empty save:** a cleared save with one level 1 starter Morty. This is experimental; see below.

## Stat formulas

These were fitted to a real 2.40.2 save and cross-checked against a community IV-calculator bot.

```
HP        = floor((base + IV + evTerm + 50) * level / 50) + 10
Atk/Def/Spd = floor((base + IV + evTerm) * level / 50) + 5      (+ seed stat bonus)
evTerm    = floor(sqrt(EV) / 4)          (no cap in campaign)
```

The HP bonus is added directly to the HP stat. Attack, defence and speed bonuses are seed stat bonuses. Stored stats on EV-trained Mortys can lag by a few points until the Morty levels up, so the checker flags only the defence bug.

## Behaviour notes

- **Storage cap:** the game UI stops at 750, but the editor raises `MortySlotsCap` automatically and never blocks an add.
- **Badges list:** if it was edited with a memory editor and has a size larger than its backing array, the game tolerates it, so the editor leaves it alone unless you edit that list.
- **Warnings, not blocks:** IV above 16 or EV above 65535 show a warning and are accepted.
- **Unknown fields** are preserved exactly as they were read.

## How it works

Saves are .NET `BinaryFormatter` streams (MS-NRBF) whose root class is `PlayerSaveData`. The editor includes its own reader and writer:

- A generic NRBF reader and writer that round-trips the save byte-for-byte (excluding stale trailing bytes the game leaves behind).
- A `Save` layer for the game's structure: lists, dictionaries, Mortys, decks and moves.
- Data tables (species, base stats, attacks, items, avatars, quests) generated from `MortyInfo.json` and game dumps.

## Building from source

Source lives in the project working folder:

```
nrbf.js  save.js  data.js  empty_data.js  app_core.js  app_mortys.js  app_extra.js
style.css  shell.html  build.js
```

```
node build.js        # writes pocket_mortys_editor.html
```

Data is regenerated with `build_data.js`; the empty-save template is regenerated with `mkempty.js` (needs Playwright and a source save). UI tests (`ui_test*.js`) use headless Playwright Chromium.

## Status — what is and isn't verified

**Verified**
- Unedited round trip is byte-identical.
- Edited outputs (fix defence, mass edit, unlock, reset, custom adds, move edits, cap above 750, deck add/edit, empty save) re-parse correctly in the editor's own parser and in an independent third-party NRBF parser.

**Not verified**
- No output has been run through the real .NET `BinaryFormatter` or the game.
- The 2.40.2 install flow is unverified; the `CloudContainer.dat` advice comes from 2.0.3.
- Editing `CloudContainer.dat` directly is untested.
- The empty save's defaults (slot cap 50, tutorial flags, starter species) are best guesses.
- The EV term is inferred from data, not read from the game.

If the game rejects a file, note what it does (crash, reset to new game, or ignore) so the cause can be narrowed down.

## Disclaimer

Unofficial fan tool, not affiliated with the game's developers. Use at your own risk, and only on your own save. Editing may affect online or cloud features.
