# pfUI 5.5.5.1

MoobStack pfUI 5.5.5.1 is a compatibility-focused maintenance release for World of Warcraft 1.12.1.

## Fixed

- Fixed an intermittent `libs/libpredict.lua` error on Vanilla clients running ClassicAPI: `attempt to perform arithmetic on local endtime (a nil value)`.
- Vanilla now registers only the original `SPELLCAST_*` event family in `libpredict`; TBC retains the `UNIT_SPELLCAST_*` event family.
- Added a defensive timestamp guard to the TBC cast-start path.
- Preserved the v5.5.5 exact same-name nameplate cast isolation behavior.

## ClassicAPI requirement

`ClassicAPI.dll` is required for the supported World of Warcraft 1.12.1 compatibility configuration of the MoobStack pfUI fork. Install it from:

https://github.com/brues-code/ClassicAPI/releases

SuperCleveRoidMacros is not required by pfUI and does not replace ClassicAPI.

## Compatibility

The addon remains `pfUI`, uses Interface 11200, and does not change SavedVariables or the public pfUI module/config interface. pfExtend, pfQuest, pfQuest-turtle, and pfUI-LocationPlus compatibility surfaces are unchanged.

The libpredict correction and the prior exact-nameplate cast fix have both passed live validation on the supported 1.12.1 environment.

See `RELEASE-NOTES-v5.5.5.1.md` and `CHANGELOG.md` for full details.
