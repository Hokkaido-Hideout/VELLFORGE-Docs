# Troubleshooting

[Documentation home](README.md)

## VellForge does not appear in the Editor

Check that the plugin is installed once, its descriptor is directly inside its plugin folder, and it is enabled in **Edit > Plugins**. Restart the Editor. Confirm that the package matches your engine build and that UVEditor, TextureGraph, GeometryScripting, and GeometryProcessing are available and enabled.

If Unreal reports missing or incompatible modules, install the matching package or rebuild from source with the matching engine toolchain. Do not rename the descriptor to resolve a module error.

## Native runtime unavailable or a DLL cannot load

Check the [native file inventory](installation.md#3-check-the-native-runtime-files). A library can exist on disk and still fail to load because one of its dependencies is missing.

Install or repair the Microsoft x64 runtime, NVIDIA driver, and CUDA 12.8 libraries from the official links in [Installation](installation.md). Restart the Editor after environment changes. A missing `cublas64_12.dll` or `cudart64_12.dll` is a CUDA library problem, not a missing model.

The current native build targets compute capability 12.0. Errors such as an unavailable kernel image on another GPU need a compatible VellForge native build, not a different checkpoint. The `sd-cli` and `vision-cli` programs are not required fixes.

## Model pack rejected

Check the exact path, byte size, and SHA-256 of every component using the [model verification instructions](models.md). Typical causes include an incomplete download, the wrong FinalCut variant, a renamed incompatible model, or a mismatched manifest/runtime version.

The four-file pack must be complete even if background removal is disabled. The currently published upstream ControlNet and BiRefNet files do not match this build's pinned identities; see [availability](models.md#download-sources-and-availability). Do not edit hashes in the manifest to suppress the error.

## Generate is disabled

Select a valid saved Static Mesh, target material slot, and UV channel. For tagged generation, select at least one base material tag. Finish or cancel the current operation and apply or discard pending UV edits. Read the LAB status message for the specific rejection.

Preset tiles start an operation immediately. Set your controls before clicking one.

## Base Texture cannot be read

Use a Texture2D with retained source pixels. Reimport the source image if needed. Confirm that its atlas layout corresponds to the preview mesh and selected UV channel. Clear the Base Texture slot to return to fresh-latent generation.

## Generation is slow or memory admission fails

Return to 1024 internal resolution and Standard quality. Close other memory-heavy applications and use one generation job at a time. The full-FP32 checkpoint intentionally uses system RAM and a bounded GPU streaming window; low GPU utilization during some phases does not by itself indicate failure.

Keep the default 16 steps while establishing a working baseline. Internal 1536/2048 generation can require substantially more memory. No fixed duration is promised. Cancellation is cooperative and may take time to reach a safe native boundary.

## Upscaling fails or no final material is published

Confirm that the admitted RealESRGAN file exists and passes verification. The LAB requires the final aligned 4096 output; it does not silently substitute a smaller reconstruction when upscaling fails. Read the diagnostic before rerunning the same request.

## Surface Processor fails

Confirm that TextureGraph is enabled and the plugin Content assets are intact. Restore the matching plugin package if the supplied graph or material parent is missing. Raw publication and Surface Processor completion are separate phases; inspect the status before treating a raw result as the final processed material.

## The result contains scenery or has the wrong scale

Describe the physical surface rather than camera, lighting, or environment. State scale explicitly: architectural stone, furniture-scale wood grain, fine armor scratches, or subtle molded plastic. Verify mesh import scale and UV texel density.

For distinct surfaces, use separate material slots and the Surface Planner. Prompt wording alone cannot reliably divide an ambiguous shared atlas into metal, leather, and cloth regions.

## Seams or background changes

Inspect UV overlaps, padding, distortion, and the channel used by the material. Use the generated derivative mesh if you generated against its UVs. Keep Average-Color UV Background enabled for ordinary seam guarding; disable it for exterior-containment comparisons. Try Smooth UV Seams after fixing the unwrap.

To preserve more of a Base Texture, reduce denoise from 0.75 and compare using the same nonzero seed. Structural guidance and hard exterior containment do not guarantee unchanged detail inside the mask.

## Information to include in a support request

Use the support contact supplied on the VellForge product listing. Include the plugin version, Unreal version, Windows version, GPU model, driver/CUDA versions, failing operation, visible diagnostic, and relevant lines from the project's `Saved/Logs/` directory or Unreal Output Log. For a reproducible generation issue, include the resolved seed, settings, model-pack identity, and a screenshot of the result when shareable.

Review logs before sharing them; remove credentials and private paths. Model weights are not needed in a support request.
