# Ego Pipeline

### From raw egocentric capture to structured, time-aligned MCAP episodes

A data-preparation and validation pipeline for multi-camera human activity recordings, combining first-person video, motion signals, hand tracks, language annotations, and episode metadata into a portable dataset format.

<p>
  <a href="https://github.com/Harjotsingh0311/ego-pipeline"><img alt="Repository" src="https://img.shields.io/badge/Repository-ego--pipeline-181717?style=for-the-badge&logo=github&logoColor=white"></a>
  <img alt="Python" src="https://img.shields.io/badge/Python-3.13-3776AB?style=for-the-badge&logo=python&logoColor=white">
  <img alt="MCAP" src="https://img.shields.io/badge/Output-MCAP-635BFF?style=for-the-badge">
  <img alt="Workflow" src="https://img.shields.io/badge/Workflow-Jupyter%20Notebooks-F37626?style=for-the-badge&logo=jupyter&logoColor=white">
</p>

---

## Overview

**Ego Pipeline** transforms fragmented egocentric recordings into episode-level data packages designed for robotics, embodied AI, human activity understanding, and multimodal learning workflows.

The dataset brings together synchronized views from a head-mounted camera, left-wrist camera, right-wrist camera, and an external camera, alongside available IMU measurements, hand-pose tracks, natural-language task labels, calibration records, and machine-readable metadata.

The primary output is **one MCAP file per episode**, with video and structured signals organized into named channels and timestamped on a common timeline. Reports and validation artifacts accompany the deliverables so that data quality, known gaps, and limitations remain inspectable.

## What the pipeline delivers

<table>
<tr>
<td width="50%" valign="top">

### 🎥 Multi-view video

- Head, left-wrist, right-wrist, and external-camera streams
- H.264 video access units in MCAP video channels
- Per-frame timestamps and consistent channel naming
- Video integrity, frame continuity, and timing checks

</td>
<td width="50%" valign="top">

### ⏱️ Time alignment

- Cross-camera timeline alignment
- Episode boundaries and task-level segmentation
- Frame-level time associations for supported signals
- Timing, overlap, and synchronization diagnostics

</td>
</tr>
<tr>
<td width="50%" valign="top">

### 🖐️ Hand and motion signals

- Left- and right-hand pose tracks
- 2D image-normalized hand coordinates
- IMU streams for the three body-mounted devices
- Explicit handling of invalid or anomalous IMU samples

</td>
<td width="50%" valign="top">

### 💬 Language and episode metadata

- Human-readable task descriptions and temporal labels
- Episode identity, timing, camera roles, and source provenance
- Structured metadata stored with each episode
- Calibration records that distinguish known values from unavailable data

</td>
</tr>
</table>

## Dataset at a glance

| Component | Description |
|---|---|
| Recording views | Head, left wrist, right wrist, external camera |
| Body-camera target rate | 30 FPS |
| Main container | MCAP |
| Video representation | H.264 access units using `foxglove.CompressedVideo` |
| Motion data | JSON IMU samples for the three body-mounted devices |
| Hand representation | Left/right hand-joint tracks in 2D normalized image coordinates |
| Language | Task instructions and time-bounded labels |
| Metadata | Episode timing, task information, camera map, tags, coverage, and provenance |
| Quality artifacts | Audit reports, coverage summaries, hypotheses/checks, manifests, hashes, and validation outputs |

## MCAP structure

Each final episode is packaged as a self-contained `.mcap` file. The MCAP uses named channels for video, IMU, hand pose, and calibration records, plus an `episode_metadata` record.

| Channel family | Contents |
|---|---|
| `/cameras/cam_high` | Head / chest-level first-person video |
| `/cameras/cam_left_wrist` | Left-wrist video |
| `/cameras/cam_right_wrist` | Right-wrist video |
| `/cameras/cam_ext` | External-camera video |
| `/ego/imu/<camera>` | IMU samples associated with each body-mounted device |
| `/ego/hand/joints/left/position` | Left-hand joint track |
| `/ego/hand/joints/right/position` | Right-hand joint track |
| `/ego/calibration/intrinsics` | Camera intrinsic-calibration record and status |
| `/ego/calibration/extrinsics` | Camera extrinsic-calibration record and status |
| `episode_metadata` | Episode identity, timestamps, task, cameras, labels, coverage, and provenance |

Video is represented as `foxglove.CompressedVideo`; structured signals use JSON schemas. Timestamps are kept on a common episode timeline, with monotonicity checks applied during final packaging.

## Quality, traceability & validation

The project includes data-quality checks and report artifacts for:

