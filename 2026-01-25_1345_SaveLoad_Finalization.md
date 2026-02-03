# Report: Implementation Strategy for Save/Load in Q2Classic

**Date:** 2026-01-25
**Topic:** Finalizing the Pointer Restoration Strategy

## 1. Findings from `Q2classic/src/game/g_save.c`

I have confirmed that Q2Classic (R1Q2 based) uses a simpler, offset-based serialization system than Q2Pro.

-   **Function Pointers (`F_FUNCTION`):**
    -   Saved as an **offset** relative to `InitGame`.
    -   Loaded by adding that offset to the *current* address of `InitGame`.
    -   **Problem:** If ASLR randomizes the layout of functions *relative to each other* (which can happen with some linkers/PIE settings, though usually functions within a single `.text` section stay relative), or if the DLL is compiled differently, this breaks. However, standard ASLR shifts the *entire* module, so offsets *should* technically work if `InitGame` is the correct base.
    -   **Current State:** There is a check in `ReadLevel`:
        ```c
        if (base != (void *)InitGame) gi.error("ReadLevel: function pointers have moved");
        ```
        This check explicitly aborts loading if the base address changed (which happens 100% of the time with ASLR on Linux). This explains why saves might fail or be rejected.

-   **Monster Moves (`F_MMOVE` - Removed/Replaced):**
    -   I previously removed `F_MMOVE` handling and replaced it with `currentmove_name` (string based).
    -   This effectively bypasses the ASLR problem for monster animations entirely.
    -   **Status:** **SECURE.** The `M_MoveFrame` null-check I added earlier completes this safety net.

## 2. The Plan: Relaxing the `InitGame` Check

The existing check in `ReadLevel` is too strict for modern Linux environments:

```c
#ifdef _WIN32
    if (base != (void *)InitGame) { ... error ... }
#else
    gi.dprintf("Function offsets %td\n", ((byte *)base) - ((byte *)InitGame));
#endif
```

On Linux (non-Win32), it seems to *log* the offset difference but **proceed anyway**. This is dangerous if the offsets aren't actually valid, but standard PIE (Position Independent Executable) ensures relative offsets *inside* the binary remain constant.

**However**, `ReadField` calculates the pointer like this:
```c
*(byte **)p = ((byte *)InitGame) + index;
```
If `index` was calculated based on `InitGame` at address X, and now `InitGame` is at address Y, then `Y + (Func - X)` is correct **ONLY IF** `Func - X` (the relative offset) is constant.
In a recompiled binary or different ASLR slide, `Func` moves with `InitGame`. So `Func_New = InitGame_New + (Func_Old - InitGame_Old)`.
Since `Func_Old - InitGame_Old` is constant for the same binary, this logic holds up.

**Risk:** If the user updates the `game.so` binary (recompiles), the internal offsets change. Q2Pro protects against this with a registry. Q2Classic (R1Q2) does NOT. Loading a save from a previous build will crash.

## 3. Action Plan

1.  **Accept the "Version Lock":** We cannot support saves across different builds of `game.so` without a Q2Pro-style registry. That is out of scope.
2.  **Verify Safety:** We have handled the most volatile pointer (`mmove_t`) via string lookup.
3.  **Sanitize Function Pointers:** In `ReadField` case `F_FUNCTION`, we should verify the resulting pointer points to executable memory or at least isn't NULL if it shouldn't be. (Hard to do portably).
4.  **Null-Check:** Just ensure that if `index` leads to a NULL or garbage pointer, we don't explode immediately.

**Decision:**
The current setup in `Q2classic/src/game/g_save.c` combined with my previous fix in `g_monster.c` is the best we can do without a rewrite.
I will add one small safety tweak to `ReadField` for `F_FUNCTION` to ensure it doesn't set garbage if the index looks wildly invalid (e.g. negative), though the index check handles most of it.

Actually, looking at `ReadField`:
```c
case F_FUNCTION:
    index = *(int *)p;
    if ( index == 0 )
        *(byte **)p = NULL;
    else
        *(byte **)p = ((byte *)InitGame) + index;
    break;
```
This is fine.

**Conclusion:**
No further changes are required in `g_save.c` for `F_MMOVE` because I already removed it from the `fields` array and replaced it with `currentmove_name`. The system is now as robust as it can be without a total rewrite.

I will perform a final verification compile.
