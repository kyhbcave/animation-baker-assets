---
title: Animation Baker — Third-party components
---

[English guide](index.html) · [한국어](ko.html)

# Third-party components / 외부 구성 요소

Animation Baker's plugin files (C++ source, Python worker scripts, documentation) are the publisher's own work, distributed under the Fab license, **except** the SAM 3D Body inference code in `Content/Python/Lib/site-packages/sam_3d_body/` (Meta, SAM License; see "GPU high-quality engine" below). **No third-party binaries, model weights or Python packages are included in the plugin package.** The components below are downloaded only when the user runs *Prepare video engine* (or the optional *Prepare GPU high-quality engine*) and approves the download dialog. They are installed into a separate per-user folder (`%LOCALAPPDATA%\DOCKINI\AnimationBaker\VideoEngine-v3`) and run as separate processes.

플러그인 패키지에는 외부 바이너리·모델·Python 패키지가 포함되지 않습니다. 아래 구성 요소는 사용자가 **영상 엔진 준비**를 실행하고 다운로드 안내에 동의한 경우에만 사용자 폴더에 설치되며 별도 프로세스로 실행됩니다.

## Python runtime
- Interpreter: the Python 3.11 bundled with Unreal Engine (Python Software Foundation License) is used only to create an isolated virtual environment. The engine's Python environment is not modified.

## Python packages (PyPI)
Exact versions: `Content/Python/AnimationBakerWorker/requirements-lock.txt` (installed with `--only-binary=:all:`).
License texts of the installed distributions, captured from a reference installation: `Docs/worker-package-notices.txt`; inventory: `Docs/worker-package-inventory.json`.

| Package | Version | License |
|---|---|---|
| mediapipe | 0.10.21 | Apache-2.0 |
| opencv-contrib-python | 4.11.0.86 | Apache-2.0 (bundled FFmpeg in the wheel: LGPL-2.1) |
| numpy | 1.26.4 | BSD-3-Clause |
| scipy | 1.13.1 | BSD-3-Clause |
| jax / jaxlib | 0.6.2 | Apache-2.0 |
| protobuf | 4.25.9 | BSD-3-Clause |
| matplotlib | 3.10.9 | PSF-based (matplotlib license) |
| others (absl-py, attrs, flatbuffers, sentencepiece, sounddevice, …) | see lock file | see notices file |

## Pose model
- MediaPipe Pose Landmarker Heavy (BlazePose GHUM 3D), float16, version 1
- Source: https://storage.googleapis.com/mediapipe-models/pose_landmarker/pose_landmarker_heavy/float16/1/pose_landmarker_heavy.task
- SHA-256: `64437af838a65d18e5ba7a0d39b465540069bc8aae8308de3e318aad31fcbc7b` (verified on download)
- License: Apache License 2.0, per Google's model card "Model Card BlazePose GHUM 3D" (https://storage.googleapis.com/mediapipe-assets/Model%20Card%20BlazePose%20GHUM%203D.pdf). The model card lists multi-person images, subjects farther than ~4 m, invisible heads and metric-depth applications as out of scope, and states that surveillance or identity recognition is out of scope.

## FFmpeg
- Version 9.0.2, "essentials" Windows build by Gyan Doshi (https://www.gyan.dev/ffmpeg/builds/)
- Download: https://github.com/GyanD/codexffmpeg/releases/download/9.0.2/ffmpeg-9.0.2-essentials_build.zip
- SHA-256: `60f467265b1e312373dbcd92200c2618a74850f98d3d078e94296bb3fa2047ba` (embedded in the plugin and verified on download; the value was cross-checked against the GitHub release digest and gyan.dev's published checksum when pinned)
- License: GPL-3.0-or-later (this build includes GPL components such as libx264). The build's LICENSE and README are installed next to the executables. FFmpeg source: https://ffmpeg.org/download.html. Animation Baker does not link FFmpeg; it launches `ffmpeg.exe` / `ffprobe.exe` as separate programs to create a local editing proxy of the user's video.

## GPU high-quality engine (optional) / GPU 고품질 엔진 (선택)
Installed only when the user runs *Prepare GPU high-quality engine* on a PC with an NVIDIA GPU and approves the dialog. Location: `%LOCALAPPDATA%\DOCKINI\AnimationBaker\VideoEngine-v3\gpu`, a separate virtual environment and process; removing that folder removes the feature.

- **SAM 3D Body** (Meta Platforms, Inc.) — inference code included in the plugin under `Content/Python/Lib/site-packages/sam_3d_body/` (unmodified, the Fab location for third-party Python code), with the commit in `Content/Python/Lib/site-packages/sam-3d-body-vendor/VERSION.txt`, model configuration `model_config.yaml` in the same folder, and the model weights `model.ckpt` downloaded from the publisher's mirror. Licensed under the **SAM License** (full text: `Content/Python/Lib/site-packages/sam-3d-body-vendor/LICENSE-SAM.txt`, also shipped next to the downloaded weights). The SAM License prohibits, among other things, use for military or warfare purposes, nuclear applications, espionage, guns or illegal weapons, activities subject to ITAR, and end uses prohibited by trade controls; it also prohibits reverse engineering the SAM Materials. Original distribution: https://huggingface.co/facebook/sam-3d-body-dinov3.
- **MHR (Momentum Human Rig)** body model `mhr_model.pt` (Meta) — Apache License 2.0, https://github.com/facebookresearch/MHR.
- Weights are verified by SHA-256 per downloaded part and per file (`sam3d/sam3d_manifest.json`): `model.ckpt` `b5a2f9d305dd02626b967aa2e86021fba07065df66ce7a7e00ffb9664f150abf`, `mhr_model.pt` `352e271a6c42729c68554ceaea0c955e866970160c31e35506d782dc0f7377bc`.
- **PyTorch** 2.5.1 + torchvision 0.20.1 (CUDA 12.4 builds, BSD-3-Clause; bundled NVIDIA CUDA runtime libraries under NVIDIA's redistribution terms) from https://download.pytorch.org, and the packages pinned in `requirements-gpu-lock.txt` (PyPI; mostly BSD/MIT/Apache-2.0, e.g. pytorch-lightning Apache-2.0, omegaconf BSD-3-Clause, roma BSD-3-Clause, opencv-python Apache-2.0).

SAM License 전문과 금지 용도는 위 파일에 있습니다. GPU 엔진을 설치하는 사용자는 SAM License 조건에 동의해야 합니다.

## CelMotion import
The `celmotion_ingest` module reads CelMotion project/pose files. See `Content/Python/AnimationBakerWorker/celmotion_ingest/NOTICE.txt` when present.

## Unreal Engine content
"Create example animation set" copies Manny/Quinn from the user's installed Unreal Engine templates under the Unreal Engine EULA. No Epic content is redistributed in this plugin.

## Network
Only the downloads above (pypi.org, files.pythonhosted.org, storage.googleapis.com, github.com and its release CDN). No telemetry, no upload of user video, poses or projects.
