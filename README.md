# Road Surface Damage Detection

An end-to-end computer vision project for detecting visible road surface damage from driving footage using classical image processing and geometric reasoning.

This repository has two main notebooks:

- [5_Final.ipynb](./5_Final.ipynb) for directly running the core detection algorithm on a video
- [Road_Damage_Project_Setup_and_Full_Workflow.ipynb](./Road_Damage_Project_Setup_and_Full_Workflow.ipynb) for creating the project folder structure and saving the full staged workflow outputs

The supporting notebooks break the pipeline into smaller parts for experimentation, visualization, and evaluation.

## Project Vision

This project was built to turn raw road video into a readable damage report in motion:

- extract frames from video
- enhance low-contrast road texture
- isolate the road surface region
- segment potential defects
- classify damage patterns
- export an annotated result video
- evaluate masks against ground-truth data

## Pipeline At A Glance

```text
Input Road Video
    |
    v
Frame Extraction
    |
    v
Adaptive Enhancement
    |
    v
Road ROI Segmentation
    |
    +--> Predicted Masks --> Evaluation Against Ground Truth
    |
    v
Damage Candidate Mask
    |
    v
Geometric Damage Classification
    |
    v
Annotated Output Video
```

## Main Notebooks

### 1. Core Algorithm Notebook

[5_Final.ipynb](./5_Final.ipynb) is the compact notebook for directly using the road damage detection algorithm.

It performs:

- video loading
- grayscale conversion
- adaptive contrast enhancement
- edge-preserving denoising
- road-region masking
- dynamic threshold-based segmentation
- contour filtering and rule-based damage classification
- output video generation with colored overlays

Default paths inside the notebook:

- `input_video_path = "input.mp4"`
- `output_video_path = "output.mp4"`

You can update those two variables at the top of the notebook before running it.

### 2. Full Project Workflow Notebook

[Road_Damage_Project_Setup_and_Full_Workflow.ipynb](./Road_Damage_Project_Setup_and_Full_Workflow.ipynb) is the project-structure notebook.

This is the notebook that:

- creates the working folders
- extracts frames from `input_video/video.mp4`
- saves enhanced frames
- saves segmentation masks
- saves detected overlays
- saves final annotated frames
- exports the stage-by-stage videos

If you want the complete project flow with all saved folders and outputs, this is the main notebook to run.

## Damage Categories

The final classifier uses contour geometry and shape descriptors to separate likely defects into:

- `Pothole` shown in red
- `Alligator crack` shown in purple
- `Longitudinal crack` shown in blue

The decision logic uses contour area, aspect ratio, convex hull solidity, circularity, and PCA-based spread ratio to filter noise and assign a class.

## How The Pipeline Works

### 1. Frame Extraction

The video is split into still frames so each processing stage can be inspected independently.

Primary notebook:

- [1_Extract_frames.ipynb](./1_Extract_frames.ipynb)

Generated output:

- `extract_frame/`

### 2. Adaptive Enhancement

The enhancement stage improves road texture visibility under low contrast or low sharpness conditions.

Enhancement logic includes:

- CLAHE for contrast recovery when brightness or variance is weak
- bilateral filtering to suppress noise while preserving defect edges
- mild sharpening when Laplacian-based sharpness is too low

Primary notebook:

- [2_final_enhace_pipeline.ipynb](./2_final_enhace_pipeline.ipynb)

Generated output:

- `enhanced_frames/`
- `enhanced_video.mp4`

### 3. Road Segmentation And Damage Detection

The road surface is isolated using a polygonal ROI focused on the visible roadway. A dynamic threshold is computed from road-region statistics, then a median blur is used to reduce isolated noise.

Detection logic includes:

- region-of-interest masking
- adaptive thresholding from road pixel mean and standard deviation
- contour filtering to remove tiny artifacts
- aspect ratio filtering to reject small round pores or pebbles
- geometry-based class assignment for potholes, alligator cracking, and standard cracks

Primary notebook:

