# 硬件扩展 - RealSense D435i

- OS: Ubuntu 24.04

## 硬件环境

- [RealSense™ depth camera D435i](https://realsenseai.com/products/depth-camera-d435i/)

## 软件环境

- [RealSense SDK](https://github.com/realsenseai/librealsense)
  - [Linux Ubuntu Installation](https://github.com/realsenseai/librealsense/blob/master/doc/installation.md)
- [RealSense ROS Wrapper](https://github.com/realsenseai/realsense-ros)

### apt 安装

```bash
source /opt/ros/jazzy/setup.bash
sudo apt install ros-$ROS_DISTRO-librealsense2*
sudo apt install ros-$ROS_DISTRO-realsense2-*
```

更多：

- 若源码安装，可加装插件 `realsense2_rviz_plugin`。
- 可运行 `realsense-viewer` 升级固件到最新版本。
  - 建议主机 PC 上升级，于板子上升级过一次失败了。

### 源码安装

安装 SDK，

```bash
sudo apt-get update && sudo apt-get upgrade

sudo apt-get install libusb-1.0-0-dev
sudo apt-get install libudev-dev
sudo apt-get install libssl-dev pkg-config libgtk-3-dev

sudo apt-get install git wget cmake build-essential

sudo apt-get install libglfw3-dev libgl1-mesa-dev libglu1-mesa-dev at

git clone --depth 1 https://github.com/realsenseai/librealsense.git

cd librealsense
./scripts/setup_udev_rules.sh

mkdir build && cd build

cmake ..
make
sudo make install
```

安装 ROS Wrapper，

```bash
source /opt/ros/jazzy/setup.bash

ROS2_WS=~/Codes/ros2_ws
mkdir -p $ROS2_WS/src

cd $ROS2_WS/src/
git clone --depth 1 -b ros2-master https://github.com/realsenseai/realsense-ros.git

cd $ROS2_WS
sudo apt-get install python3-rosdep -y
sudo rosdep init
rosdep update
rosdep install -i --from-path src --rosdistro $ROS_DISTRO --skip-keys=librealsense2 -y

# colcon build
# conda deactivate
cd $ROS2_WS
source /opt/ros/jazzy/setup.bash
export MAKEFLAGS="-j1"
# rm -r build install log
colcon build --cmake-args -DRVIZ_RGBD_PLUGIN=ON \
  --parallel-workers 1 --executor sequential

cd $ROS2_WS
. install/local_setup.bash
```

注意：

- RK3588 板子上不能并行编译，资源不够，容易卡死。
- 电源用适配器供电。只 PC USB 供电，可能电压不稳。

<!--
make -j"$(($(nproc) / 2))"
-->

报错 `cv_bridge.h`：

```bash
/mnt/nvme/Codes/ros2_ws/src/realsense-ros/realsense2_rgbd_plugin/src/realsense_rviz_plugin.cpp:22:10: fatal error: cv_bridge/cv_bridge.h: No such file or directory
   22 | #include <cv_bridge/cv_bridge.h>
```

解决：

```bash
# ros2 pkg list | grep cv_bridge
# find $(ros2 pkg prefix cv_bridge) -name "cv_bridge.hpp"
sudo ln -s cv_bridge/cv_bridge.hpp /opt/ros/jazzy/include/cv_bridge/cv_bridge.h
```

### ROS 使用

启动相机节点，

```bash
source /opt/ros/jazzy/setup.bash
# 若源码安装，还要 👇
source /home/pi/Codes/ros2_ws/install/local_setup.bash

# ros2 launch realsense2_camera rs_launch.py
ros2 launch realsense2_camera rs_launch.py \
  rgb_camera.color_profile:=640x480x15 \
  depth_module.depth_profile:=640x480x15 \
  align_depth.enable:=false \
  pointcloud.enable:=true
```

<!--
cat $(ros2 pkg prefix realsense2_camera)/share/realsense2_camera/launch/rs_launch.py
-->

RViz2 订阅查看，

```bash
source /opt/ros/jazzy/setup.bash
rviz2
```

- 设置固定坐标系
  - 左侧 `Global Options` 中将 `Fixed Frame` 设置为 `camera_link`
  - 注意：点云必须设置正确的固定坐标系才能显示
- 添加可视化显示
  - 添加彩色图像
    - `Add` → `By topic` → `/camera/camera/color/image_raw` → `Image`
  - 添加深度图像
    - `Add` → `By topic` → `/camera/camera/depth/image_rect_raw` → `Image`
  - 添加点云
    - `Add` → `By topic` → `/camera/camera/depth/color/points` → `PointCloud2`

## 使用问题

### USB 2.1 port

`realsense-viewer` 发现显示 USB 2.1 连接，应该 USB 3.2 连接。

解决办法：换 USB 线材 + USB Hub 供电。主机 PC 与板子 RK3588，都能成功。

提示信息，ROS 2 启动节点时：

```bash
# USB 2.1
[realsense2_camera_node-1] [WARN] .. [camera.camera]: Device 845112071669 is connected using a 2.1 port. Reduced performance is expected.

# USB 3.2
[realsense2_camera_node-1] [INFO] .. [camera.camera]: Device USB type: 3.2
```

<!--
# ROS 2 启动时
[realsense2_camera_node-1]  01/06 00:00:00,390 ERROR [547821969472] (librealsense-exception.h:52) Mipi device capability could not be grabbed Last Error: No such device

$ realsense-viewer
 01/06 00:00:00,143 ERROR [547797223264] (librealsense-exception.h:52) Mipi device capability could not be grabbed Last Error: No such device
-->

### 点云话题找不到

ROS 2 launch.py 启动时，发现 `pointcloud.enable:=true` 参数没用，

```bash
# 发现实际参数是 pointcloud__neon_.enable
# ros2 param dump /camera/camera
ros2 param list | grep pointcloud
ros2 param get /camera/camera pointcloud__neon_.enable

# 运行时启用点云
ros2 param set /camera/camera pointcloud__neon_.enable true

# 发现点云话题
ros2 topic list | grep points
```

或 ROS 2 run 运行，设定 `pointcloud.enable:=true` 参数：

```bash
ros2 run realsense2_camera realsense2_camera_node \
  --ros-args \
  -p rgb_camera.color_profile:=640x480x15 \
  -p depth_module.depth_profile:=640x480x15 \
  -p pointcloud__neon_.enable:=true \
  -p align_depth.enable:=true
```

源码编译 ROS Wrapper 时，可以注释掉订阅者检查，强制发布点云：

```cpp
// ros2_ws/src/realsense-ros/realsense2_camera/src/pointcloud_filter.cpp
void PointcloudFilter::Publish(rs2::points pc, const rclcpp::Time& t, const rs2::frameset& frameset, const std::string& frame_id)
{
    // {
    //     std::lock_guard<std::mutex> lock_guard(_mutex_publisher);
    //     if ((!_pointcloud_publisher) || (!(_pointcloud_publisher->get_subscription_count())))
    //         return;
    // }
```

参考：

- [#3489](https://github.com/realsenseai/realsense-ros/issues/3489)
- [#3397](https://github.com/realsenseai/realsense-ros/issues/3397)
