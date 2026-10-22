---
title: Animation Baker — User Guide
---

[한국어](ko.html) · [Third-party components](third-party.html)

# Animation Baker 1.0 — User Guide

Animation Baker is an Unreal Editor tool that **imports motion from a reference video or an existing animation → applies it to your character → lets you sculpt poses frame by frame → exports Animation Sequences, Montages or SpriteBaker projects**. It is an editor-only plugin; nothing is added to your packaged game.

> The editor UI is currently in Korean. This guide names each control with its on-screen Korean label in parentheses.

---

## 1. Requirements

| Item | Details |
|---|---|
| Engine | Unreal Engine 5.8, Win64 Editor (other versions only as listed on the Fab product page) |
| OS | Windows 10/11 64-bit |
| Video analysis | One-time internet connection (~1 GB download), 2 GB free disk space |
| Python | No separate install. An isolated environment is created with the engine's bundled Python 3.11. |
| Plugin dependencies | IK Rig, Full Body IK, Animation Data, Niagara (shipped with the engine, enabled automatically) |

## 2. Installation

1. Install to the engine from your Fab library, or copy the folder to `<Project>/Plugins/AnimationBaker`. Do not install it in both places.
2. Enable **Edit → Plugins → Animation Baker** and restart the editor.
3. Open **Tools → Animation Baker**.

## 3. Quick start

1. Top-left **Import (가져오기) → Create example animation set (예제 애니메이션 셋 만들기)**. Manny is copied from your installed Third Person template and an example project with 26 states (Idle, 8-way Walk/Jog, Jump, Attack, …) is created. If the template is missing, install *Templates and Feature Packs* from the Launcher or pick your own character and animation. Example assets are not part of this plugin.
2. Pick a state in the tree, pick a frame on the timeline, then drag the green spheres in the viewport (or select a bone in the bone tree) to change the pose.
3. **Save project (프로젝트 저장, Ctrl+S)**, then **Export… (내보내기…)**. Source assets are never overwritten.

## 4. Motion from video

### 4.1 Prepare the video engine (once)
Open **Video import (영상 가져오기)** → **Prepare video engine (영상 엔진 준비)**. A dialog lists what will be downloaded and its licenses. On approval it installs into `%LOCALAPPDATA%\DOCKINI\AnimationBaker\VideoEngine-v3`:

- Python packages (MediaPipe 0.10.21, OpenCV, NumPy, SciPy, …; pinned versions, binary wheels only)
- MediaPipe Pose Landmarker Heavy model (Apache-2.0, SHA-256 verified)
- FFmpeg 9.0.2 (gyan.dev Windows build, GPL, SHA-256 verified)

The engine's Python and your project are not modified. Analysis runs locally; video is never uploaded. To use a different Python 3.10–3.12, enter its path in the field.

### 4.2 Analyse
1. **Select video or CelMotion project…** — mp4/mov/mkv/avi/webm/m4v. **Local files only.** Use footage you shot yourself or have the right to use.
2. Drag a box around the **whole body of the target person** in the preview. **Track selected person** is on by default.
3. Set start/end frames (max 30 s) and **Analyse and apply to new state (분석 후 새 상태에 적용)**.
4. In group dances, if tracking switches to someone else, go to that frame, redraw the box and add a **tracking guide (추적 보정점)**.

Output is 30 fps **in-place** motion. Re-analysing the same file/region reuses the cached pose analysis. Expect roughly 1–2 seconds of CPU time per video frame on a desktop CPU (a 10 s clip at 30 fps ≈ 5–10 minutes). Opening a long video for the first time adds time to build its editing proxy and its preview/person index. After that, preview steps are instant, and re-analysing after adding a tracking guide re-runs tracking on the stored person candidates and only re-infers frames whose region changed.

**Subject tracking (1.0)** combines top and thigh colour/brightness, the appearance remembered from the first frames, and positional continuity. When people overlap it does not jump to a differently dressed person; those frames are reported as **subject check needed (대상 인물 확인 필요)** with the first frame, and **show detected problem frame (문제가 감지된 프레임 보기)** takes you there to add a guide. People wearing the same outfit crossing each other may still need guides.

**Identity check (대상 확인)**: tracked frames where the torso jumps away for 1–2 frames and comes back may briefly show someone else. They are counted separately as **대상 확인 N프레임**; **다음 대상 확인 프레임 보기** steps through them. If a frame shows another person, add a guide there and analyse again. A swap to someone standing at almost the same position is not detected.

