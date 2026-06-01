# 硬件扩展 - NanoPi M6 (RK3588S)

- OS: Ubuntu 24.04

[NanoPi M6]: https://wiki.friendlyelec.com/wiki/index.php/NanoPi_M6/zh

## 硬件环境

按照 [NanoPi M6][] 文档，下载固件。选的 Ubuntu 24.04 桌面版，并通过 USB 烧写系统到 eMMC。

另外，加装了一个 M.2 NVMe 硬盘。

## 系统环境

Ubuntu 24.04 系统，文档介绍了如何使用。

此外，为了远程操作，安装并启动 SSH 服务，

```bash
sudo apt install openssh-server
sudo service ssh start
ps -aux | grep ssh
```

并于 `Settings > System > Remote Desktop` 打开桌面共享。

还可设置无头模式，不依赖 HDMI 显示，

```bash
# GNOME Remote Desktop (RDP + VNC)
#  https://github.com/GNOME/gnome-remote-desktop

# 生成 TLS 证书
mkdir -p ~/.local/share/gnome-remote-desktop/
openssl req -new -newkey rsa:4096 -days 720 -nodes -x509 -subj /C=SE/ST=NONE/L=NONE/O=IKUOKUO/CN=NanoPi-M6 -out ~/.local/share/gnome-remote-desktop/tls.crt -keyout ~/.local/share/gnome-remote-desktop/tls.key

# 配置 RDP (VNC 见上述 github 地址下官方文档)
grdctl --headless rdp set-tls-key ~/.local/share/gnome-remote-desktop/tls.key
grdctl --headless rdp set-tls-cert ~/.local/share/gnome-remote-desktop/tls.crt
grdctl --headless rdp set-credentials pi pi
grdctl --headless rdp enable

# 启动
systemctl --user enable --now gnome-remote-desktop-headless.service
```

主机用 Remmina 配置 RDP 连接时，分辨率可 Custom 指定。

另外，如下挂载 M.2 NVMe 硬盘，

```bash
# 查看 NVMe 设备信息
sudo apt install nvme-cli
sudo nvme list

# 格式化
sudo mkfs.ext4 /dev/nvme0n1

# 创建挂载点
sudo mkdir -p /mnt/nvme
# 临时挂载
sudo mount /dev/nvme0n1 /mnt/nvme
# 验证挂载
df -h /mnt/nvme

# 查看 UUID
sudo blkid /dev/nvme0n1
# 编辑 fstab，配置开机自动挂载
sudo vi /etc/fstab
# 文件末尾添加：
#  UUID=你的UUID /mnt/nvme ext4 defaults 0 2
# 测试 fstab 配置，应无错误信息
sudo mount -a

# 设置权限
sudo chown -R pi:pi /mnt/nvme

# 验证结果
lsblk
df -h
```

若要设置中文输入法（自带 IBus），

- 安装中文语言包和拼音引擎，并重启 IBus 输入法框架使得安装生效

  ```bash
  sudo apt install ibus-libpinyin language-pack-zh-hans
  ibus-daemon -drx
  ```

- 系统设置中添加中文输入源
  - 打开 Settings -> Keyboard，点击 Input Sources 下 “+” 按钮
  - 弹窗里点击: Chinese -> Chinese (Intelligent Pinyin) -> 右上 Add

## 硬件连线

把 Robot 的 USB 接到板子，与 PC 一样。

## 软件环境

### Robot 环境

见『[准备/软件](../README.md)』，安装 Robot 环境。

如果挂载了硬盘 /mnt/nvme/，可以都往里安装：

```bash
# 安装 conda
wget "https://github.com/conda-forge/miniforge/releases/latest/download/Miniforge3-$(uname)-$(uname -m).sh"
bash Miniforge3-$(uname)-$(uname -m).sh -b -p /mnt/nvme/miniforge3
/mnt/nvme/miniforge3/bin/conda init

# 配置 conda 缓存
mkdir -p /mnt/nvme/conda_cache/{pkgs,envs}
conda config --add pkgs_dirs /mnt/nvme/conda_cache/pkgs
conda config --add envs_dirs /mnt/nvme/conda_cache/envs

# 配置 pip 缓存
mkdir -p /mnt/nvme/pip_cache
pip config set global.cache-dir /mnt/nvme/pip_cache

# 配置 pip 镜像
pip config set global.index-url https://pypi.tuna.tsinghua.edu.cn/simple
```
<!--
在安装 lerobot 之前，可以先安装 PyTorch CPU 版，省掉安装 CUDA 版相关环境。 ✕
(过高版本，被重装了；对应 lerobot 依赖版本要求再试)
-->
<!--
mkdir /mnt/nvme/Codes
ln -s /mnt/nvme/Codes ~/Codes
-->

更多：

- pynput 监听不到本地键盘事件，在 SSH 或 RDP Wayland 远程下启动脚本时
  - 上显示器、键盘，直接操作  ✕

    ```bash
    $ echo $XDG_SESSION_TYPE
    wayland
    ```

  - 上蓝牙模块再 Joycon 遥操作  ?
  - 上 WiFi 模块再实现远程控制  ✓

### Docker 环境

见『[NanoPi M6][]』，安装 Docker 环境。

并把 Docker 数据配置到硬盘，

```bash
# 移动 Docker 数据
$ sudo cp -r /var/lib/docker /mnt/nvme/docker

# 配置 Docker 数据路径,j加上 data-root
$ sudo vi /etc/docker/daemon.json
{
    "data-root": "/mnt/nvme/docker"
}

# 重启 Docker 服务
$ sudo systemctl restart docker
```

<!--
https://github.com/dusty-nv/jetson-containers/blob/master/docs/setup.md
-->

### ROS 2 环境

ROS 2 Jazzy Jalisco (for Ubuntu 24.04),

- Installation: https://docs.ros.org/en/jazzy/Installation.html
  - deb packages: https://docs.ros.org/en/jazzy/Installation/Ubuntu-Install-Debs.html

按照上述文档安装好 ros-jazzy-desktop 后，如下确认测试正常：

```bash
# In one terminal, source the setup file and then run a C++ talker:
source /opt/ros/jazzy/setup.bash
ros2 run demo_nodes_cpp talker

# In another terminal source the setup file and then run a Python listener:
source /opt/ros/jazzy/setup.bash
ros2 run demo_nodes_py listener
```
