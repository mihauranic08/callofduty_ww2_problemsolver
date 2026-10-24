# Call of Duty: WWII â€” crash diagnosis (2026-10-24)

## System
- CPU: Intel Core Ultra 9 275HX (24 cores / 24 threads)
- GPUs: Intel(R) Graphics (iGPU, driver 32.0.101.8331) + NVIDIA GeForce RTX 5090 Laptop GPU (driver 32.0.16.1742)
- Displays: internal 2560x1600@240 on the **Intel iGPU**; external 3840x2160@60 on the NVIDIA GPU
- RAM: 63.4 GB, Windows 11 Home 26200
- Game: Steam, SP build 22081346 / MP build 22081147, both "Fully Installed"

## Symptoms
- Event log (Application, ID 1000/1001): repeated `0xc0000005` access violations
  - `s2_mp64_ship.exe` in `ntdll.dll` @ `0x1a12d` (crashed after 33 s, 35 s, 372 s)
  - `s2_sp64_ship.exe` in itself @ `0xf2af90a` (after 133 s)
- Steam console log, 2026-10-24: game exits ~3-4 s after the Steam overlay attaches
  (i.e. right at D3D device / window creation), on several launches in a row.

## Root causes found
1. **Game renders on the Intel iGPU, not the RTX 5090.** Every WER crash report shows the Intel
   D3D11 driver loaded (`igd10umt64xe.dll`, `igc64.dll`, `igdgmm64.dll`) and no NVIDIA user-mode driver.
   No per-app GPU preference was set, and the main display hangs off the iGPU (Optimus).
2. **Nahimic (A-Volute) audio software injects into the game** (`AudioDevProps2.dll`,
   `ASM.Plugin.AudioDevProps2.dll`, `ProductInfo.dll` in every crash). Activision has
   identified Nahimic as a cause of crashes in Call of Duty titles.

## Fixes applied
- `HKCU\Software\Microsoft\DirectX\UserGpuPreferences`: `GpuPreference=2;` (High performance)
  for `s2_sp64_ship.exe` and `s2_mp64_ship.exe`.
  Revert: delete those two values.
- `NahimicService`: stopped and StartupType set to Disabled.
  Revert (admin): `Set-Service NahimicService -StartupType Automatic; Start-Service NahimicService`

## If it still crashes, try next
- NVIDIA Control Panel -> Manage 3D settings -> Program settings: set both exes to
  "High-performance NVIDIA processor"; if the laptop has a MUX switch, set it to dGPU-only/Ultimate.
- Play on the external monitor (attached to the NVIDIA GPU).
- Disable the Steam overlay for WW2.
- Uninstall Nahimic completely (it reinstalls via Windows Update as a driver component;
  may need to be hidden with the "Show or hide updates" tool).
- Limit CPU affinity (e.g. to P-cores only) in case of a high-core-count issue in the old engine.
- Steam -> Verify integrity of game files.
- Analyse the crash dump `%LOCALAPPDATA%\CrashDumps\s2_mp64_ship.exe.22196.dmp` with WinDbg (`!analyze -v`).

## Follow-up verification (2026-10-24)
- Registry: `HKCU\Software\Microsoft\DirectX\UserGpuPreferences` still has `GpuPreference=2;` for both `s2_sp64_ship.exe` and `s2_mp64_ship.exe` (High performance).
- Service: `NahimicService` is `Stopped`, startup mode `Disabled`.
- Application log: no matching Event 1000/1001 crashes dated 2026-10-24 or later. The newest matching records are MP on 2026-10-20 01:33:21 and SP on 2026-10-19 21:36:10. Thus there is no post-change crash record to compare yet. The older records remain consistent with the original pattern: MP `ntdll.dll`, exception `0xc0000005`, offset `0x1a12d` (three archived reports); SP `s2_sp64_ship.exe`, exception `0xc0000005`, offset `0xf2af90a` (one archived report).
- WER archive: all four readable game reports are dated 2026-10-19/20, before the 2026-10-24 changes. They still list Intel D3D11 modules (`igd10umt64xe.DLL`, `igc64.dll`) and A-Volute/Nahimic modules (`AudioDevProps2.dll`, `ASM.Plugin.AudioDevProps2.dll`, `ProductInfo.dll`); these reports do not list `nvwgf2umx.dll`. They cannot establish which GPU or injected modules are used after the changes. Some unrelated WER report files denied access in this user session.
- Next steps: no additional change from the 'If it still crashes, try next' list was applied because no crash newer than the changes is recorded. If a new crash occurs, first test with Steam Overlay disabled (reversible), then check NVIDIA Control Panel per-program GPU assignment and test on the external NVIDIA-connected display. These isolate overlay and Optimus routing without modifying game files. The remaining listed actions (MUX/dGPU-only, uninstalling Nahimic, CPU affinity, Steam file verification, dump analysis) are not performed; no software was uninstalled and no game files were changed.


## Launch test after fixes (2026-10-24 18:28)
- Launched SP (`s2_sp64_ship.exe`) via Steam and monitored for 5 minutes.
- NVIDIA D3D driver (`nvwgf2umx.dll`) is now loaded (Intel driver also present; normal on Optimus since the internal panel is attached to the iGPU).
- No Nahimic/A-Volute modules injected.
- Process stayed alive and responsive for 300 s (previous launches today exited after 25-45 s); no new Application Error / Hang events.
- Result: fixes appear to resolve the crash. Multiplayer not yet tested.
