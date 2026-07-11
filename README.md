# Video-based Vision-and-Language Navigation Evaluation

This repository provides a unified workspace for evaluating and reproducing three video-based Vision-and-Language Navigation models in continuous environments:

- [NaVid](https://github.com/jzhzhang/NaVid-VLN-CE)
- [Uni-NaVid](https://github.com/jzhzhang/Uni-NaVid)
- [StreamVLN](https://github.com/InternRobotics/StreamVLN)

The project supports evaluation on the VLN-CE R2R and RxR benchmarks. StreamVLN additionally provides training, DAgger data collection, co-training, and real-world deployment code.

---

## Supported Models

### NaVid

NaVid is a video-based vision-language model for continuous Vision-and-Language Navigation. It takes egocentric RGB observations and natural-language instructions as input and directly predicts low-level navigation actions.

### Uni-NaVid

Uni-NaVid extends video-based navigation to multiple embodied navigation tasks, including:

- Vision-and-Language Navigation
- Object Goal Navigation
- Embodied Question Answering
- Human Following

The model uses an online visual-token merging strategy to compress long navigation histories and predicts multiple future actions at each inference step.

### StreamVLN

StreamVLN performs navigation through online multi-turn vision-language-action dialogue. It introduces:

1. A fast-streaming dialogue context using a sliding-window KV cache.
2. A slow-updating memory context using temporal sampling and visual-token pruning.
3. Optional voxel-based 3D spatial pruning for reducing redundant visual tokens.
4. Stage-one training, DAgger collection, stage-two co-training, and real-world deployment.

---

## News

- StreamVLN provides benchmark and real-world checkpoints.
- StreamVLN supports stage-one training, DAgger collection, and stage-two co-training.
- NaVid, Uni-NaVid, and StreamVLN can be evaluated on VLN-CE R2R.
- Uni-NaVid and StreamVLN support evaluation on VLN-CE RxR.
- Multi-GPU evaluation is supported.

---

## Environment Preparation

NaVid/Uni-NaVid and StreamVLN use different Habitat versions. Separate Conda environments are recommended.

### Environment A: NaVid and Uni-NaVid

Recommended environment:

- Python 3.8
- Habitat-Sim 0.1.7
- Habitat-Lab 0.1.7

```bash
conda create -n vlnce_navid python=3.8 -y
conda activate vlnce_navid

conda install \
  -c aihabitat \
  -c conda-forge \
  habitat-sim=0.1.7=py3.8_headless_linux_856d4b08c1a2632626bf0d205bf46471a99502b7
```

Install Habitat-Lab:

```bash
mkdir -p navid_ws
cd navid_ws

git clone --branch v0.1.7 https://github.com/facebookresearch/habitat-lab.git
cd habitat-lab

python -m pip install -r requirements.txt
python -m pip install -r habitat_baselines/rl/requirements.txt
python -m pip install -r habitat_baselines/rl/ddppo/requirements.txt

python setup.py develop --all
```

Clone NaVid-VLN-CE:

```bash
cd ..
git clone https://github.com/jzhzhang/NaVid-VLN-CE.git
cd NaVid-VLN-CE

pip install -r requirements.txt
```

To evaluate Uni-NaVid, clone the repository and create a symbolic link:

```bash
cd ..
git clone https://github.com/jzhzhang/Uni-NaVid.git

cd NaVid-VLN-CE
ln -s /absolute/path/to/Uni-NaVid/uninavid uninavid
```

Verify the link:

```bash
readlink -f uninavid
```

### Environment B: StreamVLN

The official StreamVLN repository is tested with:

- Python 3.9
- PyTorch 2.1.2
- CUDA 12.4
- Habitat-Sim 0.2.4
- Habitat-Lab 0.2.4

Create the environment:

```bash
conda create -n streamvln python=3.9 -y
conda activate streamvln
```

Install Habitat-Sim and Habitat-Lab:

```bash
conda install habitat-sim==0.2.4 withbullet headless \
  -c conda-forge \
  -c aihabitat

git clone --branch v0.2.4 https://github.com/facebookresearch/habitat-lab.git
cd habitat-lab

pip install -e habitat-lab
pip install -e habitat-baselines
```

Clone StreamVLN:

```bash
cd ..
git clone https://github.com/InternRobotics/StreamVLN.git
cd StreamVLN

pip install -r requirements.txt
```

Check the environment:

```bash
python - <<'PY'
import torch
import habitat_sim

print("PyTorch:", torch.__version__)
print("CUDA available:", torch.cuda.is_available())
print("CUDA version:", torch.version.cuda)
print("Habitat-Sim imported successfully")
PY
```

---

## Workspace Structure

A recommended workspace structure is:

```text
vln_workspace/
├── habitat-lab-v0.1.7/
├── habitat-lab-v0.2.4/
│
├── NaVid-VLN-CE/
│   ├── navid/
│   ├── uninavid/
│   ├── VLN_CE/
│   ├── model_zoo/
│   ├── agent_navid.py
│   ├── agent_uninavid.py
│   ├── run.py
│   ├── analyze_results.py
│   ├── eval_navid_vlnce.sh
│   └── eval_uninavid_vlnce.sh
│
├── Uni-NaVid/
│
└── StreamVLN/
    ├── assets/
    ├── config/
    ├── llava/
    ├── realworld/
    ├── scripts/
    ├── streamvln/
    ├── trl/
    ├── requirements.txt
    └── README.md
```

NaVid/Uni-NaVid and StreamVLN should be run in their corresponding Conda environments.

---

## Data Preparation

### 1. Scene Datasets

#### Matterport3D

Matterport3D scenes are required by R2R, RxR, EnvDrop, and several trajectory datasets.

```text
data/
└── scene_datasets/
    └── mp3d/
        ├── 17DRP5sb8fy/
        │   └── 17DRP5sb8fy.glb
        ├── 1LXtFkjw3qL/
        │   └── 1LXtFkjw3qL.glb
        └── ...
```

#### Habitat-Matterport3D

HM3D scenes are required by the StreamVLN ScaleVLN subset.

```text
data/
└── scene_datasets/
    └── hm3d/
        ├── 00000-kfPV7w3FaU5/
        ├── 00001-UVdNNRcVyV1/
        └── ...
```

Verify scene files:

```bash
find -L data/scene_datasets/mp3d -name "*.glb" | head
find -L data/scene_datasets/hm3d -name "*.glb" | head
```

Check a specific scene:

```bash
SCENE="data/scene_datasets/mp3d/17DRP5sb8fy/17DRP5sb8fy.glb"

ls -lh "$SCENE"
readlink -f "$SCENE"
stat "$SCENE"
```

### 2. VLN-CE Episodes

Prepare the following datasets as required:

```text
data/
└── datasets/
    ├── r2r/
    │   ├── train/
    │   ├── val_seen/
    │   └── val_unseen/
    ├── rxr/
    │   ├── train/
    │   ├── val_seen/
    │   └── val_unseen/
    ├── envdrop/
    │   └── envdrop.json.gz
    └── scalevln/
        └── scalevln_subset_150k.json.gz
```

For the current StreamVLN benchmark checkpoint, use the updated R2R VLN-CE v1-3 data rather than the older R2R VLN-CE v1 data.

### 3. StreamVLN Trajectory Data

StreamVLN training uses pre-collected observation-action trajectories:

```text
data/
└── trajectory_data/
    ├── R2R/
    │   ├── images/
    │   └── annotations.json
    ├── RxR/
    │   ├── images/
    │   └── annotations.json
    ├── EnvDrop/
    │   ├── images/
    │   └── annotations.json
    └── ScaleVLN/
        ├── images/
        └── annotations.json
```

### 4. DAgger Data

DAgger data generated after stage-one training can be organized as:

```text
data/
└── dagger_data/
    ├── R2R/
    ├── RxR/
    └── EnvDrop/
```

Each dataset directory normally contains:

```text
images/
annotations.json
```

### 5. Co-training Data

StreamVLN stage-two training supports:

- LLaVA-Video-178K
- ScanNet/ScanQA
- MMC4-core

Recommended structure:

```text
data/
└── co-training_data/
    ├── ScanNet/
    │   ├── posed_images/
    │   ├── scanqa_annotations.json
    │   └── sqa3d_annotations.json
    ├── LLaVA-Video-178K/
    └── MMC4-core/
        ├── images/
        └── docs_shard_*.jsonl
```

---

## Model Zoo

### NaVid and Uni-NaVid

Recommended structure:

```text
NaVid-VLN-CE/
└── model_zoo/
    ├── eva_vit_g.pth
    ├── navid/
    └── uninavid/
```

A complete model directory should contain configuration, tokenizer, model shards, and the model index file.

Check the files:

```bash
find model_zoo -maxdepth 3 -type f | sort
```

### StreamVLN

StreamVLN provides two checkpoints:

- **Benchmark checkpoint:** for reproducing VLN-CE benchmark results.
- **Real-world checkpoint:** adjusted for physical-robot deployment, including reduced redundant initial turning and improved trajectory safety.

Set the local checkpoint path in the corresponding configuration or evaluation script.

---

## NaVid Evaluation

Activate the NaVid environment:

```bash
conda activate vlnce_navid
cd /path/to/NaVid-VLN-CE
```

Run the provided script:

```bash
bash eval_navid_vlnce.sh
```

Use selected GPUs:

```bash
CUDA_VISIBLE_DEVICES=0,1,2,3 bash eval_navid_vlnce.sh
```

For four evaluation splits:

```bash
CHUNKS=4
```

Single-GPU debugging:

```bash
CUDA_VISIBLE_DEVICES=0 python run.py \
  --exp-config /path/to/config.yaml \
  --exp-save video-data \
  --model-name navid \
  --split-num 1 \
  --split-id 0 \
  --model-path /path/to/navid/checkpoint \
  --result-path /path/to/results/navid
```

---

## Uni-NaVid Evaluation

Activate the NaVid environment:

```bash
conda activate vlnce_navid
cd /path/to/NaVid-VLN-CE
```

Run:

```bash
bash eval_uninavid_vlnce.sh
```

Use selected GPUs:

```bash
CUDA_VISIBLE_DEVICES=0,1,2,3 bash eval_uninavid_vlnce.sh
```

Single-GPU debugging:

```bash
CUDA_VISIBLE_DEVICES=0 python run.py \
  --exp-config /path/to/config.yaml \
  --exp-save video-data \
  --model-name uninavid \
  --split-num 1 \
  --split-id 0 \
  --model-path /path/to/uninavid/checkpoint \
  --result-path /path/to/results/uninavid
```

---

## StreamVLN Evaluation

Activate the StreamVLN environment:

```bash
conda activate streamvln
cd /path/to/StreamVLN
```

Edit the model path, dataset paths, GPU settings, and result directory in the StreamVLN evaluation script or configuration.

Run multi-GPU evaluation with KV-cache support:

```bash
sh scripts/streamvln_eval_multi_gpu.sh
```

A direct four-GPU configuration normally uses:

```bash
export CUDA_VISIBLE_DEVICES=0,1,2,3
```

and launches four worker processes or sets the distributed world size to four, depending on the script implementation.

Before evaluation, verify:

```bash
echo "CUDA_VISIBLE_DEVICES=$CUDA_VISIBLE_DEVICES"
find data/datasets/r2r/val_unseen -maxdepth 2 -type f | head
find data/scene_datasets/mp3d -name "*.glb" | head
ls -lah /path/to/streamvln/checkpoint
```

The updated official benchmark checkpoint reports:

| Benchmark | NE ↓ | OS ↑ | SR ↑ | SPL ↑ | nDTW ↑ |
|---|---:|---:|---:|---:|---:|
| R2R Val-Unseen | 4.90 | 63.6 | 56.4 | 50.2 | — |
| RxR Val-Unseen | 5.65 | — | 54.4 | 45.4 | 63.7 |

---

## StreamVLN Training

### Stage-One Training

Run distributed stage-one training:

```bash
sbatch scripts/streamvln_train_slurm.sh
```

The stage-one model is trained primarily with oracle navigation trajectories.

### DAgger Collection

Collect corrective trajectories using the stage-one model:

```bash
sh scripts/streamvln_dagger_collect.sh
```

Verify that newly generated samples are written to the configured `dagger_data` directory.

### Stage-Two Co-training

Run stage-two distributed training:

```bash
sbatch scripts/streamvln_stage_two_train_slurm.sh
```

Stage two combines navigation, DAgger, and general multimodal data.

---

## Real-World Deployment

StreamVLN includes a physical deployment pipeline for a Unitree Go2 robot.

Refer to:

```text
StreamVLN/realworld/realworld.md
```

Use the real-world checkpoint rather than the benchmark-only checkpoint. The deployment code should be configured with:

- Camera input
- Robot communication address
- Remote inference server
- Local action controller
- Model checkpoint
- Image transmission settings

---

## Monitoring

Monitor GPU usage:

```bash
watch -n 1 nvidia-smi
```

Monitor logs:

```bash
tail -f /path/to/log/file.log
```

Inspect evaluation processes:

```bash
ps -ef | grep -E "run.py|streamvln_eval.py|torchrun" | grep -v grep
```

Stop NaVid or Uni-NaVid evaluation:

```bash
pkill -f "python run.py"
```

Stop StreamVLN evaluation:

```bash
pkill -f "streamvln_eval.py"
```

---

## Evaluation Metrics

| Metric | Description |
|---|---|
| TL | Average trajectory length |
| NE | Navigation error |
| OS / OSR | Oracle success rate |
| SR | Success rate |
| SPL | Success weighted by path length |
| nDTW | Normalized dynamic time warping |

For StreamVLN efficiency analysis, also record:

- Average inference latency
- Peak GPU memory
- Number of active visual tokens
- KV-cache length
- Sliding-window reset latency
- Token-pruning ratio

---

## Troubleshooting

### Scene File Not Found

Example:

```text
Render asset template handle ... does not correspond to any existing file
```

Check the scene:

```bash
SCENE="/absolute/path/to/scene.glb"

ls -lh "$SCENE"
readlink -f "$SCENE"
stat "$SCENE"
```

A `.glb` file that is only a few bytes is usually a broken symbolic link or an incomplete download.

### Create a Scene Symbolic Link

```bash
rm -f data/scene_datasets/mp3d

ln -s \
  /absolute/path/to/mp3d \
  data/scene_datasets/mp3d
```

Verify:

```bash
readlink -f data/scene_datasets/mp3d
find -L data/scene_datasets/mp3d -name "*.glb" | head
```

### Empty Configuration Variable

If the program reports:

```text
argument --exp-config: expected one argument
```

check:

```bash
echo "$CONFIG_PATH"
test -f "$CONFIG_PATH"
```

Always quote shell variables:

```bash
--exp-config "$CONFIG_PATH"
```

### CUDA Device Mapping

When using:

```bash
CUDA_VISIBLE_DEVICES=3
```

the selected physical GPU is usually visible to the Python process as local CUDA device `0`.

Check:

```bash
CUDA_VISIBLE_DEVICES=3 python - <<'PY'
import torch

print("Visible GPUs:", torch.cuda.device_count())
print("Current device:", torch.cuda.current_device())
print("Device name:", torch.cuda.get_device_name(0))
PY
```

### StreamVLN Evaluation Performance Is Abnormally Low

Check the following:

1. Pull the latest StreamVLN code.
2. Confirm that `num_history` is correctly passed during evaluation.
3. Use the matching R2R VLN-CE v1-3 dataset for the updated checkpoint.
4. Verify the checkpoint path and configuration.
5. Confirm that each distributed rank uses a different GPU and a non-overlapping episode split.

---

## Citation

### NaVid

```bibtex
@article{zhang2024navid,
  title   = {NaVid: Video-based VLM Plans the Next Step for Vision-and-Language Navigation},
  author  = {Zhang, Jiazhao and Wang, Kunyu and Xu, Rongtao and Zhou, Gengze and Hong, Yicong and Fang, Xiaomeng and Wu, Qi and Zhang, Zhizheng and Wang, He},
  journal = {Robotics: Science and Systems},
  year    = {2024}
}
```

### Uni-NaVid

```bibtex
@article{zhang2025uninavid,
  title   = {Uni-NaVid: A Video-based Vision-Language-Action Model for Unifying Embodied Navigation Tasks},
  author  = {Zhang, Jiazhao and Wang, Kunyu and Wang, Shaoan and Li, Minghan and Liu, Haoran and Wei, Songlin and Wang, Zhongyuan and Zhang, Zhizheng and Wang, He},
  journal = {Robotics: Science and Systems},
  year    = {2025}
}
```

### StreamVLN

```bibtex
@article{wei2025streamvln,
  title   = {StreamVLN: Streaming Vision-and-Language Navigation via SlowFast Context Modeling},
  author  = {Wei, Meng and Wan, Chenyang and Yu, Xiqian and Wang, Tai and Yang, Yuqiang and Mao, Xiaohan and Zhu, Chenming and Cai, Wenzhe and Wang, Hanqing and Chen, Yilun and others},
  journal = {arXiv preprint arXiv:2507.05240},
  year    = {2025}
}
```

---

## Acknowledgements

This workspace is based on or related to:

- NaVid-VLN-CE
- Uni-NaVid
- StreamVLN
- VLN-CE
- Habitat-Lab
- Habitat-Sim
- LLaVA-Video
- LLaVA-NeXT

Please follow the licenses and dataset usage terms of each original repository and third-party dependency.

---

## License

The combined workspace does not replace the licenses of the original projects.

- Follow the NaVid and Uni-NaVid repository licenses for their code and models.
- StreamVLN is released under the Creative Commons Attribution-NonCommercial-ShareAlike 4.0 International License.
- Matterport3D, HM3D, Habitat, VLN-CE, pretrained language models, and other third-party assets retain their original licenses.
