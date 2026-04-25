# G1 虚拟键盘操作手册

这份文档用于从零配置并运行 G1 虚拟键盘流程，目标是：

- 运行 `unitree_rl_lab`
- 运行你改过的 `unitree_mujoco`
- 使用虚拟键盘控制 G1

默认你已经拿到了这两个仓库：

- `unitree_rl_lab`
- 你修改过、支持 `use_virtual_keyboard` 的 `unitree_mujoco`


## 一、整体上需要什么

要跑通这套流程，一共需要 5 类东西：

1. `unitree_sdk2`
2. `unitree_sdk2_python`
3. MuJoCo `3.2.7`
4. `unitree_rl_lab`
5. 你改过的 `unitree_mujoco`

它们的作用分别是：

- `unitree_sdk2`：给 C++ 程序用
  - `unitree_mujoco`
  - `g1_ctrl`
- `unitree_sdk2_python`：给 Python 虚拟键盘脚本用
- MuJoCo：仿真器本体


## 二、先检查本机是否已经安装

如果本机已经装过 `unitree_sdk2`、`unitree_sdk2_python` 或 MuJoCo，可以先检查，不需要重复安装。

### 1. 检查 `unitree_sdk2`

检查位置：

```bash
ls /opt/unitree_robotics
ls /opt/unitree_robotics/include/unitree
ls /opt/unitree_robotics/lib
```

正常情况下应当能看到：

- `/opt/unitree_robotics/include`
- `/opt/unitree_robotics/lib`
- `libunitree_sdk2.a`
- `libddsc.so`
- `libddscxx.so`

检查版本：

```bash
grep 'PACKAGE_VERSION' /opt/unitree_robotics/lib/cmake/unitree_sdk2/unitree_sdk2ConfigVersion.cmake
```

当前环境中检查到的版本示例：

```text
2.0.0
```

如果这些文件都在，一般说明 `unitree_sdk2` 已经安装好了。

### 2. 检查 `unitree_sdk2_python`

先检查当前 Python 环境是否能导入：

```bash
python3 -c "import unitree_sdk2py; print('unitree_sdk2py ok')"
```

如果想看安装信息：

```bash
pip show unitree_sdk2py
```

如果你保留了源码仓库，也可以看 `setup.py`：

```bash
grep "version=" /path/to/unitree_sdk2_python/setup.py
```

当前环境中看到的源码版本示例：

```text
1.0.1
```

### 3. 检查 MuJoCo

检查机器上有哪些 MuJoCo 版本：

```bash
find ~/.mujoco /root/.mujoco -maxdepth 2 -type d -name 'mujoco-*' 2>/dev/null | sort
```

检查当前使用的路径：

```bash
echo $MUJOCO_DIR
```

检查头文件版本：

```bash
grep 'mjVERSION_HEADER' $MUJOCO_DIR/include/mujoco/mujoco.h
```

版本对应关系：

- `327` 表示 MuJoCo `3.2.7`
- `336` 表示 MuJoCo `3.3.6`

这一套流程要求使用：

```text
MuJoCo 3.2.7
```

如果机器上同时有多个版本，例如：

- `mujoco-3.2.7`
- `mujoco-3.3.6`

一定要把 `MUJOCO_DIR` 指向 `3.2.7`。

### 4. 检查仓库来源

检查 `unitree_rl_lab`：

```bash
git -C ~/Desktop/project2/unitree_rl_lab remote -v
```

应当类似：

```text
https://github.com/unitreerobotics/unitree_rl_lab.git
```

检查 `unitree_mujoco`：

```bash
git -C ~/Desktop/project2/unitree_mujoco remote -v
```

应当类似：

```text
https://github.com/HeYee03/unitree_mujoco
```

如果以上检查都正常，可以直接进入编译和运行步骤，不需要重新安装。


## 三、推荐目录结构

推荐按下面这样放：

```text
~/Desktop/
  project2/
    unitree_rl_lab/
    unitree_mujoco/

/opt/
  unitree_robotics/

~/software/
  unitree_sdk2_python/

~/.mujoco/
  mujoco-3.2.7/
```

说明：

- `unitree_sdk2` 安装到 `/opt/unitree_robotics`
- `unitree_sdk2_python` 作为源码仓库可以放在任意位置
- MuJoCo 建议放在 `~/.mujoco/mujoco-3.2.7`


## 四、安装系统依赖

先安装基础依赖：

```bash
sudo apt update
sudo apt install -y \
  build-essential cmake git \
  libyaml-cpp-dev libboost-program-options-dev libfmt-dev \
  libglfw3-dev libxinerama-dev libxcursor-dev libxi-dev
```


