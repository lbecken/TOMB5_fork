# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a decompilation project for Tomb Raider: Chronicles (TOMB5). The goal is to reconstruct the original source code from PSX and PC disassembly since the original source is reportedly lost. Code is reconstructed from debugging symbols (.SYM, .MAP), TRosettastone 3.0, and GAMEWAD.OBJ.

## Build Commands

### PSXPC_N (Recommended - PSX code emulated on PC)

**Linux:**
```bash
mkdir BUILD && cd BUILD
cmake ..
make
```

**Windows (Visual Studio):**
1. Create `EXTERNAL/` folder in project root
2. Extract SDL2 to `EXTERNAL/SDL2/`
3. Extract GLEW to `EXTERNAL/glew-2.1.0/`
4. Generate CMake project for VS 2015-2019, **x86 only** (x64 unsupported)

**Dependencies:** CMake 3.0+, SDL 2.0.9+, GLEW 2.1.0+

### Native PSX Build

Requires PSyQ SDK 4.6+ installed to `c:/psyq`:
```bash
PSX_BUILD.bat      # Compile
PSX_LINK.bat       # Link
PSX_CLEAN.bat      # Clean
```

### PC Native (DirectX 7)

Requires DirectX 7 SDK. Configure include/lib paths in Visual Studio project settings.

## Architecture

### Directory Structure

- **GAME/** - Cross-platform game logic (Lara character, enemies, effects, camera, collision, etc.)
- **SPEC_PSX/** - Native PSX platform code (MIPS, PSX SDK calls)
- **SPEC_PSXPC_N/** - PSX code emulated on PC (main development target)
- **SPEC_PC_N/** - PC native version (DirectX 7)
- **EMULATOR/** - PSX emulator library implementing LIBGPU, LIBGTE, LIBCD, LIBSPU, etc.
- **TOOLS/** - WAD pack/unpack utilities, module compiler (DEL2FAB)
- **DISC/** - PSX disc/ISO generation files

### Build Targets

The same source compiles for multiple platforms via preprocessor flags:
- `PSX_VERSION`, `DISC_VERSION`, `BETA_VERSION`, `NTSC_VERSION`
- `DEBUG_VERSION`, `PSXPC_TEST`, `RELOC`, `USE_32_BIT_ADDR`

### PSX Module/Overlay System

PSX version uses 50+ dynamic code overlays (`.MOD` files) loaded at runtime. Functions accessed via `RelocPtr[ModuleIndex][FunctionIndex]`. See MODULE.md and CODEWAD.md for details.

## Code Conventions

### Function Address Markers

All functions include PSX address markers for tracking against original disassembly:
```c
int main()//10064(<), 10064(<) (F) (*) (D) (ND)
```
- Left address: internal beta version
- Right address: final version
- `(<)` or `(>)`: indicates which version the code belongs to
- `**`: produces identical code to original when compiled
- `F`: fully decompiled
- `D`: debugged, functionally same as original
- `ND`: not debugged

### Contribution Rules

**Do not alter the structure or architecture of the code.** This maintains alignment with original MIPS disassembly for comparison. Focus on:
- Resolving bugs
- Finishing existing decompiled methods
- Improving readability without changing structure
- Adding new decompiled methods

### Base Versions for Reference

- PSX NTSC v1.0 SLUS_013.11 MD5: `4EF523E708D7A7D6571F39C6E47784F9`
- PSX Internal Beta MAIN.EXE MD5: `238F15B1C6260F80DB79E31A94508354`

## Game Files Setup

### PSXPC_N with ISO
Set `DISC_VERSION=1` and name files `TOMB5.BIN`/`TOMB5.CUE`.

### PSXPC_N with extracted assets
Use `GAMEWAD_Unpack` tool to extract `GAMEWAD.OBJ`, place level files in `/DATA/` folder next to MAIN binary.
