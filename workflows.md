# Generate textures in VellForge LAB

[Documentation home](README.md)

Complete [installation](installation.md) and [model verification](models.md) first. Start with a saved Static Mesh with correct scale, normals, material slots, and non-overlapping UVs.

## First material

1. Right-click a Static Mesh in the Content Browser and choose **Edit in VellForge LAB...**. Alternatively, open **Window > VellForge LAB**, select the mesh, and use the **+** button beside the preview Mesh selector.
2. Confirm the preview mesh, target material slot, and **Guidance UV Layer**. Use UV0 unless your mesh uses another layer deliberately.
3. In **AUTO UV PREFLIGHT**, set the intended texture resolution and click **Analyze UVs**. Resolve overlaps, degenerate faces, out-of-bounds islands, or inadequate padding.
4. If repair is necessary, choose an unwrap method and click **Create UV Derivative**. This creates an asset under `/Game/VellForgeGenerated/AutoUV/`, preserving the source mesh. Apply or discard pending UV edits before generating.
5. Set the Diffuser controls using the table below. Clear **Base Texture** for a new material.
6. Click a suitable **Preset Material Library** tile. A tile starts generation immediately; configure settings before clicking it.
7. Keep the target mesh and UV layer unchanged while generation runs. **CANCEL** requests cooperative cancellation, which may wait for a native phase boundary.
8. On success, inspect the published material on the preview mesh and review its texture maps. Save the project assets you intend to keep.

## Starting settings

| Control | Initial value | Effect |
| --- | --- | --- |
| Internal Resolution | 1024 × 1024 | Native generation resolution; the LAB publishes 4096 × 4096 maps |
| Quality | Standard | Start here before trying larger internal resolutions |
| Seed | 0 for exploration | Randomizes once; record the resolved nonzero seed for repeat comparisons |
| Scheduler | DPM++ 2M / Karras | Default recipe |
| Sampling Steps | 16 | More steps increase work and do not guarantee a better material |
| Guidance Scale | 5.0 | Prompt influence; tune gradually |
| Denoise Strength | 0.75 for a Base Texture | Higher values allow more change to the supplied atlas |
| Structure Enforcement | 0.08 starting point | Structural guidance, independent from denoise; presets may provide different values |
| Wrap Mode | Clamp | Appropriate for unique mesh atlases |
| Average-Color UV Background (Seam Guard) | On | Fills unused atlas texels with each map's average covered color |
| Smooth UV Seams | Off during diagnosis | Evaluate UV quality before enabling final seam smoothing |
| Enable Surface Processor | On | Processes the final material inputs |
| Use Unreal Mesh Baker Inputs | On | Incorporates geometry-derived mesh information |

The interface admits 1–100 steps and guidance 0–30. Non-default recipes need visual review. Internal 1536 and 2048 modes cost more memory and time; publication remains 4096. Keep a nonzero seed when comparing settings, and retain the same model pack, input assets, and backend. Do not assume identical pixels across different native builds or hardware.

## Custom materials with tags

Use **Material Tag Mixer** when a preset does not describe the intended surface.

1. Enter a physical **Object Description**.
2. Select at least one base material tag.
3. Read the generated material brief below the tag grid. Catalog order determines the primary substrate, secondary structure, and later accents; selection order is not the rule.
4. Click **GENERATE TAGGED MATERIAL**.

Example for a stone pillar:

```text
Single carved architectural pillar; pale limestone at building scale; fine pores,
small age chips, dust in sheltered recesses, and restrained mineral variation;
preserve continuous carved stone and readable edges.
```

Describe construction, substrate, scale, finish, and wear. Camera angles, dramatic lighting, landscapes, and pedestals can become unwanted imagery in the texture. A standard preset supplies its own curated material brief; use tags when the Object Description should contribute.

## Transform an existing atlas

1. Assign the source Texture2D to **Base Texture**. It must retain readable source image data and match the intended mesh UV layout.
2. Start at denoise `0.75`. Lower it to preserve more of the source; raise it for larger changes.
3. Set the desired material brief or preset and start generation.
4. Compare authored boundaries and recognizable details against the original before keeping the result.

The explicit Base Texture becomes the img2img input. Clearing it restores fresh-latent generation; a preset thumbnail is not substituted for the source atlas. ControlNet guides structure, while a separate containment pass preserves pixels outside the approved UV mask after decode and upscale.

**Average-Color UV Background** is a later publication step that intentionally changes unused atlas texels. Disable it when checking byte-for-byte exterior preservation. It does not modify covered texels, and containment does not promise to preserve every detail inside the mask.

## Plan multi-material meshes

For assemblies or meshes with several material slots:

1. Select the actors or Static Mesh assets.
2. Open **MULTI-MESH SURFACE PLANNER** and click **SCAN SELECTION**.
3. Inspect each material-slot classification and UV status; correct mismatches.
4. Load one row into LAB and generate that surface.
5. Repeat for other slots.

Use separate slots for physically distinct surfaces such as steel plate, leather straps, and cloth. The planner organizes per-surface work; it does not generate every slot automatically or guarantee semantic regions on an ambiguous shared atlas.

## Review the result

- Inspect Albedo without lighting for painted shadows, scenery, or guidance colors.
- Rotate the mesh and inspect seams, grazing highlights, and texture scale.
- Check Normal for exaggerated noise, and Roughness/Specular for useful variation.
- Verify Metallic values make sense for the material.
- Record the resolved seed and settings for a candidate worth refining.

Continue with [material application and customization](materials-and-customization.md).
