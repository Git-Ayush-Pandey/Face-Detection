# Face Authenticator

> Lightweight face authentication demo using OpenCV DNN for face detection and LBPH face recognizer for identification.

---

## Project overview

This repository implements a simple face capture → train → recognize workflow:

- Uses OpenCV's DNN (Caffe `deploy.prototxt` + `res10_300x300_...caffemodel`) for face detection.
- Uses OpenCV's LBPH face recognizer (`cv2.face.LBPHFaceRecognizer_create()`) for training and prediction.
- A small command-line interface (`main.py`) provides operations to record faces for a username, train the recognizer, verify users (live webcam), and delete users.

This project is intended as a compact, educational demo — suitable for prototyping, classroom demos, or small local projects.

---

## Repository structure (important files/folders)

```
face_authenticator/
├─ main.py                 # CLI entrypoint (add user / train / verify / remove)
├─ requirement.txt         # Python dependencies (install with pip)
├─ data/                   # User folders — one folder per user containing face images
│  ├─ alice/
│  └─ bob/
├─ models/                 # Trained models + detector files
│  ├─ deploy.prototxt      # Face detector model definition (Caffe)
│  ├─ res10_300x300_...    # Pretrained Caffe weights used by the detector
│  ├─ face_model.yml       # Saved LBPH recognizer model
│  └─ labels.json          # Mapping from label id -> username
├─ penv/                   # (Included in archive) virtualenv site-packages — ignore and recreate locally
```

> NOTE: The archive includes a `penv/` virtual environment folder. **Do not** use that directly. Create a fresh virtual environment locally and install dependencies from `requirement.txt`.

---

## Requirements

- Python 3.8+ (tested on 3.10/3.11+)
- OpenCV (both `opencv-python` and `opencv-contrib-python` are referenced)
- `numpy`, `tqdm`, `imutils`

The minimal `requirement.txt` in the repo lists:

```
opencv-python
opencv-contrib-python
numpy
tqdm
imutils
```

---

## Setup (quick start)

1. Create and activate a fresh virtual environment (example using `venv`):

```bash
python -m venv venv
# mac / linux
source venv/bin/activate
# windows (powershell)
.\\venv\\Scripts\\Activate.ps1
```

2. Install dependencies:

```bash
pip install --upgrade pip
pip install -r requirement.txt
```

3. Run the CLI:

```bash
python main.py
```

`main.py` will show a small menu with options to add a user (capture faces from webcam), build/train the model, verify a user (live recognition), remove a user, and exit.

---

## Typical workflow / Usage

1. **Record faces for a new user**
   - Choose the "Add user" option in `main.py`.
   - Provide a username (folder will be created under `data/<username>`).
   - The script captures frames from your webcam, detects faces, saves preprocessed grayscale face crops (150×150) to the user folder.

2. **Train the recognizer**
   - Choose the "Train model" option.
   - The script scans `data/` for subfolders (each subfolder is treated as one identity), loads images, preprocesses them (resizing + histogram equalization), trains an LBPH recognizer, writes `models/face_model.yml`, and saves `models/labels.json` with the mapping.

3. **Verify / Recognize**
   - Choose the "Verify user" option to start webcam recognition.
   - A DNN face detector finds faces; the LBPH recognizer predicts an ID and gives a confidence score. The CLI overlays results on the live frame.

4. **Remove user**
   - Deletes the corresponding folder in `data/` and optionally removes the model; re-train after removal.

---

## How it works (brief technical notes)

- **Face detection:** The system uses a pretrained Caffe SSD face detector (`deploy.prototxt` + `.caffemodel`) to find face bounding boxes in camera frames. This is robust and reasonably fast.

- **Preprocessing:** Detected faces are converted to grayscale, resized to 150×150, and histogram-equalized (`cv2.equalizeHist`) before saving/training. Consistent preprocessing improves LBPH performance.

- **Recognition:** The Local Binary Patterns Histogram (LBPH) algorithm (OpenCV) is used for lightweight face recognition. It works well for small datasets and is fast on CPU.

- **Labels:** A `labels.json` file maps numeric label IDs used by the recognizer to the actual usernames.

---

## Re-training and updating

- Whenever you add or remove users (change `data/`), re-run the Train option to rebuild `models/face_model.yml` and update `models/labels.json`.
- Keep a backup of `models/face_model.yml` if you want to roll back.

---

## Troubleshooting & tips

- **Black/blank webcam frames / cannot access camera:** Ensure no other application is using the webcam. On Linux, permissions or wrong camera index can block access. Try changing the `cv2.VideoCapture(0)` index to 1 or 2.

- **Poor recognition accuracy:** Get more varied samples per user (different angles, lighting). Ensure faces are well-centered and not too small.

- **Missing dependencies / import errors:** Use a clean venv and `pip install -r requirement.txt`. Avoid using the archived `penv/` folder — it's platform-specific and may break on other systems.

- **Large `penv/` in repo:** The repository contains a full virtualenv (`penv/`) which bloats the archive. Delete it locally or add `penv/` to `.gitignore` for future commits.

---

## Security & Privacy notes

- This repository saves captured face images locally under `data/`. Treat these images as sensitive personal data; do not upload them to public servers or repos without consent.
- LBPH models can be spoofed more easily than deep-learning-based models; do not rely on this for high-security use-cases.

---

## Possible improvements / Next steps

- Replace LBPH with a modern embedding-based model (FaceNet, ArcFace) + classifier for much higher accuracy.
- Add CLI flags or a small GUI for easier operation.
- Implement dataset augmentation and a proper train/validation split.
- Add unit tests and CI, remove `penv/` from the repository, and add a `requirements.txt` (already present as `requirement.txt`) and a `LICENSE` file.

---

## License & authors

This README doesn't assume a license file exists in the archive. If you want a license (MIT, Apache-2.0, etc.), add a `LICENSE` file.

---

## Contact

If you want, I can:
- generate a minimal `requirements.txt` (already present but I can expand pinned versions),
- produce a `run.sh` script to automate venv creation & dependency installation,
- or convert `main.py` options into CLI arguments for easier scripting.

Tell me which of the above you'd like next.

