> ## Fork: rebuild for Skyrim 1.7.104
>
> This is a **compatibility fork** of [patchulidev/ModExplorerMenu](https://github.com/patchulidev/ModExplorerMenu).
> No feature was added, removed or changed — only what was needed to build and run on the
> current Skyrim AE runtime.
>
> Upstream Modex 3.0.1 was built against a CommonLibSSE-NG revision older than Skyrim
> **1.7.104**. On that runtime the plugin cannot find its address library and refuses to load:
>
> ```
> REL/ID.h(223): failed to open address library file
> ```
>
> This fork rebuilds it against **CommonLibSSE-NG 7.5.2** (`c7662fc`), with the porting fixes
> that version requires.
>
> ### Porting fixes
>
> | # | File | Fix |
> |---|------|-----|
> | 1 | `src/core/PrettyLog.h` | `ASSERT_MSG` left a trailing comma when expanded with no variadic argument. `__VA_OPT__(,)` removes it. |
> | 2 | `src/data/BaseObject.h` | Slot masks are `REX::EnumSet` in NG 7.x — `static_cast<int>` replaced by `.underlying()`. |
> | 3 | `src/core/Graphic.h` | NG 7.x no longer pulls in `<d3d11.h>` transitively; included explicitly. |
> | 4 | `src/ui/core/UIMenuImpl.cpp` | `BSInputDeviceManager::Reset`/`Process` renamed to `ClearInputState`/`Poll`. |
> | 5 | `src/ui/core/UIMenuImpl.cpp`, `src/ui/modules/settings/SettingsModule.cpp` | `ToggleControls` takes an extra argument — updated at all 13 call sites. |
> | 6 | `src/core/Commands.h` | `BookMenu::OpenBookMenu` is now private; replaced with the public `OpenMenuFromBaseForm`. |
> | 7 | `src/core/Hooks.cpp` | **Runtime fix.** A single trampoline allocation sized for all four hooks. |
>
> Fix 7 is the one that kept the plugin from booting even once it compiled. `SKSE::AllocTrampoline`
> **sets** `info.trampolineSize` instead of accumulating it, and `API::InitTrampoline` is guarded by
> a `std::call_once` — so only the *first* call ever creates the buffer. The four sequential calls
> (14, 8, 14, 14) therefore reserved 14 bytes in total. The first `write_call<5>` consumed all of
> them and the next hook died in `do_allocate`:
>
> ```
> SKSE/Trampoline.cpp(164): Failed to handle allocation request
> ```
>
> ### Build
>
> Same as upstream, except `--skyrim_vr=y` — the prebuilt NG package this fork resolves is built
> with VR support on, and the flags must match:
>
> ```bat
> xmake config -m releasedbg --skyrim_vr=y
> xmake build
> ```
>
> ### Licence
>
> Modex is © Patchuli, released under the **GNU General Public License v3.0**. This fork keeps that
> licence; see [LICENSE.txt](LICENSE.txt) for the complete GPL-3.0 text.
> The upstream MIT copyright and permission notice is retained in [NOTICE.txt](NOTICE.txt),
> not as an alternative license for the combined distribution. Dependency licenses remain
> applicable to their respective components. Source code for this fork is available here.
>
> Original mod: [Nexus 137877](https://www.nexusmods.com/skyrimspecialedition/mods/137877) ·
> [patchulidev/ModExplorerMenu](https://github.com/patchulidev/ModExplorerMenu)

---

![](https://capsule-render.vercel.app/api?type=waving&height=300&color=gradient&text=Modex&desc=A%20Mod%20Explorer%20Menu&descSize=20&section=header)

![GitHub last commit](https://img.shields.io/github/last-commit/patchulidev/modexplorermenu?style=for-the-badge) ![GitHub License](https://img.shields.io/github/license/patchulidev/modexplorermenu?style=for-the-badge) ![GitHub Issues or Pull Requests](https://img.shields.io/github/issues/patchulidev/modexplorermenu?style=for-the-badge) ![GitHub Release](https://img.shields.io/github/v/release/patchulidev/modexplorermenu?include_prereleases&display_name=release&style=for-the-badge) ![Static Badge](https://img.shields.io/badge/nexus-page-gray?style=for-the-badge&labelColor=orange&link=https%3A%2F%2Fwww.nexusmods.com%2Fskyrimspecialedition%2Fmods%2F137877)

This is a CommonlibSSE-NG Plugin for Skyrim SE/AE Game versions 1.5.97 - 1.6.1170. This project utilizes xmake, following the outline of the template [commonlibsse-ng-template](https://github.com/libxse/commonlibsse-ng-template/tree/main). Build instructions can be found below.

![Static Badge](https://img.shields.io/badge/Skyrim-1.5.97+-gray?style=for-the-badge&labelColor=blue) ![Static Badge](https://img.shields.io/badge/Skyrim-1.6.1170-gray?style=for-the-badge&labelColor=blue)


### Requirements
* [XMake](https://xmake.io) [2.8.2+]
* C++23 Compiler (MSVC, Clang-CL)

### Dependencies (Managed)
* [imgui](https://github.com/ocornut/imgui) [v1.91.5]
* [freetype](https://github.com/freetype/freetype) [Latest]
* [fmt](https://github.com/fmtlib/fmt) [Latest]
* [nlohmann-json](https://github.com/nlohmann/json) [v.3.12.0]
* [simpleini](https://github.com/brofield/simpleini) [Latest]
* [commonlibsse-ng](https://github.com/alandtse/CommonLibVR/) [Latest]

### Information

This project is natively maintained and built on Windows 11 using Neovim. Mileage may vary.

This project is setup for a *local* install of Commonlib in the project folder. Will require reconfiguration if you have a global instance of it.

> ***Note:*** *You may have include path issues with my xmake configuration - sorry.*

## Getting Started
```bat
git clone --recurse-submodules https://github.com/patchulidev/ModExplorerMenu
cd ModExplorerMenu
xmake config -m releasedbg --skyrim_vr=n
```

### Build
To build the project, run the following command:
```bat
xmake build
```

> ***Note:*** *This will generate a `/build/` directory in the **project's root directory** with the build output.*
> ***Note:*** *Project packages are installed locally in the .xmake directory in your workspace folder. This can be turned off*

### Build Output (Optional)
The project configuration is designed to copy the contents of `/dist/` into your mod manager `/data/`
directory after building the plugin. This requires the below environment variable set. Otherwise,
the binaries will only be distributed to `/dist/`.

If you want to redirect the build output, set one of or both of the following environment variables:

- Path to a Mod Manager mods folder: `MO2_MODS_FOLDER`

### Project Generation (Optional) (Untested)
If you want to generate a Visual Studio project, run the following command:
```bat
xmake project -k vsxmake
```

> ***Note:*** *This will generate a `vsxmakeXXXX/` directory in the **project's root directory** using the latest version of Visual Studio installed on the system.*

### Upgrading Packages (Optional)
If you want to upgrade/modify the project's dependencies, run the following commands:
```bat
xmake repo --update
xmake require --upgrade
```
Alternatively, if you want to clean and re-install project dependencies, run the following commands:
```bat
xrepo remove --all
xrepo clean
xmake f -c
```
Doing so will redownload project dependencies from source. Follow "Getting Started" afterwards.

### Clean and Reconfigure (Optional)
Similarly to CMake, you may need to clean and reconfigure your installation.
```bat
xmake f -c
xmake build
```

## Documentation
Please refer to the [Wiki](https://github.com/libxse/commonlibsse-ng-template/wiki) for more advanced topics and template guidance.
