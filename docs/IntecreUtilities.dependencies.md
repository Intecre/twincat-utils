# Dependencies of IntecreUtilities

Generated 2026-11-10 15:18 on `INTECRE-XPS15-M` from `C:\src\repos\Intecre\twincat-utils\IntecreUtilities\IntecreUtilities.sln`.

Resolved versions, installed versions and TcPkg packages describe **this machine**; the requested versions come from the project files.

## TwinCAT

| | |
|---|---|
| XAE version (tsproj TcVersion) | `3.1.4026.25` |
| PLC ProgramVersion | `3.1.4026.25` |
| TwinCAT XAE engineering | `TwinCAT.Standard.XAE` (installed: 4026.25.0) |

## Projects

| Project | Kind | File | PLC project |
|---|---|---|---|
| IntecreUtilities | TwinCAT XAE | `IntecreUtilities\IntecreUtilities.tsproj` |  |
| IntecreUtilities | PLC | `IntecreUtilities\IntecreUtilities\IntecreUtilities.plcproj` | library: Intecre.Utilities 4.2.0.0 (Intecre Ltd) |
| TestRunner | PLC | `IntecreUtilities\TestRunner\TestRunner.plcproj` | library: TestRunner 1.0.0.0 (Intecre Ltd) |

## PLC library references

| Library | Company | Requested | Resolves to | Installed versions | TcPkg package | Provided by workload | Used by |
|---|---|---|---|---|---|---|---|
| **Intecre.Utilities** | Intecre Ltd | * (placeholder) | `4.2.0.0` | 3.1.0.0, 3.1.4.0, 3.3.2.2, 3.4.1.0, 3.4.2.0, 3.4.3.0, 4.0.0.0, 4.1.0.0, 4.2.0.0 | - | - | TestRunner |
| **Tc2_Math** | Beckhoff Automation GmbH | * (placeholder) | `3.5.3.0` | 3.3.1.0, 3.4.4.0, 3.5.1.0, 3.5.3.0 | `TwinCAT.XAE.PLC.Lib.Tc2_Math` 3.5.1, 3.5.3 | TwinCAT.StandardRM.XAE 4026.25.0 | IntecreUtilities |
| **Tc2_Standard** | Beckhoff Automation GmbH | * (placeholder) | `3.4.7.0` | 3.3.2.0, 3.3.3.0, 3.4.5.0, 3.4.7.0 | `TwinCAT.XAE.PLC.Lib.Tc2_Standard` 3.4.5, 3.4.7 | TwinCAT.StandardRM.XAE 4026.25.0 | IntecreUtilities, TestRunner |
| **Tc2_System** | Beckhoff Automation GmbH | * (placeholder) | `3.10.2.0` | 3.4.22.0, 3.4.24.0, 3.5.3.0, 3.6.4.0, 3.9.1.0, 3.10.1.0, 3.10.2.0 | `TwinCAT.XAE.PLC.Lib.Tc2_System` 3.9.1, 3.10.1, 3.10.2 | TwinCAT.StandardRM.XAE 4026.25.0 | IntecreUtilities, TestRunner |
| **Tc2_Utilities** | Beckhoff Automation GmbH | * (placeholder) | `3.10.8.0` | 3.3.35.0, 3.3.47.0, 3.3.54.0, 3.6.1.0, 3.7.4.0, 3.9.2.0, 3.10.1.0, 3.10.6.0, 3.10.8.0 | `TwinCAT.XAE.PLC.Lib.Tc2_Utilities` 3.10.1, 3.10.6, 3.10.8 | TwinCAT.StandardRM.XAE 4026.25.0 | IntecreUtilities, TestRunner |
| **Tc3_IPCDiag** | Beckhoff Automation GmbH | * (placeholder) | `1.2.6.0` | 1.2.2.0, 1.2.6.0 | `TwinCAT.XAE.PLC.Lib.Tc3_IPCDiag` 1.2.2, 1.2.6 | TwinCAT.StandardRM.XAE 4026.25.0 | IntecreUtilities |
| **Tc3_Module** | Beckhoff Automation GmbH | * (placeholder) | `3.4.7.0` | 3.3.21.0, 3.3.23.0, 3.4.5.0, 3.4.7.0 | `TwinCAT.XAE.PLC.Lib.Tc3_Module` 3.4.5, 3.4.7 | TwinCAT.StandardRM.XAE 4026.25.0 | IntecreUtilities, TestRunner |
| **TcUnit** | www.tcunit.org | * (placeholder) | `1.3.1` | 1.3.1 | - | - | TestRunner |

## Transitive library dependencies

Read from each installed library's `dependencies` file; not referenced directly by any project.

| Library | Company | Requested | Resolves to | TcPkg package | Required by |
|---|---|---|---|---|---|
| Base Interfaces | System | * | `3.5.17.0` | - | TcUnit |
| CmpErrors2 Interfaces | System | * | `3.5.21.0` | - | SysDir |
| SysDir | System | * | `3.5.17.0` | - | TcUnit |
| SysFile | System | * | `3.5.17.0` | - | TcUnit |
| SysTypes2 Interfaces | System | * | `3.5.17.0` | - | SysDir, SysFile |
| Tc3_DynamicMemory | Beckhoff Automation GmbH | * | `1.1.6.0` | `TwinCAT.XAE.PLC.Lib.Tc3_DynamicMemory` 1.1.4, 1.1.6 | Tc3_IPCDiag |
| Tc3_EventLogger | Beckhoff Automation GmbH | * | `3.3.15.0` | `TwinCAT.XAE.PLC.Lib.Tc3_EventLogger` 3.3.13, 3.3.15 | Tc3_IPCDiag, Tc3_DynamicMemory |

## Setting up a fresh engineering PC

Beckhoff libraries arrive with these workloads:

- `TwinCAT.StandardRM.XAE` 4026.25.0

```bash
tcpkg install TwinCAT.Standard.XAE=4026.25.0
```

Third-party libraries with no TcPkg package on this machine; install the `.library` files with RepTool or the Library Repository dialog:

- TcUnit 1.3.1 (www.tcunit.org)
- Intecre.Utilities 4.2.0.0 (Intecre Ltd)

CODESYS system libraries, shipped with the XAE: Base Interfaces 3.5.17.0, SysDir 3.5.17.0, SysFile 3.5.17.0, SysTypes2 Interfaces 3.5.17.0, CmpErrors2 Interfaces 3.5.21.0.

## Warnings

- TcUnit: third-party library referenced as '*' - the build is not reproducible; pin a version (resolves to 1.3.1 on this machine)
- Intecre.Utilities: third-party library referenced as '*' - the build is not reproducible; pin a version (resolves to 4.2.0.0 on this machine)

