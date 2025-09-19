# Preprocessed SSBH Dataset

## Generated Content

**This folder is created by running `ssbh_preprocessing.ipynb`**

### Expected Structure After Processing
```
preprocessed/
├── rgb/               # RGB composite images (uint16 GeoTIFF)
├── height/            # Building height rasters (float32 GeoTIFF)
└── mask/              # Binary building masks (uint8 GeoTIFF)
```

### Processing Details

**RGB Processing:**
- Combines Sentinel-2 bands B4(Red), B3(Green), B2(Blue)
- Reflectance normalization (0-10000 → 0-1)
- Percentile contrast stretching (2nd-98th percentile)
- Stored as 16-bit GeoTIFF (0-65535)

**Height Processing:**
- Smart interpolation for invalid values
- All negative/NoData values handled
- Final values ≥ 0 meters

**Mask Generation:**
- Binary building masks from height data
- 0m threshold with connected component filtering
- Direct threshold conversion (no morphological operations)

### File Naming
All files follow the pattern: `cut{XX}_{tile_id}.tif`

### Next Step
After this folder is populated, run the **splitting notebook** (`ssbh_prep_split.ipynb`) to create train/valid/test splits.