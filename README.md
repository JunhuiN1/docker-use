# NaVid and Uni-NaVid: VLN-CE Evaluation

This repository provides the evaluation code for **NaVid** and **Uni-NaVid** on Vision-and-Language Navigation in Continuous Environments (VLN-CE).

The models take navigation instructions and egocentric video observations as input, and directly predict navigation actions. They do not require explicit odometry, depth observations, or map construction.

This repository contains the VLN-CE evaluation implementation for:

* **NaVid: Video-based VLM Plans the Next Step for Vision-and-Language Navigation**
* **Uni-NaVid: A Video-based Vision-Language-Action Model for Unifying Embodied Navigation Tasks**

## News

* Evaluation code for NaVid is available.
* Evaluation code for Uni-NaVid is available.
* Finetuned NaVid and Uni-NaVid checkpoints are provided.
* R2R and RxR VLN-CE evaluation are supported.
* Multi-GPU parallel evaluation is supported.

## Prerequisites

### 1. Create the Conda Environment

The evaluation environment uses Python 3.8 and Habitat-Sim 0.1.7.

```bash
conda create -n vlnce_navid python=3.8 -y
conda activate vlnce_navid
```

Install the headless version of Habitat-Sim:

```bash
conda install \
  -c aihabitat \
  -c conda-forge \
  habitat-sim=0.1.7=py3.8_headless_linux_856d4b08c1a2632626bf0d205bf46471a99502b7
```

When direct installation is unstable, download the corresponding Habitat-Sim Conda package first and install it locally:

```bash
conda install habitat-sim-package.tar.bz2
```

### 2. Install Habitat-Lab

Create a workspace and download Habitat-Lab 0.1.7:

```bash
mkdir -p navid_ws
cd navid_ws

git clone --branch v0.1.7 https://github.com/facebookresearch/habitat-lab.git
cd habitat-lab
```

Install Habitat-Lab and Habitat-Baselines:

```bash
python -m pip install -r requirements.txt
python -m pip install -r habitat_baselines/rl/requirements.txt
python -m pip install -r habitat_baselines/rl/ddppo/requirements.txt

python setup.py develop --all
```

### 3. Install NaVid-VLN-CE

Return to the workspace and clone this repository:

```bash
cd ..

git clone https://github.com/jzhzhang/NaVid-VLN-CE.git
cd NaVid-VLN-CE
```

Install the required Python packages:

```bash
pip install -r requirements.txt
```

### 4. Optional: Install Uni-NaVid

To evaluate Uni-NaVid, clone the Uni-NaVid repository and create a symbolic link to its model implementation:

```bash
git clone https://github.com/jzhzhang/Uni-NaVid.git
```

Create the symbolic link inside the NaVid-VLN-CE directory:

```bash
cd NaVid-VLN-CE

ln -s /absolute/path/to/Uni-NaVid/uninavid uninavid
```

Verify the symbolic link:

```bash
readlink -f uninavid
```

## Data Preparation

The project follows the VLN-CE data organization.

You need to prepare:

1. Matterport3D scene files
2. R2R or RxR episode files
3. Ground-truth files used to calculate navigation metrics

A recommended data structure is:

```text
data/
├── scene_datasets/
│   └── mp3d/
│       ├── 17DRP5sb8fy/
│       │   └── 17DRP5sb8fy.glb
│       ├── 1LXtFkjw3qL/
│       │   └── 1LXtFkjw3qL.glb
│       └── ...
│
└── datasets/
    ├── R2R_VLNCE_v1-3_preprocessed/
    │   ├── train/
    │   ├── val_seen/
    │   └── val_unseen/
    │
    └── RxR_VLNCE_v0/
        ├── train/
        ├── val_seen/
        └── val_unseen/
```

After downloading the datasets, update the paths in the corresponding VLN-CE configuration files.

Example configuration for R2R:

```yaml
TASK_CONFIG:
  TASK:
    NDTW:
      GT_PATH: data/datasets/R2R_VLNCE_v1-3_preprocessed/{split}/{split}_gt.json.gz

  DATASET:
    TYPE: VLN-CE-v1
    SPLIT: val_unseen
    DATA_PATH: data/datasets/R2R_VLNCE_v1-3_preprocessed/{split}/{split}.json.gz
    SCENES_DIR: data/scene_datasets/
```

Before evaluation, verify that the Matterport3D scene files are available:

```bash
find -L data/scene_datasets/mp3d -name "*.glb" | head
```

You can also check a specific scene:

```bash
SCENE="data/scene_datasets/mp3d/17DRP5sb8fy/17DRP5sb8fy.glb"

ls -lh "$SCENE"
readlink -f "$SCENE"
stat "$SCENE"
```

## Pretrained Weights

The following pretrained weights are required:

