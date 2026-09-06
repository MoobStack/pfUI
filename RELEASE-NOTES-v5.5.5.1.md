# MoobStack pfUI v5.5.5.1 Release Notes

**Status:** Published release
**Version:** 5.5.5.1
**Release date:** 2026-09-05
**Target:** World of Warcraft 1.12.1 / Interface 11200
**ClassicAPI:** Required for the supported MoobStack compatibility configuration
**Live-client validation:** Passed

## Fixed

### libpredict / ClassicAPI spellcast event compatibility

Fixed an intermittent chat error originating from `libs/libpredict.lua` when ClassicAPI is installed:

```text
ERROR: Interface\AddOns\pfUI\libs\libpredict.lua:516: attempt to perform arithmetic on local `endtime' (a nil value)
```

The error predates the MoobStack nameplate modifications. Upstream pfUI registers both Vanilla `SPELLCAST_*` events and TBC-style `UNIT_SPELLCAST_*` events. On an unmodified 1.12.1 client the TBC event family never fires, so this is normally harmless. ClassicAPI backports `UNIT_SPELLCAST_*` to 1.12.1, which exposes the latent assumption: `libpredict` interprets the event as TBC and immediately subtracts `endtime - starttime` from its legacy Vanilla cast tracker even when those values are not populated yet.

The released fix now:

- registers Vanilla `SPELLCAST_*` events only on the 1.12.1 client;
- registers TBC `UNIT_SPELLCAST_*` events only on the TBC client;
- retains ClassicAPI use elsewhere in the MoobStack fork, including exact nameplate identity/cast handling;
- adds a defensive nil guard before the TBC timing subtraction.

This keeps the fix isolated to `libpredict` event handling and does not change the pfUI namespace, SavedVariables, module API, companion-addon compatibility surface, or the already live-validated nameplate cast implementation.

## Required compatibility component

For the supported MoobStack World of Warcraft 1.12.1 configuration, install `ClassicAPI.dll` from:

https://github.com/brues-code/ClassicAPI/releases

SuperCleveRoidMacros is not required by pfUI and does not replace ClassicAPI.

## Live validation

The `libpredict` correction was tested live on the World of Warcraft 1.12.1 / OctoWoW target environment with ClassicAPI installed. The previously intermittent `endtime` nil arithmetic error no longer reproduced during normal play.

The exact same-name nameplate cast isolation from v5.5.5 remains included and unchanged in this release; that implementation had already passed live validation before publication.
