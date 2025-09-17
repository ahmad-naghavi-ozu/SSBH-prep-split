# SSBH Split Dataset

This folder contains the train/validation/test split version of the SSBH (Sentinel-1 Sentinel-2 based Building Height) dataset, organized for machine learning training workflows and building height estimation tasks.

## Overview

The SSBH split dataset provides ready-to-use train/validation/test splits with consistent formatting and balanced distribution across urban areas. This dataset combines Sentinel-2 optical imagery with crowdsourced building height annotations to create a comprehensive resource for building height estimation and segmentation tasks.

**Original Dataset**: SSBH covers 67 urban centers in China with satellite imagery from 2019 Sentinel composites.  
**Split Methodology**: Random shuffling with fixed seed for reproducibility  
**Split Ratios**: 70% Train / 10% Validation / 20% Test

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

**Height Data Processing**: Raw building height rasters from crowdsourced floor count data were processed with smart interpolation to handle invalid values. All NoData, NaN, and negative values were successfully imputed using nearest neighbor or griddata methods, ensuring all final height values are non-negative.

**Building Mask Generation**: Binary building masks were derived using a 0m height threshold. Connected component filtering (minimum 1 pixel) was applied while preserving original spatial characteristics through direct threshold conversion.

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
- **Processing**: Reflectance normalization and percentile contrast stretching (2nd-98th percentile) applied
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

### RGB Data Loading and Normalization
The RGB images are stored in uint16 format (0-65535 range) and should be normalized to [0,1] range for most machine learning applications:
```
normalized_rgb = rgb_data.astype(np.float32) / 65535.0
```

**Important Learning Rate Consideration**: The necessity to normalize the 16-bit RGB imagery to the [0,1] range before inputting it into the deep learning model will most probably impact your typical learning rate. Therefore, you may need a significantly smaller rate compared to standard uint8 RGB datasets in the [0,255] range.

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
- **Geographic scope**: Chinese urban areas (67 cities)
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

- **Data Source**: SSBH dataset covering 67 urban centers in China with 2019 Sentinel-2 composites
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
**Source**: SSBH Dataset (67 urban centers, China, 2019 Sentinel composites)  
**Total Samples**: ~5,606 samples (after validation and quality checks)  
**Processing Pipeline**: RGB enhancement, height interpolation, mask generation, 70-10-20 random split