* EVA-ViT-G vision encoder
* Finetuned NaVid checkpoint
* Finetuned Uni-NaVid checkpoint, when evaluating Uni-NaVid

Place the downloaded checkpoints under `model_zoo`.

Recommended structure:

```text
model_zoo/
├── eva_vit_g.pth
├── navid/
│   ├── config.json
│   ├── tokenizer.model
│   ├── tokenizer_config.json
│   ├── model-00001-of-00002.safetensors
│   ├── model-00002-of-00002.safetensors
│   └── model.safetensors.index.json
│
└── uninavid/
    ├── config.json
    ├── tokenizer.model
    ├── tokenizer_config.json
    ├── model-00001-of-00002.safetensors
    ├── model-00002-of-00002.safetensors
    └── model.safetensors.index.json
```

After downloading, confirm that the model files are complete:

```bash
find model_zoo -maxdepth 3 -type f | sort
```

## Project Structure

The workspace can be organized as follows:

```text
navid_ws/
├── habitat-lab/
│
└── NaVid-VLN-CE/
    ├── navid/
    ├── uninavid/
    ├── VLN_CE/
    ├── model_zoo/
    │   ├── eva_vit_g.pth
    │   ├── navid/
    │   └── uninavid/
    ├── agent_navid.py
    ├── agent_uninavid.py
    ├── analyze_results.py
    ├── eval_navid_vlnce.sh
    ├── eval_uninavid_vlnce.sh
    ├── kill_navid_eval.sh
    ├── run.py
    ├── requirements.txt
    └── README.md
```

The main files are:

| File                     | Description                           |
| ------------------------ | ------------------------------------- |
| `agent_navid.py`         | NaVid agent implementation            |
| `agent_uninavid.py`      | Uni-NaVid agent implementation        |
| `run.py`                 | Main evaluation entry                 |
| `eval_navid_vlnce.sh`    | Multi-GPU NaVid evaluation script     |
| `eval_uninavid_vlnce.sh` | Multi-GPU Uni-NaVid evaluation script |
| `analyze_results.py`     | Evaluation-result analysis            |
| `kill_navid_eval.sh`     | Stops running evaluation processes    |

## Evaluation Preparation

Before running evaluation, edit the variables in the corresponding shell script.

Example:

```bash
CHUNKS=8
MODEL_PATH="/path/to/model/checkpoint"
CONFIG_PATH="/path/to/task/config.yaml"
SAVE_PATH="/path/to/evaluation/results"
MODEL_NAME="navid"
EXP_SAVE="video-data"
```

The variables have the following meanings:

| Variable      | Description                               |
| ------------- | ----------------------------------------- |
| `CHUNKS`      | Number of evaluation splits and GPUs      |
| `MODEL_PATH`  | Path to the model checkpoint              |
| `CONFIG_PATH` | Path to the VLN-CE configuration file     |
| `SAVE_PATH`   | Directory used to save evaluation results |
| `MODEL_NAME`  | `navid` or `uninavid`                     |
| `EXP_SAVE`    | Type of evaluation outputs to save        |

Each GPU evaluates one non-overlapping portion of the complete episode set.

## NaVid Evaluation

Set the model name to `navid` in `eval_navid_vlnce.sh`:

```bash
MODEL_NAME="navid"
```

Run the evaluation:

```bash
bash eval_navid_vlnce.sh
```

To assign specific GPUs:

```bash
CUDA_VISIBLE_DEVICES=0,1,2,3 bash eval_navid_vlnce.sh
```

For four GPUs, use:

```bash
CHUNKS=4
```

For eight GPUs, use:

```bash
CHUNKS=8
```

## Uni-NaVid Evaluation

Set the model name to `uninavid` in `eval_uninavid_vlnce.sh`:

```bash
MODEL_NAME="uninavid"
```

Run the evaluation:

```bash
bash eval_uninavid_vlnce.sh
```

To use specific GPUs:

```bash
CUDA_VISIBLE_DEVICES=0,1,2,3 bash eval_uninavid_vlnce.sh
```

Uni-NaVid uses online visual-token merging during evaluation to improve inference efficiency.

## Single-GPU Evaluation

For debugging, the model can be evaluated on one GPU:

```bash
CUDA_VISIBLE_DEVICES=0 python run.py \
  --exp-config /path/to/config.yaml \
  --exp-save video-data \
  --model-name navid \
  --split-num 1 \
  --split-id 0 \
  --model-path /path/to/model/checkpoint \
  --result-path /path/to/evaluation/results
```

For Uni-NaVid, change:

```bash
--model-name uninavid
```

## Monitor Evaluation

Evaluation outputs are saved under the directory specified by `SAVE_PATH`.

A typical output directory contains:

```text
results/
├── log/
├── video/
└── data/
```

