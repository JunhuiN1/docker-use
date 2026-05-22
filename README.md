# 🤖 Embodied AI & VLN Simulation Environment (Docker)

<p align="left">
  <img src="https://img.shields.io/badge/Python-3.6%20%7C%203.13-blue.svg" alt="Python">
  <img src="https://img.shields.io/badge/Platform-Linux--64-orange.svg" alt="Platform">
  <img src="https://img.shields.io/badge/Docker-Supported-brightgreen.svg" alt="Docker">
  <img src="https://img.shields.io/badge/Framework-Habitat%20%7C%20TensorFlow-lightgrey.svg" alt="Framework">
</p>

本仓库提供了面向 **Vision-Language Navigation (VLN)** 以及 `habitat-lab` 仿真实验的工业级 Docker 环境配置方案。通过将基础 Conda 环境与 `requirements.txt` 批量依赖在构建期直接固化，实现**“环境秒级复现，数据与环境彻底解耦”**。

---

## 📌 Table of Contents
- [🛠️ File Structure](#️-file-structure)
- [🐳 Configuration Files](#-configuration-files)
- [🚀 Quick Start](#-quick-start)
  - [1. Build Image](#1-build-image)
  - [2. Run Container](#2-run-container)
- [💻 Container Lifecycle Management](#-container-lifecycle-management)
- [⚠️ Troubleshooting & Tips](#️-troubleshooting--tips)

---

## 🛠️ File Structure

在开始构建前，请确保将关键配置文件放置在物理机的同一个实验工作区内（例如 `/data/Newdisk/robotdog/` 或 `~/docker_test`）：

```text
└── workspace/
    ├── Dockerfile          # Docker 镜像构建说明书
    ├── environment.yml     # Conda 基础依赖环境配置
    └── requirements.txt    # habitat-lab 核心 pip 依赖清单
```

---

## 🐳 Configuration Files
💡 GitHub Tip: 点击下方展开标签查看精准的生产环境配置文件。
```text
FROM anaconda/miniconda:latest

# 拷贝环境配置文件到镜像内部临时目录
COPY environment.yml /tmp/environment.yml
COPY requirements.txt /tmp/requirements.txt

# 1. 配置国内高性能镜像源 (南京大学 Conda 源 + 阿里云 pip 源)
RUN conda config --remove-key channels || true && \
    conda config --add channels [https://mirrors.nju.edu.cn/anaconda/pkgs/main/](https://mirrors.nju.edu.cn/anaconda/pkgs/main/) && \
    conda config --add channels [https://mirrors.nju.edu.cn/anaconda/cloud/conda-forge/](https://mirrors.nju.edu.cn/anaconda/cloud/conda-forge/) && \
    conda config --set show_channel_urls yes && \
    mkdir -p ~/.config/pip && \
    echo "[global]\nindex-url = [https://mirrors.aliyun.com/pypi/simple/](https://mirrors.aliyun.com/pypi/simple/)\ntrusted-host = mirrors.aliyun.com" > ~/.config/pip/pip.conf

# 2. 基于 environment.yml 创建基础 Conda 环境
RUN conda env create -f /tmp/environment.yml && \
    conda clean -afy

# 3. 核心固化：在构建期将 requirements.txt 依赖直接灌入指定环境
RUN /opt/conda/envs/vlnce_ETPnav/bin/python -m pip install -r /tmp/requirements.txt

# 4. 设置环境变量并配置默认全自动激活环境
ENV PATH=/opt/conda/envs/vlnce_ETPnav/bin:$PATH
RUN echo "conda activate vlnce_ETPnav" >> ~/.bashrc

CMD ["/bin/bash", "--login"]
```
## 🚀 Quick Start
1. Build Image
在 Dockerfile 所在目录下执行编译。得益于内嵌的国内高速镜像源，系统会自动拉取并固化所有环境：
```text
docker build -t vlnce_env:v1 .
```
2. Run Container
使用以下标准指令挂载物理机大硬盘数据目录，并一键挺进实验环境：
```text
docker run -it \
  --rm \
  --gpus all \
  -v /data/Newdisk/robotdog:/workspace \
  vlnce_env:v1 \
  /bin/bash
```
[!IMPORTANT]
关于数据持久化：
容器启动命令中添加了 -v /data/Newdisk/robotdog:/workspace 挂载参数。这意味着容器内的 /workspace 目录与大硬盘绝对路径完全实时同步。在容器内做的任何代码修改、解压操作以及训练产生的模型权重，都会直接写入物理大硬盘，容器销毁后绝不丢失。


