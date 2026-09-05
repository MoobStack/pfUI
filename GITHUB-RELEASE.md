# pfUI 5.5.5

MoobStack pfUI 5.5.5 is a compatibility-focused maintenance release for World of Warcraft 1.12.1.

## Fixed

- Nameplate cast bars can now be associated with the exact visible enemy when a supported exact nameplate provider is available.
- Multiple visible enemies with the same name no longer all display a cast bar when only one is casting.
- Exact ClassicAPI binding now tolerates wrapper identity differences, nameplate-slot reuse, sparse `nameplateN` slots, and timing differences between native nameplate discovery and spellcast events.
- Existing pfUI SuperWoW behavior and legacy fallback behavior remain available.

## Dependency note

pfUI does not require SuperCleveRoidMacros. The identical-name cast isolation fix requires an exact provider such as ClassicAPI or SuperWoW; without one, pfUI falls back to its original Vanilla name-based cast tracking.

## Compatibility

The addon remains `pfUI`, uses Interface 11200, and does not change SavedVariables or the public pfUI module/config interface.

See `RELEASE-NOTES-v5.5.5.md` and `CHANGELOG.md` for full details.
