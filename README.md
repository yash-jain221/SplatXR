# Integrating 3D Gaussian Splatting into XR

This repository contains the project and assets for "Integrating 3D Gaussian Splatting into XR: A Pipeline for Photorealistic VR and AR Rendering" (authors: Keshav Tripathi, Sirat Baweja, Yash Jain).

## Summary
- End-to-end capture → reconstruction → XR pipeline using 3D Gaussian Splatting (3DGS).
- Trained models with Nerfstudio (Splatfacto), exported as `.ply`, cleaned with Blender/SuperSplat, and imported into Unity and Unreal Engine 5 for VR/AR evaluation on Meta Quest 3.
- Comparative analysis of artifact modes and engine integration issues (stereo depth-sort, floaters, aliasing).

## Key features / contributions
- Documented pipeline from smartphone video to navigable VR splat.
- Integration notes and fixes for Unity (gsplat-unity) and UE5 (XScene/Xverse, MLSLabsRenderer).
- Notes on COLMAP robustness, reconstruction quality metrics (PSNR/SSIM/LPIPS), and headset profiling.

## Repo layout
- `project.tex` — LaTeX report describing method, experiments, and conclusions.
- `xrproject/` — Unreal/Unity projects and engine content (assets, maps, uassets).
- `outputs/` — exported reconstruction / Splatfacto outputs and dataset artifacts.

See the repository for additional binary assets (Unreal `.uasset` files managed with Git LFS).

## Prerequisites
- COLMAP (for SfM) or hloc + SuperPoint/SuperGlue for more robust phone-video SfM.
- Nerfstudio with Splatfacto backend (`ns-train splatfacto`, `ns-export gaussian-splat`).
- Blender with Kiri Engine & SuperSplat add-ons (for cleanup and mesh conversions).
- Unity 6.2 (URP) or Unreal Engine 5.5 for runtime integration.
- Meta Quest 3 (for in-headset profiling and deployment).

## Quickstart (reconstruction and export)
1. Extract frames from capture video (example with FFmpeg):

```powershell
ffmpeg -i capture.mp4 -vf fps=5 frames/frame_%06d.png
```

2. Run COLMAP SfM (or hloc/SuperGlue): tune sequential matcher overlap for phone video.

3. Train 3DGS via Nerfstudio Splatfacto:

```bash
ns-train splatfacto \
  --data <dataset_path> \
  --pipeline.model.use_scale_regularization True \
  --pipeline.model.cull_alpha_thresh 0.005 \
  --pipeline.model.continue_cull_post_densification False
```

4. Export trained model as `.ply`:

```bash
ns-export gaussian-splat --model <checkpoint> --out <export_dir>
```

5. Clean exported splats in Blender with SuperSplat / Kiri Engine to remove floaters.

6. Import cleaned `.ply` into engine:
- Unity: use `gsplat-unity` to convert `.ply` → `GaussianSplatAsset` and add render pass to URP.
- UE5: import as Blueprint actor via XScene/Xverse or MLSLabsRenderer (apply 90° X-axis rotation to match UE Z-up).

## Integration notes & known issues
- Stereo rendering asymmetry in UE5: implement per-eye depth sorting in plugin render pass (workaround: prefer Unity plugin for reliable stereo).
- Floaters/background Gaussians: manage via alpha-cull tuning or semantic masking during training/cleanup.
- COLMAP on handheld video: use hloc + SuperGlue or tune sequential matcher and enable loop-closure to avoid split models.

## Experiments & metrics
- Scripts and example configs for PSNR / SSIM / LPIPS evaluation are referenced in `project.tex` and should be run on held-out views recorded during capture. Tables in the report are placeholders for measured values.
