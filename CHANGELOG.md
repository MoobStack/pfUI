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
- Hardened the legacy Vanilla combat-log fallback. If multiple visible nameplates share the same name and no exact provider can resolve the plate, pfUI now suppresses the ambiguous name-keyed cast instead of displaying the same cast on every matching plate.
- Corrected the SuperWoW cast lookup in the touched nameplate code path to reference the current `plate` object explicitly.

#### Compatibility

- No TOC version change: still `5.5.4` while this development cycle remains unreleased.
- No addon-folder rename.
- No change to the global `pfUI` namespace.
- No SavedVariables/schema changes.
- No public pfUI module/API contract changes.
- The fix is isolated to `modules/nameplates.lua`; pfExtend, pfQuest, pfQuest-turtle, and pfUI-LocationPlus integration surfaces are unchanged.
- ClassicAPI support is optional and feature-detected. Stock 1.12.1 clients continue to use pfUI's legacy behavior, with the new duplicate-name safety guard.

#### Live validation status

- Static/source review: completed.
- Mocked identity-selection validation: completed.
- Live 1.12.1 client validation: **pending**.

