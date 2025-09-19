# Train/Valid/Test Splits

## Generated Content

**This folder is created by running `ssbh_prep_split.ipynb`**

### Expected Structure After Splitting
```
prep_split/
├── train/                 # Training set (70% of samples)
│   ├── rgb/               # RGB composite images
│   ├── dsm/               # Building height rasters (Digital Surface Model)
│   └── sem/               # Binary building masks (semantic segmentation)
├── valid/                 # Validation set (10% of samples)
│   ├── rgb/
│   ├── dsm/
│   └── sem/
├── test/                  # Testing set (20% of samples)
│   ├── rgb/
│   ├── dsm/
│   └── sem/
└── split_visualization.png
```

### Split Configuration
- **Train:** 70% (~3,924 samples)
- **Valid:** 10% (~560 samples)  
- **Test:** 20% (~1,122 samples)
- **Random seed:** 42 (for reproducibility)

### Data Mapping
The splitting process maps preprocessed modalities to ML-ready names:
- `preprocessed/rgb/` → `*/rgb/` (RGB composite images)
- `preprocessed/height/` → `*/dsm/` (Digital Surface Model - building heights)
- `preprocessed/mask/` → `*/sem/` (Semantic segmentation - building masks)

### File Format
- **RGB:** 3-band uint16 GeoTIFF (normalize by /65535.0 for training)
- **DSM:** Single-band float32 GeoTIFF (height in meters)
- **SEM:** Single-band uint8 GeoTIFF (0: non-building, 1: building)

### Usage
This is your **final ML-ready dataset** for building height estimation and segmentation tasks!