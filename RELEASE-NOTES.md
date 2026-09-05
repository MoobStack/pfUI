# MoobStack pfUI Development Release Notes

**Status:** Unreleased development build  
**Version:** 5.5.4 (intentionally unchanged)  
**Development build:** `2026-09-04-nameplate-castfix4`
**Base upstream commit:** `b2f6df84`  
**Target:** World of Warcraft 1.12.1 / Interface 11200  
**Live-client validation:** Three development builds tested; current-plate/sparse-slot correction pending

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
6. Uses pfUI's Lua-registered overlay child as the authoritative identity marker when ClassicAPI returns a fresh wrapper for a default engine nameplate.
7. Performs a bounded on-demand `nameplate1`..`nameplateN` scan if an event mapping arrived before pfUI decorated the frame, then caches the exact token/GUID after the registered child matches.
8. Uses the original name-keyed cast estimator only when the visible name is unique (or when ClassicAPI exact casting is unavailable). A duplicated visible name never falls back to shared name-keyed cast state while the exact provider exists.

Once a plate has exact identity, its cast state comes only from that exact unit. If exact binding is temporarily unavailable, unique names may still use stock pfUI's estimator without ambiguity; duplicate names are held back from the estimator specifically to prevent cross-plate leakage.

### Live-test correction

The first development build compared the `Frame` object returned by `C_NamePlate.GetNamePlateForUnit()` directly with pfUI's previously discovered parent frame. Live testing showed no cast bars. ClassicAPI documents that default engine nameplates can be surfaced through fresh Lua wrapper objects, so two wrappers can represent the same native frame without being Lua-equal.

The second development build added native-handle comparison and restored pfUI's name-keyed fallback when exact binding missed. Live testing restored cast bars, but the fallback also restored the original bug: every visible same-name mob displayed the shared cast.

The current correction no longer depends on parent-wrapper identity alone. pfUI's overlay is a Lua-registered child of the native plate; the fresh ClassicAPI wrapper exposes that same registered child through `GetChildren()`, giving an exact cross-wrapper correlation. An unresolved decorated plate can therefore enumerate the live `nameplateN` tokens, identify its own exact token through the registered child, and use `C_Spell` for that unit. Duplicate visible names are not permitted to use the shared name-keyed fallback while ClassicAPI exact casting is present.

The third live build showed that cast bars were now sometimes correct but absent most of the time. Review found two deterministic implementation defects rather than another ClassicAPI ambiguity:

- the per-nameplate castbar update called `GetClassicAPIPlateUnit(plate, ...)`, where `plate` was the outer scratch variable used by the WorldFrame discovery loop, instead of passing the current `nameplate` overlay;
- the fallback `nameplate1`..`nameplate80` scan stopped at the first `UnitExists(...) == false`. ClassicAPI 1.13.4 nameplate slots can contain holes, so this could make all later live slots unreachable.

The current build passes the current overlay explicitly, scans the full bounded token range while skipping free slots, and re-attempts exact binding on ClassicAPI `UNIT_SPELLCAST_*` start/stop transitions for `nameplateN` tokens. The visible castbar itself remains pfUI's existing widget/rendering path.

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

