# PROJECT-CONTEXT.md

Source of truth for migrating this project to a new workspace. Written from the full conversation history. Convention used throughout:

- `UNKNOWN` = never stated in the conversation.
- `NOT CONFIRMED` = claimed or produced, but not verified by the user or by an actual run.
- "Claude proposed" = a recommendation made by the assistant. **The user never explicitly accepted or rejected any recommendation**; they moved on to the next step, which is treated here as implicit acceptance only.
- No Windows machine was available to the assistant. **No script produced in this project has been run by the assistant.** Only the user ran things (see section 9).

---

## 1. Project identity

| Field | Value |
|---|---|
| Project name |"Custom Windows 10 IoT LTSC gaming image" |
| Project number | 'nth' |
| Category | Windows OS image customization / debloating (DISM, offline registry, scripting) |
| Primary objective | Build a custom Windows 10 ISO from the **Windows 10 IoT Enterprise LTSC 2021** media that is stripped so the OS uses only the resources needed to run, leaving everything else for games, "smoothest ever, like a PS5 / dedicated gaming hardware" |
| Long-term vision | `UNKNOWN` beyond the primary objective. The user also said they want to **learn the ISO and its components** and want the assistant to "research for my entire project and support me whenever needed" |
| Explicit non-goal | The ISO is **not hardware-specific** ("not the system specific iso to be built"); it should still give gaming resources no compromise |

## 2. Context recovery

### 2.1 Original motivation and requirements (user's words, paraphrased)
- Wants a bare system "like Arch Linux does": cut off services, applications and everything not needed to run the system, and allocate all available resources to gaming.
- Asked first for a brief list of Windows 10 internals (must-have applications, services, skeleton) that must stay untouched.
- Then supplied the base ISO and asked what to rip out and for a systematic procedure; wants to learn the ISO structure.
- Then asked for an automated script to enumerate the OS into a directory-style text/markdown map.
- Then asked for the whole map to be split, folder by folder, into a "to be removed" file and a "to be kept" file (total 20 files) with a description of every component: if kept why / what it is, if removed why.
- Then asked for a single "ultimate" script that strips all the agreed items.

### 2.2 Technical environment (as stated or observed)
| Item | Value | Source |
|---|---|---|
| Base ISO | `en-us_windows_10_iot_enterprise_ltsc_2021_x64_dvd_257ad90f.iso` | user |
| WIM indexes | Index 1 = "Windows 10 Enterprise LTSC" (EdgeID/Edition `EnterpriseS`); Index 2 = "Windows 10 IoT Enterprise LTSC" | scan `wim_info`, user confirmed index 2 for the tasks scan |
| Image build | 10.0.19044.1288 (21H2), last serviced Oct 2021 (RollupFix 19041.1288, SSU 19041.1220, KB5003791 enablement package, a .NET rollup) | scan of index 1 |
| Host DISM | 10.0.26100.8972 | scan + user's `dism /Get-MountedImageInfo` |
| Paths used by user | `C:\iso\sources\install.wim`, `C:\mount` (earlier R/W mount), `C:\mount2` (read-only mount of index 2), `C:\OSMap` (output dir) | user commands |
| Tools | DISM + PowerShell DISM cmdlets, 7-Zip/ADK `oscdimg`, NTLite and Atlas/ReviOS (named by assistant as references; **not used**), Hyper-V/VMware/VirtualBox for testing (recommended, **NOT CONFIRMED** whether user uses one) | assistant |
| Hardware (CPU, GPU, RAM, storage type, wired vs Wi-Fi, Bluetooth) | `UNKNOWN`, asked twice, never answered |
| Games / launchers / anti-cheat | `UNKNOWN`, asked, never answered |
| Game Pass / Microsoft Store use | `UNKNOWN` (LTSC has no Store, so these do not work) |
| User location | not relevant to this project; omitted |

### 2.3 Key terminology
- **install.wim / index**: the OS image inside the ISO; this ISO holds two editions.
- **DISM**: Microsoft's image servicing tool. **Component store (WinSxS)**: hard-link store behind System32, so never delete OS files by hand.
- **FoD / capability / feature / package / provisioned Appx**: the units DISM can remove cleanly.
- **Start=4** on a service/driver key = disabled; `Start` 0 Boot, 1 System, 2 Automatic, 3 Manual.
- **Batches**: B1 low risk, B2 apply after B1 boots and games run, B3 optional (disk-only gains). Defined by the assistant, used in all files and scripts.
- **VBS/HVCI**: virtualization-based security; can cost FPS; some anti-cheats need related features.

