# 遥操作

- OS: Ubuntu 24.04

```bash
conda activate lerobot
cd lerobot

# ModuleNotFoundError: No module named 'zmq'
pip install pyzmq
```

## 准备

```bash
# 临时给端口加权限
sudo chmod 666 /dev/ttyACM0 /dev/ttyACM1
ll /dev/ttyACM*

# 或给当前用户添加 dialout 组权限
sudo usermod -a -G dialout $USER
```

```bash
# 检查控制板ID
lerobot-find-port

# 单轮：进如下文件，修改port1与port2的名称
#  lerobot/robots/xlerobot/config_xlerobot.py
class XLerobotConfig(RobotConfig):
    port1: str = "/dev/ttyACM0"  # port to connect to the bus (so101 + head camera)
    port2: str = "/dev/ttyACM1"  # port to connect to the bus (same as lekiwi setup)

# 两轮：进如下文件，修改port1与port2的名称
#  lerobot/robots/xlerobot_2wheels/config_xlerobot_2wheels.py
class XLerobot2WheelsConfig(RobotConfig):
    port1: str = "/dev/ttyACM0"  # port to connect to the bus (so101 + head camera)
    port2: str = "/dev/ttyACM1"  # port to connect to the bus (arms + 2 wheels)
```

之后，如果只检测到8个电机（实际要9/10个），那么交换上述port1与port2可解决。

```bash
# 8 个  ✕
Full expected motor list (id: model_number):
{1: 777, 2: 777, 3: 777, 4: 777, 5: 777, 6: 777, 7: 777, 8: 777}
# 9 个  ✓
{1: 777, 2: 777, 3: 777, 4: 777, 5: 777, 6: 777, 7: 777, 8: 777, 9: 777}
# 10 个  ✓
{1: 777, 2: 777, 3: 777, 4: 777, 5: 777, 6: 777, 7: 777, 8: 777, 9: 777, 10: 777}
```

## 键盘遥操作

```bash
# 单轮的，9个电机
python examples/4_xlerobot_teleop_keyboard.py
# 两轮的，10个电机
python examples/4_xlerobot_2wheels_teleop_keyboard.py
```

键盘控制指令，

```bash
================================================================================
🤖 XLeRobot 2Wheels Keyboard Control Keymap
================================================================================

📱 Base Control (Differential Drive):
    i: Forward
    k: Backward
    u: Rotate Left
    o: Rotate Right
    n: Speed Up
    m: Speed Down
    b: Quit
    🚀 Smooth Control: Linear acceleration when holding, linear deceleration when released

🦾 Left Arm Control:
   Joint Control:
    Q/E: Shoulder Pan +/- (shoulder_pan)
    R/F: Wrist Roll +/- (wrist_roll)
    T/G: Gripper +/- (gripper)
    Z/X: Pitch +/- (pitch)
   Position Control:
    W/S: X-axis +/- (x movement)
    A/D: Y-axis +/- (y movement)
   Special Functions:
    C: Reset to zero position
    Y: Execute rectangular trajectory

🦾 Right Arm Control:
   Joint Control:
    7/9: Shoulder Pan +/- (shoulder_pan)
    /*: Wrist Roll +/- (wrist_roll)
    +/-: Gripper +/- (gripper)
    1/3: Pitch +/- (pitch)
   Position Control:
    8/2: X-axis +/- (x movement)
    4/6: Y-axis +/- (y movement)
   Special Functions:
    0: Reset to zero position
    Y: Execute rectangular trajectory

👁️ Head Control:
    </>: Head Motor 1 +/- (head_motor_1)
    ,/.: Head Motor 2 +/- (head_motor_2)
    ?: Head reset to zero position

⚙️ Robot Configuration:
   Wheel Radius: 0.050m
   Wheelbase: 0.250m
   Speed Levels: 3 levels
      Level 1: Linear 0.1m/s, Angular 30°/s
      Level 2: Linear 0.2m/s, Angular 60°/s
      Level 3: Linear 0.3m/s, Angular 90°/s

🚀 Smooth Control Parameters:
   Acceleration Rate: 10.0 speed/second
   Deceleration Rate: 10.0 speed/second
   Max Speed Multiplier: 6.0x

================================================================================
🎮 Control started! Use above keys to control robot
================================================================================
```

