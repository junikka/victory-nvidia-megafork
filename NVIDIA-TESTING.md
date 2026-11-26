# NVIDIA testing guide

## Base and benchmarks

Start from [trustno1-coder `6a31e431`](https://github.com/trustno1-coder/VIctory-NVIDIA/commit/6a31e431). It has an NVIDIA-specific launch profile, a Series S interface compositor, and settings for texture synchronization and descriptor capacity. The upstream README reports about 20–22 FPS indoors on an RTX 5070 in Series S at half render scale. A [separate RTX 5080 report](https://github.com/fxceless/victory-fork/issues/1379) reaches about 25 FPS with a stack of additional patches. A [separate RTX 4080 test](https://github.com/fxceless/victory-fork/issues/1268) reports 27.2 FPS median with another mix of patches. Those numbers describe the listed configurations and scenes.

Cool Fox `30e54b4` is a valuable reference for texture synchronization and frame time. Its package reports about 30 FPS median on an RTX 4090. Use that as a comparison target for individual fixes. [Geckomacabre](https://github.com/Geckomacabre/vx-layer) is another candidate source for sampler handling and reducing GPU waits. [Kelvaroq](https://github.com/stellarr-studio/kelvaroq) is worth tracking for memory management, with its public benchmark kept separate from development previews.

## First run

Use the pinned Rust toolchain and the MSVC C++ build tools with a Windows SDK. On Windows 10, follow the [Agility SDK setup](README.md#windows-10-setup) before building. Build a diagnostics executable and start with the upstream Quality preset at 720p internal resolution. The runtime uses your own game files.

```powershell
cargo build --locked --profile diag -p victory --features diagnostics
.\scripts\play-nvidia.ps1 -Exe 'D:\Games\Example\game.exe' -Quality -Resolution 720 -NoSmoothMotion
```

Compare against the same command without `-Quality` after establishing the baseline. Record the commit, GPU, driver, CPU, RAM, cache state, internal and output resolution, and render settings. Capture the opening close-up and a fixed city route on RTX 5070, 4080/4090 and 5080-class hardware where available. Compare median and p95 frame times, missing faces or geometry, and device losses. Keep generated frames separate from rendered FPS.

## Sharing code with the AMD megafork

The NVIDIA line and the [AMD line](https://github.com/rorodev6/victory-megafork-amd) have a common ancestor (`0a435665`), but their renderers have diverged. Port focused patches and run the appropriate tests on each base. Maintain separate launch defaults for shader wave execution, descriptor budgets and upscaling. Compare related fixes on both GPUs before treating them as cross-vendor defaults.

The first shared fix is [Filip722's pipeline job completion change from fxceless](https://github.com/fxceless/victory-fork/pull/1353). It completes a failed job after a Rust panic and wakes waiters. This is a defensive recovery for Rust errors. Driver resets still need separate diagnosis. The Git history retains upstream authorship and the [README credits](README.md#credits) retain the project's upstream acknowledgements.
