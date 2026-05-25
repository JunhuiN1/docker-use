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
❗❗❗事先声明：这个README是对vlnce_ETPnav的环境配置，自己建的时候不要再使用这个名字，并注意这个环境中的habita-sim-headless是在python3.6环境下的v0.1.7版本，habitat-lab版本需要与之匹配，且habitat-sim必须是headless版本否则服务器无法运行。
```text
conda install habitat-sim-0.1.7-py3.6_headless_linux_856d4b08c1a2632626bf0d205bf46471a99502b7.tar.bz2
```
❗❗❗habitat-lab 无法从服务器直接git clone，需要自行前往https://github.com/facebookresearch/habitat-lab
```text
└── workspace/
    ├── Dockerfile          # Docker 镜像构建说明书
    ├── environment.yml     # Conda 基础依赖环境配置
    ├── habita-lab  #安装habita-lab版本需要和habita-sim-headless版本匹配
    └── data    # 存放mp3d与hm3d数据集
```

---

## 🐳 Configuration Files
💡 GitHub Tip: 点击下方展开标签查看精准的生产环境配置文件。
打开dockfile
```text
nano dockfile
```
在dockfile中直接复制粘贴以下指令
```text
FROM anaconda/miniconda:latest

# 拷贝环境配置文件到镜像内部临时目录
COPY environment.yml /tmp/environment.yml #这里的environment是针对于ETP的环境，换成另外的环境需修改

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
按ctrl+o保存，再按Enter，最后按ctrl+x退出nano编辑器

## 🚀 Quick Start
1. Build Image
在 Dockerfile 所在目录下执行编译。得益于内嵌的国内高速镜像源，系统会自动拉取并固化所有环境：
```text
docker build -t vlnce_env:v1 .
```
2. Run Container
由于服务器有的时候会出现断网的现象，防止之前开好的容器被下面的--rm删除了，所以先在tmux开个房间，它能让你的程序在服务器后台常驻，哪怕你本地电脑关机、断网，服务器上的程序也会继续跑。
```text
tmux new -s vln_run #vln_run可以换成任何你想取的名字
docker run -it --rm --gpus all -v /data/Newdisk/robotdog:/workspace vlnce_env:v1 /bin/bash
```

网络恢复后重新连上服务器，输入一句话就能瞬间回到刚才的 Docker 现场：
```text
tmux a -t vln_run
```

💡 参数解析：
--rm: 退出容器后自动销毁临时容器房间，绝不占用服务器垃圾磁盘空间。
-v: 把物理机存储/代码的绝对路径挂载为容器内的 /workspace。

[!IMPORTANT]
关于数据持久化：
容器启动命令中添加了 -v /data/Newdisk/robotdog:/workspace 挂载参数。这意味着容器内的 /workspace 目录与大硬盘绝对路径完全实时同步。在容器内做的任何代码修改、解压操作以及训练产生的模型权重，都会直接写入物理大硬盘，容器销毁后绝不丢失。

## 环境验收与代码运行
进入容器后，系统已通过 .bashrc 全自动激活 好了你的 vlnce_ETPnav 环境，直接就是要求的系统版本：
```text
# 1. 验证 Python 版本（应输出项目指定的旧版本，如 Python 3.6.x）
python --version

# 2. 验证 NVIDIA 显卡是否成功打通
nvidia-smi  

# 3. 展开你的科研实验
cd /workspace
python 你的主程序.py
```
##关闭/退出容器
```text
exit
```
##💡 Tips查看容器状态 (常驻与监控)

如果你想在物理机上确认当前有哪些 Docker 环境正在运行，可以在物理机终端执行以下命令：
查看当前正在运行的容器：
```text
docker ps
```
查看服务器上所有留存的“环境模具”(镜像)：
```text
docker images
```