Analyze the current evaluation results with:

```bash
python analyze_results.py --path /path/to/evaluation/results
```

Continuously refresh the statistics:

```bash
watch -n 1 python analyze_results.py --path /path/to/evaluation/results
```

Monitor GPU usage:

```bash
watch -n 1 nvidia-smi
```

Inspect the running processes:

```bash
ps -ef | grep -E "run.py|eval_navid|eval_uninavid" | grep -v grep
```

## Stop Evaluation

Stop all evaluation processes using:

```bash
bash kill_navid_eval.sh
```

Alternatively:

```bash
pkill -f "python run.py"
```

Before terminating processes, you can inspect them with:

```bash
ps -ef | grep "python run.py" | grep -v grep
```

## Evaluation Metrics

The evaluation reports standard VLN metrics:

| Metric | Description                     |
| ------ | ------------------------------- |
| TL     | Average trajectory length       |
| NE     | Navigation error                |
| OS     | Oracle success rate             |
| SR     | Success rate                    |
| SPL    | Success weighted by path length |

## Reference Results

The repository reports the following evaluation performance:

| Model     | Benchmark |   TL |   NE |   OS |   SR |  SPL |
| --------- | --------- | ---: | ---: | ---: | ---: | ---: |
| NaVid     | R2R Val   | 10.7 | 5.65 | 49.2 | 41.9 | 36.5 |
| NaVid     | R2R Test  | 11.3 | 5.39 | 52.0 | 45.0 | 39.0 |
| NaVid     | RxR Val   | 15.4 | 5.72 | 55.6 | 45.7 | 38.2 |
| Uni-NaVid | R2R Val   | 9.22 | 4.96 | 57.4 | 51.8 | 47.7 |
| Uni-NaVid | RxR Val   | 18.4 | 5.67 | 66.4 | 56.1 | 44.5 |

## Troubleshooting

### Habitat-Sim cannot find an MP3D scene

Example error:

```text
Render asset template handle ... does not correspond to any existing file
```

Check whether the scene file exists:

```bash
ls -lh data/scene_datasets/mp3d/SCENE_ID/SCENE_ID.glb
```

Check whether a symbolic link is valid:

```bash
readlink -f data/scene_datasets/mp3d
```

List all valid scene files:

```bash
find -L data/scene_datasets/mp3d -name "*.glb" | head
```

### The model path does not exist

Check the model directory:

```bash
MODEL_PATH="/path/to/model/checkpoint"

echo "$MODEL_PATH"
ls -lah "$MODEL_PATH"
```

### Configuration argument is empty

If the program reports:

```text
argument --exp-config: expected one argument
```

verify that the variable has been defined:

```bash
echo "$CONFIG_PATH"
test -f "$CONFIG_PATH"
```

Always quote path variables:

```bash
--exp-config "$CONFIG_PATH"
```

### CUDA device mapping

Check the GPUs visible to the current process:

```bash
CUDA_VISIBLE_DEVICES=0 python - <<'PY'
import torch

print("CUDA available:", torch.cuda.is_available())
print("Visible GPU count:", torch.cuda.device_count())

if torch.cuda.is_available():
    print("Current device:", torch.cuda.current_device())
    print("GPU:", torch.cuda.get_device_name(0))
PY
```

## Citation

Please cite NaVid when using the NaVid model or evaluation code:

```bibtex
@article{zhang2024navid,
  title   = {NaVid: Video-based VLM Plans the Next Step for Vision-and-Language Navigation},
  author  = {Zhang, Jiazhao and Wang, Kunyu and Xu, Rongtao and Zhou, Gengze and Hong, Yicong and Fang, Xiaomeng and Wu, Qi and Zhang, Zhizheng and Wang, He},
  journal = {Robotics: Science and Systems},
  year    = {2024}
}
```

Please cite Uni-NaVid when using the Uni-NaVid model:

```bibtex
@article{zhang2025uninavid,
  title   = {Uni-NaVid: A Video-based Vision-Language-Action Model for Unifying Embodied Navigation Tasks},
  author  = {Zhang, Jiazhao and Wang, Kunyu and Wang, Shaoan and Li, Minghan and Liu, Haoran and Wei, Songlin and Wang, Zhongyuan and Zhang, Zhizheng and Wang, He},
  journal = {Robotics: Science and Systems},
  year    = {2025}
}
```

## Acknowledgements

This repository is built upon the following open-source projects:

* LLaMA-VID
* VLN-CE
* Habitat-Lab
* Habitat-Sim

We thank the authors and contributors of these projects for releasing their code, models, and datasets.

## License

This repository is released under the MIT License.

Please also follow the licenses of Habitat-Lab, Habitat-Sim, VLN-CE, Matterport3D, pretrained language models, and all other third-party components used in this project.
