You are helping fix Call of Duty: WWII (Steam) crashing on this Windows 11 laptop
(Intel Core Ultra 9 275HX, Intel iGPU + RTX 5090 Laptop GPU, Optimus).

Read DIAGNOSIS.md in this repo first. It lists what was found and what was already changed:
- GPU preference "High performance" set for s2_sp64_ship.exe / s2_mp64_ship.exe
- NahimicService disabled

Your tasks:
1. Verify the changes from DIAGNOSIS.md are still in place (registry + service state).
2. Check the Windows Application event log for any new s2_sp64_ship.exe / s2_mp64_ship.exe crashes
   newer than 2026-10-24 and compare them with the earlier ones (module, exception code, offset).
3. If WER reports exist under C:\ProgramData\Microsoft\Windows\WER\ReportArchive, confirm whether the
   game now loads the NVIDIA driver (nvwgf2umx.dll) instead of the Intel one (igd10umt64xe.dll),
   and whether any Nahimic/A-Volute DLLs are still injected.
4. If crashes continue, work through the "If it still crashes, try next" list in DIAGNOSIS.md,
   explaining each change before you make it. Do not delete game files or uninstall software
   without asking.
5. Append your findings to DIAGNOSIS.md under a new dated heading.
