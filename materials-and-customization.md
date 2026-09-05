# Apply and customize generated materials

[Documentation home](README.md)

## Find generated assets

The normal LAB preset/tag workflow publishes textures, creates a material instance, and applies it to the LAB preview. Standard project locations are:

| Asset | Content Browser path |
| --- | --- |
| Generated maps | `/Game/VellForgeGenerated/Presets/<MeshName>/<CategoryName>/` |
| Material instances | `/Game/VellForgeGenerated/Materials/<MeshName>/UV<N>/` |
| Auto UV derivative meshes | `/Game/VellForgeGenerated/AutoUV/` |

Names may include part of the request cache hash. Keep the textures and material instance from the same generation together. Material IDs and Mesh Guidance are diagnostic assets, not Base Color replacements.

## Apply the material to your scene

1. Inspect the result in LAB, then locate the generated material instance in the Content Browser.
2. Select the intended mesh actor in the level and assign the material instance to the corresponding **Materials** slot in Details. This sets the component override for that actor.
3. To change the mesh asset's default material instead, open the Static Mesh Editor and edit its material slot deliberately; this can affect other uses of that asset.
4. If generation used an Auto UV derivative, use that derivative mesh in the scene. The original mesh may not share its UV layout.
5. Confirm that the material uses the same UV channel selected during generation, inspect under your scene lighting, and save the edited assets and level.

A successful LAB preview is not confirmation that every actor in the level has been updated.

## Use the maps in your own material

Create a project-owned material and connect the corresponding textures:

| Map | Material input | Interpretation |
| --- | --- | --- |
| Diffuse Albedo / Albedo | Base Color | sRGB color |
| Normal | Normal | Linear tangent-space normal data; use the texture's appropriate normal sampler/settings |
| Ambient Occlusion | Ambient Occlusion | Linear grayscale |
| Roughness | Roughness | Linear grayscale |
| Specular | Specular | Linear grayscale reflectance proxy, not emitted light |
| Metallic, when produced by the Surface Processor | Metallic | Linear material data |

Preserve the generated texture settings when reusing assets. If exporting and reimporting, restore each map's intended color space and sampler settings. Do not use sRGB sampling for scalar data or normal vectors. The native five-map output is not packed ORM; use named outputs rather than assuming channel packing.

The current workflow does not generate Emissive. Effects such as glowing lava require your own emissive mask and material logic.

## Refine appearance

Use LAB controls for Normal Strength and roughness, metallic, or specular constraints to refine a candidate. Keep the seed fixed while comparing one change at a time. The Surface Processor can combine generated maps with Unreal mesh-baker inputs and exposes the resulting material inputs for inspection.

For a custom material graph, work in project Content. Duplicate the relevant assets before editing shared plugin content so future plugin updates do not overwrite your custom work. Preserve asset references through Unreal's Content Browser operations rather than moving `.uasset` files in Explorer.

## Developer extension points

Source modifications use native C++20 modules:

| Module or resource | Responsibility |
| --- | --- |
| `VellForgeLab` | Slate UI, mesh/UV preparation, job orchestration, previews, asset publication |
| `IVellForgeGenerativeTextureProvider` | Request/output boundary between the LAB and generation provider |
| `VellForgeStableDiffusion` | Model validation and native inference integration |
| `Resources/VellForgeMaterialPresets/VellForgeMaterialPresets.json` | Curated preset catalog and material defaults |
| `VellForgeCompiler` | Separate producer shader-compilation infrastructure |

Keep UObject and Slate mutations on the Game Thread. Run inference asynchronously through the provider boundary, using immutable request data and cooperative cancellation. Native-model changes must update admission, request/cache identity, and tests together. Do not bypass hash, memory, bounds, or UV-containment checks to admit a replacement model.

See [source build prerequisites](installation.md#optional-build-or-modify-the-source) before rebuilding. Generation is an Editor authoring operation; these pages do not describe a runtime in-game AI API or the separate protected shader-product packaging workflow.
