# Installation and prerequisites

[Documentation home](README.md)

## Requirements

| Component | Requirement for this integration |
| --- | --- |
| Platform | 64-bit Windows; Unreal Editor on Win64 |
| Unreal Engine | Stock UE 5.8; use the engine build supported by your plugin package |
| Editor rendering | DirectX 12 and Shader Model 6 |
| Native inference GPU | The current CUDA library targets Blackwell compute capability 12.0; RTX 5060 with 8 GiB VRAM is the development reference device |
| NVIDIA software | Compatible NVIDIA display driver and CUDA 12.8 runtime components |
| Microsoft runtime | Microsoft Visual C++ v14 Redistributable, x64 |
| Background removal | Vulkan-capable graphics driver and the matching vision.cpp runtime |
| Model storage | Approximately 16.89 GB for the four admitted model files, plus the plugin, download staging, caches, generated assets, and Unreal project storage |

The 13.88 GB checkpoint uses CPU-resident parameters and bounded GPU streaming. Its size is not a VRAM requirement, and an 8 GiB GPU does not imply that 8 GiB of system RAM is sufficient. No general minimum system-RAM specification has been qualified for this documentation. Larger internal resolutions increase time and memory requirements.

CPU fallback within a successfully loaded native backend does not remove Windows DLL dependencies or establish support for non-NVIDIA hardware with this CUDA build.

## 1. Install the external prerequisites

Close Unreal Editor before installing or repairing prerequisites.

### NVIDIA driver

Download the driver for your exact GPU and Windows version from [NVIDIA's official driver page](https://www.nvidia.com/en-us/drivers/). Follow the installer and restart if requested. The GPU driver also provides the Vulkan support used by the background-removal backend.

### Microsoft Visual C++ runtime

Open Microsoft's [Visual C++ Redistributable download page](https://learn.microsoft.com/en-us/cpp/windows/latest-supported-vc-redist?view=msvc-170), select the **x64 v14** package, and install it. If already installed, use **Repair** when troubleshooting missing Microsoft runtime DLLs. Microsoft requires the runtime to be at least as recent as the compiler used to build the application.

### CUDA 12.8

The current native library depends on CUDA 12 DLLs, including `cudart64_12.dll` and `cublas64_12.dll`. A display driver alone does not install every CUDA Toolkit runtime library needed by VellForge.

1. Open the official [CUDA Toolkit 12.8 download archive](https://developer.nvidia.com/cuda-12-8-0-download-archive).
2. Select Windows, x86_64, and your Windows version.
3. Download and run the NVIDIA installer. Install the CUDA runtime and library components; installing the complete Toolkit is a straightforward way to provide them.
4. Follow NVIDIA's [Windows installation guide](https://docs.nvidia.com/cuda/archive/12.8.0/cuda-installation-guide-microsoft-windows/index.html).
5. Restart Unreal Editor after installation so it inherits the updated environment.

If Windows still cannot locate the CUDA libraries, check that the installation's `bin` directory is on `PATH`, normally `C:\Program Files\NVIDIA GPU Computing Toolkit\CUDA\v12.8\bin`. Do not copy unrelated DLL versions into the Unreal Engine directory.

The CUDA compiler is a developer tool; normal texture generation calls the supplied native library. The Vulkan SDK is also a developer prerequisite, not an application users must run alongside VellForge.

## 2. Install the plugin

If your distribution offers an engine-managed installation, use its installer for the supported engine version. For a project-local plugin archive:

1. Close the project and extract the plugin folder into `<YourProject>/Plugins/HKHVellForge/`.
2. Confirm that `VellForgeTool.uplugin` is directly inside that folder. Avoid a second nested copy of the plugin directory.
3. Open the project in the matching Unreal Editor version.
4. Open **Edit > Plugins**, search for **VellForge**, and enable it. Restart when requested.
5. Keep its engine plugin dependencies enabled: **UVEditor**, **TextureGraph**, **GeometryScripting**, and **GeometryProcessing**.
6. In Project Settings, use **DirectX 12** as the Windows Default RHI and enable the SM6 shader format. Restart if requested.

Keep the descriptor name `VellForgeTool.uplugin` unchanged. Its filename is the Unreal plugin identifier even when the enclosing folder is `HKHVellForge`.

If Unreal asks to rebuild missing modules, use the source-build instructions below or obtain the package for your engine version. Repeatedly copying DLLs from another engine version will not repair module compatibility.

## 3. Check the native runtime files

The provider loads these VellForge-supplied libraries from the installed plugin's `Binaries/Win64/` directory:

```text
stable-diffusion.dll
visioncpp.dll
ggml-base.dll
ggml.dll
ggml-cpu.dll
ggml-vulkan.dll
```

Obtain matching libraries through the VellForge distribution. Upstream binaries are not drop-in replacements for its pinned integration. A source package also needs the corresponding headers, import libraries, and native DLLs under `Source/ThirdParty/`; UnrealBuildTool stages the runtime dependencies during the build.

`sd-cli.exe` and `vision-cli.exe` are development probes. They are not applications users must install or launch to generate textures. No ComfyUI installation, Python worker, local web server, or workflow JSON is required.

## 4. Install the models and confirm setup

Follow [Model pack installation](models.md), then open **Window > VellForge LAB**. Inspect any diagnostic before generating. A working viewport alone does not prove that the native libraries or model pack are ready.

Run the [first-material workflow](workflows.md) with the standard 1024 internal resolution. Confirm that generation completes, maps are published into project Content, and the material appears on the preview mesh.

## Optional: build or modify the source

This section applies to a complete VellForge developer checkout, which includes the root build scripts and pinned patches. A plugin-only archive may not include those developer files.

Install Visual Studio with the Unreal-compatible C++ toolchain and Windows SDK. Native rebuilds additionally require Git, CMake, Ninja, CUDA 12.8, the MSVC 14.44 toolset used by the scripts, and the [Vulkan SDK](https://vulkan.lunarg.com/sdk/home). Use Epic's [Visual Studio setup guidance](https://dev.epicgames.com/documentation/en-us/unreal-engine/setting-up-visual-studio-development-environment-for-cplusplus-projects-in-unreal-engine) for the UE toolchain.

From the checkout root, with the admitted BiRefNet file already installed:

```powershell
.\Build-StableDiffusionCpp.ps1 -VisualStudioRoot 'C:\Program Files\Microsoft Visual Studio\18\Community' -ToolsetVersion '14.44'
.\Build-VisionCpp.ps1 -VisualStudioRoot 'C:\Program Files\Microsoft Visual Studio\18\Community' -ToolsetVersion '14.44'
```

Adjust `VisualStudioRoot` to the actual installation. The scripts fetch pinned upstream source on first use and apply the required local patches. This developer download step is separate from local-only inference. Do not replace headers, import libraries, and DLLs independently.

Build the host from the checkout root using your installed stock engine:

```powershell
$projectFile = Join-Path (Get-Location).Path 'VellForge.uproject'
& 'C:\Program Files\Epic Games\UE_5.8\Engine\Build\BatchFiles\Build.bat' `
    VellForgeHostEditor Win64 Development `
    "-Project=$projectFile" -WaitMutex -NoHotReloadFromIDE
```

The native source pins are [stable-diffusion.cpp](https://github.com/leejet/stable-diffusion.cpp) commit `de298c225bed97c3f9026b73cd7b71e7879bd41b` and [vision.cpp](https://github.com/Acly/vision.cpp) commit `26a752912d49f6c4ff4545b35a1bdf7400d349ed`. Retain the VellForge patches and build settings. Changing the native ABI or GPU target requires new compatibility and inference testing.
