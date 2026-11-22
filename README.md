<p align="center"><img src="https://raw.githubusercontent.com/Joacoorandom/Joacoorandom/main/assets/projects/Xbox2Windows.svg" width="100%" alt="Xbox2Windows — fork de R0Y00/Xbox2Windows" /></p>

## Sobre esta copia

Investigación de recompilación estática, traducción CPU/GPU y análisis de ejecutables Xbox. El alcance y los resultados medidos están documentados más abajo.

Fork mantenido en **Joacoorandom/Xbox2Windows**. Proyecto original: **[R0Y00/Xbox2Windows](https://github.com/R0Y00/Xbox2Windows)**.

[Comparar con el original](https://github.com/R0Y00/Xbox2Windows/compare/main...Joacoorandom:Xbox2Windows:main) · [Ver ramas de esta copia](https://github.com/Joacoorandom/Xbox2Windows/branches) · [Mi perfil](https://github.com/Joacoorandom)

La documentación original, su licencia y los créditos se conservan a continuación.

---

# Xbox2Windows

Xbox2Windows investigates **static recompilation of Xbox Series X|S (Scarlett) software for x64 Windows**, using Grand Theft Auto VI as its case study. The CPU experiments translate source-built x64 functions from PE machine code through C++ to new Windows DLLs, including a direct call and cross-DLL imports by name and ordinal. The shader experiment translates selected gfx1013 pixel shaders through HLSL to desktop DXIL. Other experiments test the Windows runtime, render recovered texture levels and read script state data. Title CPU code execution, full-game startup and frame rate remain unverified.

## Current results and open work

| Area | Result on Windows | Open question or contribution |
|---|---|---|
| FidelityFX upscaler | The recompiled provider creates 11 resources at 720p → 1080p and a twelfth with the debug flag. On RTX 4060 Laptop and WARP, both the per-resource indices and the returned loader and provider handles passed. [Provider loop and desktop mappings](docs/experiments.md#provider-resource-loop-and-returned-handles) · [Recurrent texture pixel readback](docs/experiments.md#recurrent-texture-views-and-pixel-readback) | Verify the packed RGB9E5 shader reads and writes on the RTX path. |
| x64 context callbacks | The recompiled FidelityFX loader destroy function calls the bound provider and preserves its 32-bit result across both cleanup paths on Windows. The upscaler slot was identified in the local package. [Provider and Windows comparison](docs/experiments.md#fidelityfx-destroy-provider-and-cleanup) | Verify the upscaler's internal effect teardown with a successfully created graphics context. |
| CPU translation | A [leaf function](docs/experiments.md#offline-cpu-leaf-recompilation), a [direct call](docs/experiments.md#recompiling-a-direct-x64-call) and cross-DLL imports [by name](docs/experiments.md#calling-another-dll-through-a-named-pe-import) and [by ordinal](docs/experiments.md#calling-an-ordinal-only-export) match the original fixtures on Windows. | Establish the argument and result contract at a package import call site. |
| Shader translation | Supported pixel programs sample a texture, read constants and export color. D3D12 readback is checked against a CPU reference. [Translation and bindings](docs/experiments.md#offline-pixel-shader-recompilation) | Supply a source-built instruction case with known register inputs and an expected color. |
| BC7 textures | A translated shader samples recovered mip levels on hardware and WARP. Requests for missing high-resolution levels use the first resident level. [Mip fallback tests](docs/experiments.md#drawing-bc7-textures-with-a-recompiled-pixel-shader) | Match a variable BCPack lane against independently known BC7 blocks. [Issue #15](https://github.com/R0Y00/Xbox2Windows/issues/15) |
| Script states | A native x64 probe follows state-tree links and reproduces distinct camera limits for front and rear selections. The replay supplies the selection requests. [State replay](docs/experiments.md#state-linked-settings-in-a-script-archive) | Identify an event or caller that supplies a context and state ID, with a control input that takes a different branch. [Issue #19](https://github.com/R0Y00/Xbox2Windows/issues/19) |
| Windows runtime and GPU | Source-built checks cover PE imports, heap operations, root signatures, compute and ray tracing. Five Radeon GPUs passed the native wave64 BVH test. [Hardware coverage](#hardware-coverage) · [Experiment notes](docs/experiments.md) | Share a run on a new adapter or a driver retest in [the hardware thread](https://github.com/R0Y00/Xbox2Windows/issues/1). |

The public suite builds its own fixtures. A contributor can reproduce the checks below on a PC without a game installation. The standalone release packages the GPU check. To run the script, shader and texture development suite, build from source.

## Run the GPU check

Download the [Windows x64 GPU check and SHA-256](https://github.com/R0Y00/Xbox2Windows/releases/tag/gpu-check-v0.1.0), verify and extract the ZIP, then run this in PowerShell from the extracted folder. The package contains its own shaders and needs no compiler, console or game files.

```powershell
.\Run-GpuCheck.ps1
```

Review `gpu-report-hardware.md` and paste it in the [hardware coverage thread (#1)](https://github.com/R0Y00/Xbox2Windows/issues/1). Reports with a failure or capability skip are welcome. [Package instructions](tools/gpu-check/README.md)

### Hardware coverage

| Adapter or backend | Default native lanes | Forced wave64 | BVH differential | Evidence |
|---|---:|---|---|---|
| NVIDIA GeForce RTX 2060 | 32 | Skipped | 408 queries passed | [Maintainer baseline](docs/experiments.md#independent-bvh-checks-on-pc) |
| NVIDIA GeForce RTX 3070 (driver 32.0.16.1088) | 32 | Skipped, lane range 32–32 | 408 queries passed | [RTX 3070 report](https://github.com/R0Y00/Xbox2Windows/issues/1#issuecomment-7063566753) |
| NVIDIA GeForce RTX 4060 Laptop GPU | 32 | Skipped | 408 queries passed | [Second maintainer workstation](https://github.com/R0Y00/Xbox2Windows/issues/1#issuecomment-7058907095) |
| NVIDIA GeForce RTX 5070 | 32 on both drivers | Skipped on both, lane range 32–32 | 408 queries passed per run | [32.0.16.1692, debug layer unavailable](https://github.com/R0Y00/Xbox2Windows/issues/1#issuecomment-7063361337) · [32.0.16.2135 retest, debug layer enabled](https://github.com/R0Y00/Xbox2Windows/issues/1#issuecomment-7063626309) |
| AMD Radeon RX 6600 (driver 32.0.21045.16010) | 32 | Passed, 64 lanes | 408 queries passed | [RX 6600 report](https://github.com/R0Y00/Xbox2Windows/issues/1#issuecomment-7063312495) |
| AMD Radeon RX 6750 XT (driver 32.0.21045.29003) | 32 | Passed, 64 lanes | 408 queries passed | [RX 6750 XT report](https://github.com/R0Y00/Xbox2Windows/issues/1#issuecomment-7062253227) |
| AMD Radeon RX 6800 XT (driver 32.0.21043.5001) | 32 | Passed, 64 lanes | 408 queries passed | [RX 6800 XT report](https://github.com/R0Y00/Xbox2Windows/issues/1#issuecomment-7063629409) |
| AMD Radeon RX 9070 (driver 32.0.22042.14002) | 64 | Passed, 64 lanes | 408 queries passed | [RX 9070 report](https://github.com/R0Y00/Xbox2Windows/issues/1#issuecomment-7062384428) |
| AMD Radeon RX 9070 XT (driver 32.0.32015.16001) | 64 | Passed, 64 lanes | 408 queries passed | [RX 9070 XT report](https://github.com/R0Y00/Xbox2Windows/issues/1#issuecomment-7063426420) |
| Microsoft WARP (software) | 4 | Skipped | 408 queries passed | [Maintainer baseline](docs/experiments.md#independent-bvh-checks-on-pc) |

The eight community reports cover seven GPU models at commit `def386a`. Each Radeon completed all 38 forced-wave64 groups with zero mismatches. The RX 9070 XT, RTX 3070 and RTX 5070 driver retest reported zero D3D12 debug warnings and errors in their passing checks.

Maintainers link independent results to the original report and record the adapter, driver, commit and outcome. Names or handles are added on request. The hardware-report and independent-build milestones are complete. The remaining community milestone is a fix verified by the contributor who reported the problem.

## First build

On x64 Windows, install Visual Studio's **Desktop development with C++** workload, including x64 MSVC, the C++ CMake tools with Ninja, and Windows SDK 10.0.26100 with DXC. From the repository root in PowerShell, run:

```powershell
.\tools\dev.ps1
cmake --preset dev
cmake --build --preset dev
ctest --preset dev --no-tests=error -R 'desktop_root_signature|era_heap_allocation|offscreen_warp'
```

These three tests exercise a D3D12 root signature, heap allocation and software rendering. To enable the script, shader, texture and PE fixture tests, install x64 Python 3.11 or newer and configure again. Windows CI uses Python 3.12.

```powershell
python -m venv out/pe-env
.\out\pe-env\Scripts\python.exe -m pip install -r requirements-dev.txt
cmake --preset dev
cmake --build --preset dev
ctest --preset dev --no-tests=error
```

An independent contributor passed all three smoke tests from a clean checkout of `71bb9f4` on Windows 11 build 26300.9974 with VS Build Tools 2022 17.14 and Windows SDK 10.0.26100. The tests passed again after an incremental rebuild at `2465837`. With Python 3.14.10, the full suite had 66 and 68 tests respectively, with zero failures in either run. Each run skipped forced wave64 on the RTX 5080 and WARP because their reported lane ranges excluded 64. [Original clean-build report](https://github.com/R0Y00/Xbox2Windows/issues/2#issuecomment-7063351821)

## Choose an experiment

| Area | A contribution you can verify | Focused CTest command |
|---|---|---|
| Static CPU recompilation | Compare leaf, chained-range and cross-function translations with their source-built originals and negative controls. | `ctest --preset dev --no-tests=error -R '^cpu_' -V` |
| Script states | Reproduce a parent/child lookup, an ambiguous selection or a damaged sibling link with a synthetic tree. | `ctest --preset dev --no-tests=error -R '^script_(state_literals\|path_links)$' -V` |
| Texture recovery | Check a packed mip tail, a first refill byte, or a resident mip drawn on the software adapter. | `ctest --preset dev --no-tests=error -R '^(bcpack_prefix_and_mip_tail\|bc7_draw_warp)$' -V` |
| Static shader recompilation | Translate the single-texture pixel shader fixtures to desktop DXIL and check every output channel. [Supported instructions and bindings](docs/experiments.md#offline-pixel-shader-recompilation) | `ctest --preset dev -R '^pixel_aot_' -V` |
| D3D12 and ray tracing | Compare the CPU reference, custom BVH shader and DXR on another adapter. Record the driver and native wave width. | `ctest --preset dev -R '^bvh_(differential\|wave64)_hardware$' -V` |
| Windows runtime | Add a source-built allocation case with a measurable Windows result. | `ctest --preset dev -R 'era_heap\|desktop_memory_query'` |
| PE tooling | Reproduce an import-resolution edge case with a DLL or EXE built from source. | `ctest --preset dev -R 'static_exe_intake\|conditional_import_paths'` |
| Build and documentation | Try the first build from a clean clone. Identify the first missing prerequisite or unclear instruction. | Use the three-test command above. |

For task ownership, evidence requirements and PR review, see the [contribution guide](CONTRIBUTING.md). [Open an issue](https://github.com/R0Y00/Xbox2Windows/issues/new/choose) · [Community conduct](CODE_OF_CONDUCT.md) · [Private security reports](SECURITY.md)

## GTA VI case study and provenance

The installation metadata inspected for this case study lists a Scarlett executable. The main executable was absent from the inspected files. The public regression suite uses source-built fixtures and independently built shaders. [Intake methods and evidence](docs/experiments.md#static-executable-intake)

## License

Original project code and documentation use the [MIT License](LICENSE). Third-party games, assets, SDKs and trademarks belong to their respective owners. Xbox2Windows is an independent community project, unaffiliated with Rockstar Games, Take-Two, Microsoft or AMD.

