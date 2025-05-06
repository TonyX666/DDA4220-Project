*The models are uploaded to https://huggingface.co/X666/finetuned-instruct-pix2pix/tree/main*


# RoboTwin Environment Setup

## 1. Installation and Environment Configuration

### Vulkan

```bash
sudo apt install libvulkan1 mesa-vulkan-drivers vulkan-tools
```

### Basic Environment

```bash
conda create -n robotwin python=3.8
conda activate robotwin
pip install torch==2.4.1 torchvision sapien==3.0.0b1 scipy==1.10.1 mplib==0.1.1 gymnasium==0.29.1 trimesh==4.4.3 open3d==0.18.0 imageio==2.34.2 pydantic zarr openai huggingface_hub==0.25.0 -i https://pypi.tuna.tsinghua.edu.cn/simple/
```

### RoboTwin Project Download and PyTorch3D Installation

```bash
git clone https://github.com/TianxingChen/RoboTwin.git
cd RoboTwin
cd third_party/pytorch3d_simplified && pip install -e . && cd ../..
```

## 2. Download and Extract Assets

```bash
python ./script/download_asset.py
unzip aloha_urdf.zip && unzip main_models.zip
```

## 3. Modify mplib Library Code

- **File**: `planner.py`
  - **Line 71**: Comment out `convex=True` using `#`
  - **Line 848**: Remove `or collide`

## 4. Install Diffusion Policy (DP) & 3D Diffusion Policy (DP3)

### DP

```bash
cd policy/Diffusion-Policy
pip install -e .
cd ../..
```

### DP3

```bash
cd policy/3D-Diffusion-Policy/3D-Diffusion-Policy && pip install -e . && cd ..
pip install zarr==2.12.0 wandb ipdb gpustat dm_control omegaconf hydra-core==1.2.0 dill==0.3.5.1 einops==0.4.1 diffusers==0.11.1 numba==0.56.4 moviepy imageio av matplotlib termcolor -i https://pypi.tuna.tsinghua.edu.cn/simple/
cd ../..
```





# RoboTwin Data Collection Process

## Step 1: Run Tasks

The following tasks are executed:

- `bash run_task.sh blocks_stack_easy 0`
- `bash run_task.sh block_hammer_beat 0`
- `bash run_task.sh block_handover 0`

Each task produces episodes from `episode0` to `episode99`, with a total of 300 folders, saved as follows:

- `data/blocks_stack_easy/episodes`
- `data/block_hammer_beat/episodes`
- `data/block_handover/episodes`

## Step 2: Extract Head Camera Images

For each folder in the above directories, run the `pkl2jpg` script to extract the head camera image data. The images are saved in the respective folder where the pkl files are located, with filenames formatted as `episode{i}_frame{j}.jpg`. The pkl files are deleted after extraction.

## Step 3: Match Frames and Prompts

For each folder, match the `i`-th frame and the `(i+50)`-th frame, along with their corresponding prompts. Save this data in a CSV file, and finally, save it as a dataset.







# Fine-Tuning Environment Setup

## Environment Configuration

```bash
conda create -n ip2p python=3.8
conda activate ip2p
pip install torch==2.4.1 torchvision==0.19.1 torchaudio==2.4.1 --index-url https://download.pytorch.org/whl/cu121
git clone https://github.com/huggingface/instruction-tuned-sd.git
cd instruction-tuned-sd
pip install -r requirements.txt
pip3 install -U xformers==0.0.28.post1 --index-url https://download.pytorch.org/whl/cu121
```

## Model Download

```bash
git-lfs clone https://hf-mirror.com/timbrooks/instruct-pix2pix
```

## Dataset and Fine-Tuning Configuration

- Upload the dataset to the `instruction-tuned-sd` folder.
- Modify `trian.sh` to update:
  - Model path
  - Dataset path
  - Output path

## Modify `finetune_instruct_pix2pix.py` File

- **Line 39**: Add `load_from_disk`
- **Line 652**: Change `load_dataset` to `load_from_disk`
- **Line 655**: Remove `cache_dir=args.cache_dir,`
- **Line 656**: Remove `use_auth_token=True`
- **Line 121**: Change `default` to `'edited_image'`
- **Line 438**: Modify the function to load local images
- **Line 1077**: Change `download_image` to `load_image`

## Start Fine-Tuning

```bash
accelerate config default
bash trian.sh
```









# Training Environment Setup

## Environment Configuration

```bash
conda create -n ip2p python=3.8
conda activate ip2p
pip install torch==2.4.1 torchvision==0.19.1 torchaudio==2.4.1 --index-url https://download.pytorch.org/whl/cu121
git clone https://github.com/huggingface/instruction-tuned-sd.git
cd instruction-tuned-sd
pip install -r requirements.txt
pip3 install -U xformers==0.0.28.post1 --index-url https://download.pytorch.org/whl/cu121
```

## Dataset and Fine-Tuning Configuration

- Upload the dataset to the `instruction-tuned-sd` folder.
- Modify `trian.sh` to update:
  - Model ID
  - Dataset path
  - Output path

## Modify `train_instruct_pix2pix.py` File

- **Line 39**: Add `load_from_disk`
- **Line 584**: Change `load_dataset` to `load_from_disk`
- **Line 587**: Remove `cache_dir=args.cache_dir,`
- **Line 110**: Change `default` to `'original_image'`
- **Line 381**: Modify the function to load local images
- **Line 925**: Change `download_image` to `load_image`

## Start Training

```bash
accelerate config default
bash trian.sh
```
