# LivePortrait Emotion API — Setup Guide

Complete setup instructions for running LivePortrait facial expression generation on a Vast.ai GPU server.

---

## 1. Provisioning the Server on Vast.ai

### Choose the right template

Select the **PyTorch (Vast)** template — it comes with PyTorch, CUDA, and Python pre-installed, so you skip the longest part of dependency setup.

### GPU and instance settings

When you click the play button on the template, configure these before launching:

| Setting | Value |
|---|---|
| GPU | RTX 3090 / RTX 4090 / A100 (16GB+ VRAM minimum) |
| Disk space | 50GB minimum |
| CUDA version | 12.1 or 12.9 — both work fine |
| Jupyter | ON |
| SSH | ON |

> Avoid GPUs under 16GB VRAM — LivePortrait will run out of memory during inference.

---

## 2. First Steps After the Instance Starts

### Connect via SSH

Vast.ai gives you an SSH command in the dashboard. It looks like:

```bash
ssh -p PORT root@SERVER_IP -L 8080:localhost:8080
```

The `-L 8080:localhost:8080` flag forwards Jupyter to your local browser at `http://localhost:8080`.

---

## 3. Server Setup — Run in Order

### Step 1 — Check your GPU

```bash
nvidia-smi
```

You should see `RTX 4090` or `A100` with `CUDA Version: 12.x`. If this fails, the instance has no GPU — contact Vast.ai support.

### Step 2 — Check Python and PyTorch

```bash
python --version
# Should be 3.10.x or 3.11.x

python -c "import torch; print('PyTorch:', torch.__version__); print('CUDA:', torch.cuda.is_available())"
# Should print: PyTorch: 2.x.x   CUDA: True
```

If CUDA shows `False`, run `nvidia-smi` again — sometimes the driver needs a moment after instance start.

### Step 3 — Install system dependencies

```bash
apt-get update -y && apt-get install -y \
    git \
    ffmpeg \
    libgl1 \
    libglib2.0-0 \
    wget \
    curl
```

### Step 4 — Clone LivePortrait

```bash
cd ~
git clone https://github.com/KwaiVGI/LivePortrait
cd LivePortrait
ls
# You should see: src/  assets/  inference.py  requirements.txt  app.py  etc.
```

### Step 5 — Install Python dependencies

```bash
pip install -r requirements.txt
```

This takes 2–4 minutes. Then verify the key packages installed correctly:

```bash
python -c "
import cv2
import numpy
import torch
import onnxruntime
print('cv2:', cv2.__version__)
print('numpy:', numpy.__version__)
print('torch:', torch.__version__)
print('onnxruntime:', onnxruntime.__version__)
print('All good!')
"
```

### Step 6 — Download model weights

```bash
pip install huggingface_hub

python -c "
from huggingface_hub import snapshot_download
print('Downloading LivePortrait weights (~2GB)...')
snapshot_download(
    repo_id='KwaiVGI/LivePortrait',
    local_dir='./pretrained_weights',
    ignore_patterns=['*.git*', 'README*', 'docs*']
)
print('Done!')
"
```

This takes 3–5 minutes. When finished, verify:

```bash
ls pretrained_weights/liveportrait/
# Should show: base_models/  retargeting_models/
```

### Step 7 — Fix ONNX Runtime for CUDA 12+

The default `onnxruntime-gpu` expects CUDA 11 libraries. Replace it:

```bash
pip uninstall onnxruntime onnxruntime-gpu -y
pip install onnxruntime==1.18.0

python -c "
import onnxruntime as ort
print('ORT version:', ort.__version__)
print('Providers:', ort.get_available_providers())
"
```

> **Note:** If your server runs CUDA 13.x (check with `nvcc --version`), install the CPU-only version — `onnxruntime==1.18.0` without the `-gpu` suffix. The main LivePortrait inference still runs on GPU via PyTorch. Only face detection uses ONNX Runtime, and it runs fast on CPU.

### Step 8 — Upload your face photo

From your **local machine**, open a new terminal and run:

```bash
scp -P PORT your_photo.png root@SERVER_IP:~/LivePortrait/assets/my_photo.png
```

Photo requirements for best results:
- Front-facing, looking at camera
- Face takes up 40–70% of the frame
- Good even lighting, no harsh shadows
- At least 512×512 pixels
- JPG or PNG format
- No sunglasses, no heavy side profile, no multiple faces

### Step 9 — Test LivePortrait with a sample

```bash
cd ~/LivePortrait

python inference.py \
  -s assets/examples/source/s6.jpg \
  -d assets/examples/driving/d0.mp4 \
  --output_dir ./output/test/
```