## Joycon 遥操作

安装操作库，

```bash
git clone --depth 1 https://github.com/box2ai-robotics/joycon-robotics.git
cd joycon-robotics

pip install -e .
sudo apt-get update
sudo apt-get install -y dkms libevdev-dev libudev-dev cmake git
make install

# 替换修改版本代码
export XR_DIR=`pwd`/XLeRobot
export JR_DIR=`pwd`/joycon-robotics
cp $XR_DIR/software/joyconrobotics/* $JR_DIR/joyconrobotics/

# 更好的蓝牙连接
# sudo apt install joycond
sudo systemctl enable --now joycond
sudo systemctl status joycond
```

再蓝牙配对。

操作单臂，

```bash
python examples/6_so100_joycon_ee_control.py
```

操作 Robot，

```bash
# 单轮
python examples/7_xlerobot_teleop_joycon.py
# 两轮
python examples/7_xlerobot_2wheels_teleop_joycon_smooth.py
```

Joycon 控制指令，

```bash
================================================================================
🤖 XLeRobot 2Wheels Joy-Con Control Instructions
================================================================================

📱 Base Control (Differential Drive):
    X: Rotate Left
    B: Rotate Right
    Y: Forward
    A: Backward
    🚀 Smooth Control: Linear acceleration when holding, linear deceleration when released

🦾 Right Arm Control:
   Joystick Control:
    - Vertical Joystick: X-axis and Z-axis movement (forward/backward)
    - Horizontal Joystick: Y-axis movement (left/right)
    - R Button: Z-axis up
    - Right Stick Button: Z-axis down
   Gripper Control:
    - ZR Button: Hold to toggle open/close direction, continue holding for linear open/close
    - D-pad: Control head motors
   🚀 Smooth Control: Linear acceleration/deceleration for shoulder_pan, shoulder_lift, elbow_flex

🦾 Left Arm Control:
   Joystick Control:
    - Vertical Joystick: X-axis and Z-axis movement (forward/backward)
    - Horizontal Joystick: Y-axis movement (left/right)
    - L Button: Z-axis up
    - Left Stick Button: Z-axis down
   Gripper Control:
    - ZL Button: Hold to toggle open/close direction, continue holding for linear open/close
    - D-pad: Control head motors
   🚀 Smooth Control: Linear acceleration/deceleration for shoulder_pan, shoulder_lift, elbow_flex

👁️ Head Control:
   Left Joy-Con D-pad:
    - Up/Down: Head Motor 2 +/-
    - Left/Right: Head Motor 1 +/-

⚙️ Robot Configuration:
   Wheel Radius: 0.050m
   Wheelbase: 0.250m
   Speed Levels: 3 levels
      Level 1: Linear 0.1m/s, Angular 30°/s
      Level 2: Linear 0.2m/s, Angular 60°/s
      Level 3: Linear 0.3m/s, Angular 90°/s

🚀 Smooth Control Parameters:
   Base Control:
     Acceleration Rate: 10.0 speed/second
     Deceleration Rate: 10.0 speed/second
     Max Speed Multiplier: 5.0x
   Arm Control (shoulder_pan, shoulder_lift, elbow_flex):
     Acceleration Rate: 5.0 degrees/second
     Deceleration Rate: 8.0 degrees/second
     Max Speed Multiplier: 2.0x

================================================================================
🎮 Control started! Use Joy-Con to control robot
================================================================================
```

## 手臂遥操作

- [Open Arms Mini](https://github.com/pkooij/open-arms-mini)
