# AIMeE Bug Fixes

A Stationeers plugin mod that fixes 16 bugs with the AIMeE mining robot, in single player and multiplayer.

[Steam Workshop page](https://steamcommunity.com/sharedfiles/filedetails/?id=3674960841)

## Requirements

- [BepInEx](https://github.com/BepInEx/BepInEx) 5.x
- [StationeersLaunchPad](https://github.com/StationeersLaunchPad/StationeersLaunchPad) (includes LaunchPadBooster, which the mod uses for multiplayer sync; nothing extra to install)

## Multiplayer / dedicated servers

**Install the mod on the server and on every player's client.**

- AIMeE's driving and mining logic runs only on the server, so most fixes need the server copy.
- Since v0.16.0, the mod sends its own idle position sync (Bug 15) using LaunchPadBooster networking. Booster checks that the server and each joining player run the same networked mods, so a player without AimeeBugFixes can't join a server running it, and vice versa.
- Any AimeeBugFixes version is accepted, so being one release behind won't block joining.
- If LaunchPadBooster is missing or older than 0.3 (very old StationeersLaunchPad), only Bug 15 is skipped; every other fix still loads.

To confirm it loaded, check `BepInEx/LogOutput.log` for:

```
AIMeE Bug Fixes v0.16.0 loading...
  [Bug15] Registered AIMeeSyncMessage with LaunchPadBooster
AIMeE Bug Fixes: N patches applied, 0 failed
```

## What it fixes

| # | Bug | Fix |
|---|-----|-----|
| 1 | Stops mining after the first ore | Reset the mining cancellation token after each mine |
| 2 | Hops/teleports while turned off | Skip unstuck/pathing logic when off or unpowered |
| 3 | Frozen tread animation on clients | Estimate wheel RPM from synced velocity |
| 4 | Stops exploring when no ore is nearby | Always drive when the queue is empty; scan range restored to 32 |
| 5 | Picking her up launches the player | Zero motor/brake/steering while dragged |
| 6 | `MineablesInQueue` always 0 on clients | Client-side capped estimate |
| 7 | `MineablesInVicinity` overcounts | Per-ore bounds check |
| 8 | Carrying her into terrain launches the player | Disable wheel colliders while carried |
| 9 | Teleports when stuck for 60 s | Physics-based escape instead of a position snap |
| 10 | Mines distant ore first | Queue sorted by distance |
| 11 | Circles aimlessly | Post-reverse cooldown and steering reset |
| 12 | Mined ore stays visible on clients | Inject client voxel sync events |
| 13 | Targets ore in caves/tunnels | Column scan rejects ore under a cave ceiling |
| 14 | Stale ore queue after relocation | Clear queue on pickup, power-on, and TargetX/Y/Z writes |
| 15 | Idle AIMeE shows in the ground / tipped over for distant players on dedicated servers | Server resyncs idle AIMeE's position to all clients every 5 s (by StormCircuit) |
| 16 | Idle AIMeE sinks on clients | Clear client-side velocity while the server reports her idle |

Full write-ups are in [Developer Files/](Developer%20Files/):

- [AIMEE_BUGS_OVERVIEW.md](Developer%20Files/AIMEE_BUGS_OVERVIEW.md): summary table
- [BUG_REPORT.md](Developer%20Files/BUG_REPORT.md): root-cause analysis for each bug
- [DEVELOPER_RECOMMENDATIONS.md](Developer%20Files/DEVELOPER_RECOMMENDATIONS.md): suggested native fixes for the game code
- [BUG_15_NETWORK_DESYNC.md](Developer%20Files/BUG_15_NETWORK_DESYNC.md): the dedicated-server idle desync investigation

## Credits

- **FlorpyDorp**: author
- **[StormCircuit](https://github.com/StormCircuit)**: dedicated-server testing and investigation, and the Bug 15 idle network sync fix

## Debug overlay

Type `aimeedebug` (or `aimeedebug on|off`) in the in-game console to show per-AIMeE stats (mode, queue, vicinity, target, velocity) and enable detailed logging.

## Building

1. Install the .NET SDK (the project targets `net48`).
2. If Stationeers isn't installed at the default Steam path, edit `StationeersPath` in `AimeeBugFixes.csproj`.
3. Build:

   ```
   dotnet build -c Release
   ```

4. Optional: copy the DLL and `About/` into your local mods folders:

   ```
   dotnet msbuild -t:Package
   ```

Output goes to `bin/AimeeBugFixes.dll`.