**Joint anchors (관절 고정)**: after an analysis the preview shows the subject's shoulders, elbows, wrists, hips, knees and ankles. Drag a wrong point to anchor it on that frame (orange; the four torso points are anchored with it). Re-analysing uses the anchored positions as they are on that frame and picks the person who matches them. The 6 frames on each side follow only where their own joint was uncertain; every other frame is still detected on its own. **다음 관절 한계 프레임 보기** steps through the frames flagged by the joint-limit check first. Anchor a few frames, not many.

**Shot-change check (1.0)**: if the selected range contains a cut or transition, analysis stops before inference and reports the frame. Choose a range inside one shot.

### 4.3 Review and correct
- **Pink lines** on timeline bone rows mark confidence below 45%. **Next review frame (다음 보정 프레임 →)** jumps to the frame and bone.
- Since 1.0, **feet gliding on the floor** are also marked pink (see 4.4).
- **Poses outside joint limits** (knee or elbow bent backwards, impossible hip, shoulder, spine or neck angles) are also marked pink, and the result panel lists the count per joint and the first frame. They mostly come from video depth errors, so check them in a side camera and fix them. The analysis option **관절 한계 밖 자세 자동 보정** (experimental, off by default) keeps ankle and wrist positions and turns knee/elbow bend direction and head direction back into range. GPU high-quality 3D greatly reduces violations.
- Confidence is not a measured accuracy. Compare interpolated or estimated spans with the source video and fix them.

### 4.4 Foot grounding
- **Foot grounding (발 접지 보정)**, on by default, places the actual shoe sole of your mesh on the floor and locks its horizontal position during contacts detected in the video. Unreachable targets move the pelvis instead of stretching bones.
- Grounding v2 (1.0): contacts the video detector missed are inferred when **the mesh sole stays within 2 cm of the floor, slower than 1.5 m/s, for at least 0.12 s**. Airborne frames are never inferred.
- Feet that still glide fast along the floor without a lock are not altered; they are **flagged for review**.
- Grounding v3: if placing a foot would bend the knee sideways or backwards, the knee is turned the least amount around the hip→foot axis to bring it back inside the leg's range of motion. Foot position and knee bend angle stay the same. States saved earlier keep v2.
- States saved before 1.0 keep their previous grounding so existing work does not change. Re-import the video to use v2.
- **Floor height (바닥 높이, cm)** adjusts the floor; turn grounding off to compare with the raw motion.

### 4.5 CelMotion poses
Reads `.celmotion` v1 projects and CelMotion-format `.npz`. NPZ without image size requires **fixed pelvis height (골반 높이 고정)**.

### 4.6 GPU high-quality 3D (optional, experimental)
With an NVIDIA GPU (6 GB memory or more), the 3D pose can be estimated with **SAM 3D Body** (Meta). 2D tracking, identity checks, review flags and foot grounding stay the same; only the 3D joints change.

- **Prepare:** in the video window, **Prepare GPU high-quality engine (optional)** → confirm the dialog. PyTorch (CUDA 12.4) and the model (about 7.5 GB) are installed into `VideoEngine-v3\gpu`. The base video engine is not touched.
- **Use:** once prepared, **GPU high-quality 3D** appears in the analysis options. A 30-second range takes about 3 extra minutes (measured on an RTX 3090).
- **Effect (internal measurements):** fewer wide stances baked as kneeling in the side view, less hand/foot jitter and less foot sliding. Fast spins can get worse limb directions than the base engine.
- **On failure:** if the GPU run fails (out of memory, driver issues), analysis continues with the base 3D and the result notes the reason.
- **License:** SAM 3D Body is under the SAM License, which prohibits military/warfare, weapons, nuclear and espionage uses, among others. See `Docs/THIRD_PARTY.md`.

## 5. Pose editing

- **Drag a green sphere = move**, **Shift+drag = move in screen depth**. Translate arrows (W) and rotate rings (E) are shown together.
- Click / right-click a sphere → numeric tool window: XYZ position/rotation, **position pin / rotation pin**, allowance 0–100, per-axis rotation/translation limits.
- An edit spreads smoothly (quintic falloff) to neighbouring frames; change the range in the tool window or choose **current frame only**.
- Small wrist moves are solved within the arm first; shoulders and torso join progressively. **Only this part (이 부위만 조절)** disables torso participation.
- Undo/redo: Ctrl+Z / Ctrl+Y (or Ctrl+Shift+Z). One drag is one history entry; Esc during a drag cancels it.
- Navigation: type a frame number, ◀▶ single frame, Page Up/Down = previous/next edit, Home/End, Space = play/pause.
- Right-hand **camera views** (left/right/back) detach into windows on double-click.
- Dragging a sphere on an **onion-skin** (previous/next frame mesh) edits only that neighbour frame.
- Inserting/deleting/copying timeline columns retimes notifies, curves and sync markers.
- **Blend layers** bake another animation range in with weight, fade and bone mask.
- **Sockets**: right-click a bone → create socket. Working attachment points; the source Skeleton is not modified.

