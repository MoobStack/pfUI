# MoobStack pfUI Changelog

This is the living development changelog for the MoobStack maintenance fork of pfUI.

The runtime addon identity remains `pfUI` for compatibility with pfUI modules and companion addons. Development stays on the current version until a release is explicitly finalized; intermediate fixes are recorded here without changing the TOC version or creating a release tag.

## 5.5.4 - Development

### 2026-09-04

#### Fixed

- Nameplate cast bars now use exact ClassicAPI nameplate unit identity when the optional `C_NamePlate` and `C_Spell` APIs are available.
  - Each visible nameplate is associated with its own `nameplateN` unit token and GUID.
  - `C_Spell.UnitCastingInfo` and `C_Spell.UnitChannelInfo` are queried for that exact unit instead of using the creature name as the cast-state key.
  - Identically named enemies no longer inherit another matching enemy's cast bar when exact ClassicAPI identity is available.
- Preserved pfUI's existing SuperWoW GUID cast path as a separate exact-identity provider.
- Corrected the initial ClassicAPI nameplate association after live testing showed cast bars disappearing.
  - ClassicAPI can expose fresh Lua wrappers for the same default engine nameplate; pfUI now compares the underlying native frame handle instead of relying only on Lua wrapper equality.
  - `NAME_PLATE_UNIT_ADDED` mappings are cached by native frame handle so an event that arrives before pfUI decorates the plate is retained and bound when the overlay is created.
  - Cached unit tokens/GUIDs are revalidated against the native frame before use and cleared on removal/recycling.
- Restored pfUI's original name-keyed cast fallback whenever exact ClassicAPI identity cannot be resolved. A transient exact-provider miss must not remove cast bars that stock pfUI would otherwise display.
- Corrected the second live-test failure where that restored fallback also restored the original duplicate-name cast leakage.
  - Exact ClassicAPI parent correlation now uses pfUI's Lua-registered overlay child as an authoritative identity marker across fresh native nameplate wrappers.
  - Unresolved plates perform a bounded on-demand scan of ClassicAPI's contiguous `nameplateN` unit tokens and bind the matching exact unit/GUID.
  - When ClassicAPI exact casting is available, a duplicated visible name is never allowed to fall back to pfUI's name-keyed `libcast` state; this prevents one mob's cast from being copied to its same-name neighbors.
  - Unique-name plates retain the legacy fallback if exact binding is temporarily unavailable.
- Corrected the SuperWoW cast lookup in the touched nameplate code path to reference the current `plate` object explicitly.

#### Compatibility

- No TOC version change: still `5.5.4` while this development cycle remains unreleased.
- No addon-folder rename.
- No change to the global `pfUI` namespace.
- No SavedVariables/schema changes.
- No public pfUI module/API contract changes.
- The fix is isolated to `modules/nameplates.lua`; pfExtend, pfQuest, pfQuest-turtle, and pfUI-LocationPlus integration surfaces are unchanged.
- ClassicAPI support is optional and feature-detected. Stock 1.12.1 clients continue to use pfUI's original legacy cast behavior.

#### Live validation status

- Static/source review: completed.
- Mocked identity-selection validation: completed.
- First live 1.12.1 test of the initial implementation: **failed** (nameplate cast bars were suppressed).
- Second live build: **failed** (cast bars returned, but same-name cast leakage also returned).
- Registered-child exact-identity build: **pending live validation**.

