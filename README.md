# Win11Themes
This is a repository for Windows 11 Themes. I am currently working on an aero theme. Most aero themes I have come across have two main issues:
* They don't work with Microsoft Office nicely.
* The menus look like crap.

So, I am trying to make a good-looking Windows 7 theme that works well. Here is the latest release.

# Notes
There are two main versions right now. One has blue, aero-like borders and the other doesn't. However, for the border version to work correctly to work you need to use Windhawk. I made a fork of "Bring Back the Borders!" by teknixstuff for this, and it should (hopefully) work. Copy and paste this into a new windhawk mode and compile then activate it. Here is the source code:

```
// ==WindhawkMod==
// @id              w11-dwm-fix-fork
// @name            Bring Back the Borders! - Fork
// @description     Restores borders and corners
// @version         1.1.0
// @author          teknixstuff
// @github          https://github.com/teknixstuff
// @twitter         https://twitter.com/teknixstuff
// @homepage        https://teknixstuff.github.io/
// @include         dwm.exe
// @architecture    x86-64
// ==/WindhawkMod==

// Source code is published under The MIT License.
// Email teknixstuff@gmail.com for any help or info

// ==WindhawkModReadme==
/*...*/
// ==/WindhawkModReadme==

#include <windhawk_utils.h>
#include <windows.h>

bool HighContrastNow = true;



bool (*WINAPI IsHighContrastMode_Original)();
bool WINAPI IsHighContrastMode_Hook() {
    return HighContrastNow;
}

bool (*WINAPI SystemParametersInfoW_Original)(UINT uiAction, UINT uiParam, PVOID pvParam, UINT fWinIni);
bool WINAPI SystemParametersInfoW_Hook(UINT uiAction, UINT uiParam, PVOID pvParam, UINT fWinIni) {
    bool retval = SystemParametersInfoW_Original(uiAction, uiParam, pvParam, fWinIni);
    if (uiAction == SPI_GETHIGHCONTRAST) {
        HIGHCONTRASTW* contrast;
        contrast = (HIGHCONTRASTW*)pvParam;
        contrast->dwFlags |= HCF_HIGHCONTRASTON;
    }
    return retval;
}





BOOL Wh_ModInit() {
    Wh_Log(L">");

    HMODULE udwm = GetModuleHandle(L"udwm.dll");
    if (!udwm) {
        Wh_Log(L"udwm.dll isn't loaded");
        return FALSE;
    }
    HMODULE user32 = GetModuleHandle(L"user32.dll");
    if (!user32) {
        Wh_Log(L"user32.dll isn't loaded");
        return FALSE;
    }

    WindhawkUtils::SYMBOL_HOOK symbolHooksUser32[] = {
        {
            {LR"(int __cdecl RealSystemParametersInfoW(unsigned int,unsigned int,void *,unsigned int))"},
            (void**)&SystemParametersInfoW_Original,
            (void*)SystemParametersInfoW_Hook,
        },
    };

    HookSymbols(user32, symbolHooksUser32, ARRAYSIZE(symbolHooksUser32));

    WindhawkUtils::SYMBOL_HOOK symbolHooks[] = {

        {
            {LR"(public: static bool __cdecl CDesktopManager::IsHighContrastMode(void))"},
            (void**)&IsHighContrastMode_Original,
            (void*)IsHighContrastMode_Hook,
        },
    };
    return HookSymbols(udwm, symbolHooks, ARRAYSIZE(symbolHooks));
}

void Wh_ModUninit() {
    Wh_Log(L">");
}
```
