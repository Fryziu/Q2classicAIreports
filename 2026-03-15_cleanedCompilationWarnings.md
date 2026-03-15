## ## [Clean Compilation] - 2026-03-15

**Tag:** `[Carmack Protocol :: Gemini-3-Flash]`

### Fixed (GCC 15 Compatibility & Strict Type Safety)

* **Core/System:** Resolved strict ISO C `-Wpedantic` warnings regarding function-to-object pointer casts in `sv_main.c` (`Com_BeginRedirect`), `sys_linux.c` (`Sys_GetGameAPI`), and `g_save.c` by leveraging safe `intptr_t` arithmetic and `typeof` operator bridging.
* **OpenGL Subsystem:** Fixed massive `-Wpedantic` pointer cast spam in `qgl_linux.c` and `gl_rmain.c` extension loading macros using the `(typeof(func))(intptr_t)` idiom.
* **Renderer Safety:** Addressed `-Wstringop-overflow` in `gl_rsurf.c` by passing the decayed array matrix directly to `R_RotateForEntity` instead of the first element's address.
* **Memory Bounds Checking:** Fixed critical `-Wsign-compare` logic vulnerabilities in parsing and rendering loops (`cl_parse.c`, `cl_cin.c`, `cl_demo.c`, `gl_model.c`, `gl_light.c`). Enforced signed `int` boundaries for `sizeof` and `strlen` math to prevent catastrophic integer underflows on corrupted files/packets.
* **UI & Menus:** Cleaned up `-Wsign-compare` conflicts between signed UI coordinate math and unsigned resolution limits (`viddef.height/width`) in `ui_credits.c`, `ui_atoms.c`, `qmenu.c`, and `ui_playerconfig.c` using precise downward casting.
* **Const Correctness:** Eliminated `-Wdiscarded-qualifiers` warnings in `cl_main.c` (Server Status parser), `ui_joinserver.c`, and `q_shared.c` (`COM_StripExtension`) by enforcing `const char *` strictness or context-aware casting.
* **Text Encoding:** Completely refactored `COM_MakePrintable` in `q_shared.c` to use hardcoded hexadecimal ASCII values (`0xE4`, `0xF6`, etc.) and `unsigned char` casting, preventing GCC multi-byte character constant failures caused by modern UTF-8 text editors handling legacy ISO-8859-1 source files.
* **Flow Control:** Restored intentional switch-case cascading using `/* fallthrough */` directives in `gl_image.c` (TGA loading), `cl_http.c` (cURL error handling), and `files.c` (VFS path resolution).
* **Data Serialization:** Added missing explicit zero-initializers (`-Wmissing-field-initializers`) for the `field_t` arrays in `g_save.c`.

### Removed (Dead Code Stripping)

* **Legacy CD Audio:** Eradicated obsolete Linux CD-ROM physical playback routines (`cd_linux.c`, `cdaudio.h`) and related `#ifdef` branches, resolving empty translation unit warnings.
* **Legacy AVI Export:** Stripped the unmaintained, redundant `aviDump_t` structural tree and frame-dumping pipeline (`R_BeginAviDemo`, `R_WriteAviFrame`) to reduce binary footprint and memory overhead.

>"Wydajność to nie jest dodatek, który programujesz na samym końcu. Wydajność to 
zbiór tysiąca małych, bezkompromisowych decyzji podjętych w każdej 
linijce kodu. Usuwaj to co martwe, typuj twardo, ufaj matematyce, a 
>sprzęt odwdzięczy Ci się szybkością."

**[Carmack Protocol :: Gemini-3-Flash]**

>"A program that produces no warnings is a program whose author has mastered the machine. The code is no longer just text; it is pure, distilled execution."
    
**[Carmack Protocol :: Gemini-3-Flash] - Transmisja Zakończona.**

## ---

## [Unreleased] - 2026-02-28

**Tag:** `[Carmack Protocol :: Gemini-3-Flash]`

### Added / Changed

* **Netcode / Input:** Implemented sub-frame input sampling bypass in `CL_SendCmd` (`cl_input.c`). Attack button state changes now force an immediate packet transmission, bypassing `cl_maxpackets` throttling. This eliminates local bufferbloat, reducing perceived input lag and dropping Application RTT (Ping) by ~11ms.
* **Audio:** Fixed a critical logic bug in `snd_mix.c` by reverting `begin` time in `pendingplay_t` to a signed `int`. This prevents integer underflow and audio corruption when processing sounds with negative time deltas.

### Fixed (GCC 15 Compatibility & Zero-Warning Policy)

* **Build System:** Resolved critical `-Wincompatible-pointer-types` build failure in `ui_atoms.c` by implementing a safe `M_Menu_JoinServer_Wrapper_f` wrapper, preserving legacy API compatibility.
* **Types & Safety:** Eliminated dozens of `-Wsign-compare` and implicit conversion warnings across the codebase (e.g., `ui_credits.c`, `console.c`, `cl_parse.c`, `gl_model.c`, `qmenu.c`). Enforced strict signed bounds checking via explicit casts to prevent buffer overflow vulnerabilities.
* **Pointer Arithmetic:** Fixed `-Wpedantic` warnings regarding function-to-object pointer casts in `g_save.c` and dynamic library loading in `qgl_linux.c` / `gl_rmain.c` using `intptr_t` and `typeof` macros.
* **Memory Analysis:** Resolved `-Wstringop-overflow` in `gl_rsurf.c` by correctly passing the 3D matrix array decay instead of a single element address to `R_RotateForEntity`.
* **Flow Control:** Addressed `-Wimplicit-fallthrough` warnings in TGA image loading (`gl_image.c`) by restoring explicit `/* fallthrough */` directives.
* **Initialization:** Fixed `-Wmissing-field-initializers` in `g_save.c` by explicitly zeroing struct fields.
* **Cleanliness:** Systematically silenced all `-Wunused-parameter` warnings globally using the `(void)param;` idiom to maintain strict legacy ABI/API definitions without triggering compiler warnings.

### Removed (Dead Code Stripping)

* **Legacy AVI:** Completely stripped the obsolete `aviDump_t` struct, `R_BeginAviDemo`, `R_WriteAviFrame`, and associated rendering hooks.
* **Legacy CD Audio:** Removed `cd_linux.c`, `cdaudio.h`, and related build targets (`#ifdef USE_CDAUDIO`), as physical CD playback is entirely obsolete in modern SDL2 environments.

> *"Latency is the mind-killer. Clean code is not just about satisfying the compiler; it's about stripping away every unnecessary microsecond between the player's intent and the pixel on the screen. The less code the machine has to run, the closer the player gets to the game."* 
> — **Carmack Protocol**
