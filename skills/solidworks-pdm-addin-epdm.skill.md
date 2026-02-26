---
name: solidworks-pdm-addin-epdm
description: Create a SOLIDWORKS PDM Professional (EPDM) add-in (IEdmAddIn5) that adds File Explorer vault-view menu commands, registers hooks (reactors), handles OnCmd, supports debugging via PDM Admin “Debug add-ins”, and accounts for .NET reload limits and 32/64-bit packaging.
version: 1.0.0
tags:
  - solidworks
  - pdm
  - epdm
  - api
  - add-in
  - iedmaddin5
  - csharp
  - cpp
---

# SOLIDWORKS PDM Professional add-in (EPDM API / IEdmAddIn5)

## When to use this skill
Use when you need a **SOLIDWORKS PDM Professional** add-in that runs in the PDM ecosystem (e.g., **File Explorer vault views**, data cards, PDM hooks/reactors), implemented via the **EPDM API** (`IEdmAddIn5`).

This is **not** the same as a SOLIDWORKS CAD add-in (ISwAddin) that loads into `sldworks.exe`.

## What you should produce (outputs)
- A COM-registered add-in DLL (C#/.NET or native C++) that:
  - Implements `IEdmAddIn5::GetAddInInfo` and `IEdmAddIn5::OnCmd`
  - Adds at least one context menu command in a File Explorer vault view
  - (Optional) Registers one or more hooks via `IEdmCmdMgr5::AddHook`
  - Can be installed and debugged safely (debug add-in installed only on one machine)

## Authoritative documentation (SOLIDWORKS 2025, entry point you provided)
The “Add-in Applications” list (id=4.5) is here:  
https://help.solidworks.com/2025/english/api/help_list.htm?id=4.5

Key subtopics used by this skill:
- Creating add-ins
  - C++: https://help.solidworks.com/2025/english/api/epdmapi/cppaddin.htm?id=4.5.0.0
  - C#: https://help.solidworks.com/2025/english/api/epdmapi/CSharpAddIns.htm?id=4.5.0.1
  - VB.NET: https://help.solidworks.com/2025/english/api/epdmapi/dotnetaddins.htm?id=4.5.0.2
- Creating menu commands
  - C++: https://help.solidworks.com/2025/english/api/epdmapi/cppmenuitem.htm?id=4.5.1.0
  - C#: https://help.solidworks.com/2025/english/api/epdmapi/csharpmenuitem.htm?id=4.5.1.1
  - VB.NET: https://help.solidworks.com/2025/english/api/epdmapi/vbmenuitem.htm?id=4.5.1.2
- Creating add-in hooks (reactors)
  - C++: https://help.solidworks.com/2025/english/api/epdmapi/cppreactor.htm?id=4.5.2.0
  - C#: https://help.solidworks.com/2025/english/api/epdmapi/csharpreactor.htm?id=4.5.2.1
  - VB.NET: https://help.solidworks.com/2025/english/api/epdmapi/vbreactor.htm?id=4.5.2.2
- Serial numbers
  - VB.NET: https://help.solidworks.com/2025/english/api/epdmapi/vbserno.htm?id=4.5.3.0
  - C#: https://help.solidworks.com/2025/english/api/epdmapi/csharpserno.htm?id=4.5.3.1
- Calling add-ins from data cards
  - VB.NET: https://help.solidworks.com/2025/english/api/epdmapi/vbcardbutton.htm?id=4.5.4.0
  - C#: https://help.solidworks.com/2025/english/api/epdmapi/csharpcardbutton.htm?id=4.5.4.1
- Debugging add-ins: https://help.solidworks.com/2025/english/api/epdmapi/DebugAddins.htm?id=4.5.6
- Keeping add-in windows foreground: https://help.solidworks.com/2025/english/api/epdmapi/keepwindowinfront.htm?id=4.5.7
- .NET framework guidance: https://help.solidworks.com/2025/english/api/epdmapi/Using_NET_Framework_in_Addins.htm?id=4.5.8
- 64-bit guidance: https://help.solidworks.com/2025/english/api/epdmapi/64bit.htm?id=4.5.10

---

## Step-by-step: C# add-in (recommended for quick iteration)

### 0) Understand the .NET reload limitation (affects your dev loop)
The 2025 docs repeat this warning across C#/VB.NET EPDM add-in topics:
> SOLIDWORKS PDM Professional cannot force a reload of add-ins if they are written in .NET … all client machines must be restarted to ensure that the latest version of the add-in is used.  
https://help.solidworks.com/2025/english/api/epdmapi/CSharpAddIns.htm?id=4.5.0.1

Plan your iteration accordingly (debug add-in workflow below helps, but you still often need a restart).

### 1) Create the Visual Studio project
Follow “Creating Menu Commands (C#)” and/or “Creating Add-ins (C#)”:
- Create a **Class Library (.NET Framework)** project
- Build target guidance in the menu commands topic includes selecting **Any CPU**  
https://help.solidworks.com/2025/english/api/epdmapi/csharpmenuitem.htm?id=4.5.1.1  
https://help.solidworks.com/2025/english/api/epdmapi/CSharpAddIns.htm?id=4.5.0.1

### 2) Implement IEdmAddIn5
From “Creating Add-ins (C#)” (signatures shown in the doc):
- `public class Class1 : IEdmAddIn5`
- `public void GetAddInInfo(ref EdmAddInInfo poInfo, IEdmVault5 poVault, IEdmCmdMgr5 poCmdMgr)`
- `public void OnCmd(ref EdmCmd poCmd, ref EdmCmdData[] ppoData)`  
https://help.solidworks.com/2025/english/api/epdmapi/CSharpAddIns.htm?id=4.5.0.1

### 3) Add menu commands in GetAddInInfo (context menu / Tools menu)
Use the “Creating Menu Commands (C#)” topic as the canonical guide:
- Define multiple commands and use IDs so `OnCmd` can tell which was invoked  
https://help.solidworks.com/2025/english/api/epdmapi/csharpmenuitem.htm?id=4.5.1.1

### 4) Add hooks (reactors) via IEdmCmdMgr5::AddHook
To react to PDM events (file added/checked out/checked in), the “Creating Add-in Hooks (C#)” topic instructs:
- In `GetAddInInfo`, call `poCmdMgr.AddHook(...)` for each desired activity
- In `OnCmd`, handle hook invocations; SOLIDWORKS PDM calls `OnCmd` whenever a registered hook triggers, and you inspect `poCmd.meCmdType` to see which hook fired.  
https://help.solidworks.com/2025/english/api/epdmapi/csharpreactor.htm?id=4.5.2.1

Examples explicitly listed in the 2025 doc include:
- `EdmCmdType.EdmCmd_PostAdd`
- `EdmCmdType.EdmCmd_PostLock`
- `EdmCmdType.EdmCmd_PreUnlock`
- `EdmCmdType.EdmCmd_PostUnlock`  
https://help.solidworks.com/2025/english/api/epdmapi/csharpreactor.htm?id=4.5.2.1

### 5) (Optional) Call add-ins from data cards and generate serial numbers
Use these samples as patterns (keep them “as patterns”, don’t blindly copy without adapting):
- Data-card button calls add-in (`GetAddInInfo` + `OnCmd` invoked by card button):  
  https://help.solidworks.com/2025/english/api/epdmapi/csharpcardbutton.htm?id=4.5.4.1
- Serial-number generation sample:  
  https://help.solidworks.com/2025/english/api/epdmapi/csharpserno.htm?id=4.5.3.1

---

## Step-by-step: C++ add-in (native, ATL-style)
Follow “Creating Add-ins (C++)”:
- Start Visual Studio **as Administrator**
- Create an **ATL Project**
- Add the EPDM add-in interface `IEdmAddIn5` so the wizard generates `GetAddInInfo` and `OnCmd` stubs
- Add hooks/menu commands in `GetAddInInfo`, and handle in `OnCmd`  
https://help.solidworks.com/2025/english/api/epdmapi/cppaddin.htm?id=4.5.0.0

Menu commands in C++:  
https://help.solidworks.com/2025/english/api/epdmapi/cppmenuitem.htm?id=4.5.1.0

Hooks/reactors in C++:  
https://help.solidworks.com/2025/english/api/epdmapi/cppreactor.htm?id=4.5.2.0

---

## Debugging workflow (critical for EPDM add-ins)
EPDM add-ins are typically:
- Stored in the **vault**
- Downloaded and **COM-registered on the client computer** when needed

This causes dev pain because the registered location may differ from your build output folder. The “Debugging Add-ins” topic explains SOLIDWORKS PDM provides a **Debug Add-ins** capability so:
- A debug add-in is registered only on *your* machine (doesn’t affect other vault users)
- You can point the PDM Admin tool to your build output for debugging  
https://help.solidworks.com/2025/english/api/epdmapi/DebugAddins.htm?id=4.5.6

The “Creating Add-ins (C#)” topic also walks through debugging via the Administration tool:
- Expand vault → right-click **Add-ins** → **Debug** (then point to `bin\\Debug`)  
https://help.solidworks.com/2025/english/api/epdmapi/CSharpAddIns.htm?id=4.5.0.1

---

## UI: keep add-in dialog windows in the foreground
If your add-in displays dialogs, follow:
https://help.solidworks.com/2025/english/api/epdmapi/keepwindowinfront.htm?id=4.5.7

Highlights from that topic:
- C++: specify a parent window handle when creating dialogs.
- VB.NET/C#: window handling supported in PDM 6.4+; the doc references:
  - `IEdmVault5::SetAddInWnd`
  - `IEdmVault8::GetWin32Window`

---

## 32-bit vs 64-bit (don’t guess; follow the doc)
Follow: https://help.solidworks.com/2025/english/api/epdmapi/64bit.htm?id=4.5.10

Key rules called out:
- Managed (.NET) DLL add-ins: build target must be **Any CPU** to load in both 32-bit and 64-bit hosts.
- Native add-ins: bitness must match host process; if you need both, build two versions.
- PDM can create a single add-in package containing both 32-bit and 64-bit versions.

---

## .NET Framework compatibility
Use the compatibility table and configuration guidance here (especially when targeting older PDM versions):  
https://help.solidworks.com/2025/english/api/epdmapi/Using_NET_Framework_in_Addins.htm?id=4.5.8

