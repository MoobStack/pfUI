# MoobStack pfUI Development Release Notes

**Status:** Unreleased development build  
**Version:** 5.5.4 (intentionally unchanged)  
**Base upstream commit:** `b2f6df84`  
**Target:** World of Warcraft 1.12.1 / Interface 11200  
**Live-client validation:** Initial build failed; corrected build pending

## Development policy

This fork keeps pfUI's runtime identity and extension contracts intact. Until a release is explicitly finalized, fixes accumulate under version `5.5.4`; no release tag should be created and the TOC version should not be incremented.

The compatibility baseline includes addons that consume or coexist with pfUI, including:

- pfExtend
- pfQuest
- pfQuest-turtle
- pfUI-LocationPlus / pfUI-LocPlus

Changes should avoid renaming the `pfUI` addon folder, replacing the `pfUI` global namespace, changing SavedVariables unnecessarily, or breaking the existing module/config framework expected by external pfUI modules.

## Current development fix: exact nameplate cast identity

### Problem

On Vanilla pfUI, enemy cast information is normally reconstructed from the combat log and indexed by unit name. If several visible enemies have the same name, a cast attributed to that name can therefore be displayed on every matching nameplate even though only one individual unit is casting.

Example:

- Three identical-name murlocs are visible.
- Only one begins a cast.
- Stock pfUI can show the same cast bar beneath all three plates.

### Implementation

When ClassicAPI's optional nameplate and spell APIs are available, pfUI now:

1. Listens for `NAME_PLATE_UNIT_ADDED` and `NAME_PLATE_UNIT_REMOVED`.
2. Associates the underlying pfUI nameplate frame with the exact `nameplateN` token and GUID.
3. Revalidates cached tokens before use so recycled nameplate slots cannot be mistaken for a previous unit.
4. Queries `C_Spell.UnitCastingInfo(unit)` and `C_Spell.UnitChannelInfo(unit)` for the exact plate identity.
5. Keeps the existing SuperWoW GUID path intact when SuperWoW is the available exact provider.
6. Compares ClassicAPI nameplate frames by their underlying native frame handle when Lua wrapper identity is not stable.
7. Caches `NAME_PLATE_UNIT_ADDED` mappings by that native handle so early events can be bound after pfUI creates its overlay.
8. Falls back to pfUI's original name-keyed cast estimator whenever exact frame identity cannot be resolved.

The fallback is intentionally display-safe: a temporary ClassicAPI association miss can re-expose stock pfUI's duplicate-name limitation, but it must never make all cast bars disappear. Once a plate has exact identity, its cast state comes only from that exact unit.

### Live-test correction

The first development build compared the `Frame` object returned by `C_NamePlate.GetNamePlateForUnit()` directly with pfUI's previously discovered parent frame. Live testing showed no cast bars. ClassicAPI documents that default engine nameplates can be surfaced through fresh Lua wrapper objects, so two wrappers can represent the same native frame without being Lua-equal.

The corrected build therefore compares the wrapper's native frame handle (`frame[0]`) when direct equality fails and preserves the original cast fallback if exact binding is temporarily unavailable.

### Why this approach

The fix does not replace pfUI's `libcast`, alter global casting functions, or change the public pfUI extension interface. It adds an optional exact provider only at the nameplate display boundary, keeping the compatibility risk small for companion addons.

## Live test checklist

Test with nameplate cast bars enabled and, where applicable, ClassicAPI loaded:

1. Spawn or locate at least three hostile NPCs with exactly the same displayed name.
2. Keep all three nameplates visible.
3. Cause only one of them to cast.
   - Expected: only the actual caster receives a cast bar.
4. Let a different same-name NPC cast next.
   - Expected: the cast bar moves to that individual plate only.
5. Have two same-name NPCs cast at overlapping times if possible.
   - Expected: each casting unit has its own independent bar and timing.
6. Interrupt or kill a caster mid-cast.
   - Expected: that unit's bar clears without affecting the other same-name plates.
7. Let a nameplate leave range/despawn and allow another unit to occupy a recycled nameplate slot.
   - Expected: no stale cast state transfers to the new unit.
8. Test a channeled spell.
   - Expected: only the exact channeling unit receives the channel bar when timing is available.
9. Test a unique-name caster.
   - Expected: ordinary cast bars continue to function.
10. Reload the UI while several nameplates are already present.
    - Expected: exact identities are recovered and no duplicate-name cast leakage appears.
11. Smoke-test pfExtend, pfQuest, pfQuest-turtle, and pfUI-LocationPlus.
    - Expected: normal loading and functionality; no missing `pfUI` globals/modules/config data.

## Known limitation retained by design

On a client without an exact nameplate identity/cast provider, the Vanilla combat log still cannot distinguish two simultaneously visible mobs that have the same name. In that environment this fork preserves stock pfUI's behavior rather than suppressing cast bars. The identical-name isolation fix therefore depends on an exact provider such as ClassicAPI or SuperWoW.