Should produce `output/test/s6--d0_concat.mp4` in 10–20 seconds. If you see `Done!` with no errors, everything is working.

---

## 4. Running the Emotion API

### Install Flask

```bash
pip install flask
```

### Start the API

```bash
cd ~/LivePortrait
python emotion_api.py
```

Wait for the ready message (~40 sec model load, one time only):

```
Ready in 42.3s
==================================================
API running — available emotions:
  neutral / happy / sad / angry / surprised
  fearful / disgusted / excited / calm
==================================================
```

### Keep it running after you close SSH

```bash
apt-get install -y screen
screen -S api

cd ~/LivePortrait
python emotion_api.py

# Detach (keeps running): Ctrl+A then D
# Reattach later:         screen -r api
```

---

## 5. API Endpoints

| Method | Endpoint | Description |
|---|---|---|
| GET | `/health` | Server status + available emotions |
| GET | `/emotions` | List emotion names |
| GET | `/movements` | List head movement patterns |
| POST | `/generate` | Returns MP4 video |
| POST | `/generate/image` | Returns JPEG image (faster) |
| POST | `/generate/movement` | Emotion + head movement combined |

### Example requests

```bash
# Health check
curl http://SERVER_IP:5000/health

# Generate happy video
curl -X POST http://SERVER_IP:5000/generate \
     -H "Content-Type: application/json" \
     -d '{"emotion": "happy"}' \
     --output happy.mp4

# Generate sad video, 5 seconds
curl -X POST http://SERVER_IP:5000/generate \
     -H "Content-Type: application/json" \
     -d '{"emotion": "sad", "seconds": 5}' \
     --output sad.mp4

# Single image (faster, ~500ms)
curl -X POST http://SERVER_IP:5000/generate/image \
     -H "Content-Type: application/json" \
     -d '{"emotion": "surprised"}' \
     --output surprised.jpg

# Happy face nodding yes
curl -X POST http://SERVER_IP:5000/generate/movement \
     -H "Content-Type: application/json" \
     -d '{"emotion": "happy", "movement": "yes"}' \
     --output happy_nodding.mp4
```

### Request parameters

**POST /generate**
```json
{
  "emotion":   "happy",
  "seconds":   3,
  "intensity": 1.0
}
```

**POST /generate/movement**
```json
{
  "emotion":   "happy",
  "movement":  "yes",
  "seconds":   3,
  "loops":     2,
  "intensity": 1.0
}
```

---

## 6. Available Emotions and Movements

### Emotions
| Name | Description |
|---|---|
| neutral | Resting face |
| happy | Smile, raised cheeks |
| sad | Frown, head slightly down |
| angry | Brows down, narrowed eyes |
| surprised | Wide eyes, open mouth |
| fearful | Raised brows, slight head turn |
| disgusted | Nose scrunch, lip raised |
| excited | Wide smile, wide eyes |
| calm | Relaxed, slight smile |

### Head movements
| Name | Description |
|---|---|
| yes | Nods down twice — agreement |
| no | Shakes left-right — disagreement |
| thinking | Looks up and to the right |
| shy | Looks down and away |
| confident | Chin up, slight turn |
| lookround | Scans left then right |

---

## 7. Expected Performance

```
Server startup (one time):      ~40 sec
Single image (/generate/image): ~500ms
3-second video (/generate):     ~8–15 sec
10-second video:                ~20–30 sec
All 9 emotions (batch):         ~5–8 sec
```

---

## 8. Troubleshooting

**CUDA not available after start**
Run `nvidia-smi` — wait 30 seconds and try again. GPU driver sometimes needs a moment.

**ONNX Runtime libcublasLt error**
Follow Step 7 above to replace onnxruntime with the CPU version.

**"ModuleNotFoundError: src"**
Always `cd ~/LivePortrait` before running any script.

**Face crops incorrectly**
Your photo has the face too small or off-centre. Crop manually to ~512×512 with the face centred before uploading.

**Expressions look weak**
Increase intensity: `"intensity": 1.3` in your request body.

**Out of memory error**
Your GPU has less than 16GB VRAM. Upgrade to RTX 3090 / 4090 / A100 on Vast.ai.

---

## 9. VS Code SSH Setup (optional)

Add this to your VS Code SSH config (`Ctrl+Shift+P` → `Remote-SSH: Open SSH Configuration File`):

```
Host vastai-liveportrait
    HostName SERVER_IP
    User root
    Port PORT
    LocalForward 8080 localhost:8080
    LocalForward 5000 localhost:5000
```

Then install the **Claude Code** extension in VS Code and select **"Install in SSH: SERVER_IP"** to run it directly on the remote server.
