# SOLIDWORKS add-in skills

This repo contains **Cursor skills** for building SOLIDWORKS add-ins, based on the official **SOLIDWORKS 2025 API Web Help**.

## Skills

- `skills/solidworks-cad-addin-iswaddin.skill.md`  
  Create a **SOLIDWORKS (CAD) desktop add-in** using `ISwAddin` (C# focus, with notes for other languages). Covers COM + registry registration, `ConnectToSW` / `DisconnectFromSW`, CommandManager UI, callbacks, icons, toolbars, and packaging pointers.

- `skills/solidworks-pdm-addin-epdm.skill.md`  
  Create a **SOLIDWORKS PDM Professional (EPDM) add-in** using `IEdmAddIn5`. Covers menu commands in File Explorer vault views, hooks/reactors via `IEdmCmdMgr5::AddHook`, debugging via PDM Admin “Debug add-ins”, and 32/64-bit + .NET considerations.