## 五、安装 `unitree_sdk2`

这是 C++ SDK，必须装。

建议安装到 `/opt/unitree_robotics`：

```bash
git clone https://github.com/unitreerobotics/unitree_sdk2.git
cd unitree_sdk2
mkdir -p build && cd build
cmake .. -DCMAKE_INSTALL_PREFIX=/opt/unitree_robotics
sudo make install -j$(nproc)
```

安装完后，应该能看到：

```bash
/opt/unitree_robotics/include
/opt/unitree_robotics/lib
```

例如：

```bash
ls /opt/unitree_robotics/lib
```

应当包含类似这些库：

- `libunitree_sdk2.a`
- `libddsc.so`
- `libddscxx.so`


## 六、安装 MuJoCo 3.2.7

这一套流程建议使用：

```text
MuJoCo 3.2.7
```

不要用 `3.3.6`，因为当前这份 `unitree_mujoco` 的代码更适配 `3.2.7`。

建议解压到：

```bash
~/.mujoco/mujoco-3.2.7
```

然后设置环境变量：

```bash
export MUJOCO_DIR=$HOME/.mujoco/mujoco-3.2.7
```

如果你是 `root` 环境，也可以是：

```bash
export MUJOCO_DIR=/root/.mujoco/mujoco-3.2.7
```

为了长期生效，可以写进 `~/.bashrc`：

```bash
echo 'export MUJOCO_DIR=$HOME/.mujoco/mujoco-3.2.7' >> ~/.bashrc
source ~/.bashrc
```


## 七、安装 `unitree_sdk2_python`

这是给虚拟键盘脚本用的 Python SDK，也必须装。

可以把源码放在任意位置，例如：

```bash
git clone https://github.com/unitreerobotics/unitree_sdk2_python.git ~/software/unitree_sdk2_python
```

然后在你自己的 Python 环境里安装：

```bash
cd ~/software/unitree_sdk2_python
pip install -e .
```

还需要安装 Python 依赖：

```bash
pip install pygame cyclonedds
```

安装完后检查：

```bash
python3 -c "import unitree_sdk2py; print('unitree_sdk2py ok')"
```


## 八、准备仓库

建议把两个仓库放在同一级目录：

```bash
mkdir -p ~/Desktop/project2
cd ~/Desktop/project2
```

先克隆两个仓库：

```bash
git clone https://github.com/unitreerobotics/unitree_rl_lab.git
git clone https://github.com/HeYee03/unitree_mujoco.git
```

克隆完成后，再把单独提供的两个脚本覆盖到 `unitree_rl_lab/scripts/` 目录：

```text
unitree_rl_lab/scripts/virtual_keyboard_publisher.py
unitree_rl_lab/scripts/run_g1_cpp_bridge.sh
```

也就是说，这两个文件不要直接使用仓库里原始版本，而是使用单独发给你的版本。

注意：

- `unitree_mujoco` 必须是你修改过、支持虚拟键盘的版本
- `unitree_rl_lab/scripts/virtual_keyboard_publisher.py` 和 `unitree_rl_lab/scripts/run_g1_cpp_bridge.sh` 需要替换成单独提供的版本
- 如果仓库里还有根目录下的 `virtual_keyboard_publisher.py`，以你单独提供并实际约定使用的版本为准


## 九、配置 `unitree_mujoco`

编辑：

```text
unitree_mujoco/simulate/config.yaml
```

G1 虚拟键盘模式建议改成：

```yaml
robot: "g1"
robot_scene: "scene_29dof.xml"

domain_id: 0
interface: "lo"

use_joystick: 0
use_virtual_keyboard: 1

print_scene_information: 1
enable_elastic_band: 1
```

关键配置是：

- `use_joystick: 0`
- `use_virtual_keyboard: 1`
- `domain_id: 0`
- `interface: "lo"`


## 十、编译 `unitree_mujoco`

```bash
cd ~/Desktop/project2/unitree_mujoco/simulate
rm -rf build
mkdir build && cd build
cmake ..
make -j$(nproc)
```

编译成功后应该有：

```bash
~/Desktop/project2/unitree_mujoco/simulate/build/unitree_mujoco
```


## 十一、编译 `g1_ctrl`

```bash
cd ~/Desktop/project2/unitree_rl_lab/deploy/robots/g1_29dof
rm -rf build
mkdir build && cd build
cmake ..
make -j$(nproc)
```

编译成功后应该有：

