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
# GNOME Remote Desktop
#  https://github.com/GNOME/gnome-remote-desktop

# 生成 TLS 证书
mkdir -p ~/.local/share/gnome-remote-desktop/
openssl req -new -newkey rsa:4096 -days 720 -nodes -x509 -subj /C=SE/ST=NONE/L=NONE/O=IKUOKUO/CN=NanoPi-M6 -out ~/.local/share/gnome-remote-desktop/tls.crt -keyout ~/.local/share/gnome-remote-desktop/tls.key

# 配置
grdctl --headless rdp set-tls-key ~/.local/share/gnome-remote-desktop/tls.key
grdctl --headless rdp set-tls-cert ~/.local/share/gnome-remote-desktop/tls.crt
grdctl --headless rdp set-credentials pi pi
grdctl --headless rdp enable

# 启动
systemctl --user enable --now gnome-remote-desktop-headless.service
```

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

## 硬件连线

把 Robot 的 USB 接到板子，与 PC 一样。

## 软件环境

### Robot 环境

见『[准备/软件](../README.md)』，安装 Robot 环境。

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