### 2.4 Chronology of the conversation
1. **Turn 1.** User asks for the Windows 10 skeleton to keep. Assistant gives a layered map (core, game-required services, runtimes, safe-to-remove, remove-with-caution) and a reality check (gains are mostly frametime consistency, low single-digit %; PS5 smoothness comes from fixed hardware). It claimed Windows 10 support ended October 2025.
2. **Turn 2.** User states the ISO is IoT Enterprise LTSC 2021. Assistant corrects: IoT LTSC 2021 is supported to **January 2032**; gives ISO anatomy, a 12-step DISM procedure, tiers of removal, gaming settings, measurement advice.
3. **Turn 3.** User asks for a script to automate Step 4 enumeration. Assistant creates `Build-OSMap.ps1`.
4. **Turn 4.** User uploads `OSMap.zip` (scan output). Assistant reads all of it and produces the 20-file `OSMap_Decisions/` set plus `Get-OfflineTasks.ps1`. Key findings: wrong index scanned, stale patch level, scheduled-task gap (see 2.5).
5. **Turn 5.** User runs `Get-OfflineTasks.ps1`: 213 tasks written, then `ERROR: Access is denied.` at hive unload. Assistant explains the hive-lock cause and fixes.
6. **Turn 6.** User uploads `tasks_registry.md`, states it is from index 2, shows `dism /Get-MountedImageInfo` (only `C:\mount2`, index 2, read-only). Assistant regenerates the two 07 files from the real list, creates `Apply-TaskPolicy.ps1`, `Apply-OfflinePolicy.ps1`, `OfflinePolicy.csv`, and gives a runbook.
7. **Turn 7.** User asks for an "ultimate" script. Assistant creates `Build-GamingImage.ps1` and fixes a reserved-keyword bug (`Do`).
8. **Turn 8 (this).** Context migration request.

### 2.5 Corrections and changes of direction (important)
| # | Earlier claim / state | Later correction | Effect |
|---|---|---|---|
| 1 | Assistant: "Windows 10 reached end of support Oct 2025" (Turn 1) | LTSC editions continue; IoT Enterprise LTSC 2021 to 13 Jan 2032 (Turn 2) | Base choice validated |
| 2 | Turn 2 said "IoT Enterprise LTSC 2021 supported until 2032" without noting the ISO has two indexes | The scanned **index 1 is the non-IoT Enterprise LTSC, support ends 12 Jan 2027**; IoT (index 2) ends 13 Jan 2032. Verified by web search | Build must use index 2; all later work targets index 2 |
| 3 | Scan was from index 1 | Tasks scan (Turn 6) was from index 2; the rest of the scan (packages, services, drivers, filesystem) is **still index 1 only** | See CONTEXT-GAPS #1 |
| 4 | Assistant listed `MMCSS` as a service | In this image it is a **kernel driver** (`mmcss.sys`), kept | Cosmetic |
| 5 | Scheduled tasks: scan found 0 task files | Tasks live in the registry TaskCache, which `Build-OSMap.ps1` did not read; `Get-OfflineTasks.ps1` found 213 | 07 files replaced with real data |
| 6 | First draft of 09 grouped `hnetcfg.dll` into the removable peer-networking group and `WMPhoto.dll` into WMP | `hnetcfg.dll` hosts the Windows Firewall COM API used by game installers; `WMPhoto.dll` is the JPEG XR codec. Both now KEEP | Rules fixed |
| 7 | `SSDPSRV` first set to disable in Batch 1 | Some apps use Windows' UPnP API for port mapping; moved to **Batch 2** | Changed in files and `OfflinePolicy.csv` |
| 8 | Turn 1 sorted Firewall as "fine to disable" | Final decision: **keep `mpssvc` and `BFE`** | Superseded |
| 9 | `Build-GamingImage.ps1` helper named `Do` | `do` is a PowerShell keyword; renamed `Act` | Fixed |

