---
name: solidworks-cad-addin-iswaddin
description: Create a SOLIDWORKS (CAD) desktop add-in using the SOLIDWORKS API SDK templates or by implementing ISwAddin/SwAddin manually; includes COM/registry registration, ConnectToSW/DisconnectFromSW lifecycle, CommandManager UI, callbacks, icons, and toolbar considerations.
version: 1.0.0
tags:
  - solidworks
  - sldworks
  - api
  - add-in
  - iswaddin
  - csharp
  - com
---

# SOLIDWORKS CAD add-in (ISwAddin) — create, register, and wire UI

## When to use this skill
Use when you need to create a **SOLIDWORKS desktop add-in** (loaded by SOLIDWORKS itself; not a PDM vault add-in), implemented as a COM-visible DLL that SOLIDWORKS loads via **ISwAddin**.

## What you should produce (outputs)
- A C# (or VB.NET / C++/CLI / C++) add-in project that:
  - Implements `SolidWorks.Interop.swpublished.ISwAddin` (`ConnectToSW`, `DisconnectFromSW`)
  - Registers itself so it appears in **Tools > Add-ins...**
  - Adds at least one command (menu/toolbar/CommandManager) with correct callback/enable signatures
  - Has proper icons (including scaled icons if using .NET templates)

## Authoritative documentation (SOLIDWORKS 2025)
- Add-in creation routes (templates/wizards): https://help.solidworks.com/2025/english/api/sldworksapiprogguide/Overview/SolidWorks_API_Add-Ins,_Project_Templates,_and_Wizards.htm?id=1.2.3.0
- Use .NET add-in templates: https://help.solidworks.com/2025/english/api/sldworksapiprogguide/Overview/SolidWorks_CSharp_and_VB.NET__Project_Templates.htm?id=1.2.3.1
- Manual “SwAddin” approach (ISwAddin + registry): https://help.solidworks.com/2025/english/api/sldworksapiprogguide/Overview/Using_SwAddin_to_Create_a_SolidWorks_Addin.htm?id=1.2.3.5
- CommandManager & command groups: https://help.solidworks.com/2025/English/api/sldworksapiprogguide/Overview/CommandManager_and_CommandGroups.htm?id=e695b1d69d874993be07528d907b2e75
- Callback/enable method signatures: https://help.solidworks.com/2025/English/api/sldworksapiprogguide/Overview/Add-in_Callbacks.htm
- Scaled add-in icons: https://help.solidworks.com/2025/English/api/sldworksapiprogguide/Overview/Add-in_Icons.htm
- Toolbar behavior/IDs: https://help.solidworks.com/2025/English/api/sldworksapiprogguide/Overview/Add-in_Toolbars.htm
- Concrete reference implementation (C#): https://help.solidworks.com/2025/english/api/sldworksapi/Create_TaskPaneView_Add-in_Example_CSharp.htm

---

## Step-by-step: create the project (recommended: .NET template)

### 0) Install the SOLIDWORKS API SDK templates
Follow the “Create Task Pane View Add-in Example (C#)” prerequisites:
- Install the SOLIDWORKS API SDK templates (the docs reference running `SOLIDWORKS API SDK.msi` from the SOLIDWORKS installation’s `apisdk` folder):  
  https://help.solidworks.com/2025/english/api/sldworksapi/Create_TaskPaneView_Add-in_Example_CSharp.htm

### 1) Create a new add-in project in Visual Studio
Use the SOLIDWORKS API SDK add-in template:
- `SwCSharpAddin` (C#) or `SwVBAddin` (VB.NET) per:  
  https://help.solidworks.com/2025/english/api/sldworksapiprogguide/Overview/SolidWorks_CSharp_and_VB.NET__Project_Templates.htm?id=1.2.3.1

### 2) Ensure COM visibility + identity
In your main add-in class (commonly `SwAddin`):
- Ensure `[ComVisible(true)]`
- Ensure a stable `[Guid("...")]` (new GUID per add-in)
- Ensure `[SwAddin(Description=..., Title=..., LoadAtStartup=...)]` is present (as shown in the C# TaskPaneView example):  
  https://help.solidworks.com/2025/english/api/sldworksapi/Create_TaskPaneView_Add-in_Example_CSharp.htm

### 3) Implement COM registration that SOLIDWORKS uses to find your add-in
SOLIDWORKS add-ins must be registered as COM servers and then listed under the SOLIDWORKS add-in registry key (see “Using SwAddin to Create a SOLIDWORKS Add-In”):
- Register via:
  - `Regsvr32.exe` (unmanaged C++ and managed C++/CLI add-ins)
  - `RegAsm.exe` (C# and VB.NET add-ins)  
  https://help.solidworks.com/2025/english/api/sldworksapiprogguide/Overview/Using_SwAddin_to_Create_a_SolidWorks_Addin.htm?id=1.2.3.5

For .NET add-ins, use `[ComRegisterFunction]` / `[ComUnregisterFunction]` to create/remove the keys (the TaskPaneView example shows exactly what SOLIDWORKS expects):

- **HKLM**: `HKEY_LOCAL_MACHINE\SOFTWARE\SolidWorks\Addins\{GUID}`
  - Default value: `0` (or `1` to enable in Add-in Manager per SwAddin doc)
  - `Description`: string (shown in Add-in Manager)
  - `Title`: string (shown in Add-in Manager)
- **HKCU**: `HKEY_CURRENT_USER\Software\SolidWorks\AddInsStartup\{GUID}`
  - Default value: DWORD `0/1` controlling load-at-startup (`LoadAtStartup`)

Reference code pattern (from the TaskPaneView add-in example):
https://help.solidworks.com/2025/english/api/sldworksapi/Create_TaskPaneView_Add-in_Example_CSharp.htm

**Important:** writing HKLM requires elevated permissions on Windows; plan your install story accordingly (admin install or installer writes keys).

### 4) Implement ISwAddin lifecycle correctly
Implement:
- `bool ConnectToSW(object ThisSW, int Cookie)` (called when the add-in is loaded)
- `bool DisconnectFromSW()` (called when SOLIDWORKS is closing or the add-in is disabled)

In `ConnectToSW`, the SwAddin manual notes you can call:
- `ISldWorks::SetAddinCallbackInfo` (SOLIDWORKS “holds onto this object and makes callbacks”)
- `ISldWorks::AddMenuItem3`
- `ISldWorks::AddToolbar4`  
https://help.solidworks.com/2025/english/api/sldworksapiprogguide/Overview/Using_SwAddin_to_Create_a_SolidWorks_Addin.htm?id=1.2.3.5

The C# TaskPaneView example shows calling:
- `iSwApp.SetAddinCallbackInfo(0, this, addinID);`  
https://help.solidworks.com/2025/english/api/sldworksapi/Create_TaskPaneView_Add-in_Example_CSharp.htm

### 5) Add UI: CommandManager + CommandGroups (recommended pattern)
Use:
- `ISldWorks::GetCommandManager` to get the one `ICommandManager` instance
- Create one or more `ICommandGroup`s via `ICommandManager::CreateCommandGroup`
- Add commands via `ICommandGroup::AddCommandItem2`
- Activate via `ICommandGroup::Activate`

This is covered in:
https://help.solidworks.com/2025/English/api/sldworksapiprogguide/Overview/CommandManager_and_CommandGroups.htm?id=e695b1d69d874993be07528d907b2e75

#### Callback + enable method signatures (critical)
Your menu/toolbar command callbacks and enable methods must use one of the supported signatures:
- No parameters
- One `string` parameter (supported in SOLIDWORKS 2012 SP0+)  
https://help.solidworks.com/2025/English/api/sldworksapiprogguide/Overview/Add-in_Callbacks.htm

### 6) Icons: provide scaled icons (especially for .NET templates)
SOLIDWORKS Add-in Manager shows an icon next to your add-in name, and supports scaled icons (as of SOLIDWORKS 2022). The docs describe:
- .NET templates include default icons of various sizes
- A **post-build event** renames them to match the DLL name and puts them next to the DLL
- You can replace the default icons with your own scaled icon set  
https://help.solidworks.com/2025/English/api/sldworksapiprogguide/Overview/Add-in_Icons.htm

The Add-in Icons doc references scaled sizes (example set): **16, 20, 32, 40, 64, 96, 128** (e.g., `MySwAddin_20.png`, `MySwAddin_40.png`, etc.).

If you don’t place icons beside the DLL, the doc describes setting an `Icon` value under the add-in registry key (e.g., `HKEY_LOCAL_MACHINE\SOFTWARE\SOLIDWORKS\AddIns\{CLSID}\Icon`), with version-specific variants:  
https://help.solidworks.com/2025/English/api/sldworksapiprogguide/Overview/Add-in_Icons.htm

### 7) Toolbars: handle toolbar IDs during development vs release
The toolbar docs explain:
- First run “instantiates” toolbars and stores **toolbar IDs in the registry**
- During development, you may need to clear relevant registry entries before changing toolbar structure
- For release versions, assign new toolbar IDs to avoid collisions across versions  
https://help.solidworks.com/2025/English/api/sldworksapiprogguide/Overview/Add-in_Toolbars.htm

The CommandManager doc calls out where toolbar definitions live and how to remove them, e.g.:
- `HKEY_CURRENT_USER\Software\SOLIDWORKS\SOLIDWORKS <version>\User Interface\Custom API Toolbars\<index>`
- `HKEY_CURRENT_USER\Software\SOLIDWORKS\SOLIDWORKS <version>\User Interface\Toolbars\...`  
https://help.solidworks.com/2025/English/api/sldworksapiprogguide/Overview/CommandManager_and_CommandGroups.htm?id=e695b1d69d874993be07528d907b2e75

---

## Packaging/deployment (Windows Installer)
If you need a deployable installer, follow:
https://help.solidworks.com/2025/english/api/sldworksapiprogguide/Miscellaneous/Create_Setup_Project_to_Distribute_SolidWorks_Add-in.htm?format=P&value=

---

## Debug checklist (pragmatic)
1. Confirm COM registration succeeds (for .NET, `regasm /codebase`-style workflows or installer; for unmanaged, `regsvr32`).
2. Confirm registry keys exist:
   - `HKLM\SOFTWARE\SolidWorks\Addins\{GUID}`
   - `HKCU\Software\SolidWorks\AddInsStartup\{GUID}`
3. Start SOLIDWORKS → **Tools > Add-ins...** and confirm:
   - Add-in appears with correct Title/Description
   - Load-at-startup behavior matches expectation
4. Attach debugger to `sldworks.exe`, set breakpoints in `ConnectToSW` and your command callbacks.
5. If UI commands don’t show, verify CommandGroup IDs/activation logic per CommandManager guide, and reset toolbar registry entries during development as needed.

