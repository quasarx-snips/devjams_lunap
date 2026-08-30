# LunaP

LunaP is a lunar terrain analysis webapp for identifying hazards, comparing
landing sites, and planning risk-aware traversal routes. The frontend is a
static HTML experience, while a local Flask backend runs CLAHE enhancement,
ONNX crater segmentation, OpenCV morphology, landing-site scoring, A* route
planning, and 3D terrain generation.

## Features

- Upload and preview a lunar image.
- Enhance the image with CLAHE local-contrast processing.
- Generate an integrated hazard overlay using the crater U-Net model plus
  crater, boulder, dark-shadow, and steep-slope detectors.
- Score up to five landing sites using flatness, smoothness, clearance, usable
  area, and hazard fraction.
- Drop start and goal pins and generate Distance Priority, Balanced, and Max
  Safety routes.
- View the selected routes projected onto a Plotly 3D terrain surface.
- Store the raw and enhanced images in IndexedDB so pages can share large
  images without relying on localStorage limits.

## Project Layout

```text
FinalDeploy/
├── landing_page.html       # Visual entry page
├── upload.html             # Image selection and CLAHE handoff
├── analysis.html           # Hazard, landing, route, and 3D dashboard
├── repo.html               # Project/repository page
├── team.html               # Team page
├── app.py                  # Flask API server
├── main.py                 # ONNX + tiled hazard-map engine
├── terrain_module.py       # Landing, routing, and 3D calculations
├── storage.js              # IndexedDB wrapper used by the frontend
├── requirements.txt        # Python dependencies
├── models/                 # ONNX model files (Presently stored in Kaggle)
├── outputs/                # Generated overlay PNG and risk-map files
└── temp/                   # Short-lived backend input images
```

The notebook and PDF in the repository contain the original exploration and
prototype material used to develop the terrain modules.

## Requirements

- Windows, macOS, or Linux
- Python 3.10 or newer
- A browser with IndexedDB, JavaScript, and Canvas support
- `crater_unet.onnx` and its external-data sidecar, if the model uses one

## Installation

From the project root:

```powershell
python -m venv .venv
.\.venv\Scripts\Activate.ps1
pip install -r requirements.txt
```

On macOS or Linux:

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

## Model Setup

By default, `app.py` looks for:

```text
models/crater_unet.onnx
```

If the ONNX model uses external data, keep the sidecar beside it:

```text
models/crater_unet.onnx
models/crater_unet.onnx.data
```

To use a model in another location, set `CRATER_MODEL_PATH` before starting
Flask.

```powershell
$env:CRATER_MODEL_PATH = "C:\path\to\crater_unet.onnx"
```

Check the model configuration with:

```powershell
Invoke-WebRequest http://localhost:5000/api/health
```

The response should contain `"status": "ok"` and `"model_found": true`.

## Running The App

Use two terminals from the project root.

Terminal 1, Flask API:

```powershell
.\.venv\Scripts\Activate.ps1
python app.py
```

The API runs at `http://localhost:5000`.

Terminal 2, static frontend server:

```powershell
python -m http.server 8000
```

Open:

```text
http://localhost:8000/landing_page.html
```

Serving the pages over HTTP is recommended. The frontend is configured to
call `http://localhost:5000`, and the backend enables `flask-cors` for this
separate local origin. Opening the HTML directly with `file://` can produce
browser-specific CORS behavior. Also kindly download the models from [Kaggle](https://huggingface.co/Dumbface/Lunar_Crater_Detector)

## User Flow

1. Open `landing_page.html` and continue to the upload page.
2. Select a lunar image in `upload.html`.
3. Click `Process image` to call `/api/enhance`. The raw image and enhanced
   baseline are saved in IndexedDB under `lunapImage` and
   `lunapEnhancedImage`.
4. Continue to `analysis.html` and click `Generate Hazard Map`.
5. Use `Find Landing Sites` to display ranked site markers and scores.
6. Click the baseline image once for a start pin and once for a goal pin,
   then click `Plan Routes`.
7. Review the route overlays and the Plotly 3D terrain view.

Landing-site, route, and 3D requests send the current enhanced baseline to
the backend. If the matching hazard map is not cached, the backend computes
it automatically. The cache is in memory and is intended for one local user;
restarting Flask clears it.

## Backend API

All endpoints below are served from `http://localhost:5000`.

| Endpoint | Method | Request body | Purpose |
|---|---|---|---|
| `/api/health` | GET | None | Reports server status and model availability. |
| `/api/enhance` | POST | `{ "image": "<data URL>" }` | Returns a CLAHE-enhanced PNG data URL. |
| `/api/hazard-map` | POST | `{ "image": "<data URL>" }` | Runs the hazard engine and returns an overlay data URL and output path. |
| `/api/landing-sites` | POST | `{ "image": "<data URL>" }` | Returns ranked landing-site records and the hazard image. |
| `/api/route` | POST | `{ "image": "<data URL>", "start": {"x": 0, "y": 0}, "goal": {"x": 1, "y": 1} }` | Returns three risk-aware paths. |
| `/api/terrain3d` | POST | `{ "image": "<data URL>" }` | Returns a downsampled terrain mesh and cached routes/pins. |

Directly entering `/api/landing-sites` in the browser uses GET, but the route
only accepts POST, so a `405 Method Not Allowed` response is expected. Use the
frontend or a POST-capable client to call it.

## Processing Details

`main.py` processes images in 512-pixel tiles with 100-pixel overlap and uses
up to four CPU worker processes for ONNX inference. The generated overlay is
written to `outputs/` along with a matching `_risk.npy` numeric risk map.

`terrain_module.py` reuses that exact risk map. Its route planner max-pools
large maps before A* so the three strategy searches remain responsive, then
maps the resulting paths back to the original image coordinates.

## Troubleshooting

### `Failed to fetch`

- Confirm Flask is running with `python app.py`.
- Confirm `http://localhost:5000/api/health` returns successfully.
- Open the frontend from `http://localhost:8000`, not a stale tab or an
  unrelated static-server directory.
- Restart Flask after changing `app.py`; the development server runs with
  the reloader disabled.
- Check the Flask terminal for model, ONNX, or processing errors.

### Model not found

Place the model in `models/` or set `CRATER_MODEL_PATH`. If the model uses
external data, the `.onnx.data` file must remain beside the `.onnx` file.

### Landing sites or routes say to run the hazard map first

Make sure the request includes the current enhanced image. The supplied
frontend does this automatically. A request without an image requires an
existing in-memory hazard result.

### Browser shows a CORS error

Keep `flask-cors` installed and restart Flask. Also check that the frontend
uses the same API base configured in `upload.html` and `analysis.html`:

```js
const API_BASE = 'http://localhost:5000';
```

### Old routes or 404 responses after editing

Stop duplicate Flask processes and start the backend from this project root.
At startup, `app.py` prints its registered API routes; verify that the route
you need appears there.

## Generated Files And Privacy

Generated overlays and risk maps accumulate in `outputs/`. Temporary decoded
baseline files in `temp/` are removed after processing. Images stored in the
browser's IndexedDB remain until the site data is cleared or the application
removes them.

The current backend uses a single in-memory `LAST_RESULT` cache and has no
authentication, database, user isolation, or production deployment hardening.
It is intended for local demonstration and development.
