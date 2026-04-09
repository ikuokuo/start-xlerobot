# 安装 LeRobot

- OS: Ubuntu 24.04

## 安装 LeRobot 环境

- [官方安装指南](https://huggingface.co/docs/lerobot/installation)
  - [Miniconda](https://www.anaconda.com/docs/getting-started/miniconda/install)

创建 Conda 环境，

```bash
conda create -n lerobot python=3.12
```

建议新建，与仿真独立，不然可能有依赖冲突。

```bash
conda activate lerobot

conda install ffmpeg -c conda-forge
ffmpeg -version  # ffmpeg version 8.0
ffmpeg -encoders | grep libsvtav1  # libsvtav1 check supported
# or,
# sudo apt install ffmpeg
```

```bash
git clone --depth 1 https://github.com/huggingface/lerobot.git
cd lerobot
pip install -e .
pip install -e ".[feetech]"

lerobot-info
```

<!--
mani-skill 3.0.0b22 requires gymnasium==0.29.1, but you have gymnasium 1.2.3 which is incompatible.

pip install gymnasium==0.29.1
-->

## 安装 XLeRobot 资源

```bash
export XR_DIR=`pwd`/XLeRobot
export LR_DIR=`pwd`/lerobot

# 软链接 XLeRobot 资源
ln -s $XR_DIR/software/src/model/SO101Robot.py $LR_DIR/src/lerobot/model/
ln -s $XR_DIR/software/src/robots/xlerobot $LR_DIR/src/lerobot/robots/
ln -s $XR_DIR/software/examples/* $LR_DIR/examples/
```