## 3. Decision register

Status key: **Proposed** = Claude's recommendation, user proceeded (implicit acceptance, not explicitly confirmed). **Superseded** = replaced later. **Open** = depends on missing user input.

| # | Decision | Reason | Status | Evidence |
|---|---|---|---|---|
| D1 | Build on **index 2 (IoT Enterprise LTSC)**, export only that index | Support to 2032 vs 12 Jan 2027 for index 1 | Proposed; user later scanned/mounted index 2 | Turns 4, 6 |
| D2 | Never modify the source WIM; work on an exported copy | Safety/reproducibility | Proposed | Turn 7 script |
| D3 | Integrate latest SSU + CU + .NET CU **before** removals and before disabling Windows Update | Image is Oct 2021 level | Proposed; **not yet done** | Turns 4, 6, 7 |
| D4 | Remove via DISM/NTLite/service disable; **never hand-delete System32 files** | WinSxS hard links break servicing | Proposed | Turn 4 (09 files) |
| D5 | Disable services by `Start=4` offline; never delete service keys or `.sys` files | Reversible | Proposed | Turns 2, 6 |
| D6 | Batch model B1/B2/B3 with dry-run before apply | Risk control | Proposed | Turns 4-7 |
| D7 | Disable Windows Update (wuauserv, UsoSvc, WaaSMedicSvc + policy) and patch manually/offline; keep `TrustedInstaller`/WinSxS | Avoid background update work while keeping servicing | Proposed | 06, script |
| D8 | `TrustedInstaller` is Automatic in this image: set to **Manual (Start=3)**, do not disable | Needed for servicing | Proposed | 06 KEEP, `OfflinePolicy.csv` |
| D9 | Defender stack (WinDefend, WdNisSvc, WdBoot/WdFilter/WdNisDrv/MsSecFlt, SecurityHealthService, wscsvc, SecurityHealth Run value, SecHealthUI, policy) disabled in **B2 only**, user must accept no real-time AV | Scan stutter vs security | Proposed; **Open** (anti-cheat requirements unknown) | 06, script |
| D10 | Keep Wi-Fi (`WlanSvc`, `Wcmsvc`) and Bluetooth stacks, and all NIC/Wi-Fi inbox drivers, until hardware is known | Avoid no-network first boot | Proposed; **Open** | 05/06 KEEP |
| D11 | Keep `mpssvc` (firewall) and `BFE` | Launchers, VPN, anti-cheat use BFE; cheap protection | Proposed | 06 KEEP |
| D12 | Disable `SysMain`, `WSearch` (keep Search package), `Spooler`, telemetry (`DiagTrack`, `dmwappushservice`), `OneSyncSvc`, Xbox services, Hyper-V guest services, RDP server, etc. | Background cost / unused stacks | Proposed; SysMain assumes SSD/NVMe (**NOT CONFIRMED**) | 06 REMOVE |
| D13 | Keep `defragsvc` | TRIM/retrim for SSD health | Proposed | 06 KEEP |
| D14 | Keep Edge (Chromium) until another browser is installed; keep WebView2 runtime; disable `edgeupdate*` in B2 | Need a browser and launchers use WebView2 | Proposed | 09 KEEP, 06 |
| D15 | Remove IE, WMP, Hello Face, speech/OCR/handwriting, Fax/Print FoD, QuickAssist, StepsRecorder, OneSync, PowerShell 2.0, etc. via DISM | Unused | Proposed | 01-03 REMOVE |
| D16 | Keep Notepad, DirectX database FoD, en-US language features, `UserExperience-Desktop` shell components, .NET 4.8 pieces | Needed/gaming-relevant | Proposed | 01-03 KEEP |
| D17 | **Enable .NET 3.5** from `sources\sxs`; leave DirectPlay available but disabled | Old games/mod tools | Proposed (the one "addition") | 02 KEEP |
| D18 | Keep all `c_fs*` file-system filter class INFs, `apppatch`, `hnetcfg.dll`, `winscard.dll`, `winspool.drv`/`spoolss.dll`, `mstsc` | Anti-cheat minifilters, compat shims, firewall COM API | Proposed | 05/09 |
| D19 | Inbox driver-store pruning (05) is **Batch 3 / optional**; DISM can't remove inbox drivers; no runtime gain | Disk only | Proposed | 05 REMOVE header |
| D20 | Scheduled tasks: disable 163 (B1 141, B2 20, B3 2), keep 50 (DirectX adapter cache/database, .NET NGEN, ScheduledDefrag, time sync, TPM maintenance, Secure-Boot-Update, activation, etc.), applied at first boot with `Disable-ScheduledTask` | Tasks can't be switched off cleanly offline | Proposed | 07, scripts |
| D21 | VBS/HVCI off only with explicit `-DisableVBS`; check anti-cheat first | Possible FPS gain vs anti-cheat requirements | Proposed; **Open** | Build script |
| D22 | Keep WinRE (`Recovery`) | Disk-only cost; useful while iterating | Proposed | 09 KEEP |
| D23 | Keep Storage Spaces (`spaceport`, `smphost` decision conditional), `RasMan`/SSTP VPN stack, `W32Time`, `msiserver`, `Appinfo`, `AppXSvc`, `BITS` | Common needs | Proposed | 06 KEEP |
| D24 | SystemApps removal list (B1: ContentDeliveryManager, PeopleExperienceHost, BioEnrollment, AssignedAccessLockApp, CallingShellApp, XboxGameCallableUI, ParentalControls; B2: legacy Edge x2, SecHealthUI; B3: five more) | Unused UWP shell apps | Proposed; riskiest B1 step | 09, script |
| D25 | Xbox/Game Pass stack not supported; `XblAuthManager`, `XblGameSave`, `XboxGipSvc`, `XboxNetApiSvc` disabled | LTSC has no Store/Gaming Services | Proposed; depends on D-unknown (Game Pass use) | 06 |
| R1 | Rejected/Superseded: disabling the firewall; removing `SearchEngine-Client-Package`; hand-deleting System32 DLLs; removing WinRE; removing NIC drivers now | See sections above | Superseded/rejected by assistant | Turns 1-4 |

