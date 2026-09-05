# Model pack installation

[Documentation home](README.md)

VellForge reads model files from the installed plugin's `LocalModels/SDXL/` directory. It checks the manifest, component sizes, SHA-256 hashes, and runtime compatibility before native inference. It does not download, convert, or repair model files automatically.

## Download sources and availability

The current model identity is **VellForge SDXL FinalCut 13GB FP32 0.3-development/runtime-abi-8**. Install the exact files admitted by the manifest supplied with your plugin version.

| Component | Acquisition instructions |
| --- | --- |
| FinalCut checkpoint | The recorded publisher source is [Felldude FinalCut SDXL, version 2119886](https://civitai.com/models/1864658/finalcut-sdxl?modelVersionId=2119886). Select the 13GB full-FP32 variant and verify the hash below. The smaller variant and Lightning checkpoints are incompatible. Publisher access may require an account; availability was not independently confirmed during this documentation check. |
| RealESRGAN | Download `RealESRGAN_x4plus.safetensors` from the [Comfy-Org Safetensors repackaging](https://huggingface.co/Comfy-Org/Real-ESRGAN_repackaged/blob/main/RealESRGAN_x4plus.safetensors). Its published size and SHA-256 match the current manifest. This is a model-file download; installing ComfyUI is unnecessary. |
| SDXL Scribble ControlNet | Obtain the exact manifest-matching artifact from the VellForge publisher. The public [xinsir model](https://huggingface.co/xinsir/controlnet-scribble-sdxl-1.0) currently has a different SHA-256 and is not a verified substitute for this build. |
| BiRefNet | Obtain the exact manifest-matching GGUF from the VellForge publisher. The public [Acly BiRefNet GGUF files](https://huggingface.co/Acly/BiRefNet-GGUF) currently have different SHA-256 values and are not verified substitutes for this build. |

**Current availability limitation:** a verified public download location for the admitted ControlNet and BiRefNet artifacts has not been established. A new installation cannot complete model admission without those exact files. Contact the VellForge publisher through the product listing if they are absent from your authorized distribution. Renaming a different download does not make it compatible.

Review the model publisher's terms before downloading and using weights. The current development checkpoint is not included under an established VellForge redistribution approval. Installing it locally does not establish permission to redistribute it with a plugin or game.

## Install the files

1. Close Unreal Editor.
2. Locate the folder containing `VellForgeTool.uplugin`. All paths below are relative to that folder, including for an engine-installed plugin.
3. Keep the `model.manifest.json` supplied with your matching plugin version.
4. Download the exact compatible model artifacts and place them in the following locations. If a verified download has a different filename, rename it to the required name without modifying its contents.
5. Verify every size and SHA-256 before launching the Editor.

```text
LocalModels/
  SDXL/
    model.manifest.json
    diffuser/
      SDXL_Finalcut.safetensors
    control_net/
      HKH_Core_ControlNet-Scribble_SDXL-1.0/
        diffusion_pytorch_model.safetensors
    upscaler/
      RealESRGAN_x4plus.safetensors
    bg_removal/
      BirefNet.gguf
```

The FinalCut checkpoint includes both CLIP encoders and the SDXL VAE. Do not add a separate VAE or text encoder. LoRAs, Flux RefControl adapters, external CLIP Vision, and multi-reference composition are not supported by this profile.

All four files are required by current pack admission, even when an operation does not use background removal or ControlNet. Loading remains request-driven; requiring their installation does not mean all models are simultaneously resident in GPU memory.

## Exact file identity

| File | Bytes | SHA-256 |
| --- | ---: | --- |
| `SDXL_Finalcut.safetensors` | 13,875,721,488 | `b334ed89c3b8982287037cbb0339ade901cc02cf5111b0d8d37c8bc4408fae8b` |
| `diffusion_pytorch_model.safetensors` | 2,502,139,104 | `053e9427d3936ac2f1ed766d43cfc119a49a44938aa2e3475c7f01e4a0c476f3` |
| `RealESRGAN_x4plus.safetensors` | 66,857,836 | `37f9a931c215f040aa6d50f711f2cb115f713c46df1d0d6469a8bd7bfe9a60bb` |
| `BirefNet.gguf` | 440,372,864 | `4ef5969052247e339c2c49a21b3ea91760f06deda8da456ccff757b4b66de71c` |

To inspect all installed components, edit the first line and run this read-only PowerShell snippet:

```powershell
$pluginRoot = 'D:\YourProject\Plugins\HKHVellForge'
$modelRoot = Join-Path $pluginRoot 'LocalModels\SDXL'
$manifest = Get-Content -LiteralPath (Join-Path $modelRoot 'model.manifest.json') -Raw | ConvertFrom-Json
foreach ($property in $manifest.components.PSObject.Properties) {
    $component = $property.Value
    $file = Join-Path $modelRoot $component.file
    if (-not (Test-Path -LiteralPath $file -PathType Leaf)) {
        Write-Output "$($property.Name): MISSING"
        continue
    }
    $sizeMatches = (Get-Item -LiteralPath $file).Length -eq [long]$component.bytes
    $hashMatches = (Get-FileHash -LiteralPath $file -Algorithm SHA256).Hash -ieq $component.sha256
    [pscustomobject]@{
        Component = $property.Name
        SizeMatches = $sizeMatches
        HashMatches = $hashMatches
    }
}
```

Every component must report `True` for both checks. Hashing these large files takes time. This check compares files to the supplied manifest; the provider additionally validates the manifest's pinned identity and canonical pack hash. Editing the manifest to accept another model will not qualify it.

## Confirm in Unreal

Restart the Editor and open VellForge LAB. Read the diagnostic if model validation fails. Once the pack is admitted, generate a small initial candidate with the [standard workflow](workflows.md). Keep the default recipe while checking installation: 1024 internal resolution, DPM++ 2M/Karras, 16 steps, CFG 5.0.
