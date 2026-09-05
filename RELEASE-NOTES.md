# MoobStack pfUI Development Release Notes

**Status:** Unreleased development
**Base public version:** 5.5.5
**Development date:** 2026-09-05
**Target:** World of Warcraft 1.12.1 / Interface 11200
**ClassicAPI:** Required for the supported MoobStack compatibility configuration
**Live-client validation:** Pending

## Current development changes

### libpredict / ClassicAPI spellcast event compatibility

Fixed an intermittent chat error originating from `libs/libpredict.lua` when ClassicAPI is installed:

```text
ERROR: Interface\AddOns\pfUI\libs\libpredict.lua:516: attempt to perform arithmetic on local `endtime' (a nil value)
```

The error predates the MoobStack nameplate modifications. Upstream pfUI registers both Vanilla `SPELLCAST_*` events and TBC-style `UNIT_SPELLCAST_*` events. On an unmodified 1.12.1 client the TBC event family never fires, so this is normally harmless. ClassicAPI backports `UNIT_SPELLCAST_*` to 1.12.1, which exposes the latent assumption: `libpredict` interprets the event as TBC and immediately subtracts `endtime - starttime` from its legacy Vanilla cast tracker even when those values are not populated yet.

The development fix now:

- registers Vanilla `SPELLCAST_*` events only on the 1.12.1 client;
- registers TBC `UNIT_SPELLCAST_*` events only on the TBC client;
- retains ClassicAPI use elsewhere in the MoobStack fork, including exact nameplate identity/cast handling;
- adds a defensive nil guard before the TBC timing subtraction.

This keeps the fix isolated to `libpredict` event handling and does not change the pfUI namespace, SavedVariables, module API, companion-addon compatibility surface, or the already live-validated nameplate cast implementation.

## Required compatibility component

For the supported MoobStack World of Warcraft 1.12.1 configuration, install `ClassicAPI.dll` from:

https://github.com/brues-code/ClassicAPI/releases

SuperCleveRoidMacros is not required by pfUI and does not replace ClassicAPI.

## Live-test focus

1. Cast several normal cast-time healing spells and confirm no `libpredict.lua` error appears.
2. Interrupt/cancel casts and confirm incoming-heal prediction clears normally.
3. Test mouseover/click-cast healing if used.
4. Confirm the previously fixed identical-name nameplate cast bars still behave correctly.
