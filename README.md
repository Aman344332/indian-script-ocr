# 🇮🇳 Indian OCR — Full Program

AI-powered text extraction for **all 22 official Indian languages**.  
Supports handwritten and printed images.  
4 ways to use: **Web App · REST API · CLI · Batch Processor**

---

## Languages Supported (All 22)

| Language | Native | Script | EasyOCR | Tesseract |
|----------|--------|--------|---------|-----------|
| Hindi | हिन्दी | Devanagari | ✓ | ✓ |
| Bengali | বাংলা | Bengali | ✓ | ✓ |
| Tamil | தமிழ் | Tamil | ✓ | ✓ |
| Telugu | తెలుగు | Telugu | ✓ | ✓ |
| Kannada | ಕನ್ನಡ | Kannada | ✓ | ✓ |
| Malayalam | മലയാളം | Malayalam | ✓ | ✓ |
| Punjabi | ਪੰਜਾਬੀ | Gurmukhi | ✓ | ✓ |
| Gujarati | ગુજરાતી | Gujarati | ✓ | ✓ |
| Odia | ଓଡ଼ିଆ | Odia | — | ✓ |
| Marathi | मराठी | Devanagari | ✓ | ✓ |
| Urdu | اردو | Perso-Arabic RTL | ✓ | ✓ |
| Assamese | অসমীয়া | Bengali | — | ✓ |
| Maithili | मैथिली | Devanagari | — | ✓ |
| Sanskrit | संस्कृत | Devanagari | — | ✓ |
| Nepali | नेपाली | Devanagari | ✓ | ✓ |
| Dogri | डोगरी | Devanagari | — | ✓ |
| Konkani | कोंकणी | Devanagari | — | ✓ |
| Bodo | বোড়ো | Devanagari | — | ✓ |
| Sindhi | سنڌي | Perso-Arabic RTL | — | ✓ |
| Kashmiri | كشميري | Perso-Arabic RTL | — | ✓ |
| Manipuri | মণিপুরী | Meitei Mayek | — | ✓ |
| Santali | ᱥᱟᱱᱛᱟᱲᱤ | Ol Chiki | — | ✓ |

---

## Install

```bash
# 1. Clone / download this project
cd indian-ocr

# 2. Create virtual environment
python -m venv ocr_env
source ocr_env/bin/activate        # Mac/Linux
ocr_env\Scripts\activate           # Windows

# 3. Install Python libraries
pip install -r requirements.txt

# 4. Install Tesseract (system package)
# Ubuntu:  sudo apt-get install tesseract-ocr tesseract-ocr-hin tesseract-ocr-tam ...
# Mac:     brew install tesseract tesseract-lang
# Windows: download from github.com/UB-Mannheim/tesseract/wiki

# 5. Run setup check
python setup.py
```

---

## Usage — 4 Ways

### 1. Command Line (CLI)

```bash
# Single image — Hindi with EasyOCR (default)
python cli/ocr.py photo.jpg

# Choose language and backend
python cli/ocr.py scan.png --language tamil --backend tesseract

# Save result to file
python cli/ocr.py image.jpg --language marathi --output result.txt

# Batch folder — process all images, save CSV
python cli/ocr.py scans/ --batch --language hindi --output results.csv

# Try ALL supported languages on one image
python cli/ocr.py image.jpg --all-languages

# List all 22 languages with their properties
python cli/ocr.py --list-languages
```

### 2. REST API

```bash
# Start the API server
uvicorn api.server:app --reload --port 8000

# Open interactive docs
open http://localhost:8000/docs
```

```python
import requests

# Single image OCR
with open("image.jpg", "rb") as f:
    res = requests.post("http://localhost:8000/ocr",
        files={"file": f},
        data={"language": "hindi", "backend": "easyocr"})
print(res.json())

# Batch ZIP upload
with open("images.zip", "rb") as f:
    res = requests.post("http://localhost:8000/ocr/batch",
        files={"file": f},
        data={"language": "tamil", "backend": "tesseract"})
job_id = res.json()["job_id"]

# Poll for results
import time
while True:
    status = requests.get(f"http://localhost:8000/ocr/status/{job_id}").json()
    print(f"{status['done']}/{status['total']} done")
    if status["status"] in ("done", "error"):
        break
    time.sleep(2)
```

### 3. Web App (React)

```bash
# One-time React setup
cd web
npx create-react-app . --template minimal
cp App.jsx src/App.jsx
npm install axios

# Start web app (make sure API is running first)
npm start
# Opens http://localhost:3000
```

Features: drag-and-drop upload, language selector, backend picker,
batch ZIP processing, OCR history, copy-to-clipboard.

### 4. Python Code (Direct)

```python
from src.ocr_engine import OCREngine

# ── EasyOCR (recommended) ──────────────────────────────────────
engine = OCREngine(language="hindi", backend="easyocr")
result = engine.predict("image.jpg")
print(result.text)
print(f"Confidence: {result.confidence:.1f}%")

# ── Tesseract (fast, no GPU) ───────────────────────────────────
engine = OCREngine(language="tamil", backend="tesseract")
result = engine.predict("tamil_doc.png")

# ── Batch folder ───────────────────────────────────────────────
results = engine.predict_folder(
    folder="my_images/",
    save_csv="output/results.csv"
)

# ── From numpy array (e.g. from OpenCV) ───────────────────────
import cv2
img = cv2.imread("image.jpg")
result = engine.predict_array(img)
```

---

## Train Your Own Custom Model

When EasyOCR/Tesseract accuracy isn't good enough for your specific
images, train a custom CRNN model on your own dataset.

```bash
# 1. Put your images in:
#    data/processed/train/images/  + labels.csv (filename, text)
#    data/processed/val/images/    + labels.csv
#    data/processed/test/images/   + labels.csv

# 2. Build vocabulary
python -c "
from src.vocabulary import Vocabulary
v = Vocabulary()
v.build(['data/processed/train/labels.csv',
         'data/processed/val/labels.csv'], min_freq=3)
v.save('data/vocab/vocab.json')
"

# 3. Start training (edit CFG dict in trainer.py first)
python train/trainer.py

# 4. Use trained model
engine = OCREngine(
    language='hindi',
    backend='custom',
    model_path='models/checkpoints/best_model.pth',
    vocab_path='data/vocab/vocab.json'
)
result = engine.predict('image.jpg')
```

---

## Project Structure

```
indian-ocr/
├── src/
│   ├── languages.py      ← all 22 language configs
│   ├── ocr_engine.py     ← EasyOCR + Tesseract + custom model
│   ├── preprocess.py     ← image cleaning pipeline
│   ├── vocabulary.py     ← char ↔ integer mapping
│   ├── model.py          ← CRNN neural network
│   └── dataset.py        ← PyTorch data loader
├── api/
│   └── server.py         ← FastAPI REST API
├── cli/
│   └── ocr.py            ← command line tool
├── web/
│   └── App.jsx           ← React web application
├── train/
│   └── trainer.py        ← training loop
├── data/                 ← your images go here
├── models/               ← trained model checkpoints saved here
├── results/              ← batch results and plots saved here
├── requirements.txt
├── setup.py              ← run this first
└── README.md
```

---

## API Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/` | Health check |
| GET | `/health` | API status |
| GET | `/languages` | All 22 languages info |
| POST | `/ocr` | Single image OCR |
| POST | `/ocr/batch` | Batch ZIP upload |
| GET | `/ocr/status/{id}` | Batch job status |