## 6. Animation set projects

One project asset stores **Group (character) → Motion set → State → Variant**. Each state has loop, root lock and output range settings; every branch keeps its own edits and undo history. ↶/↷ below the tree undo structural changes. Recovery copies are written about every 2 s to `Saved/AnimationBaker/ProjectRecovery`; they are not a substitute for backups.

- The tabs at the top left switch between **Import · Linked work · Content · Blending (가져오기 · 연계 작업 · 콘텐츠 · 블렌딩)**; the set tree stays visible below them.
- **+State / +Set / +Group / +Variant** puts the cursor in the new name field with the name selected, ready to type. Double-click a tree item to rename it.
- A state that has no motion yet and still has its default name (Idle, State) takes its name from the first motion put into it, e.g. MM_Attack_01 → Attack_01. Names already in use are left alone.
- Closing the tab or the editor with an unsaved set asks whether to save.

## 7. Export

**Export all…** chooses the scope (all/group/set/state/variant), outputs (Animation Sequence, per-state Montage, SpriteBaker project), naming pattern (`{project} {group} {motion} {state} {variant}`), in-place or root motion, notify/curve inclusion, montage slot and blend times. Name collisions get a new number; sources are never overwritten.

**SpriteBaker** (separate product): if a SpriteBaker version with AnimationBridge v2 is installed and enabled, a new SpriteBaker project is created and opened. Otherwise a JSON hand-off and asset list are written to `Saved/AnimationBaker/Exports`.

## 8. Compatibility

| Input / target | Status |
|---|---|
| UE Animation Sequence → same skeleton | Bone tracks, frame rate and root motion preserved |
| Manny ↔ Quinn | Automatic retargeting verified |
| Custom humanoid rigs | 17 body parts auto-mapped by name aliases and hierarchy; manual mapping or an existing IK Retargeter otherwise |
| Direct BVH import, non-humanoids | Not supported |

## 9. Limitations (read before buying)

- Video capture is a **single-camera estimate**. Depth, limb twist, occluded and off-screen limbs are estimated and may need manual correction.
- The pose model (MediaPipe BlazePose GHUM 3D) is designed for **one person per image, within about 4 m of the camera, head visible**. Multi-person footage is supported through subject tracking and guides, with lower quality.
- Output is **in-place**. Horizontal travel paths, individual finger/toe joints and arbitrary camera motion are not reconstructed; camera compensation only softens small zoom/pan.
- In in-place output, feet of a travelling or weight-shifting performer may slide; such frames are flagged for review.
- Up to 30 s per analysis, 30 fps output.
- UI language: Korean.

## 10. Troubleshooting

| Symptom | Fix |
|---|---|
| "Prepare the video engine first" | Run 4.1. After switching engine versions, preparing again rebuilds the environment. |
| Package installation failed | Check internet/proxy and `setup.log` in the job folder. Corporate firewalls must allow pypi.org, files.pythonhosted.org, storage.googleapis.com and github.com. |
| FFmpeg integrity check failed | The download was corrupted or blocked; prepare again. For an internal mirror set `ANIMATIONBAKER_FFMPEG_DIR` to a folder containing ffmpeg.exe and ffprobe.exe. |
| Tracking switches person | Redraw the box at that frame to add a tracking guide. |
| Bake fails with a foot target error | Adjust the floor height or disable grounding and inspect the pose. |
| Cache grows | The **video engine** area of the video window shows the size of proxies + media index, analysis caches and job folders. **정리 (Clean)** removes everything except the open video; above the **cache limit** (default 20 GB) the least recently used videos are removed automatically, and job folders older than 7 days are deleted. You can also delete `%LOCALAPPDATA%\DOCKINI\AnimationBaker\VideoEngine-v3\cache` (rebuilt on demand). |
| Move the runtime | Set the environment variable `ANIMATIONBAKER_RUNTIME_ROOT`. |

## 11. Data and privacy

Analysis and editing run locally. The plugin does not transmit video, poses or projects. Network access happens only while preparing the video engine, to download the components above.

## 12. Support

Use the support link on the Fab product page. Include the editor version, logs from `Saved/Logs`, and `status.json` / `error.log` from the video job folder (`Saved/AnimationBaker/Jobs/<ID>`). You do not need to send the video itself.
