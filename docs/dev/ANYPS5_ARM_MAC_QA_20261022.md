# AnyPS5-ARM Apple Silicon QA — 2026-10-22

Test platform: Apple M4 Pro Mac mini, macOS 26.3, Rosetta 2, Vulkan/MoltenVK.
Development branch: `feature/anyps5-arm-upstream-sync-20261022`.
This branch uses the current macOS port plus the preserved macOS entry-signing fix.

## Relinker improvements
- Preserve guest executable entry code during macOS signing (existing fork fix).
- Normalize PS5 system-library `.sprx` imports to the available host `.prx` names for Mach-O imports. Keep `$ORIGIN` guest-module paths unchanged.
- Extend macOS Mach-O dependency fixture coverage for `libkernel.sprx`, `libSceVideoOut.sprx` and `libScePad.sprx`.
- Do not register `macos_fixtures` for `ANYPS5_RELINKER_ONLY=ON`: those runtime cases need PRX libraries not built in that configuration.

## Toolchain and automated results
- AppleClang 21.0.0, CMake 4.4.4, Ninja 1.13.2.
- Configure with `-DANYPS5_RELINKER_ONLY=ON -DBUILD_TESTING=ON`.
- Build: 239 Ninja build steps succeeded initially; patch rebuild succeeded.
- CTest: **232/232 passed** after applying the registration and import fixes.
- Relinker-only builds do **not** validate whole-game gameplay without host PRX runtimes.

## Local homebrew test inputs
All release binaries came from the title developers' GitHub Releases; source archives and converted apps remain outside this repository.
- Celeste Classic PS5 — `darkxex/PS5_Celeste`, release `1.00`, `PPSA59322.zip`, SHA-256 `96861b81c1fbaac818080d1bf958b575e6b44d2c258684e1931f2940cee2fb43`.
- Raptor: Call of the Shadows PS5 — `ChaDea1/RaptorPS5`, release `v1.0.0-rc.2`, SHA-256 `b6620582b5561f36fe1cfbb8f2866f88c9670c850beef1bc9270b1eb3b0106f6`.
- SharpEmu Breakout demo — `sharpemu/sharpemu-demo`, release `49e6b3b`, SHA-256 `2413f587b6f310c7b0f5ae3f76019de53d5a419141e7392b48096648eb74def4`.

## Results by title
| Title | ELF / Mach-O conversion | Runtime observation | Status |
| --- | --- | --- | --- |
| Celeste Classic PS5 | Decrypted SELF unwrapped, modules relinked | Game startup logged; Vulkan device creation reported unavailable | Graphics blocker, not playable |
| Raptor PS5 | Decrypted SELF unwrapped, modules relinked | First-launch menu shown at ~46 FPS, M4 Pro Vulkan swapchain and DualSense detected; built-in free-shareware download fails HTTP 0 / 0x80431063 | Menu working; gameplay awaiting legally sourced data |
| SharpEmu Breakout | Direct PS5 ELF relinked; host `.sprx` issue fixed | Breakout graphics shown at ~59.9 FPS; keyboard Enter started a live game, score reached 10; DualSense detected | Gameplay verified with keyboard; physical controller inputs unverified |

The prior separate DOOM homebrew test (`v0.2.1-arm + pp1`) reported 600 seconds of playable rendering at ~35 FPS; it was **not** revalidated under this new branch.

## Remaining work
1. Diagnose Celeste's direct Vulkan device selection / RADV assumptions against MoltenVK.
2. Provide or obtain authorized Raptor shareware game data, validate gameplay and save persistence.
3. Validate physical DualSense button mappings and audio playback across all test apps.
4. Verify full x86-64 runtime build with host PRX libraries separately from relinker-only CTest.
5. Integrate verified compatibility into PlayPulsive behind opt-in flags; do not replace the working launcher.

No commercial PS5 game dumps, ROMs, firmware, keys or unpublished assets are included.
