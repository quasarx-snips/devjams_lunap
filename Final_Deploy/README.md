# LunaP Backend

Wires your `main.py` hazard engine (U-Net + morphology) and a CLAHE
enhancement step behind a small Flask API, so the web frontend can call
real Python processing instead of a fake progress bar.

## 1. Install

```bash
cd backend
python -m venv venv
venv\Scripts\activate        # Windows
# source venv/bin/activate   # macOS/Linux
pip install -r requirements.txt
```

## 2. Add your model

Copy `crater_unet.onnx` **and** `crater_unet.onnx.data` (the external-data
sidecar ONNX split off) into `backend/models/`, so you have:

```
backend/models/crater_unet.onnx
backend/models/crater_unet.onnx.data
```

Both files must sit in the same folder — onnxruntime resolves the `.data`
file by relative name next to the `.onnx` file.

Don't want to move the files? Instead set an environment variable pointing
at wherever they already live, e.g. on Windows:

```powershell
$env:CRATER_MODEL_PATH = "C:\Users\bijan\Desktop\DevJams\Module1_2_Integrated\models\crater_unet.onnx"
```

## 3. Run

```bash
python app.py
```

You should see Flask start on `http://localhost:5000`. Sanity-check it:

```bash
curl http://localhost:5000/api/health
```

`"model_found": true` means the ONNX model was located correctly.

## 4. Point the frontend at it

`upload.html` and `analysis.html` both define:

```js
const API_BASE = 'http://localhost:5000';
```

near the top of their `<script>` blocks. Leave it as-is if you're running
everything on one machine; change it if the backend is hosted elsewhere
(and make sure that host allows CORS via `flask-cors`).

## API

| Endpoint            | Method | Body                          | Returns                                  |
|----------------------|--------|--------------------------------|-------------------------------------------|
| `/api/health`        | GET    | —                               | model path + whether it was found         |
| `/api/enhance`       | POST   | `{ "image": "<data URL>" }`    | `{ success, image: "<data URL>" }` (CLAHE) |
| `/api/hazard-map`    | POST   | `{ "image": "<data URL>" }`    | `{ success, image: "<data URL>", output_path }` |

`/api/hazard-map` expects the **CLAHE-enhanced** image (the output of
`/api/enhance`) as its baseline, matching your intended pipeline:
raw upload → CLAHE baseline → hazard map.

## Notes

- Tiling, ONNX inference, and morphology run exactly as written in your
  `main.py` (unmodified) — `ProcessPoolExecutor` with 4 workers, 512px
  tiles, 100px overlap.
- A processed image on a laptop CPU can take anywhere from several seconds
  to a couple of minutes depending on resolution — the frontend shows a
  progress state while it waits.
- Temp baseline files written to `backend/temp/` are deleted after each
  request; final overlays accumulate in `backend/outputs/` (same
  `_integrated_overlay.png` naming your script already uses).
