Install and setup livepotrait in gpu server
check gpu : nvidia-smi
# 3. Verify PyTorch + CUDA
python -c "import torch; print(torch.cuda.is_available(), torch.version.cuda)"
# Should print: True  12.x

apt-get update -y && apt-get install -y \
    git \
    ffmpeg \
    libgl1 \
    libglib2.0-0 \
    wget \
    curl

    git clone https://github.com/KwaiVGI/LivePortrait
cd LivePortrait

pip install -r requirements.txt

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


CUDA 13.0 — that's very new. There's no official onnxruntime-gpu build for CUDA 13 yet. Here's how to fix it:

The fix for CUDA 13.0
bash# 1. Remove existing onnxruntime
pip uninstall onnxruntime onnxruntime-gpu -y

# 2. Install CPU-only onnxruntime
# (face detection runs on CPU anyway — no real speed loss)
pip install onnxruntime==1.18.0

# 3. Verify
python -c "
import onnxruntime as ort
print('ORT version:', ort.__version__)
print('Providers:', ort.get_available_providers())
"
# Will show: ['CPUExecutionProvider']
# That's fine — the main LivePortrait inference still uses PyTorch on GPU




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