- **Input integrity:** source inventory, SHA-256 checksums, duplicate identification, and verified copies.
- **Video quality:** stream properties, frame counts, decoding problems, freeze/black-frame checks, and bitrate summaries.
- **IMU quality:** schema and sampling checks, timing irregularities, repeated/zero rows, and orientation-related diagnostics.
- **Synchronization:** cross-device offsets, timeline alignment, task boundaries, and timestamp consistency.
- **Hand tracking:** left/right track coverage, detection/tracking state, review sheets, and continuity-aware track assignment.
- **Language labels:** non-empty task labels, span coverage, and label review outputs.
- **Final delivery:** MCAP channel/schema checks, message and timestamp checks, metadata validation, manifests, checksums, and a documented deviation register.

The final validator is intended to assess the delivered files independently against the target vendor format. Use its generated reports as the source of truth for the status of a particular batch.

## Important data notes

- **Hand pose is 2D image-normalized.** It is not metric 3D pose and should not be interpreted as world-space geometry.
- **Camera calibration is unavailable in the current capture.** Intrinsic and extrinsic values are explicitly represented as unavailable/null with status information; no calibration values are fabricated.
- **IMU values are raw and uncalibrated.** Units are recorded as assumptions where they have not been independently confirmed.
- **Known deviations remain visible.** Review the vendor specification and validation/deviation reports before using the dataset for tasks that require metric pose or calibrated multi-camera geometry.

## Repository layout

```text
ego-pipeline/
├── configs/       # pipeline and episode configuration
├── notebooks/     # end-to-end processing, audit, assembly, and validation notebooks
├── code/          # supporting code, where applicable
├── pyproject.toml # project dependencies and metadata
├── uv.lock        # locked dependency resolution
└── .python-version
```

Runtime data and generated outputs are organized separately from tracked source files. Depending on the stage and local run, these data directories include:

```text
raw/       # frozen source inputs
interim/   # intermediate arrays, timestamps, and processing artifacts
episodes/  # segmented episode assets and per-frame tables
mcap/      # MCAP builds and final episode files
calib/     # calibration records and inputs
reports/   # quality reports, manifests, validation results, and review artifacts
```

Large raw and derived data files are not intended to be committed to Git.

## Notebook index

| Notebook | Purpose |
|---|---|
| `01_setup_and_inventory.ipynb` | Environment, input inventory, file integrity, and provenance |
| `02_imu_audit.ipynb`, `02b_imu_extra_checks.ipynb` | IMU quality, timing, anomalies, and device-level diagnostics |
| `03_video_audit.ipynb` | Video stream and frame-integrity audit |
| `04_pairing_and_clock.ipynb` | Clip pairing and timeline alignment |
| `05_task_segmentation.ipynb` | Task/episode boundary definitions |
| `06_timestamps_and_metadata.ipynb` | Episode timestamps and metadata |
| `07_video_preparation*.ipynb` | Video normalization and H.264 preparation |
| `08*` notebooks | MCAP assembly, external-camera synchronization, and IMU integration |
| `09*` notebooks | Hand-pose generation, hand-channel integration, and track assignment |
| `10_language_labels.ipynb` | Language labels and temporal spans |
| `11_calibration.ipynb` | Calibration records and availability/deviation reporting |
| `12_final_mcap*.ipynb` | Final episode MCAP assembly |
| `13_validator.ipynb` | Independent delivery validation and review artifacts |

Exact filenames may include working-copy suffixes in the current repository; the table groups related notebooks by their role.

## Getting started

### Requirements

- Windows 11 or a compatible Python environment
- Python 3.13
- [`uv`](https://docs.astral.sh/uv/)
- `ffmpeg` and `ffprobe` on `PATH`
- VS Code with the Python and Jupyter extensions

### Set up the environment

```powershell
git clone https://github.com/Harjotsingh0311/ego-pipeline.git
cd ego-pipeline
uv sync
```

Open the repository in VS Code and select the project's `.venv` Python interpreter as the notebook kernel. The notebooks expect the project configuration and input data to be available at the paths described in `configs/config.yaml` and the relevant notebook documentation.

> **Data required:** raw recordings and generated artifacts are excluded from version control. Cloning the repository alone does not include the capture dataset or final MCAP files.

## Intended use

The resulting episodes are intended to support downstream workflows such as:

- Robotics and embodied-AI data preparation
- Egocentric action and task understanding
- Multimodal video, motion, pose, and language experiments
- Dataset QA, synchronization research, and annotation review
- MCAP-compatible inspection and integration tooling

## License & data access

No license or public dataset-access terms are declared here. Confirm the repository owner’s terms and any applicable participant, capture, or vendor restrictions before redistributing source recordings or derived data.

---

<div align="center">

**Ego Pipeline** · Structured episodes for multimodal, first-person data

</div>
