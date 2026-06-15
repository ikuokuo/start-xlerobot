# Web 远控

- OS: Ubuntu 24.04

```bash
XLeRobot/
  software/src/robots/xlerobot/xlerobot_host.py  # 单轮 ZMQ 服务
  software/src/robots/xlerobot_2wheels/README.md  # 两轮 ZMQ 服务说明
  web_control/README.md  # Web 控制说明
```

## 前提

[patch/260615.txt](../patch/260615.txt) 记录了最新并验过的 commit hash。

其中 XLeRobot 要 `git apply 260615.patch 260615_cam.patch` 再准备 lerobot。

或者，直接：

```bash
git clone --depth 1 -b dev https://github.com/ikuokuo/XLeRobot.git
```

<!--
scp 260615.patch pi@192.168.199.154:/mnt/nvme/Codes/XLeRobot/
-->

## Robot 启动 ZMQ 服务

```bash
conda activate lerobot
cd lerobot
pip install -e .

PYTHONPATH=src python -m lerobot.robots.xlerobot_2wheels.xlerobot_2wheels_host --robot.id=my_xlerobot_2wheels

# 启用相机
PYTHONPATH=src python -m lerobot.robots.xlerobot_2wheels.xlerobot_2wheels_host --robot.id=my_xlerobot_2wheels --camera.path /dev/video40
#  更多参数
PYTHONPATH=src python -m lerobot.robots.xlerobot_2wheels.xlerobot_2wheels_host --robot.id=my_xlerobot_2wheels \
--camera.path /dev/video40 --camera.fourcc MJPG --camera.width 320 --camera.height 240 --camera.fps 5 --camera.warmup 3
```

注意: 上述是两轮下的命令。

<!--
nc -zv 192.168.199.154 5555
nc -zv 192.168.199.154 5556
-->

<!--
lsusb | grep -i cam

for d in /dev/video*; do
  echo "$d: $(v4l2-ctl -d "$d" --list-formats 2>/dev/null | grep -o "MJPG" | head -1)"
done

v4l2-ctl -d /dev/video40 --list-formats-ext
ffmpeg -f v4l2 -video_size 640x480 -i /dev/video40 -frames:v 1 /tmp/test.jpg -y

$ PYTHONPATH=src python -c "
import cv2
cap = cv2.VideoCapture('/dev/video40', cv2.CAP_V4L2)
print(f'Opened: {cap.isOpened()}')
print(f'Backend: {cap.getBackendName()}')
cap.set(cv2.CAP_PROP_FOURCC, cv2.VideoWriter_fourcc(*'MJPG'))
cap.set(cv2.CAP_PROP_FRAME_WIDTH, 640)
cap.set(cv2.CAP_PROP_FRAME_HEIGHT, 480)
cap.set(cv2.CAP_PROP_FPS, 5)
ret, frame = cap.read()
print(f'Ret: {ret}, Frame: {frame.shape if ret else None}')
cap.release()
"
Opened: True
Backend: V4L2
Ret: True, Frame: (480, 640, 3)
-->

## Host 启动 Web 后端

```bash
cd XLeRobot/
cd web_control/server

python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
cp .env.example .env

# 编辑 .env
ROBOT_TYPE=xlerobot
UI_HOST=0.0.0.0
UI_PORT=8000
ROBOT_HOST=192.168.199.154
ROBOT_PORT_CMD=5555
ROBOT_PORT_DATA=5556

pip install pyzmq

python main.py
```

## Host 启动 Web 前端

```bash
cd XLeRobot/
cd web_control/client

npm install

cp .env.example .env

# 编辑 .env
VITE_SERVER_PROTOCOL=http
VITE_SERVER_HOST=localhost
VITE_SERVER_PORT=8000

npm run dev
```

打开 http://localhost:5173/ 操作控制。
