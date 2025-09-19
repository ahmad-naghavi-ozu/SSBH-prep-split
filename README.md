# SSBH Benchmark Dataset - Preprocessed and Split Version

This repository contains the comprehensively preprocessed and train/validation/test split version of the SSBH (Sentinel-1 Sentinel-2 based Building Height) dataset. The dataset features advanced preprocessing pipelines including smart interpolation, 16-bit RGB enhancement, and quality validation, organized with professional splits for machine learning training workflows and building height estimation tasks.

## Overview

This provides a comprehensively preprocessed and analysis-ready version of the SSBH dataset with professional train/validation/test splits. The main contribution is the sophisticated preprocessing pipeline that transforms raw satellite data into ML-ready formats with enhanced quality and consistent formatting.

**Key Features**:
- **Advanced Preprocessing**: Smart interpolation, 16-bit RGB enhancement, and quality-validated processing
- **Analysis-Ready Format**: All invalid values imputed, consistent data types, optimized storage
- **Professional Splits**: 70% Train / 10% Validation / 20% Test with fixed seed reproducibility
- **Coverage**: 62 urban centers in China with 67 data cuts from 2019 Sentinel-2 composites (~5,606 samples)

## Download

The preprocessed and split dataset is available for download from Google Drive:

**🔗 [Download SSBH Split Dataset](https://drive.google.com/drive/folders/1zRrXZeGMRDpaZcKSCqFtd9nK84BUO9rZ?usp=sharing)**

The download includes all train/validation/test splits with RGB imagery, building height rasters, and binary masks organized in the directory structure described below.

## Dataset Structure

```
_prep_split/
├── train/                 # Training set (70% of samples)
│   ├── rgb/               # RGB composite images (3-band GeoTIFF, uint16)
│   ├── dsm/               # Building height rasters (single-band GeoTIFF, float32)
│   └── sem/               # Binary building masks (single-band GeoTIFF, uint8)
├── valid/                 # Validation set (10% of samples)
│   ├── rgb/               # RGB composite images
│   ├── dsm/               # Building height rasters
│   └── sem/               # Binary building masks
├── test/                  # Test set (20% of samples)
│   ├── rgb/               # RGB composite images
│   ├── dsm/               # Building height rasters
│   └── sem/               # Binary building masks
├── split_visualization.png # Sample visualization from each split (see Visualization section)
└── README.md              # This file
```

### File Naming Convention

All files follow a consistent naming pattern across modalities:
- **Format**: `cut{XX}_{tile_id}.tif`
- **Example**: `cut01_0001.tif`, `cut23_0045.tif`

The same base filename appears across all modalities (rgb/, dsm/, sem/) within each split folder.

### Data Processing Pipeline

The dataset underwent comprehensive preprocessing to create analysis-ready data:

**RGB Processing**: Sentinel-2 optical bands (B4-Red, B3-Green, B2-Blue) were combined into RGB composites. Original reflectance values (0-10000) were normalized to [0,1], enhanced using percentile contrast stretching (2nd-98th percentile), then scaled to uint16 format (0-65535) for efficient storage.

  - **Reflectance Normalization**: Raw satellite data measures the proportion of electromagnetic radiation reflected back from Earth's surface. Sentinel-2 stores these as integers from 0-10,000 (representing 0-100% reflectance), which are converted to the standard 0-1 range by dividing by the scale factor (10,000).

  - **Percentile Stretching**: A contrast enhancement technique that improves image visualization by:
    1. Finding the 2nd percentile (very dark values) and 98th percentile (very bright values) for each RGB channel
    2. Remapping these to become the new 0 and 1 (black and white) points
    3. Removing extreme outliers while enhancing contrast using the full dynamic range
    4. Applied per channel to maintain color balance

**Height Data Processing**: Raw building height rasters from crowdsourced floor count data were processed with smart interpolation to handle invalid values. All NoData, NaN, and negative values were successfully imputed using nearest neighbor or griddata methods, ensuring all final height values are non-negative.

**Building Mask Generation**: Binary building masks were derived using a 0m height threshold. Connected component filtering (minimum 1 pixel) was applied while preserving original spatial characteristics through direct threshold conversion. The 1-pixel minimum is appropriate given that each pixel covers 100m² (10m × 10m) area, representing a substantial building footprint.

## Split Configuration

### Split Ratios
- **Training Set**: 70% of samples (primary dataset for model training)
- **Validation Set**: 10% of samples (hyperparameter tuning and model selection)
- **Test Set**: 20% of samples (final evaluation and performance reporting)

### Splitting Methodology
1. **Data Validation**: All samples validated for completeness across RGB, height, and mask modalities
2. **Random Shuffling**: Samples randomly shuffled with fixed seed (42) for reproducibility
3. **Proportional Splitting**: Samples divided according to specified ratios (70-10-20)
4. **File Organization**: Complete copies of files organized into split directory structure
5. **Quality Verification**: File counts and integrity verified across all splits and modalities

### Quality Assurance
- **Completeness Check**: Only samples with all three modalities (RGB, height, mask) included
- **Balanced Distribution**: Random splitting ensures representative distribution across cuts
- **Reproducibility**: Fixed random seed (42) allows identical splits across runs
- **File Integrity**: shutil.copy2() preserves metadata and timestamps
- **Processing Validation**: All height interpolation successful (100% success rate), no NoData values remain

## Data Specifications

### RGB Imagery (`*/rgb/` folders)
- **Source**: Sentinel-2 bands B4 (red), B3 (green), B2 (blue) from 2019 annual composites
- **Format**: 3-band GeoTIFF
- **Data type**: uint16 (0-65535 range)
- **Spatial resolution**: 10m per pixel
- **Tile size**: 256×256 pixels
- **Processing**: 
  - **Reflectance normalization**: Raw values (0-10000) converted to physical reflectance (0-1) representing 0-100% surface reflection
  - **Percentile contrast stretching**: 2nd-98th percentile enhancement applied per channel to improve contrast while preserving color balance
  - **Dynamic range optimization**: Final scaling to uint16 format for efficient storage
- **Usage**: Convert to [0,1] range for model training: `rgb_normalized = rgb_data.astype(np.float32) / 65535.0`

### Height Data (`*/dsm/` folders)
- **Source**: Building height rasters derived from crowdsourced floor count data
- **Format**: Single-band GeoTIFF
- **Data type**: float32
- **Units**: Meters
- **Spatial resolution**: 10m per pixel
- **Tile size**: 256×256 pixels
- **Data Range**: All values ≥ 0 (no NoData values - all gaps successfully imputed)
- **Processing**: Smart interpolation applied to handle original invalid values, ensuring all final height values are non-negative

### Building Masks (`*/sem/` folders)
- **Source**: Binary masks derived from processed height data using threshold-based segmentation
- **Format**: Single-band GeoTIFF
- **Data type**: uint8 (0: non-building, 1: building)
- **Spatial resolution**: 10m per pixel
- **Tile size**: 256×256 pixels
- **Processing**: 0m height threshold applied with connected component filtering (minimum 1 pixel), preserving original spatial characteristics

## Data Usage Guidelines

### 📢 SSBH Dataset Loading Update

⚠️ **Special handling needed for SSBH RGB files (16-bit TIFF):**

**Primary method (faster):**
```python
rgb = cv2.imread(path, cv2.IMREAD_ANYDEPTH | cv2.IMREAD_COLOR)
rgb = cv2.cvtColor(rgb, cv2.COLOR_BGR2RGB)  # BGR→RGB
if rgb.dtype == np.uint16:
    rgb = rgb.astype(np.float32) / 65535.0  # 16-bit normalization
```

**Fallback:**
```python
rgb = tifffile.imread(path)
if rgb.dtype == np.uint16:
    rgb = rgb.astype(np.float32) / 65535.0
```

✅ **DSM & SEM files:** Load normally with PIL
```python
dsm = np.array(Image.open(dsm_path)).astype(np.float32)
sem = np.array(Image.open(sem_path)).astype(np.uint8)
```

**📝 CV2 Flags:**
- `IMREAD_ANYDEPTH`: Preserves original bit depth (16-bit)
- `IMREAD_COLOR`: Loads in color mode (3 channels)
- `|` operator: Combines both flags (bitwise OR)

**🌍 Note:** SSBH is the first dataset with *true geospatial 16-bit RGB TIFFs* [0,65535], unlike previous datasets that used TIFF format but had standard [0,255] values.

**⚡ Learning Rate Impact:** The necessity to normalize 16-bit RGB imagery to [0,1] range will most probably impact your typical learning rate. You may need a significantly smaller rate compared to standard uint8 RGB datasets in the [0,255] range.

## Split Statistics and Distribution

### Sample Distribution
The dataset is split with careful attention to maintaining representative distribution:

- **Geographic Coverage**: Samples from all 67 cuts represented across splits
- **Urban Diversity**: Random splitting ensures varied urban densities in each split
- **Building Complexity**: Mixed building heights and mask patterns in all splits

### Expected Sample Counts
Based on the complete dataset after preprocessing and validation:
- **Training**: ~3,924 samples (70%)
- **Validation**: ~560 samples (10%)
- **Test**: ~1,122 samples (20%)

*Note: Exact counts depend on the number of valid samples after data quality checks*

### Cut Distribution Analysis
The splitting process analyzes and reports:
- Number of unique cuts represented in each split
- Average samples per cut in each split
- Range of cuts (cut01-cut67) covered

## Training and Evaluation Guidelines

### Recommended Approach
- **Training**: Use train split (70%) for model training
- **Validation**: Use valid split (10%) for hyperparameter tuning and model selection  
- **Testing**: Use test split (20%) for final performance evaluation only

### Evaluation Metrics
**For Height Estimation**: RMSE, MAE (in meters), R² coefficient  
**For Building Segmentation**: IoU, F1 Score, Precision, Recall

### Data Characteristics to Consider
- **Geographic scope**: Chinese urban areas (62 cities, 67 data cuts)
- **Temporal consistency**: 2019 Sentinel-2 composites throughout
- **Height accuracy**: Based on crowdsourced building floor count data
- **Spatial resolution**: 10m per pixel, 256×256 pixel tiles
- **Building focus**: Original height data represents buildings exclusively

## Visualization

### Split Sample Visualization
A comprehensive visualization shows samples from each split with all modalities:

![Split Visualization](split_visualization.png)

The visualization displays:
- 3 samples from each split (train/valid/test)
- All modalities (RGB, Height, Building Mask) for each sample
- Clear visual separation between different samples
- Sample counts for each split labeled

## Reproducibility

### Split Generation
The splitting process ensures reproducibility through:
- **Fixed random seed**: 42 (for consistent shuffling)
- **Deterministic ordering**: Sorted sample names before shuffling
- **Complete documentation**: All parameters and steps recorded

### Regenerating Splits
To regenerate identical splits, use the configuration:
- Train Ratio: 70%, Valid Ratio: 10%, Test Ratio: 20%
- Random Seed: 42 (for reproducibility)
- Processing: Fixed deterministic ordering with sorted sample names before shuffling

## Storage and Performance

### Disk Usage
- **Total size**: ~2x preprocessed dataset size (files are copied, not moved)
- **Per-sample size**: ~1.5MB (RGB: ~1MB, Height: ~256KB, Mask: ~64KB)
- **Training set**: ~5.9GB (assuming 3,924 samples)
- **Validation set**: ~840MB (assuming 560 samples)
- **Test set**: ~1.7GB (assuming 1,122 samples)

### Loading Performance
- **Recommended batch size**: 16-32 samples for 8GB GPU
- **Data loading workers**: 4-8 workers for optimal I/O performance
- **Memory considerations**: Consider memory mapping for very large training runs

## Important Processing Notes

### Key Technical Considerations
- **RGB Data**: Stored in uint16 format, requires normalization to [0,1] for training. May require smaller learning rates than standard uint8 data.
- **Reflectance Values**: Represent true physical surface reflectance (0-100%) derived from Sentinel-2's calibrated measurements, providing more meaningful spectral information than simple pixel intensities.
- **Contrast Enhancement**: Percentile stretching (2nd-98th percentile) enhances visual contrast while avoiding saturation from extreme outliers, making imagery more suitable for both visualization and model training.
- **Height Data**: All values are non-negative (≥0m) after successful interpolation of original invalid values.
- **Building Masks**: Direct threshold conversion (0m) with no morphological operations, preserving original spatial characteristics.
- **Spatial Context**: Each 256×256 pixel tile covers 2.56km × 2.56km at 10m resolution, suitable for building-level analysis.

### Known Limitations
- Height accuracy depends on crowdsourced floor count data quality
- Geographic scope limited to Chinese urban areas
- 10m pixel size limits detection of very small buildings
- RGB-only spectral information may limit some applications

## Citation

If you use this split dataset, please cite the original SSBH paper:

```bibtex
@inproceedings{ma2025ssbh,
  title={SSBH: a remote sensing building height dataset for deep learning},
  author={Ma, Yao and Wang, Hao and Cao, Changhao},
  booktitle={Sixth International Conference on Geoscience and Remote Sensing Mapping (GRSM 2024)},
  volume={13506},
  pages={388--394},
  year={2025},
  organization={SPIE}
}
```

## Processing History

- **Data Source**: SSBH dataset covering 62 urban centers in China (67 data cuts) with 2019 Sentinel-2 composites
- **Preprocessing**: RGB enhancement, height interpolation (100% success rate), and binary mask generation
- **Splitting**: Random 70-10-20 split with seed 42 for reproducibility
- **Version**: v1.0 (2025)

## Contact and Support

For questions about the dataset:
- Check the comprehensive data loading examples provided above
- Verify data integrity using the provided loading and visualization functions
- Report issues with specific sample IDs and detailed error descriptions

---

**Split Version**: v1.0  
**Split Date**: 2025  
**Source**: SSBH Dataset (62 urban centers, 67 data cuts, China, 2019 Sentinel composites)  
**Total Samples**: ~5,606 samples (after validation and quality checks)  
**Processing Pipeline**: RGB enhancement, height interpolation, mask generation, 70-10-20 random split