- [3_segmentation_and_detection_pipeline.ipynb](./3_segmentation_and_detection_pipeline.ipynb)

This notebook is mainly for stage-level testing and visualization on sample enhanced frames. The project folders and saved output videos are produced by [Road_Damage_Project_Setup_and_Full_Workflow.ipynb](./Road_Damage_Project_Setup_and_Full_Workflow.ipynb), not by this visualization notebook.

### 4. Quantitative Evaluation

The project also includes a simple evaluation notebook that compares predicted masks against manually prepared ground-truth masks.

Metrics reported:

- Accuracy
- Precision
- Recall
- F1-score
- IoU

Primary notebook:

- [4_data_analysis.ipynb](./4_data_analysis.ipynb)

Required folders:

- `Ground_Truth_Masks/`
- `segmented_frames/`

### 5. Full Multi-Stage Workflow

If you want the larger staged workflow that produces all intermediate frame folders and output videos in one run, use:

- [Road_Damage_Project_Setup_and_Full_Workflow.ipynb](./Road_Damage_Project_Setup_and_Full_Workflow.ipynb)

This notebook uses:

- `input_video/video.mp4` as input
- `extract_frame/`, `enhanced_frames/`, `segmented_frames/`, `detected_frames/`, and `final_frames/` as generated folders
- `Final_Road_Damage_Detection.mp4` as the main final rendered video

## Setup

### Requirements

- Python 3.9 or newer
- Jupyter Notebook or JupyterLab

### Recommended Packages

Install the dependencies used across the notebooks:

```bash
pip install opencv-python numpy matplotlib pandas notebook
```

### Optional Virtual Environment

```bash
python -m venv .venv
.venv\Scripts\activate
pip install opencv-python numpy matplotlib pandas notebook
```

## Quick Start

### Option 1: Run The Core Algorithm Notebook

Use this when you want the shortest path to run the detection algorithm on a single video.

1. Place your input video in the project root.
2. Rename it to `input.mp4` or update `input_video_path` inside [5_Final.ipynb](./5_Final.ipynb).
3. Launch Jupyter:

```bash
jupyter notebook
```

4. Open `5_Final.ipynb`.
5. Run all cells from top to bottom.
6. Collect the rendered output video from the configured `output_video_path`.

### Option 2: Run The Full Project Workflow

Use this when you want every intermediate stage saved for inspection and analysis.

1. Place the source video at `input_video/video.mp4`.
2. Open [Road_Damage_Project_Setup_and_Full_Workflow.ipynb](./Road_Damage_Project_Setup_and_Full_Workflow.ipynb).
3. Run all cells in order.
4. Review intermediate frames and the final exported videos.

## Repository Structure

```text
.
|-- 1_Extract_frames.ipynb
|-- 2_final_enhace_pipeline.ipynb
|-- 3_segmentation_and_detection_pipeline.ipynb
|-- 4_data_analysis.ipynb
|-- 5_Final.ipynb
|-- Road_Damage_Project_Setup_and_Full_Workflow.ipynb
|-- README.md
|-- .gitignore
|-- input_video/                # ignored
|-- extract_frame/              # ignored
|-- enhanced_frames/            # ignored
|-- segmented_frames/           # ignored
|-- detected_frames/            # ignored
|-- final_frames/               # ignored
|-- Ground_Truth_Masks/         # ignored
`-- *.mp4                       # ignored
```

## Output Artifacts

Depending on which notebook you run, the project can generate:

- extracted frame images
- enhanced grayscale frames
- binary segmentation masks
- detected damage overlays
- final annotated output frames
- stage-by-stage preview videos
- final road damage detection video

Typical output files:

- `enhanced_video.mp4`
- `segmented_video.mp4`
- `detected_video.mp4`
- `Final_Road_Damage_Detection.mp4`
- `output.mp4`

## Publishing Notes

This repository is configured to stay lightweight when pushed to GitHub:

- videos are ignored
- generated folders are ignored
- the main published notebooks are `5_Final.ipynb` and `Road_Damage_Project_Setup_and_Full_Workflow.ipynb`
