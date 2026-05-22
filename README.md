# docker-use
docker use
# Embodied AI & VLN 仿真实验环境配置指南 (Docker 版)

本指南用于指导如何在 GPU 服务器上基于 Docker 快速搭建 Vision-Language Navigation (VLN) 及 `habitat-lab` 的标准隔离实验环境。通过将 Conda 环境与 `requirements.txt` 批量依赖直接固化至 Docker 镜像中，实现“环境随用随建，数据永不变形”。

---

# 🛠️ 环境准备与文件拓扑

在开始构建前，请确保将以下配置文件放置在物理机的同一个实验目录中（例如 `~/docker_test`）：

```text
└── docker_test/
    ├── Dockerfile          # Docker 构建说明书
    ├── environment.yml     # Conda 基础环境配置文件
    └── requirements.txt    # 从 habitat-lab 中提取的 pip 依赖清单


#🐳 核心配置文件编写
1. Dockerfile
创建并编辑 Dockerfile，集成南京大学 Conda 镜像源与阿里云 pip 镜像源，并在构建期自动固化 requirements.txt 依赖：
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

##🚀 黄金三步：从构建到挺进实验环境
第一步：一键构建 Docker 镜像
在 Dockerfile 所在目录下执行编译。由于配置了国内镜像源，系统会高速全自动下载并打包所有依赖：
docker build -t vlnce_env:v1 .

#💻 容器生命周期管理指令 (核心使用指南)
1. 打开/启动容器 (进入实验环境)
利用 -v 挂载参数将物理机大硬盘的数据目录映射至容器内部的 /workspace（实现数据与环境分离，退出后数据永不丢失），同时开启 --gpus all 挂载物理显卡：
docker run -it --rm --gpus all -v /data/Newdisk/robotdog:/workspace vlnce_env:v1 /bin/bash

#💡 参数解析：

--rm: 退出容器后自动销毁临时容器房间，绝不占用服务器垃圾磁盘空间。

-v: 把物理机存储/代码的绝对路径挂载为容器内的 /workspace。

#进入容器后，系统已通过 .bashrc 全自动激活 好了你的 vlnce_ETPnav 环境。

# 验证环境
python --version  # 应输出项目指定的旧版本（如 3.6.x）
nvidia-smi        # 应正常看到物理机显卡

2. 关闭/退出容器
当你的实验跑完了，或者今天的工作结束了，直接在容器内部终端输入：

exit

敲回车后：你会瞬间退回到服务器本来的物理系统。这个临时的 Docker 运行环境会被自动擦除，而你跑出来的实验数据和修改的代码，都已经完好无损地保存在物理机大硬盘上了。

3. 查看容器状态 (常驻与监控)
如果你想在物理机上确认当前有哪些 Docker 环境正在运行，可以在物理机终端执行以下命令：

查看当前正在运行的容器：

Bash
docker ps

查看服务器上所有留存的“环境模具”(镜像)：

Bash
docker images

⚠️ 常见踩坑与实战 Tips
Permission denied (拒绝访问) 报错：
若在容器内以 root 权限解压了文件（如 unzip habitat-lab.zip），物理机普通用户在通过 MobaXterm 等工具修改文本时会报权限拒绝。

最佳解法：直接在 Docker 终端内运行 nano <文件路径> 进行修改；或者在容器内安装完 nano 后（apt-get update && apt-get install -y nano）进行可视化无缝编辑。

容器的生命周期：
由于启动时添加了 --rm 参数，在容器内部使用 apt-get install 额外安装的系统小工具（如 unzip, nano）在输入 exit 退出后不会被保存。如需永久保留这些工具，请直接写进 Dockerfile 的 RUN 指令中。
