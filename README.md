# Road Surface Damage Detection

End-to-end computer vision pipeline that detects and classifies road surface damage (potholes, alligator cracks, and longitudinal cracks) from driving video and produces annotated outputs for evaluation.

This project processes road video frame by frame, enhances weak road texture, isolates likely damage regions, classifies defects into interpretable categories, and produces annotated visual outputs for inspection and evaluation.

Current benchmark snapshot from `4_data_analysis.ipynb`:

- Ground-truth frames found: `7`
- Evaluated class instances: `14`
- Macro Accuracy: `0.9683`
- Macro Precision: `0.3035`
- Macro Recall: `0.4787`
- Macro F1: `0.3336`
- Macro IoU: `0.2303`

## Table Of Contents

- [Project Summary](#project-summary)
- [Google Drive Assets](#google-drive-assets)
- [Pipeline Overview](#pipeline-overview)
- [Notebook Guide](#notebook-guide)
- [Detailed Methodology](#detailed-methodology)
- [Evaluation Method](#evaluation-method)
- [Latest Results](#latest-results)
- [Setup](#setup)
- [How To Run](#how-to-run)
- [Expected Project Structure](#expected-project-structure)
- [Generated Outputs](#generated-outputs)
- [Publishing Notes](#publishing-notes)
- [Current Limitations](#current-limitations)

## Project Summary

The system is designed to convert raw driving footage into a structured visual analysis of road defects. It focuses on three practical damage categories:

- `Pothole`
- `Alligator crack`
- `Longitudinal crack`

The project is built primarily with:

- OpenCV for image and video processing
- NumPy for numeric operations
- Pandas for evaluation summaries
- Matplotlib for notebook visualization
- Jupyter notebooks for staged experimentation and reproducibility

Main project capabilities:

- extract road video into frame sequences
- enhance low-contrast or low-sharpness road surfaces
- restrict analysis to a road-shaped region of interest
- segment dark defect-like regions using dynamic thresholding
- filter noise using contour size and shape logic
- classify visible damage by geometric rules
- generate overlay-based detection videos
- compare color-coded detections with ground-truth masks

## Google Drive Assets

Large assets are intentionally kept out of GitHub and shared separately through Google Drive:

[Google Drive assets folder](https://drive.google.com/drive/folders/14sU1XjRcyDfUeCKwZO1fmA1a6-ZzktDk?usp=sharing)

That shared folder is the intended place for large or generated files such as:

- input videos
- extracted frames
- enhanced frames
- segmented frames
- detected frames
- final annotated frames
- ground-truth masks
- zipped dataset bundles
- exported stage videos
- final rendered videos

If you clone this repository and do not see those folders or videos inside Git, that is expected. The repository is intentionally lightweight, and those artifacts are meant to live locally or in the shared Google Drive folder.

## Pipeline Overview

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
    v
Damage Candidate Mask
    |
    v
Geometric Damage Classification
    |
    +--> Color-Coded Detection Frames --> Evaluation Against Ground Truth
    |
    v
Annotated Output Video
```

High-level processing stages:

1. Read a road video and extract still frames.
2. Convert frames to grayscale and enhance weak road texture.
3. Restrict processing to the visible road region using a polygonal ROI.
4. Compute a dynamic threshold from road-region intensity statistics.
5. Generate binary candidate masks for possible damage.
6. Filter and classify contours into road-damage classes.
7. Produce videos and frame folders for every major stage.
8. Evaluate color-coded detections against manually prepared masks.

## Notebook Guide

### `5_Final.ipynb`

This is the compact notebook for running the core detector directly on a video with minimal setup.

It includes:

- adaptive enhancement
- road ROI segmentation
- dynamic thresholding
- contour-based classification
- output video writing

Default paths:

- `input_video_path = "input.mp4"`
- `output_video_path = "output.mp4"`

Use this notebook when you want the shortest end-to-end path from a single input video to a final annotated output video.

### `Road_Damage_Project_Setup_and_Full_Workflow.ipynb`

This is the most complete notebook in the repository. It creates and uses the staged project folders, processes the video through every pipeline step, and saves intermediate artifacts for inspection.

It handles:

- folder creation
- frame extraction
- grayscale enhancement
- segmentation mask generation
- detection overlay generation
- final annotation generation on original frames
- export of stage-by-stage videos

Generated folders used by this workflow:

- `extract_frame/`
- `enhanced_frames/`
- `segmented_frames/`
- `detected_frames/`
- `final_frames/`

Generated videos used by this workflow:

- `enhanced_video.mp4`
- `segmented_video.mp4`
- `detected_video.mp4`
- `Final_Road_Damage_Detection.mp4`

### `1_Extract_frames.ipynb`

This notebook extracts all frames from:

- `input_video/video.mp4`

and saves them to:

- `extract_frame/`

It is useful if you want to inspect or reuse the raw frames independently of the full pipeline.

### `2_final_enhace_pipeline.ipynb`

This notebook focuses on the enhancement stage and visually compares enhancement quality on sample frames.

It reports simple image-quality indicators before and after enhancement, including:

- mean intensity
- standard deviation
- Laplacian sharpness

It also plots grayscale histograms to help inspect contrast changes.

### `3_segmentation_and_detection_pipeline.ipynb`

This notebook is mainly for stage-level visualization and debugging. It tests segmentation and classification on a few selected enhanced frames and displays:

- the input frame
- the generated binary mask
- the filtered classification result

Use it when you want to validate the quality of segmentation and classification decisions before running the full workflow.

### `4_data_analysis.ipynb`

This notebook performs the quantitative evaluation. It compares the color-coded detection overlays against manually prepared ground-truth masks and reports per-class and macro-average metrics.

## Detailed Methodology

### 1. Frame Extraction

The video extraction stage reads the road video frame by frame and writes each frame to disk using a zero-padded naming pattern:

- `frame_00000.jpg`
- `frame_00001.jpg`
- `frame_00002.jpg`

Why this matters:

- every later notebook can operate deterministically on the same frame names
- matching between predictions and ground truth becomes easier
- debugging becomes easier because every stage can be inspected visually

### 2. Adaptive Enhancement

The enhancement stage works on grayscale frames and tries to improve damaged-road visibility without strongly distorting structure.

The logic used in the core pipeline is:

- compute mean intensity of the grayscale frame
- compute intensity standard deviation
- compute image sharpness using Laplacian variance
- if the frame is too dark or too flat, apply CLAHE
- apply bilateral filtering to preserve edges while reducing noise
- if the frame is too blurry, apply mild sharpening

Enhancement conditions:

- apply CLAHE when `mean < 90` or `std < 40`
- use CLAHE parameters `clipLimit=2.0` and `tileGridSize=(8, 8)`
- apply sharpening when `sharpness < 50`

Enhancement details:

- `5_Final.ipynb` uses a resolution-aware bilateral kernel size based on frame width
- `2_final_enhace_pipeline.ipynb` uses a fixed bilateral filter configuration for stage testing
- sharpening is performed with `cv2.addWeighted(enhanced, 1.5, blur, -0.5, 0)`

Why this stage is important:

- cracks are often low-contrast against asphalt
- weak lighting can suppress texture variation
- denoising before thresholding helps prevent random false positives

### 3. Road ROI Segmentation

The segmentation stage does not process the entire image equally. Instead, it focuses on a polygonal road-shaped region of interest that covers the lower and central portion of the frame where the road is expected to appear.

The ROI polygon is defined relative to frame size:

- left bottom: `(0, h)`
- left mid: `(0, 0.5h)`
- upper-left road point: `(0.1w, 0.3h)`
- upper-right road point: `(0.9w, 0.3h)`
- right mid: `(w, 0.5h)`
- right bottom: `(w, h)`

After masking the frame to the ROI:

- compute mean `mu` of road pixels
- compute standard deviation `sigma` of road pixels
- compute a dynamic multiplier as `clip(sigma / 25.0, 0.8, 1.8)`
- compute the threshold as `mu - dynamic_multiplier * sigma`
- clamp the threshold into the safe range `[10, 180]`
- threshold with `cv2.THRESH_BINARY_INV`
- apply a resolution-aware median blur to remove isolated noise

The median blur kernel is based on frame width and forced to remain odd, with a minimum size of `3`.

Why this works:

- darker-than-road regions are likely to contain cracks or holes
- road brightness varies across videos, so a fixed threshold is less stable
- ROI masking prevents sky, vehicles, and roadside structures from dominating statistics

### 4. Damage Candidate Filtering

Once the binary candidate mask is produced, contours are extracted and filtered to remove obvious false positives.

Filtering logic includes:

- remove microscopic contours below `total_pixels * 0.00003`
- compute minimum-area rectangle for each contour
- reject zero-width or zero-height rectangles
- compute aspect ratio as `max(width, height) / min(width, height)`
- reject small rounded regions when `area < total_pixels * 0.0003` and `aspect_ratio < 2.5`

This small-round-shape filter is important because asphalt pores, pebbles, and minor texture noise can otherwise be mistaken for real damage.

### 5. Geometric Damage Classification

The final classification is rule-based and uses contour geometry rather than a learned model.

Features used:

- contour area
- convex hull area
- solidity
- convex-hull circularity
- PCA-based spread ratio
- aspect ratio

Class rules used in the workflow:

- classify as `Pothole` when `area > total_pixels * 0.0015`, `h_circ > 0.50`, and `solidity > 0.40`
- classify as `Alligator crack` when `area > total_pixels * 0.001`, `spread_ratio > 0.10`, and `solidity < 0.65`
- otherwise classify as `Longitudinal crack`

Color mapping:

- `Pothole` is drawn in red as `(0, 0, 255)`
- `Alligator crack` is drawn in purple as `(255, 0, 255)`
- `Longitudinal crack` is drawn in blue as `(255, 0, 0)`

Visualization style:

- a filled contour is drawn on an overlay image
- contour boundaries are drawn on a detection image
- both are blended with `cv2.addWeighted(overlay, 0.35, detection, 0.65, 0)`

The full workflow saves two visually useful outputs:

- `detected_frames/` contains the color-coded detection overlays
- `final_frames/` contains the final visualization rendered on original frames

## Evaluation Method

The evaluation notebook does not compare raw binary segmentation masks from `segmented_frames/`. Instead, it compares class-specific color-coded masks reconstructed from:

- `Ground_Truth_Masks/`
- `detected_frames/`

Evaluation workflow:

1. Collect ground-truth `.png` files from `Ground_Truth_Masks/`.
2. Collect predicted `.jpg` or `.png` files from `detected_frames/`.
3. Match files by extracting the frame number from the filename.
4. Read both images in color.
5. Reconstruct class-specific binary masks from the color overlays.
6. Dilate predicted masks with a `15 x 15` kernel.
7. Compute per-class TP, TN, FP, and FN.
8. Skip class and frame pairs where both masks are empty.
9. Report averages by class and macro averages across classes.

Color-mask extraction logic:

- `Alligator` is detected by high red and blue relative to green, with red and blue remaining reasonably similar
- `Pothole` is detected by strong red dominance
- `Crack` is detected by strong blue dominance

Metrics reported:

- Accuracy
- Precision
- Recall
- F1-score
- IoU

Why dilation is used:

- line thickness in hand-labeled masks and predicted overlays may differ
- a small tolerance helps reduce unfair penalties caused only by drawing thickness

Important note:

- the notebook currently uses the label `Logititude Crack` in code and output
- this README keeps that same label in the results table so the documentation matches the notebook output exactly

## Latest Results

Latest per-class average performance from `4_data_analysis.ipynb`:

| Class | Accuracy | Precision | Recall | F1 | IoU |
| --- | ---: | ---: | ---: | ---: | ---: |
| Alligator | 0.9881 | 0.2847 | 0.2938 | 0.2803 | 0.2186 |
| Logititude Crack | 0.9342 | 0.1199 | 0.5659 | 0.1921 | 0.1114 |
| Pothole | 0.9827 | 0.5058 | 0.5763 | 0.5285 | 0.3609 |

Overall system macro average:

| Accuracy | Precision | Recall | F1 | IoU |
| ---: | ---: | ---: | ---: | ---: |
| 0.9683 | 0.3035 | 0.4787 | 0.3336 | 0.2303 |

Interpretation at a glance:

- `Pothole` is currently the strongest class in this evaluation snapshot
- `Alligator` detection is conservative and shows lower recall
- `Logititude Crack` has moderate recall but low precision, suggesting more false positives
- overall accuracy is high, but class-level overlap metrics show room for improvement

## Setup

### Requirements

- Python 3.9 or newer
- Jupyter Notebook or JupyterLab

### Install Dependencies

```bash
pip install opencv-python numpy matplotlib pandas notebook
```

### Optional Virtual Environment

```bash
python -m venv .venv
.venv\Scripts\activate
pip install opencv-python numpy matplotlib pandas notebook
```

## How To Run

### Option 1: Run The Compact End-To-End Notebook

Use this option when you want the simplest direct run.

1. Place your source video in the project root.
2. Rename it to `input.mp4`, or update `input_video_path` inside `5_Final.ipynb`.
3. Start Jupyter with:

```bash
jupyter notebook
```

4. Open `5_Final.ipynb`.
5. Run all cells from top to bottom.
6. Collect the final annotated video from the configured `output_video_path`.

### Option 2: Run The Full Multi-Stage Workflow

Use this option when you want all intermediate folders and videos.

1. Place the source video at `input_video/video.mp4`.
2. Open `Road_Damage_Project_Setup_and_Full_Workflow.ipynb`.
3. Run all cells in order.
4. Inspect the generated folders:

- `extract_frame/`
- `enhanced_frames/`
- `segmented_frames/`
- `detected_frames/`
- `final_frames/`

5. Inspect the generated videos:

- `enhanced_video.mp4`
- `segmented_video.mp4`
- `detected_video.mp4`
- `Final_Road_Damage_Detection.mp4`

### Option 3: Run Individual Stage Notebooks

Use this option when you want to debug or inspect a specific stage.

- run `1_Extract_frames.ipynb` for frame extraction only
- run `2_final_enhace_pipeline.ipynb` to inspect enhancement quality and histograms
- run `3_segmentation_and_detection_pipeline.ipynb` to visualize masks and classifications on sample frames

### Option 4: Run Quantitative Evaluation

Use this option when you already have:

- labeled masks inside `Ground_Truth_Masks/`
- predicted detection overlays inside `detected_frames/`

Then:

1. Open `4_data_analysis.ipynb`.
2. Run the notebook from top to bottom.
3. Review the printed per-class and macro-average results.

### Option 5: Download Shared Assets

Use this option when you need the large files that are intentionally not stored in GitHub.

1. Open the shared [Google Drive assets folder](https://drive.google.com/drive/folders/14sU1XjRcyDfUeCKwZO1fmA1a6-ZzktDk?usp=sharing).
2. Download the required frames, videos, masks, or zipped archives.
3. Place them back into the same project-root folder structure expected by the notebooks.

## Expected Project Structure

Tracked in Git:

```text
.
|-- 1_Extract_frames.ipynb
|-- 2_final_enhace_pipeline.ipynb
|-- 3_segmentation_and_detection_pipeline.ipynb
|-- 4_data_analysis.ipynb
|-- 5_Final.ipynb
|-- Road_Damage_Project_Setup_and_Full_Workflow.ipynb
|-- README.md
`-- .gitignore
```

Generated locally or downloaded from Google Drive:

```text
.
|-- input_video/
|   `-- video.mp4
|-- extract_frame/
|-- enhanced_frames/
|-- segmented_frames/
|-- detected_frames/
|-- final_frames/
|-- Ground_Truth_Masks/
|-- enhanced_video.mp4
|-- segmented_video.mp4
|-- detected_video.mp4
|-- Final_Road_Damage_Detection.mp4
`-- Ground_Truth_Masks.zip
```

## Generated Outputs

Depending on which notebook you run, the project may generate:

- raw extracted frame images
- enhanced grayscale road frames
- binary damage candidate masks
- color-coded detection overlays
- final annotated frames on original imagery
- stage-specific videos
- final project output video

Typical output folders:

- `extract_frame/`
- `enhanced_frames/`
- `segmented_frames/`
- `detected_frames/`
- `final_frames/`

Typical output videos:

- `enhanced_video.mp4`
- `segmented_video.mp4`
- `detected_video.mp4`
- `Final_Road_Damage_Detection.mp4`
- `output.mp4`

## Publishing Notes

This repository is intentionally lightweight for GitHub.

The `.gitignore` is configured so that:

- all root-level folders are ignored
- root-level videos are ignored
- root-level archives are ignored
- generated project artifacts are not committed
- notebooks and documentation remain publishable

In practice, this means:

- the repository tracks the notebooks and documentation
- generated frames and videos stay local or in Google Drive
- heavy artifacts do not bloat the Git history

## Current Limitations

This project is intentionally classical and interpretable, but that also creates limitations:

- classification is rule-based rather than learned from a large labeled dataset
- camera angle assumptions are built into the ROI shape
- lighting, shadows, lane markings, and road stains can still cause false positives
- some fine cracks may be under-segmented or fragmented
- the current evaluation sample is still small
- class naming in evaluation contains the typo `Logititude Crack`

Potential future improvements:

- replace rule-based classification with a trained detector or segmenter
- make ROI estimation adaptive to camera pose
- calibrate thresholds across more road and lighting conditions
- expand the ground-truth dataset for stronger evaluation
- correct the class label typo in `4_data_analysis.ipynb`
