# MoobStack pfUI v5.5.5 Release Notes

**Status:** Published release  
**Version:** 5.5.5  
**Release date:** 2026-09-04  
**Base upstream commit:** `b2f6df84`  
**Target:** World of Warcraft 1.12.1 / Interface 11200  
**Live-client validation:** Passed

## Summary

This release fixes pfUI nameplate cast bars for groups of visible enemies that share the same displayed name.

Vanilla pfUI normally reconstructs hostile casts from name-based combat-log information. When several visible enemies have the same name, that legacy state can be shared between their nameplates, causing one enemy's cast bar to appear beneath every matching plate.

pfUI 5.5.5 adds an optional exact-identity nameplate cast path. When a supported exact provider is available, each pfUI nameplate is associated with its individual unit token/GUID and queried for that unit's cast or channel state. Same-name enemies therefore no longer inherit another unit's cast bar.

## Fixed

- Isolated hostile cast bars to the individual visible nameplate that is actually casting when exact nameplate identity is available.
- Added guarded ClassicAPI nameplate integration using `C_NamePlate`, `UnitGUID`, `C_Spell.UnitCastingInfo`, and `C_Spell.UnitChannelInfo`.
- Added `NAME_PLATE_UNIT_ADDED` / `NAME_PLATE_UNIT_REMOVED` tracking and guarded `UNIT_SPELLCAST_*` refresh handling.
- Correlated ClassicAPI nameplate wrappers with pfUI's decorated native plate through pfUI's registered overlay child rather than relying on Lua wrapper equality.
- Revalidated cached unit tokens/GUIDs so recycled nameplate slots cannot inherit stale cast state.
- Corrected recovery scanning so free `nameplateN` slots do not prevent later active slots from being found.
- Corrected the per-nameplate update path to resolve the current overlay rather than the WorldFrame discovery scratch variable.
- Preserved pfUI's existing SuperWoW GUID cast path.
- Preserved the stock name-based fallback for clients that do not expose an exact provider.

## Runtime dependency for this fix

pfUI itself does **not** require ClassicAPI, SuperWoW, or SuperCleveRoidMacros to load.

For the specific 5.5.5 fix that distinguishes multiple visible enemies with the same name, an **exact nameplate identity/cast provider is required**:

- **ClassicAPI:** supported and used by the new exact nameplate path. This is the provider used for live validation of this release.
- **SuperWoW:** pfUI's pre-existing exact GUID path is retained as an alternative provider where available.
- **SuperCleveRoidMacros:** **not required**. It is a separate addon and is not consulted by pfUI's nameplate cast implementation.

Without ClassicAPI or SuperWoW, pfUI continues to function and retains its original Vanilla name-based cast behavior. That legacy API cannot reliably distinguish multiple visible mobs with identical names, so the original duplicate-name limitation may remain in that environment.

## Compatibility

The compatibility surface was intentionally kept unchanged:

- addon folder remains `pfUI`
- global namespace remains `pfUI`
- Interface remains `11200`
- SavedVariables names and schema are unchanged
- existing pfUI module/config APIs are unchanged
- pfExtend, pfQuest, pfQuest-turtle, and pfUI-LocationPlus integration surfaces were not changed

## Live validation

The final pre-release build (`2026-09-04-nameplate-castfix4`) was tested live and confirmed to restore reliable cast-bar display while fixing the same-name duplication issue.

Expected final behavior:

1. A visible casting enemy receives a cast bar even when it is not the active target.
2. Non-casting enemies with the same name do not receive that cast bar.
3. Cast-bar ownership follows the individual unit rather than the displayed name.
4. Ordinary unique-name nameplate casting continues to work.

## Installation

For World of Warcraft 1.12.1:

1. Remove or replace the existing `Interface\AddOns\pfUI` folder.
2. Extract the clean release ZIP into `Interface\AddOns`.
3. Confirm the resulting path is `Interface\AddOns\pfUI\pfUI.toc`.
4. Restart the client or reload the UI where appropriate.

Existing pfUI SavedVariables and profiles are retained.

## Upstream

MoobStack pfUI is a maintenance fork of Shagu's pfUI. The runtime addon identity is intentionally preserved for compatibility with pfUI plugins and companion addons.