**Suggestions not yet implemented or confirmed:** MSI-mode interrupts for GPU/NIC, timer-resolution tests, HAGS test, network (Nagle/RSS) tuning, Defender exclusions if Defender is kept, LatencyMon/CapFrameX/PresentMon baseline measurement, `autounattend.xml` generation, driver injection, comparing the build against Atlas OS/ReviOS playbooks. Only the Ultimate Performance power plan, `powercfg -h off`, Game DVR off and Game Mode on are scripted (Build-GamingImage first-boot/policies).

## 4. Current project state

### Completed (delivered by the assistant)
- Anatomy/procedure guidance (Turns 1-2).
- `Build-OSMap.ps1` (enumeration script) and the user's successful scan of **index 1** (uploaded `OSMap.zip`).
- `OSMap_Decisions/` 20 files (00-09, REMOVE + KEEP). Every scanned item was classified exactly once (generator asserted full coverage). 07 was later regenerated from real data.
- `Get-OfflineTasks.ps1` and the user's run (213 tasks; end-of-run hive unload error).
- `Apply-TaskPolicy.ps1`, `Apply-OfflinePolicy.ps1`, `OfflinePolicy.csv`.
- `Build-GamingImage.ps1` (one-shot pipeline; untested).
- Verified support dates by web search (Enterprise LTSC 2021 to 12 Jan 2027, IoT Enterprise LTSC 2021 to 13 Jan 2032).

### In progress
- User is at the stage of having a read-only mount of index 2 (`C:\mount2`) and the decisions/scripts; has **not reported running** `Apply-OfflinePolicy.ps1`, `Apply-TaskPolicy.ps1` or `Build-GamingImage.ps1` (`NOT CONFIRMED`).

### Pending
- Re-scan index 2 with `Build-OSMap.ps1` and diff against the index-1 scan.
- Download latest SSU/CU/.NET CU from the Microsoft Update Catalog (assistant reported October 2026 CU as **KB5126657, build 19044.7807**: from a news source, `NOT CONFIRMED` against the Catalog).
- Dry run, then B1 build, VM test, then B2, then optionally B3.
- Provide hardware/anti-cheat/Game Pass info to settle conditional decisions.
- Unattended install file and driver injection (storage/NIC) if needed.
- Baseline and post-change performance measurement.