```bash
~/Desktop/project2/unitree_rl_lab/deploy/robots/g1_29dof/build/g1_ctrl
```

如果你的本地仓库用了别的构建目录，例如 `build_codex`，也可以，但建议统一用标准的 `build/`。


## 十二、如何启动

进入 `unitree_rl_lab` 根目录：

```bash
cd ~/Desktop/project2/unitree_rl_lab
```

打开 3 个终端。

终端 A：

```bash
./run_g1_cpp_bridge.sh sim
```

终端 B：

```bash
./run_g1_cpp_bridge.sh ctrl -n lo
```

终端 C：

```bash
./run_g1_cpp_bridge.sh vkb --domain-id 0 --interface lo
```


## 十三、虚拟键盘界面怎么用

当前实际使用的脚本是：

```text
unitree_rl_lab/virtual_keyboard_publisher.py
```

它现在提供：

- 底部可点击按钮：`STAND / WALK / STOP`
- 左右两个可拖动的虚拟摇杆
- 键盘兜底控制

### 1. 推荐操作方式

直接用鼠标：

- 点击 `STAND`
- 点击 `WALK`
- 点击 `STOP`
- 拖动左摇杆和右摇杆

### 2. 键盘兜底映射

- `WASD`：左摇杆
- `IJKL`：右摇杆
- `Space`：`A`
- `Left Shift`：`B`
- `Q`：`X`
- `E`：`Y`
- `R`：`L1`
- `F`：`L2`
- `T`：`R1`
- `G`：`R2`
- 方向键：`up/down/left/right`

状态切换对应关系：

- `STAND`：`L2 + Up`
- `WALK`：`R1 + X`
- `STOP`：`L2 + B`


## 十四、正常运行时应该看到什么

如果一切正确：

- `sim` 会打开 G1 的 MuJoCo 仿真窗口
- `ctrl` 会打印：
  - `Waiting for connection to robot...`
  - `Connected to robot.`
  - `FSM: Start Passive`
- `vkb` 会打开虚拟控制器界面

然后建议按这个顺序操作：

1. 点击 `STAND`
2. 等机器人进入站立
3. 点击 `WALK`
4. 拖动摇杆控制
5. 点击 `STOP`


## 十五、常见问题

### 1. `Could not find mujoco`

原因：

- 没装 MuJoCo
- MuJoCo 版本不对
- 没设置 `MUJOCO_DIR`

解决：

- 使用 `3.2.7`
- 设置正确的 `MUJOCO_DIR`
- 删掉 `build` 后重新 `cmake ..`


### 2. `dds/dds.hpp: No such file or directory`

原因：

- `unitree_sdk2` 没安装到 `/opt/unitree_robotics`

解决：

- 重新安装 `unitree_sdk2`


### 3. `ImportError: No module named unitree_sdk2py`

原因：

- 当前 Python 环境没装 `unitree_sdk2_python`

解决：

```bash
cd ~/software/unitree_sdk2_python
pip install -e .
```


### 4. MuJoCo 能开，但控制器没反应

重点检查：

1. `unitree_mujoco/simulate/config.yaml` 是否真的设置了：
   - `use_joystick: 0`
   - `use_virtual_keyboard: 1`
2. `sim`、`ctrl`、`vkb` 是否都使用：
   - `domain_id = 0`
   - `interface = lo`
3. 虚拟键盘窗口是否真的在运行
4. `unitree_mujoco` 是否是你修改过的仓库


### 5. build 目录权限报错

原因：

- 有时用 `root` 编译过一次，后面换普通用户继续编译

解决：

```bash
rm -rf build
mkdir build
```

或者修复权限：

```bash
sudo chown -R $USER:$USER build
```


## 十六、快速检查清单

在排查问题前，先确认这几条：

- `/opt/unitree_robotics` 里有 `include` 和 `lib`
- `unitree_sdk2_python` 已安装进当前 Python 环境
- MuJoCo 是 `3.2.7`
- `MUJOCO_DIR` 配对了
- `config.yaml` 开了 `use_virtual_keyboard`
- `unitree_mujoco` 编译成功
- `g1_ctrl` 编译成功
- 3 个终端都启动了


## 十七、总结

这套环境最核心的关系是：

- `unitree_sdk2`：给 C++ 用
- `unitree_sdk2_python`：给 Python 虚拟键盘用
- MuJoCo：必须用 `3.2.7`
- 启动顺序固定：
  - `sim`
  - `ctrl`
  - `vkb`

只要这四块都对，虚拟键盘流程就能跑起来.
