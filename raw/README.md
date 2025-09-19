# Raw SSBH Dataset

## Instructions

**Extract the original SSBH dataset cuts here.**

### Expected Structure
```
raw/
├── cut1/
│   ├── sentinel2_AREA_b2/     # Blue band (B2) files
│   ├── sentinel2_AREA_b3/     # Green band (B3) files  
│   ├── sentinel2_AREA_b4/     # Red band (B4) files
│   └── _bhr/                  # Building height files
├── cut2/
├── cut3/
├── ...
└── cut67/
```

### How to Set Up

1. **Download** the original SSBH dataset (67 zip files) from the official source
2. **Extract** each `cutX.zip` file into this `raw/` folder
3. **Verify** you have folders `cut1/` through `cut67/` 
4. **Run** `ssbh_preprocessing.ipynb` to process the raw data

### Notes
- Each cut contains multiple 256×256 pixel tiles
- Sentinel-2 bands are stored separately (B2, B3, B4)
- Building height data is in the `_bhr/` subfolder
- Total dataset size: ~67 cuts covering 62 urban centers in China

### Next Step
After extracting all cuts here, run the **preprocessing notebook** to generate the `preprocessed/` folder.