### Blocked
- Conditional decisions (Wi-Fi/Bluetooth, Defender, VBS, SysMain, Sensors, PIN/Hello services, Storage Spaces) wait for user hardware and anti-cheat information.

### Not started
- `autounattend.xml`, driver injection, ISO creation test, VM test results, performance benchmarks, GPU/BIOS tuning guidance, post-install tuning beyond the power plan.

### Exact end point
The last user action in the project was reporting that the index 2 mount is `C:\mount2` (read-only) and supplying `tasks_registry.md`. The assistant then delivered the tasks decisions, the apply scripts, and `Build-GamingImage.ps1`. The user then asked for this context migration. No result from running `Build-GamingImage.ps1` exists.

## 5. Deliverable inventory (files produced; paths as delivered in `/mnt/user-data/outputs/` in the original workspace)

| File | Purpose | Run/verified? |
|---|---|---|
| `Build-OSMap.ps1` | Mounts a WIM read-only (or uses `-MountDir`), writes `00_image`..`09_filesystem` reports (md/csv/raw) | User ran it (index 1); output accepted |
| `Get-OfflineTasks.ps1` | Reads registry TaskCache of an image into `tasks_registry.md` | User ran it; wrote 213 tasks; unload error `Access is denied` |
| `OSMap_Decisions/` + `OSMap_Decisions.zip` | 20 REMOVE/KEEP decision files | Not "run"; coverage asserted by generator |
| `OfflinePolicy.csv` | 273-line policy: services/drivers/RunValue, with batch and expected old Start | Not run |
| `Apply-OfflinePolicy.ps1` | Dry-run-by-default offline Start=4 editor with revert log | Not run |
| `Apply-TaskPolicy.ps1` | Disables listed scheduled tasks on the installed OS | Not run |
| `Build-GamingImage.ps1` | End-to-end pipeline, dry-run unless `-Execute` | Not run, only syntax-structure checked |

Counts from the decisions (index 1 scan unless noted): packages 46 (32 remove/14 keep); features 132 (124/8); capabilities 22 installed + 324 absent; Appx 0; DISM inbox drivers 658 (361 optional-remove/297 keep); Win32 services 250 + kernel drivers 360; **tasks (index 2) 213 (163 disable/50 keep)**; startup 1; filesystem 283 folder/app rows plus 4,283 System32 files (28 groups) and 412 driver files (108 disable/304 keep). `OfflinePolicy.csv` batches: services B1 143 / B2 15, drivers B1 42 / B2 9 / B3 62, RunValue B2 1.

## 6. Risks and limitations
- Scripts are **unrun**; first real run may fail (updates integration, DISM feature removal order, takeown/icacls on SystemApps, oscdimg).
- Index-1 scan used to decide an index-2 build.
- Expected performance gain is **modest** (frametime consistency, idle RAM/CPU); large FPS gains come from drivers/BIOS/cooling (assistant's statement; no measurements taken).
- Disabling Defender/Update lowers security; image has no automatic patching.
- SystemApps deletion can leave dangling app registrations (possible Start/Settings issues).
- Several decisions rest on unstated hardware (SSD/NVMe, desktop, no touch/sensors).
- Licensing/activation of IoT edition: user should confirm their key matches the edition (assistant noted once).

## 7. User working preferences (supported by the conversation only)
- Wants **exhaustive, item-by-item** treatment ("each and every item", "take time", "look back into each and every item in the zip file and subfolders").
- Prefers **automation** (scripts) and **structured files** (directory-like text/md) over prose.
- Wants descriptions for every component: purpose, and why kept/removed.
- Wants to **learn** the OS/ISO internals while building.
- Wants the assistant to "research" and "support whenever needed".
- Writes brief informal messages; pastes raw PowerShell output.
- Ordering preference: stepwise ("lets move to the next step").
- Expressed objective weight: gaming resources must not be compromised.
- Budget, safety requirements, preferred explanation depth beyond the above: `UNKNOWN`. Dry-run/batching is the assistant's own safety design, not a stated user requirement.
- Things to avoid: nothing explicitly stated. Observed: the user supplies data when asked, but did not answer the hardware questions.
