# Animation Baker — download assets

Files for the **Animation Baker** Unreal Engine plugin by DOCKINI STUDIO.

- **Documentation:** https://kyhbcave.github.io/animation-baker-assets/ (English) · [한국어](https://kyhbcave.github.io/animation-baker-assets/ko.html)

## GPU high-quality engine files (release `sam3d-v1`)

The plugin's optional *Prepare GPU high-quality engine* step downloads these files after the user approves a consent dialog, and verifies each one by SHA-256. You do not need to download them by hand.

| File | Contents | License |
|---|---|---|
| `model.ckpt.part1`, `model.ckpt.part2` | SAM 3D Body model weights (`sam-3d-body-dinov3`), unmodified, split below 2 GiB | SAM License (`LICENSE-SAM.txt`) |
| `mhr_model.pt.part1` | Momentum Human Rig body model, unmodified | Apache License 2.0 |
| `LICENSE-SAM.txt`, `NOTICE.txt` | License text and notice | — |
| `sam3d_hosting.json` | Sizes and SHA-256 of every part and file | — |

SAM 3D Body is by Meta Platforms, Inc. (original distribution: https://huggingface.co/facebook/sam-3d-body-dinov3) and is redistributed here under the SAM License, a copy of which accompanies the files. The prohibited uses in the SAM License apply to every recipient, including use for military or warfare purposes, nuclear applications, espionage, guns or illegal weapons, and any use subject to ITAR or prohibited by trade controls. MHR is by Meta (https://github.com/facebookresearch/MHR).

This repository does not contain the plugin itself; it is sold on Fab.
