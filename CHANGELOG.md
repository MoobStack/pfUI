# MoobStack pfUI Changelog

This is the living development changelog for the MoobStack maintenance fork of pfUI.

The runtime addon identity remains `pfUI` for compatibility with pfUI modules and companion addons. Development changes are accumulated here and finalized into numbered releases only after live validation.

## 5.5.5.1 - 2026-09-05

### 2026-09-05

#### Fixed

- Prevented an intermittent `libs/libpredict.lua` cast-start error on Vanilla clients running ClassicAPI: `attempt to perform arithmetic on local endtime (a nil value)`.
  - Upstream pfUI registers both Vanilla `SPELLCAST_*` and TBC `UNIT_SPELLCAST_*` events unconditionally because the unsupported event family normally never fires on a stock client.
  - ClassicAPI intentionally backports `UNIT_SPELLCAST_*` to 1.12.1, which caused pfUI's Vanilla `libpredict` sender to enter its TBC-only path and subtract cast timestamps returned by pfUI's legacy `UnitCastingInfo` before that tracker had valid timing.
  - `libpredict` now registers only the event family appropriate to the detected client: Vanilla uses `SPELLCAST_*`; TBC uses `UNIT_SPELLCAST_*`.
  - Added a defensive timestamp guard to the TBC cast-start path so a transient missing cast result cannot raise an arithmetic error.

#### Documentation

- Clarified that `ClassicAPI.dll` is required for the supported World of Warcraft 1.12.1 compatibility configuration of the MoobStack pfUI fork and linked the official ClassicAPI releases page.
- Clarified that SuperCleveRoidMacros is not a pfUI dependency and does not replace ClassicAPI.

#### Live validation status

- Static/source validation: completed.
- Live 1.12.1 validation of the `libpredict` correction: **passed**. The previously intermittent `endtime` nil arithmetic error no longer reproduced during normal play.

## 5.5.5 - 2026-09-04

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
- Corrected the third live-test failure where exact casts appeared only intermittently.
  - Fixed the castbar update call site to resolve the current `nameplate` overlay rather than the outer WorldFrame-discovery scratch variable `plate`. The stale scratch variable could point at a completely different nameplate, making exact-unit resolution depend on discovery order.
  - Corrected the bounded `nameplateN` recovery scan to tolerate free slots. ClassicAPI 1.13.4 keeps nameplate slots stable for each plate lifetime, so a removed middle plate can leave a hole while higher-numbered slots remain valid.
  - Added guarded `UNIT_SPELLCAST_START`, `UNIT_SPELLCAST_CHANNEL_START`, `UNIT_SPELLCAST_STOP`, and `UNIT_SPELLCAST_CHANNEL_STOP` listeners. Nameplate spellcast transitions now provide an additional authoritative opportunity to re-bind the exact `nameplateN` token at the moment cast state changes.
- Corrected the SuperWoW cast lookup in the touched nameplate code path to reference the current `plate` object explicitly.

#### Compatibility

- Published as pfUI `5.5.5` after successful live validation of the exact-nameplate cast fix.
- No addon-folder rename.
- No change to the global `pfUI` namespace.
- No SavedVariables/schema changes.
- No public pfUI module/API contract changes.
- The fix is isolated to `modules/nameplates.lua`; pfExtend, pfQuest, pfQuest-turtle, and pfUI-LocationPlus integration surfaces are unchanged.
- ClassicAPI support is optional and feature-detected. Stock 1.12.1 clients continue to use pfUI's original legacy cast behavior. Exact isolation of same-name nameplate casts requires an exact identity provider such as ClassicAPI or SuperWoW.

#### Live validation status

- Static/source review: completed.
- Mocked identity-selection validation: completed.
- First live 1.12.1 test of the initial implementation: **failed** (nameplate cast bars were suppressed).
- Second live build: **failed** (cast bars returned, but same-name cast leakage also returned).
- Registered-child exact-identity build: **failed** (exact cast bars appeared intermittently; current-plate resolver and sparse-slot recovery defects identified).
- Current-plate/sparse-slot correction build (`2026-09-04-nameplate-castfix4`): **passed live validation**. Cast bars remained visible and were isolated to the individual same-name unit actually casting.

