# City-Wide Pavement Distress Detection using Aerial Imagery and Deep Learning
## Project Overview
Urban road infrastructure is subject to continuous deterioration due to traffic, environmental conditions, and aging, making timely and scalable monitoring essential. Traditional inspections are labor-intensive and costly, while many automated approaches rely on ground-based or low-altitude imagery, limiting their coverage for city-wide assessments.

This project develops a geospatial framework for **pavement distress detection** using high-resolution (10 cm GSD) aerial imagery over Zurich, Switzerland, combined with **deep learning-based object detection**. Road segments are extracted from aerial orthophotos to create a binary-labeled dataset distinguishing Pavement Distress from Non-Pavement Distress, reducing false positives and enhancing model focus.

Two state-of-the-art object detection models, **YOLOv8x** and **YOLOv11x**, were evaluated. YOLOv11x achieved superior performance, with a mean Average Precision (mAP@0.5) of 38.0% for pavement distress detection. Multi-stage transfer learning and combined training with external low-altitude UAV datasets (HighRPD) were explored but resulted in performance degradation due to domain shift.

The final model was applied to untiled aerial road images across Zurich using the **Slicing Aided Hyper Inference (SAHI)** framework, enabling efficient large-scale detection. This project demonstrates the practicality of using **open-access aerial imagery and deep learning** for automated urban infrastructure monitoring and provides a benchmark for future work in city-wide pavement distress detection.

## Repository Overview
This repository contains all the data, code, and results for detecting pavement distress in aerial imagery of Zurich. The goal of this repository is to ensure full reproducibility of the research. The workflow is divided into four main Python scripts located in the `02_Code/` directory..

---

## Folder Structure

The repository is organized as follows:

```
.
├── 01_Data/
│   ├── 01_Raw/
│   │   ├── groundworks_exports/
│   │   ├── HighRPD/
│   │   ├── Models/
│   │   ├── Zurich_road.gpkg
│   │   ├── Zurich_rail.gpkg
│   │   └── swissimage_urls.csv
│   └── 02_Processed/
│       ├── zurich_roads_buffered_cleaned.gpkg
│       ├── orthophotos/
│       ├── zurich_road_segments_clipped/
│       ├── zurich_tiled_512x512/
│       ├── zurich_yolo_dataset/
│       └── combined_Dataset/
│   ├── annotated_dataset.zip
├── 02_Code/
│   ├── 01_data_preparation.py
│   ├── 02_yolo_dataset_creation.py
│   ├── 03_train_yolo.py
│   ├── 04_inference_sahi.py
│   └── requirements.txt
├── 03_Results/
│   ├── mapped_distress.gpkg
│   └── training_runs/
└── README.md
```

---

## Setup Instructions

### 1. Environment Setup

Create a Python virtual environment and install all the required packages using the `requirements.txt` file.

```bash
pip install -r 02_Code/requirements.txt
```

### 2. Data Placement

Before running any scripts, place the necessary raw data into the `01_Data/01_Raw/` directory:

- **HighRPD Dataset:** Download the dataset from [Mendeley Data](https://data.mendeley.com/datasets/sywswj7djj/1) and place its contents (e.g., `images/`, `labels/` folders) inside `01_Data/01_Raw/HighRPD/`.
- **Groundworks Exports:** Place your exported STAC annotation folders inside `01_Data/01_Raw/groundworks_exports/`.
- **Zurich Geospatial Data:** Place `Zurich_road.gpkg`, `Zurich_rail.gpkg`, and the `swissimage_urls.csv` (should be downloaded from the [swisstopo site](https://www.swisstopo.admin.ch/en/orthoimage-swissimage-10) by filtering for the Zurich municipality.) inside `01_Data/01_Raw/`.
- **Pre-trained Models:** Download the YOLOv8x and YOLOv11x pre-trained weights (`yolov8x.pt`, `yolov11x.pt`) from Ultralytics and place them in `01_Data/01_Raw/Models/`.
- `swissimage_urls.csv` should be downloaded from the swisstopo site by filtering using the municipality "Zurich" and placed inside the folder.

### 3. Using the Pre-prepared Dataset (Optional)

The `annotated_dataset.zip` file contains the final, processed `zurich_yolo_dataset`. If you wish to skip the data preparation steps and go directly to training, unzip this file into the `01_Data/02_Processed/` directory.

---

## Workflow and Scripts

All scripts should be run from the `02_Code/` directory.

### Step 1: Prepare Raw Geospatial Data and Images

This script handles everything from buffering the raw road data and masking out railways to downloading the orthophotos, pruning them, and tiling the selected segments for annotation.

```bash
python 01_data_preparation.py
```

### Step 2: Create the YOLO Dataset

This script processes the STAC exports from your annotation tool, converts the GeoJSON labels to YOLO format, and splits the data into train, validation, and test sets.

```bash
python 02_yolo_dataset_creation.py
```

### Step 3: Train the Models

This script can run any of the four training experiments from the thesis. Use the `--experiment` flag to choose one. The results of each run will be saved in `03_Results/training_runs/`.

- **To run Experiment A (YOLOv8x on Zurich):**

```bash
python 03_train_yolo.py --experiment A
```

- **To run Experiment B (YOLOv11x on Zurich):**

```bash
python 03_train_yolo.py --experiment B
```

- **To run Experiment C (Multi-stage Transfer Learning):**

```bash
python 03_train_yolo.py --experiment C
```

- **To run Experiment D (Combined Dataset with Augmentation):**

```bash
python 03_train_yolo.py --experiment D
```

### Step 4: Run Inference with SAHI

After training, use this script to run inference on the full road segment images using the best model. The script requires the name of the training run folder where the `best.pt` weights are stored.

- **Example using the model from Experiment B:**

```bash
python 04_inference_sahi.py --model_run_name B_YOLOv11x_Zurich
```

The final georeferenced detections will be saved as a GeoPackage file in the `03_Results/mapped_distress.gpkg/` directory.

Here is a sample output from the YOLOv11 inference:

![Detection Example](detection_example-1.png)
**Figure 1:** Detection of pavement distress on aerial imagery. Blue boxes indicate identified distresses. Inset images show zoomed-in views of selected areas.

## Contact

For questions, suggestions, or issues related to this project, please contact jonahesther.s2k@gmail.com

## License

This project is licensed under the [MIT License](LICENSE). 

