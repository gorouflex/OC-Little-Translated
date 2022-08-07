# ACPI Basics
<details>
<summary><strong>TABLE of CONTENTS</strong> (click to reveal)</summary>

- [A brief Introduction to ACPI](#a-brief-introduction-to-acpi)
  - [What is ACPI?](#what-is-acpi)
  - [Introduction to ACPI Hotpatching](#introduction-to-acpi-hotpatching)
  - [Important ACPI Tables for hackintoshing](#important-acpi-tables-for-hackintoshing)
    - [`DSDT.aml`](#dsdtaml)
    - [`SSDT-xxxx.aml`](#ssdt-xxxxaml)
- [ACPI Renames and Hotpatches](#acpi-renames-and-hotpatches)
  - [General Patching Guidelines](#general-patching-guidelines)
  - [ACPI patching in OpenCore](#acpi-patching-in-opencore)
    - [ACPI Quirks in OpenCore](#acpi-quirks-in-opencore)
- [ACPI Source Language (ASL) Basics](#acpi-source-language-asl-basics)
  - [Preface](#preface)
  - [ACPI](#acpi)
  - [Why to prefer SSDTs over a patched DSDT](#why-to-prefer-ssdts-over-a-patched-dsdt)
- [ASL](#asl)
  - [ASL Guidelines](#asl-guidelines)
  - [Common ASL Data Types](#common-asl-data-types)
  - [ASL Variables Definition](#asl-variables-definition)
  - [ASL Assignment](#asl-assignment)
  - [ASL Calculation](#asl-calculation)
  - [ASL Logic](#asl-logic)
  - [Defining Methods in ASL](#defining-methods-in-asl)
- [ACPI Preset Functions](#acpi-preset-functions)
  - [`_OSI` (Operating System Interfaces)](#_osi-operating-system-interfaces)
  - [`_STA` (Status)](#_sta-status)
  - [`_CRS` (Current Resource Settings)](#_crs-current-resource-settings)
- [ASL flow Control](#asl-flow-control)
  - [Branch control `If` & `Switch`](#branch-control-if--switch)
    - [`If`](#if)
    - [`ElseIf`, `Else`](#elseif-else)
    - [`Switch`, `Case`, `Default`, `BreakPoint`](#switch-case-default-breakpoint)
  - [Loop control](#loop-control)
    - [`While` & `Stall`](#while--stall)
    - [`For`](#for)
- [`External` Quote](#external-quote)
- [ASL CondRefOf](#asl-condrefof)
- [ASL to AML Conversion Table](#asl-to-aml-conversion-table)
- [SSDT Loading Sequence](#ssdt-loading-sequence)
  - [Examples](#examples)
- [CREDITS & RESOURCES](#credits--resources)
</details>

## A brief Introduction to ACPI
"ACPI… the final frontier…". No, not really. But this is how overwhelmed most Hackintosh users feel the first time, they open an `.aml` file and look inside it. Understanding what to make of all of it seems like an expedition of epochal proportions impossible to grasp. And that's who this introduction is for. But first things first…

### What is ACPI?
**ACPI** stands for Advanced Configuration & Power Interface. It is an architecture-independent power management and configuration standard originally developed by Intel, Microsoft, Toshiba and other manufacturers who formed the ACPI special interest group. In October 2013, the assets of the ACPI specifications were transferred to the UEFI Forum. The latest version of the ACPI Specification was released in January 2021.

The ACPI Form describes a machine's hardware information in `.aml` format and does not have any driver capabilities of its own. However, the correct ACPI is required for a piece of hardware to work properly, otherwise it can lead to boot failures or system crashes. 

Each computer mainboard ships with a set of binary files inside the BIOS, the ACPI Tables/Forms. ACPI serves as an interface layer between the operating system and system firmware, as shown below:

![](https://uefi.org/specs/ACPI/6.4/_images/acpi-overview.png)</br>
**Source**: [**UEFI.org**](https://uefi.org/specs/ACPI/6.4/Frontmatter/Overview/Overview.html)

The number and content of ACPI tables varies from used mainboard and chipset - it even *can* vary between different BIOS versions as well.

### Introduction to ACPI Hotpatching

Unlike Windows, macOS has no hardware detection capabilities because it doesn't need it, since Macs use a fixed set/range of hardware components. So therefore Hackintoshes require fixes for certain ACPI Tables so that macOS can "understand" them. Having the correct ACPI for macOS is the foundation of a stable Hackintosh.

Although `kexts` handle a lot of patching tasks nowadays, it may be necessary to create additional patches to enable certain features (like enabling Thunderbolt or fixing Sleep issues, etc.). The preferred method to do so is **Hotpatching**.

Hotpatching means that ACPI tables or parts of them are manipulated on the fly, during system start. The original ACPI tables are extracted, then patched on the fly and handed over to macOS for further processing. There are two main techniques for hotpatching which are often combined: 1) replacing certain character in the text of ACPI tables (binary renaming) and 2) replacing or adding tables to existing ACPI tables, usually SSDTs.

In order to create Hotpatches, we need to extract – or as we say, dump – the original ACPI tables from the BIOS. Some Tools can extract a machine's ACPI Form - like `SSDTTime` in Windows or Boot Managers like `OpenCore` and `Clover`.

Since ACPI tables are binary files, we need a decompiler to read and edit them, such as: [MaciASL](https://github.com/acidanthera/MaciASL) for macOS or [QtiASL](https://github.com/ic005k/QtiASL) for Windows. When opening these files, especially the `DSDT.aml`, many errors may occur. It is important to note that in most cases, these errors are generated by the decompiler – they are not present in the ACPI forms provided by the machine.

### Important ACPI Tables for hackintoshing

#### `DSDT.aml`
As mentioned, the `DSDT` or [**Differentiated System Description Table**](https://uefi.org/specs/ACPI/6.4/05_ACPI_Software_Programming_Model/ACPI_Software_Programming_Model.html#differentiated-system-description-table-dsdt) is the most important ACPI Table because it includes most of the devices of a mainboard, its features and the way they are powered. This is the primary source for researching possible fixes to turn a PC mainboard into what macOS recognizes as an iMac mainboard, for example.

#### `SSDT-xxxx.aml`
`SSDTs`or [**Secondary System Description Tables**](https://uefi.org/specs/ACPI/6.4/05_ACPI_Software_Programming_Model/ACPI_Software_Programming_Model.html?highlight=ssdt#secondary-system-description-table-ssdt) are tables which can be added to, modify or replace specific parts or sections of the DSDT. This is another important category of ACPI Tables because these can be completely written by user and thereby fix issues, add fake devices which macOS wants to see or improve CPU Power Management. This includes tables such as: `SSDT-PLUG.aml`, `SSDT-PM`, `SSDT-AWAC.aml`, etc.

Other relevant ACPI Tables are: `APIC`, `BGRT`, `DMAR`, `ECDT`, `FACP`.

For more details about ACPI in general, please refer to the official [**ACPI Specification**](https://uefi.org/specs/ACPI/6.4/index.html) For an introduction to the ASL language, please refer to chapter [**ACPI Source Language Reference**](https://uefi.org/specs/ACPI/6.4/19_ASL_Reference/ACPI_Source_Language_Reference.html?highlight=asl%20syntax).

The following sections will help you to get a deeper understanding about ACPI, ASL and Binary Renames, so you can edit SSDT files. Click on a triangle to unfold its content.

## ACPI Renames and Hotpatches

Try to avoid ACPI binary renames and patches such as `HDAS` to `HDEF`, `EC0` to `EC`, `SSDT-OC-XOSI`, etc. whenever possible. Especially renaming of methods (`MethodObj`) such as `_STA`, `_OSI`, etc. should be performed with caution. Nowadays, a lot of renames are handled by kexts like **AppleALC** and **WhateverGreen** anyway.

### General Patching Guidelines

- Whenever possible, use SSDTs to inject preset variables and (Fake) Devices since this method is very reliable and ACPI conform (if done correctly). **Recommended approach**. 
- `Binary Renames`: The binary rename method is especially effective for computers running only macOS. On multi-boot systems with different Operating Systems these patches should be used with **caution** since binary renames apply to all systems which can cause issues. The best way to avoid such issues is to bypass OpenCore when booting into a different OS altogether, so no patches are injected. Or use Clover instead, since it does not inject patches into other OSes.
- No OS patches are required. For components that do not work properly use custom SSDT patches to enable them instead. For special OS requirements, use the `SSDT-XOSI` Patch.
- For Brightness Control Keys to work, some machines do not require extra patches. You can use `PS2 Keyboard Mapping` instead to achieve the same results.
- For now, the vast majority of Notebooks require the `0D6D Patch` to fix `Instant Wake` issues.
- Almost all Laptops require additional, device-specific renames and patches for the Battery Percentage Indicator to work. But recently, a new kext called [ECEnabler](https://github.com/1Revenger1/ECEnabler) was released which enables macOS to read the battery status provided the Embedded Controller, so no more patching is required. It doesn't work in all cases, but it's a good idea to give it a try first.
- Most ThinkPad Laptops require the `PTSWAKTTS` patch to stop the Power Button LED from pulsing after waking up from sleep.
- For machines with a dedicated Sleep Button: if pressing the Sleep Button crashes the system, use the `PNP0C0E Sleep Correction Method` to fix it.

You may need to disable or enable certain components in order to solve specific problems.

### ACPI patching in OpenCore 
OpenCore applies ACPI changes globally to *every* operating system (unlike Clover) in the following order:

1. `Patch` is processed
2. `Delete` is processed
3. `Add` is processed
4. `Quirks` are processed

#### ACPI Quirks in OpenCore
The following section refers to patching other ACPI tables (apart from `DSDT.aml`) in OpenCore with integrated Patches or "Quirks", as shown here:

![ACPI_quirks](https://user-images.githubusercontent.com/76865553/166452836-80cf06a7-3337-4a32-88b1-ac822c5fb43d.png)

Quirk               | Affected Table(s) | Description | Fixes
--------------------|-------------------|-------------|--------------
**FadtEnableReset** | **`FACP.aml`**    |Fixed ACPI Description Table (FADT). In the [ACPI Specification](https://uefi.org/specs/ACPI/6.4/05_ACPI_Software_Programming_Model/ACPI_Software_Programming_Model.html#fixed-acpi-description-table-fadt), FADT defines various static system information related to configuration and power management.| If holding the **Power Button** does not invoke the "Restart, Sleep, Cancel, Shutdown" menu, enable this quirk. If this doesn't fix it, try adding [**SSDT-PMC.aml**](https://github.com/5T33Z0/OC-Little-Translated/tree/main/01_Adding_missing_Devices_and_enabling_Features/PMC_Support_(SSDT-PMC)).</br> `Low Power S0 Idle` state. The **FACP.aml** form characterizes the machine type and determines the power management method. If `Low Power S0 Idle` = `1`, it's an `AOAC` (Always On Always Connected) system. See the [About AOAC](https://github.com/5T33Z0/OC-Little-Translated/tree/main/04_Fixing_Sleep_and_Wake_Issues/Fixing_AOAC_Machines) section for more details.
**NormalizeHeaders** | All Table Headers | Clear ACPI Header fields | Removes illegal characters from ACPI Headers. Only required for macOS 10.13
**ResetLogoStatus** |**`BGRT.aml`**      | Bootstrap graphics resource table | According to the [`ACPI Specs`](https://www.acpica.org/documentation), the `Displayed` item of the form should = `0`. However, some vendors have written non-zero data to the `Displayed` entry for some reason, which may cause the screen refresh to fail during the boot phase. The patch sets `Displayed` to `0`. **Note:** Not all machines have this table.
**RebaseRegions** | All Tables | Relocates ACPI Memory Regions | ACPI forms have memory regions with both dynamically allocated as well as fixed addresses. This quirk turns dynamically allocated addresses into fixed ones which can help with patched `DSDTs`. **Caution**: This patch is very dangerous and should not be chosen unless relocating memory regions solves boot crashes!
**ResetHwSig** | **`FACS.aml`**| Sets Hardware Signature to `0` | `Hardware Signature` is part of the **FACS.aml** table. It's calculated after the system has started based on the hardware configuration. If this value changes after the machine wakes up from a **Hibernate** state, the system will not recover correctly. **Note:** If the system has **Hibernation** disabled, you do not need to this quirk!
**SyncTableIds**| **`SLIC.aml`** | Microsoft Software Licensing table |Fixes `SLIC` table causing licensing issues in older Windows versions.
**DisableIoMapper** | **`DMAR.aml`** | Kernel Quirk to disable Vt-d and [dropping the DMAR table](https://github.com/5T33Z0/OC-Little-Translated/tree/main/00_About_ACPI/).| Usually, only early Mac systems need this patch. But with the release of macOS Monterey this has become relevant again for getting some 2.5 and 10 gig Ethernet Cards to work.

**NOTE**: For more info about ACPI Tables in general, please refer to the official [ACPI Specs](https://uefi.org/specs/ACPI/6.4/05_ACPI_Software_Programming_Model/ACPI_Software_Programming_Model.html#acpi-system-description-tables).

## ACPI Source Language (ASL) Basics

> The provided explanations in this Section are based on the following Post at PCBeta Forums by the User suhetao: "[DIY DSDT Tutorial Series, Part 1: ASL (ACPI Source Language) Basics](http://bbs.pcbeta.com/forum.php?mod=viewthread&tid=944566&archive=2&extra=page%3D1&page=1)"
>
> - Reformatted for Markdown by Bat.bat (williambj1) on 2020-2-14, with some additions and corrections.
> - Translated from Chinese into English and edited by [5T33Z0](https://github.com/5T33Z0).

### Preface
The following information is based on the documentation of the [ACPI Specifications](https://uefi.org/specs/ACPI/6.4/) provided by the Unified Extensible Firmware Interface Forum (UEFI.org). Since I am not a BIOS developer, it is possible that there could be mistakes in the provided ASL examples.

Did you ever wonder what a `DSDT` or `SSDT` is and what it does? Or how these rename patches that you have in your `config.plist` work? Well, after reading this, you will know for sure!

### ACPI
**`ACPI`** stands for `Advanced Configuration & Power Interface`. In the ACPI, peripheral devices and system hardware features of the platform are described in (1) the **`DSDT`** (Differentiated System Description Table), which is loaded at boot and (2) in SSDTs (Secondary System Description Tables), which are loaded *dynamically* at run time. ACPI is literally just a set of tables of texts to provide operating systems with some basic information about the used hardware. **`DSDTs`** and **`SSDTs`** are just *two* of the many tables that make up a system's ACPI – but very important ones for us.

### Why to prefer SSDTs over a patched DSDT
A common problem with Hackintoshes is missing ACPI functionality when trying to run macOS on X86-based Intel and AMD systems, such as: Networking not working, USB Ports not working, CPU Power Management not working correctly, screens not turning off when the lid is closed, Sleep and Wake not working, Brightness controls not working and so on.

These issues stem from DSDTs made with Windows support in mind on one hand and Apple not sticking to ACPI tables which conform to the ACPI specs 100 % for their hardware on the other. These issues can be addressed by dumping, patching and injecting a patched DSDT during boot, replacing the original.

Since a DSDT can change when updating the BIOS, injecting an older DSDT on top of a newer one can cause conflicts and break macOS functionalities. Therefore, *dynamic patching* with SSDTs is highly recommended over using a patched DSDT. Plus the whole process is much more efficient, transparent and elegant. And that's why you should avoid patched DSDTs – especially those from MaLDon/Olarila!

## ASL
A notable feature of `ACPI` is a specific proprietary language to compile ACPI tables. This language is called `ASL` (ACPI Source Language), which is at the center of this article. After an ASL is compiled, it becomes AML (ACPI Machine Language), which can be executed by the operating system. Since ASL is a language, it has its own rules and guidelines.

### ASL Guidelines

1. The variable defined in the `DefinitionBlock` must not exceed 4 characters, and not begin with digits. Just check any DSDT/SSDT – no exceptions.
2. `Scope` is similar to `{}`. There is one and there is only one `Scope`. Therefore, DSDT begins with:

   ```asl
   DefinitionBlock ("xxxx", "DSDT", 0x02, "xxxx", "xxxx", xxxx)
   {
   ```
   and is ended by

   ```asl
   }
   ```

   This is called the `Root Scope`.

The `xxxx` parameters refer to the `File Name`、`OEMID`、`Table ID` and `OEM Version`. The third parameter is based on the second parameter. As shown above, if the second parameter is **`DSDT`**, the third parameter must be `0x02`. Other parameters are free to fill in.

1. Methods and variables beginning with an underscore `_` are reserved for operating systems. That's why some ASL tables contain `_T_X` trigger warnings after decompiling.

2. A `Method` always contains either a `Device` or a `Scope`. As such, a `Method` _cannot_ be defined without a `Scope`. Therefore, the example below is **invalid** because the Method is followed by a `DefinitionBlock`:

   ```asl
   Method (xxxx, 0, NotSerialized)
   {
       ...
   }
   DefinitionBlock ("xxxx", "DSDT", 0x02, "xxxx", "xxxx", xxxx)
   {
       ...
   }
   ```

3. `\_GPE`,`\_PR`,`\_SB`,`\_SI`,`\_TZ` belong to root scope `\`.

   - `\_GPE` &rarr; ACPI Event handlers
   - `\_PR` &rarr; CPU
   - `\_SB` &rarr; Devices and Busses
   - `\_SI` &rarr; System indicator
   - `\_TZ` &rarr; Thermal zone

	Components with different attributes are placed below/inside the corresponding Scope. For example:

   - `Device (PCI0)` is placed inside `Scope (\_SB)`

     ```asl
     Scope (\_SB)
     {
         Device (PCI0)
         {
             ...
         }
         ...
     }
     ```

   - CPU related information is placed in Scope (_PR)

     > CPUs can have various scopes, for instance `_PR`,`_SB`,`_SCK0`

     ```asl
     Scope (_PR)
     {
         Processor (CPU0, 0x00, 0x00000410, 0x06)
         {
             ...
         }
         ...
     }
     ```

   - `Scope (_GPE)` places event handlers

      ```asl
      Scope (_GPE)
      {
          Method (_L0D, 0, NotSerialized)
          {
              ...
          }
          ...
      }
      ```

      Yes, methods can be placed here. Caution, methods beginning with **`_`** are reserved for operating systems.
5. `Device (xxxx)` also can be recognized as a scope, it contains various descriptions to devices, e.g. `_ADR`,`_CID`,`_UID`,`_DSM`,`_STA`.
6. Symbol `\` quotes the root scope; `^` quotes the superior scope. Similarly, `^` is superior to `^^`.
7. Symbol `_` is meaningless, it only completes the 4 characters, e.g. `_OSI`.
8. For better understanding, ACPI releases `ASL+(ASL2.0)`, it introduces C language's `+-*/=`, `<<`, `>>` and logical judgment `==`, `!=` etc.
9. Methods in ASL can accept up to 7 arguments; they are represented by `Arg0` to `Arg6` and cannot be customized.
10. Local variables in ASL can accept up to 8 arguments represented by `Local0`~`Local7`. Definitions are not necessary, but should be initialized, in other words, assignment is needed.

### Common ASL Data Types

|    ASL    |
| :-------: |
| `Integer` |
| `String`  |
|  `Event`  |
| `Buffer`  |
| `Package` |

### ASL Variables Definition

- Define Integer:

  ```asl
  Name (TEST, 0)
  ```

- Define String:
  
  ```asl
  Name (MSTR,"ASL")
  ```

- Define Package:

  ```asl
  Name (_PRW, Package (0x02)
  {
      0x0D,
      0x03
  })
  ```

- Define Buffer Field (6 available types in total):</br>
	
	| Create statement |   size    | Syntax                           |
	| :--------------: | :-------: |----------------------------------|
	| CreateBitField   |  1-Bit    | CreateBitField (AAAA, Zero, CCCC)
	| CreateByteField  |  8-Bit    | CreateByteField (DDDD, 0x01, EEEE)
	| CreateWordField  |  16-Bit   | CreateWordField (FFFF, 0x05, GGGG)
	| CreateDWordField |  32-Bit   | CreateDWordField (HHHH, 0x06, IIII)
	| CreateQWordField |  64-Bit   | CreateQWordField (JJJJ, 0x14, KKKK)
	| CreateField      | any size. | CreateField (LLLL, Local0, 0x38, MMMM)

	It is not necessary to announce its type when defining a variable.

### ASL Assignment

```asl
Store (a,b) /* legacy ASL */
b = a      /*   ASL+  */
```

**Examples**:

```asl
Store (0, Local0)
Local0 = 0

Store (Local0, Local1)
Local1 = Local0
```

### ASL Calculation

|  ASL+  |  Legacy ASL|  Examples        
| :----: | :--------: |:------------------------------------------------------------------------- |
|   +    |    Add     | `Local0 = 1 + 2`<br/>`Add (1, 2, Local0)`                                 |
|   -    |  Subtract  | `Local0 = 2 - 1`<br/>`Subtract (2, 1, Local0)`                            |
|   *    |  Multiply  | `Local0 = 1 * 2`<br/>`Multiply (1, 2, Local0)`                            |
|   /    |   Divide   | `Local0 = 10 / 9`<br/>`Divide (10, 9, Local1(remainder), Local0(result))` |
|   %    |    Mod     | `Local0 = 10 % 9`<br/>`Mod (10, 9, Local0)`                               |
|   <<   | ShiftLeft  | `Local0 = 1 << 20`<br/>`ShiftLeft (1, 20, Local0)`                        |
|   >>   | ShiftRight | `Local0 = 0x10000 >> 4`<br/>`ShiftRight (0x10000, 4, Local0)`             |
|   --   | Decrement  | `Local0--`<br/>`Decrement (Local0)`                                       |
|   ++   | Increment  | `Local0++`<br/>`Increment (Local0)`                                       |
|   &    |    And     | `Local0 = 0x11 & 0x22`<br/>`And (0x11, 0x22, Local0)`                     |
| &#124; |     Or     | `Local0 = 0x01`&#124;`0x02`<br/>`Or (0x01, 0x02, Local0)`                 |
|   ~    |    Not     | `Local0 = ~(0x00)`<br/>`Not (0x00,Local0)`                                |
|        |    Nor     | `Nor (0x11, 0x22, Local0)`                                                |

Read `ACPI Specification` for more details

### ASL Logic

|  ASL+  |   Legacy ASL  | Examples                                                         |
| :----: | :-----------: | :----------------------------------------------------------------|
|   &&   |     LAnd      |  `If (BOL1 && BOL2)`<br/>`If (LAnd(BOL1, BOL2))`                 |
|   !    |     LNot      |  `Local0 = !0`<br/>`Store (LNot(0), Local0)`                     |
| &#124; |      LOr      |  `Local0 = (0`&#124;`1)`<br/>`Store (LOR(0, 1), Local0)`         |
|   <    |     LLess     |  `Local0 = (1 < 2)`<br/>`Store (LLess(1, 2), Local0)`            |
|   <=   |  LLessEqual   |  `Local0 = (1 <= 2)`<br/>`Store (LLessEqual(1, 2), Local0)`      |
|   >    |   LGreater    |  `Local0 = (1 > 2)`<br/>`Store (LGreater(1, 2), Local0)`         |
|   >=   | LGreaterEqual |  `Local0 = (1 >= 2)`<br/>`Store (LGreaterEqual(1, 2), Local0)`   |
|   ==   |    LEqual     |  `Local0 = (Local0 == Local1)`<br/>`If (LEqual(Local0, Local1))` |
|   !=   |   LNotEqual   |  `Local0 = (0 != 1)`<br/>`Store (LNotEqual(0, 1), Local0)`       |

Only two results from logical calculation - `0` or `1`

### Defining Methods in ASL

1. Define a Method:

   ```asl
   Method (TEST)
   {
       ...
   }
   ```

2. Defines a method containing 2 parameters and applies local variables `Local0`~`Local7`

   Numbers of parameters are defaulted to `0`

   ```asl
   Method (MADD, 2)
   {
       Local0 = Arg0
       Local1 = Arg1
       Local0 += Local7
   }
   ```

3. Define a method contains a return value:
  
   ```asl
   Method (MADD, 2)
   {
       Local0 = Arg0
       Local1 = Arg1
       Local0 += Local1
  
       Return (Local0) /* return here */
   }
   ```

   ```asl
   Local0 = 1 + 2            /* ASL+ */
   Store (MADD (1, 2), Local0)  /* Legacy ASL */
   ```

4. Define serialized method:

   If not define `Serialized` or `NotSerialized`, default as `NotSerialized`

   ```asl
   Method (MADD, 2, Serialized)
   {
       Local0 = Arg0
       Local1 = Arg1
       Local0 += Local1
       Return (Local0)
   }
   ```

   It looks like `multi-thread synchronization`. In other words, only one instance can exist in the memory when the method is stated as `Serialized`. Normally the application creates one object, for example:

   ```asl
   Method (TEST, Serialized)
   {
       Name (MSTR,"I will succeed")
   }
   ```

   If we state `TEST` as shown above and call it from two different methods:

   ```asl
   Device (Dev1)
   {
        TEST ()
   }
   Device (Dev2)
   {
        TEST ()
   }
   ```
   If we execute `TEST` in `Dev1`, then `TEST` in `Dev2` will wait until the first one finalized. If we state:

   ```asl
   Method (TEST, NotSerialized)
   {
       Name (MSTR, "I will succeed")
   }
   ```

   when one of `TEST` called from `Devx`, another `TEST` will be failed to create `MSTR`.

## ACPI Preset Functions

### `_OSI` (Operating System Interfaces)

It is easy to acquire the current operating system's name and version when applying the `_OSI` method. For example, we could apply a patch that is specific to Windows or macOS.

`_OSI` requires a string which must be picked from the table below.

|   OS                         |      String     |
| :-------------------------- | :------------- |
| macOS                        | `"Darwin"`      |
| Linux (and Linux-based OS) | `"Linux"`       |
| FreeBSD                      | `"FreeBSD"`     |
| Windows                      | `"Windows 20XX"`|

> Notably, different Windows versions require a unique string, read:  
> <https://docs.microsoft.com/en-us/windows-hardware/drivers/acpi/winacpi-osi>

When `_OSI`'s string matches the current system, it returns `1` since the `If` condition is valid.

```asl
If (_OSI ("Darwin")) /* judge if the current system is macOS */
```

### `_STA` (Status)

**⚠️ CAUTION: Two types of `_STA` exist! Do not confuse it with `_STA` from `PowerResource`!**

5 types of bit can be return from `_STA` method, explanations are listed below:

| Bit     | Explanation                          |
| :-----: | :----------------------------------- |
| Bit [0] | Set if the device is present. |
| Bit [1] | Set if the device is enabled and decoding its resources. |
| Bit [2] | Set if the device should be shown in the UI.|
| Bit [3] | Set if the device is functioning properly (cleared if device failed its diagnostics).|
| Bit [4] | Set if the battery is present.|

We need to transfer these bits from hexadecimal to binary. `0x0F` transferred to `1111`, meaning enable it(the first four bits); while `Zero` means disable. 

We also encounter `0x0B`,`0x1F`. `1011` is a binary form of `0x0B`, meaning the device is enabled and not is not allowed to decode its resources. `0X0B` often utilized in ***`SSDT-PNLF`***. `0x1F` (`11111`)only appears to describe battery device from laptop, the last bit is utilized to inform Control Method Battery Device `PNP0C0A` that the battery is present.

> In terms of `_STA` from `PowerResource`
>
> `_STA` from `PowerResource` only returns `One` or `Zero`. Please read `ACPI Specification` for detail.

### `_CRS` (Current Resource Settings)
`_CRS` returns a `Buffer`, it is often utilized to acquire touchable devices' `GPIO Pin`,`APIC Pin` for controlling the interrupt mode.

## ASL flow Control

ASL also has methods to control flow.

- Switch
  - Case
  - Default
  - BreakPoint
- While
  - Break
  - Continue
- If
  - Else
  - ElseIf
- Stall

### Branch control `If` & `Switch`

#### `If`

   The following codes check if the system is `Darwin`, if yes then`OSYS = 0x2710`

   ```asl
   If (_OSI ("Darwin"))
   {
       OSYS = 0x2710
   }
   ```

#### `ElseIf`, `Else`

   The following codes check if the system is `Darwin`, and if the system is not `Linux`, if yes then `OSYS = 0x07D0`

   ```asl
   If (_OSI ("Darwin"))
   {
       OSYS = 0x2710
   }
   ElseIf (_OSI ("Linux"))
   {
       OSYS = 0x03E8
   }
   Else
   {
       OSYS = 0x07D0
   }
   ```

#### `Switch`, `Case`, `Default`, `BreakPoint`

   ```asl
   Switch (Arg2)
   {
       Case (1) /* Condition 1 */
       {
           If (Arg1 == 1)
           {
               Return (1)
           }
           BreakPoint /* Mismatch condition, exit */
       }
       Case (2) /* Condition 2 */
       {
           ...
           Return (2)
       }
       Default /* if condition is not match，then */
       {
           BreakPoint
       }
   }
   ```

### Loop control

#### `While` & `Stall`

```asl
Local0 = 10
While (Local0 >= 0x00)
{
    Local0--
    Stall (32)
}
```

`Local0` = `10`,if `Local0` ≠ `0` is false, `Local0`-`1`, stall `32μs`, the codes delay `10 * 32 = 320 μs`。

#### `For`

`For` from `ASL` is similar to `C`, `Java`

```asl
for (local0 = 0, local0 < 8, local0++)
{
    ...
}
```

`For` shown above and `While` shown below are equivalent

```asl
Local0 = 0
While (Local0 < 8)
{
    Local0++
}
```

## `External` Quote

|  Quote Types   | External SSDT Quote| Quoted                   |
| :------------: | :----------------- | :----------------------- |
|   UnknownObj   | `External (\_SB.EROR, UnknownObj`             | (avoid to use)                                                          |
|     IntObj     | `External (TEST, IntObj`                      | `Name (TEST, 0)`                                                        |
|     StrObj     | `External (\_PR.MSTR, StrObj`                 | `Name (MSTR,"ASL")`                                                     |
|    BuffObj     | `External (\_SB.PCI0.I2C0.TPD0.SBFB, BuffObj` | `Name (SBFB, ResourceTemplate ()`<br/>`Name (BUF0, Buffer() {"abcde"})` |
|     PkgObj     | `External (_SB.PCI0.RP01._PRW, PkgObj`        | `Name (_PRW, Package (0x02) { 0x0D, 0x03 })`                            |
|  FieldUnitObj  | `External (OSYS, FieldUnitObj`                | `OSYS,   16,`                                                           |
|   DeviceObj    | `External (\_SB.PCI0.I2C1.ETPD, DeviceObj`    | `Device (ETPD)`                                                         |
|    EventObj    | `External (XXXX, EventObj`                    | `Event (XXXX)`                                                          |
|   MethodObj    | `External (\_SB.PCI0.GPI0._STA, MethodObj`    | `Method (_STA, 0, NotSerialized)`                                       |
|    MutexObj    | `External (_SB.PCI0.LPCB.EC0.BATM, MutexObj`  | `Mutex (BATM, 0x07)`                                                    |
|  OpRegionObj   | `External (GNVS, OpRegionObj`                 | `OperationRegion (GNVS, SystemMemory, 0x7A4E7000, 0x0866)`              |
|  PowerResObj   | `External (\_SB.PCI0.XDCI, PowerResObj`       | `PowerResource (USBC, 0, 0)`                                            |
|  ProcessorObj  | `External (\_SB.PR00, ProcessorObj`           | `Processor (PR00, 0x01, 0x00001810, 0x06)`                              |
| ThermalZoneObj | `External (\_TZ.THRM, ThermalZoneObj`         | `ThermalZone (THRM)`                                                    |
|  BuffFieldObj  | `External (\_SB.PCI0._CRS.BBBB, BuffFieldObj` | `CreateField (AAAA, Zero, BBBB)`                                        |

> DDBHandleObj is rare, no discussion


## ASL CondRefOf

`CondRefOf` is useful to check the object is existed or not.

```asl
Method (SSCN, 0, NotSerialized)
{
    If (_OSI ("Darwin"))
    {
        ...
    }
    ElseIf (CondRefOf (\_SB.PCI0.I2C0.XSCN))
    {
        If (USTP)
        {
            Return (\_SB.PCI0.I2C0.XSCN ())
        }
    }

    Return (Zero)
}
```

The codes are quoted from **`SSDT-I2CxConf`**. When system is not MacOS, and `XSCN` exists under `I2C0`, it returns the original value.

## ASL to AML Conversion Table

> `ASL` is an abbreviation for **A**CPI **S**ource **L**anguage, i.e. the ACPI Source Code. `AML` on the other hand is its binary counterpart, the **A**CPI **M**achine **L**anguage – a bytecode language computers can understand.

The following table can be regarded as the quasi dictionary for translating ASL to AML and vice versa. 

Here's an Example: the well-known `_DSM` to `XDSM` binary rename consists of the "Find" value: `5F44534D` and the "Replace" value `5844534D`. This all seems kind of random at first, but in fact it is not. If you take a look in the binary column, you can see that the underscore "`_`" has a value of "5F" (we omit the leading zeros), "D" has "44", "S" is "3S" and "M" corresponds to "4d" – which equals "`_DSM`" in binary. And binary "58" "44" "53" "4D" equals to "`XDSM`" in ASL. And that's how you can read and translate between ASL and AML and create your own renames, if necessary.

|          ASL           | Binary (AML)|
| :--------------------: | :---------: |
|          ZERO          |   `0x00`    |
|          ONE           |   `0x01`    |
| **##################** | **-------** |
|         ALIAS          |   `0x06`    |
|          Name          |   `0x08`    |
| **##################** | **-------** |
|          Byte          |   `0x0a`    |
|          Word          |   `0x0b`    |
|         DWORD          |   `0x0c`    |
|         STRING         |   `0x0d`    |
|         QWORD          |   `0x0e`    |
| **##################** | **-------** |
|         Scope          |   `0x10`    |
|         Buffer         |   `0x11`    |
|        Package         |   `0x12`    |
|      VAR_PACKAGE       |   `0x13`    |
|         Method         |   `0x14`    |
|        Externel        |   `0x15`    |
|       DUAL_NAME        |   `0x2e`    |
|       MULTI_NAME       |   `0x2f`    |
| **##################** | **-------** |
|           A            |   `0x41`    |
|           B            |   `0x42`    |
|           C            |   `0x43`    |
|           D            |   `0x44`    |
|           E            |   `0x45`    |
|           F            |   `0x46`    |
|           G            |   `0x47`    |
|           H            |   `0x48`    |
|           I            |   `0x49`    |
|           J            |   `0x4a`    |
|           K            |   `0x4b`    |
|           L            |   `0x4c`    |
|           M            |   `0x4d`    |
|           N            |   `0x4e`    |
|           O            |   `0x4f`    |
|           P            |   `0x50`    |
|           Q            |   `0x51`    |
|           R            |   `0x52`    |
|           S            |   `0x53`    |
|           T            |   `0x54`    |
|           U            |   `0x55`    |
|           V            |   `0x56`    |
|           W            |   `0x57`    |
|           X            |   `0x58`    |
|           Y            |   `0x59`    |
|           Z            |   `0x5a`    |
|           \            |   `0x5c`    |
|           ^            |   `0x5e`    |
|           _            |   `0x5f`    |
| **##################** | **-------** |
|         Local0         |   `0x60`    |
|         Local1         |   `0x61`    |
|         Local2         |   `0x62`    |
|         Local3         |   `0x63`    |
|         Local4         |   `0x64`    |
|         Local5         |   `0x65`    |
|         Local6         |   `0x66`    |
|         Local7         |   `0x67`    |
| **##################** | **-------** |
|          Arg0          |   `0x68`    |
|          Arg1          |   `0x69`    |
|          Arg2          |   `0x6a`    |
|          Arg3          |   `0x6b`    |
|          Arg4          |   `0x6c`    |
|          Arg5          |   `0x6d`    |
|          Arg6          |   `0x6e`    |
| **##################** | **-------** |
|         Store          |   `0x70`    |
|         Refor          |   `0x71`    |
|          Add           |   `0x72`    |
|         Concat         |   `0x73`    |
|        Suntract        |   `0x74`    |
|       INCREMENT        |   `0x75`    |
|       DECREMENT        |   `0x76`    |
|        MULTIPLY        |   `0x77`    |
|         DIVIDE         |   `0x78`    |
|       SHIFT_LEFT       |   `0x79`    |
|      SHIFT_RIGHT       |   `0x7a`    |
|          AND           |   `0x7b`    |
|          NAND          |   `0x7c`    |
|           OR           |   `0x7d`    |
|          NOR           |   `0x7e`    |
|          XOR           |   `0x7f`    |
|          NOT           |   `0x80`    |
|   FIND_SET_LEFT_BIT    |   `0x81`    |
|   FIND_SET_RIGHT_BIT   |   `0x82`    |
|        DEREF_OF        |   `0x83`    |
|       CONCAT_RES       |   `0x84`    |
|          MOD           |   `0x85`    |
|         NOTIFY         |   `0x86`    |
|        SIZE_OF         |   `0x87`    |
|         INDEX          |   `0x88`    |
|         MATCH          |   `0x89`    |
|   CREATE_DWORD_FIELD   |   `0x8a`    |
|   CREATE_WORD_FIELD    |   `0x8b`    |
|   CREATE_BYTE_FIELD    |   `0x8c`    |
|    CREATE_BIT_FIELD    |   `0x8d`    |
|      OBJECT_TYPE       |   `0x8e`    |
|   CREATE_QWORD_FIELD   |   `0x8f`    |
|          LAND          |   `0x90`    |
|          LOR           |   `0x91`    |
|          LNOT          |   `0x92`    |
|         LEQUAL         |   `0x93`    |
|        LGREATER        |   `0x94`    |
|         LLESS          |   `0x95`    |
|       TO_BUFFER        |   `0x96`    |
|     TO_DEC_STRING      |   `0x97`    |
|     TO_HEX_STRING      |   `0x98`    |
|       TO_INTEGER       |   `0x99`    |
|       TO_STRING        |   `0x9c`    |
|      COPY_OBJECT       |   `0x9d`    |
|          MID           |   `0x9e`    |
|        CONTINUE        |   `0x9f`    |
|           IF           |   `0xa0`    |
|          ELSE          |   `0xa1`    |
|         WHILE          |   `0xa2`    |
|          NOOP          |   `0xa3`    |
|         RETURN         |   `0xa4`    |
|         BREAK          |   `0xa5`    |
|      BREAK_POINT       |   `0xcc`    |
|          ONES          |   `0xff`    |
| **##################** | **-------** |
|    **Ext. Op. EXT**    | **`0x5b`**  |
| **##################** | **-------** |
|         MUTEX          |   `0x01`    |
|         EVENT          |   `0x02`    |
|      COND_REF_OF       |   `0x12`    |
|      CREATE_FIELD      |   `0x13`    |
|       LOAD_TABLE       |   `0x1f`    |
|          LOAD          |   `0x20`    |
|         STALL          |   `0x21`    |
|         SLEEP          |   `0x22`    |
|        ACQUIRE         |   `0x23`    |
|         SIGNAL         |   `0x24`    |
|          WAIT          |   `0x25`    |
|         RESET          |   `0x26`    |
|        RELEASE         |   `0x27`    |
|        FROM_BCD        |   `0x28`    |
|         TO_BCD         |   `0x29`    |
|         UNLOAD         |   `0x2a`    |
|        REVISION        |   `0x30`    |
|         DEBUG          |   `0x31`    |
|         FATAL          |   `0x32`    |
|         TIMER          |   `0x33`    |
|         REGION         |   `0x80`    |
|         FIELD          |   `0x81`    |
|         DEVICE         |   `0x82`    |
|       PROCESSOR        |   `0x83`    |
|       POWER_RES        |   `0x84`    |
|      THERMAL_ZONE      |   `0x85`    |
|      INDEX_FIELD       |   `0x86`    |
|       BANK_FIELD       |   `0x87`    |
|      DATA_REGION       |   `0x88`    |

## SSDT Loading Sequence

Typically, SSDT patches are targeted at the machine's ACPI (either the DSDT or other SSDTs). Since the original ACPI is loaded prior to SSDT patches, there is no need for SSDTs in the `Add` list to be loaded in a specific order. But there are exceptions to this rule. For example, if you have 2 SSDTs (SSDT-X and SSDT-Y), where SSDT-X defines a `device` which SSDT-Y is "cross-referencing" via a `Scope`, then these the two patches have to be loaded in the correct order/sequence for the whole patch to work. Generally speaking, SSDTs being "scoped" into have to be loaded prior to the ones "scoping".

### Examples

- **Patch 1**：***SSDT-XXXX-1.aml***
  
  ```asl
  External (_SB.PCI0.LPCB, DeviceObj)
  Scope (_SB.PCI0.LPCB)
  {
      Device (XXXX)
      {
          Name (_HID, EisaId ("ABC1111"))
      }
  }
  ```
- **Patch 2**：***SSDT-XXXX-2.aml***

  ```asl
  External (_SB.PCI0.LPCB.XXXX, DeviceObj)
  Scope (_SB.PCI0.LPCB.XXXX)
  {
        Method (YYYY, 0, NotSerialized)
       {
           /* do nothing */
       }
    }
  ```
- SSDT Loading Sequence in `config.plist` under `ACPI/Add`: 

  ```XML
  Item 1
            path    <SSDT-XXXX-1.aml>
  Item 2
            path    <SSDT-XXXX-2.aml>
  ```

## CREDITS & RESOURCES
- ACPI [Specifications](https://uefi.org/htmlspecs/ACPI_Spec_6_4_html/)
- ASL Tutorial by acpica.org ([PDF](https://acpica.org/sites/acpica/files/asl_tutorial_v20190625.pdf)). Good starting point if you want to get into fixing your `DSDT` with `SSDT` hotpatches.