# Dermoscopy Feature Extraction

Complete pipeline for extracting ABCDE clinical features + PyRadiomics features from dermoscopy lesion images (ISIC 2017 dataset).

---

## Project Structure
```
dermoscopy_features/
├── feature_extraction.py          # Main extraction script
├── requirements.txt               # Python dependencies
├── README.md                      # This file
├── data/
│   ├── masked_images/             # Input: original x binary mask images
│   │   ├── train/                 # 2000 images
│   │   ├── val/                   # 150 images
│   │   └── test/                  # 600 images
│   └── predicted_masks_clean/     # Input: binary segmentation masks
│       ├── train/
│       ├── val/
│       └── test/
├── outputs/
│   ├── features.csv               # All 2750 images, 234 features
│   ├── features_train.csv         # Train split only (2000 rows)
│   ├── features_val.csv           # Val split only (150 rows)
│   └── features_test.csv          # Test split only (600 rows)
└── venv310/                       # Python 3.10 virtual environment
```

---

## Features Extracted (234 total columns)

| Group | Prefix | Count | Description |
|-------|--------|-------|-------------|
| Asymmetry | asym_ | 5 | Axis-aligned, PCA, compactness, elongation |
| Border | border_ | 6 | Irregularity, fractal dim, convexity, solidity |
| Color | color_, col_ | 48 | RGB/HSV/LAB stats + ABCDE 6-colour fractions |
| Texture | glcm_, lbp_, gabor_ | 23 | GLCM x4, LBP, 12 Gabor filters |
| Evolution | lesion_, extent... | 6 | Size ratios, bounding box, eccentricity |
| Histogram | hist_ | 16 | Energy, entropy, percentiles, IQR, MAD |
| Shape | shape_, hu_, fourier_ | 24 | Sphericity, Hu moments, Fourier descriptors |
| GLRLM | glrlm_ | 7 | SRE, LRE, GLN, RLN, RP, LGRE, HGRE |
| PyRadiomics | rx_ | 88 | firstorder(18), shape2D(9), glcm(24), glrlm(16), glszm(16), ngtdm(5) |
| Identity | image_id, split | 2 | Image name and train/val/test label |

---

## Environment

- Python: 3.10.20 (inside venv310/)
- Key packages: numpy 1.26.4, opencv-python-headless 4.8.1, SimpleITK 2.5.3, pyradiomics 3.0.1

---

## START OF EVERY CODESPACE SESSION

Open the terminal and run:
```bash
# Step 1 - activate the virtual environment (auto if bashrc was set up)
source /workspaces/radiomics/dermoscopy_features/venv310/bin/activate

# Step 2 - confirm everything works
python -c "from radiomics import featureextractor; import cv2; print('All ready')"

# Step 3 - go to project folder
cd /workspaces/radiomics/dermoscopy_features
```

If bashrc was set up correctly you will see this automatically on terminal open:
```
venv310 activated
```

---

## END OF EVERY CODESPACE SESSION

Run these before closing:
```bash
# Step 1 - save command history
cd /workspaces/radiomics/dermoscopy_features
history > outputs/terminal_history_$(date +%Y%m%d).txt

# Step 2 - commit and push everything to GitHub
git add -A
git commit -m "Session $(date +%Y-%m-%d): save work"
git push origin main

# Step 3 - confirm push worked
git log --oneline -3

# Step 4 - download CSVs to your computer via VS Code Explorer right-click > Download
#   outputs/features.csv
#   outputs/features_train.csv
#   outputs/features_val.csv
#   outputs/features_test.csv
```

---

## Running the Feature Extraction
```bash
source /workspaces/radiomics/dermoscopy_features/venv310/bin/activate

python feature_extraction.py \
  --masked_root data/masked_images \
  --mask_root   data/predicted_masks_clean \
  --output      outputs/features.csv \
  --max_dim     512 \
  --workers     4
```

---

## Verifying the Output
```bash
python -c "
import pandas as pd
df = pd.read_csv('outputs/features.csv')
print(f'Rows    : {len(df)}')
print(f'Columns : {len(df.columns)}')
print(f'Splits  : {df[\"split\"].value_counts().to_dict()}')
rx = [c for c in df.columns if c.startswith(\"rx_\")]
print(f'rx_ cols: {len(rx)}')
print(f'Missing : {df.isnull().sum().sum()}')
"
```

Expected:
```
Rows    : 2750
Columns : 234
Splits  : {'train': 2000, 'test': 600, 'val': 150}
rx_ cols: 88
Missing : 0
```

---

## If Codespace Gets Deleted (30+ days inactive)
```bash
# Install Python 3.10
sudo add-apt-repository ppa:deadsnakes/ppa -y
sudo apt-get update -y --allow-insecure-repositories
sudo apt-get install -y python3.10 python3.10-dev python3.10-venv python3.10-distutils
sudo apt-get install -y libgl1 libglib2.0-0

# Create venv and install packages IN THIS EXACT ORDER
cd /workspaces/radiomics/dermoscopy_features
python3.10 -m venv venv310
source venv310/bin/activate
pip install --upgrade pip setuptools wheel
pip install numpy==1.26.4
pip install SimpleITK
pip install pyradiomics==3.0.1 --no-build-isolation
pip install opencv-python-headless==4.8.1.78 tqdm matplotlib pandas scipy

# Make venv auto-activate on terminal open
echo 'source /workspaces/radiomics/dermoscopy_features/venv310/bin/activate' >> ~/.bashrc

# Verify
python -c "
from radiomics import featureextractor
import cv2, numpy, SimpleITK as sitk
print('numpy    :', numpy.__version__)
print('cv2      :', cv2.__version__)
print('SimpleITK:', sitk.__version__)
print('All ready')
"
```

---

## Re-downloading Data (if needed)
```bash
pip install gdown

# Masked images
mkdir -p data/masked_images/{train,val,test}
cd data/masked_images
gdown 1T_TlE5BY27j5uu7OL_bTzQovBcFTaPVJ -O test/test.zip
gdown 1c6RMof-xb4P1dRxCWuX8GSfb_PYzQO9X -O train/train.zip
gdown 1lKbUvDul-17WS8OvQW50P-sgO2GlA76T -O val/val.zip
unzip test/test.zip -d test/
unzip train/train.zip -d train/
unzip val/val.zip -d val/
rm test/test.zip train/train.zip val/val.zip

# Clean masks
mkdir -p /workspaces/radiomics/dermoscopy_features/data/predicted_masks_clean/{train,val,test}
cd /workspaces/radiomics/dermoscopy_features/data/predicted_masks_clean
gdown 1oZz2hfx_zXhLdwMCC_IlBhzygcUeaiW1 -O test/test.zip
gdown 1KW-hGFUCe4auIfdV6dgRQkJLM0lBMEI3 -O train/train.zip
gdown 1IOybjxrDCTCGtCSIJ1IUTCMmQ5EbUqEi -O val/val.zip
unzip test/test.zip -d test/
unzip train/train.zip -d train/
unzip val/val.zip -d val/
rm test/test.zip train/train.zip val/val.zip
```

---

## Important Notes

- PyRadiomics requires Python 3.10 — does NOT work on Python 3.12
- numpy must be pinned to 1.26.4 — pyradiomics compiled against numpy 1.x
- opencv must be headless version — no display needed
- Always use --workers 1 when debugging
- data/ folder is NOT committed to git (too large) — only CSVs and scripts
